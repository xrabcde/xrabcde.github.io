---
layout: post
title: JPA/No offset 방식으로 페이징 성능 개선하기
date: 2023-10-09
excerpt: "offset 방식에서 no offset 방식으로 개선하는 방법과 성능 차이에 대해 알아보자"
tags: [spring, jpa, pagination, pageable]
comments: true
---

## 개요
[지난번 글](https://xrabcde.github.io/jpa-pageable/)에서는 JPA 에서 제공하는 Pageable 인터페이스의 소스 코드와 기본 동작 원리에 대해 살펴봤다. 
이전 글에서 살펴본 것처럼 Pageable 인터페이스를 활용하면 쉽게 페이지네이션을 구현할 수 있지만, 
서비스가 커지고 데이터의 양이 많아짐에 따라 로딩속도가 느려지는 성능 문제를 야기할 수 있다. 

그런 시점에 페이징 성능을 개선할 수 있는 방법으로는 페이지 번호와 사이즈를 이용하지 않고 
다음 데이터를 호출하는 **no offset 방식**이 있다. 
쉽게 생각해서 페이지 번호로 이동하는 형태가 아닌 `더보기` 버튼으로 다음 페이지를 호출하는 무한 스크롤 형태를 떠올리면 된다.
이번 글에서는 offset 방식에서 no offset 방식으로 변경하는 방법과 그에 따른 성능 차이까지 알아보자.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/3f79a45f-cb35-4600-9832-b1bf0581ab36">
<figcaption align=center>이미지 출처: smashingmagazine</figcaption>
</div>

## 기본 Pagination 방법 (offset 기반)
JPA에서 제공하는 기본 인터페이스를 활용한 기본 페이지네이션 방법은 offset 기반 방식에 해당한다. 
offset 기반 방식이란 offset, 즉 **페이지 번호를 이용해 가져올 데이터의 위치를 파악하는 방식**이다. 
아래 코드는 이전 글에서 사용했던 예제 코드로, Pageable 인터페이스를 활용해 모든 유저를 조회하는 기본 페이지네이션 방식이다.

```kotlin
// Controller
@GetMapping("/users")
fun findAll(pageable: Pageable): ResponseEntity<UsersResponse> {
    val usersResponse = userService.findAllUsers(pageable)
    return ResponseEntity.ok(usersResponse)
}

// Service
fun findAllUsers(pageable: Pageable): UsersReponse {
    val findUsers = users.findAllUsersWithPaging(pageable)
    return UsersReponse.from(findUsers)
}

// Repository
fun findAllUsersWithPaging(pageable: Pageable): Page<Task>
```

이렇게 Pageable 인터페이스를 활용하면 아주 쉽게 페이지네이션을 구현할 수 있다는 장점이 있지만,
데이터의 양이 많아지면 쿼리 속도가 확연히 느려지게 된다.
그 이유는 오프셋의 동작방식과 실제 발생하는 쿼리를 보면 알 수 있다.

클라이언트에서 넘겨받는 페이지 번호와 페이지 사이즈를 가지고 서버에서 오프셋 값을 구한다.
`오프셋 = (페이지 번호 - 1) * 페이지 사이즈` 이기 때문에 페이지 사이즈가 20인 경우 아래와 같이 오프셋 값이 늘어나게 된다. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/a70743e0-9f83-4e1e-94c3-351bcaeef4c1">
</div>

```sql
SELECT * FROM user LIMIT 페이지사이즈 OFFSET 오프셋;
SELECT * FROM user LIMIT 20 OFFSET 0;       # 1~20 출력
SELECT * FROM user LIMIT 20 OFFSET 20;      # 21~40 출력
SELECT * FROM user LIMIT 20 OFFSET 40;      # 41~60 출력
...
SELECT * FROM user LIMIT 20 OFFSET 999980;  # 999981~100000 출력
```

이 방식은 데이터의 양이 적거나 앞 페이지를 조회할 경우 문제가 되지 않지만, 
데이터의 양이 많은 상황에서 뒷 페이지를 조회할 경우(즉, offset 값이 늘어날 경우) 굉장히 느려진다.
그 이유는 매번 데이터를 조회할 때마다 offset 개수만큼의 행을 전부 스캔한 뒤 (full scan) 필요한 데이터를 반환하기 때문이다. 

데이터의 개수가 100,000개인 테이블에서 20개의 데이터를 조회하는 쿼리를 통해 실제 수행시간을 확인해보자. 더미 데이터는 [Mockaroo](https://www.mockaroo.com/)를 이용해 생성해주었다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/0d0ac61d-63fd-4bfc-a7da-813bf09b63a2">
</div>

첫 페이지를 조회하는 경우(OFFSET 0)의 응답시간은 **0.0014초**인 반면, 마지막 페이지를 
조회하는 경우(OFFSET 99980)의 응답시간은 **0.056초**인 것을 확인할 수 있다. 
데이터의 양이 10만 개 정도밖에 되지 않는데도 약 40배의 성능 차이가 난다.
이렇게 OFFSET 기반 방식을 사용하면 데이터의 양이 많은 경우 마지막 페이지를 조회할 때 
확연한 속도차이가 나고 이는 데이터의 양이 많아질수록 더 심해진다.

뿐만 아니라, 이 방식은 offset을 기반으로 위치를 탐색하기 때문에 데이터의 중복과 누락이 발생할 수도 있다.
offset 방식은 데이터의 값은 고려하지 않고 n번째 데이터인지만 확인한 뒤 가져오기 때문에
그 사이에 추가되거나 삭제되는 데이터들을 알 수 없다. 
따라서, 중간에 추가되어 행이 뒤로 밀리는 경우 같은 값이 중복해서 노출될 수도 있고 
중간에 삭제되어 행이 앞으로 밀리는 경우 특정 행은 누락될 수도 있다. 

## 개선한 Pagination 방법 (no offset 기반)
그렇다면 no offset 기반 방식은 무엇일까?
no offset 페이지네이션이란, 말 그대로 offset을 사용하지 않고 페이지네이션을 구현하는 것이다.
즉, 페이지 번호를 사용하는 대신 탐색 위치를 찾을 무언가가 있어야 한다.
클라이언트에서 페이지 번호를 넘겨주는 대신에 마지막 조회 ID 값을 넘겨주고, 
서버에서는 이 값을 인덱스로 빠르게 찾아 **매번 첫 페이지를 읽는 것과 같은 성능을 보이게 된다.**

```sql
SELECT * FROM user WHERE id > 마지막조회ID LIMIT 페이지사이즈;
SELECT * FROM user WHERE id > 20 LIMIT 20;        # 21~40 출력
SELECT * FROM user WHERE id > 40 LIMIT 20;        # 41~60 출력
...
SELECT * FROM user WHERE id > 999980 LIMIT 20;    # 999981~100000 출력
```

위에서 확인했던 것과 같은 방법으로 데이터의 개수가 100,000개인 테이블에서 20개의 데이터를 조회하는 쿼리를 통해 no offset 방식의 실제 수행시간을 확인해보자. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/4ca98184-ff4d-443e-a925-e069b39d43a1">
</div>

이번엔 첫 페이지와 마지막 페이지 조회 속도가 각각 **0.00049초**와 **0.00069초**로
거의 차이가 없는 것을 확인할 수 있다. 
실제로 수행 시간을 비교해보니 기존 방식에 비해 월등히 성능이 좋다는 것을 알 수 있다.
그렇다면 Pageable 인터페이스를 사용하던 기존 코드에서 이 방식으로 코드를 개선해보자.

```kotlin
// Controller
@GetMapping("/users")
fun findAll(@RequestParam lastUserId: Long): ResponseEntity<UsersResponse> {
    val usersResponse = userService.findAllUsers(lastUserId)
    return ResponseEntity.ok(usersResponse)
}

// Service - PageRequest 사용하지 않는 경우
fun findAllUsers(lastUserId: Long): UsersReponse {
    val findUsers = users.findTop20ByIdGreaterThan(lastUserId)
    return UsersReponse.from(findUsers)
}

// Repository
fun findTop20ByIdGreaterThan(id: Long): List<User>

// Service - PageRequest 사용하는 경우
fun findAllUsers(lastUserId: Long, size: Int = 20): UserResponse {
    val pageRequest = PageRequest.ofSize(size)
    val findUsers = users.findAllByIdGreaterThan(lastUserId, pageRequest)
    return UsersResponse.from(findUsers)
}

// Repository
fun findAllByIdGreaterThan(id: Long, pageRequest: PageRequest): List<User>
```

기존에 Pageable 인터페이스를 통해 페이지 번호를 파라미터로 받는 부분은
`/users?lastUserId=2048` 과 같이 lastId를 받도록 수정한다. 
Pageable을 넘겨받던 Repository 부분은 id를 이용해 다음 탐색 위치를 찾도록 쿼리를 수정해준다. 
이 때, 페이지 사이즈를 결정짓는 LIMIT 의 사용을 위해 JPA Named query를 활용해 `Top20` 과 같이 작성할수도 있지만 이렇게 되면 페이지 사이즈가 변경될 수 있는 상황에서는 유연하게 대처하기
어려우므로 Service 에서 PageRequest를 만들어 넘겨주는 방식으로 구현할 수도 있다. 

## 결론
Pageable 인터페이스를 활용하면 아주 쉽게 offset 기반의 페이지네이션을 구현할 수 있다는 장점이 있지만,
데이터의 양이 많아질 경우 로딩 속도 저하, 데이터의 중복/누락이 발생할 수 있기 때문에
no offset 기반의 방식도 고려할 필요가 있다. 
no offset 기반 방식의 경우 offset 기반 방식보다 조회 시 뛰어난 성능을 보여주긴 했지만
구현이 조금 더 복잡하다는 단점이 있고 특히 정렬을 사용하는 경우 구현 난이도는 더 올라간다.
두 방식 모두 각각의 장단점이 있기 때문에 서비스의 특징에 맞는 방법을 적절하게 선택하는 것이 좋겠다.

## References
- [1. 페이징 성능 개선하기 - No Offset 사용하기](https://jojoldu.tistory.com/528)
- [No Offset 쿼리로 Paging 성능 개선하기](https://velog.io/@cmsskkk/No-Offset-Paging-ngrinder2)
- [no offset 페이지네이션](https://velog.io/@pood/no-offset-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98)