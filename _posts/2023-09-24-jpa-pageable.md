---
layout: post
title: JPA/Pageable 인터페이스와 동작 원리
date: 2023-09-24
excerpt: "Spring Data JPA에서 페이지네이션과 정렬 시 사용하는 Pageable과 PageRequest에 대해 알아보자"
tags: [spring, jpa, pagination, pageable]
comments: true
---

## 페이지네이션이란?
대부분의 SNS, 게시판, 할 일 목록 등 리스트 형태의 서비스는 한 번에 모든 데이터를 보여주지 않고 쪽수별로 나눠서 보여주거나 스크롤을 할 때마다 추가 데이터를 받아오는 형태로 보여준다. 이렇게 사용자의 액션에 따라 필요한 만큼만 잘라서 보여줌으로써 서버와 DB의 부하를 줄이는 기술을 **페이지네이션**이라고 부른다.  
페이지네이션은 개발자가 직접 구현할 수도 있지만, Spring Data JPA에서 제공하는 Pageable 인터페이스를 사용하여 페이징과 정렬을 쉽게 적용할 수도 있다. 이번 글에서는 Pageable 인터페이스가 어떻게 구현되어있는지 살펴보고 실제로 적용될 때 어떻게 동작하는지 알아보자.

## Pageable
먼저 Spring 프레임워크 하위에 Pageable 소스코드를 확인해보자. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/9adee6cb-e1dc-40f0-89f4-61ff8a43ca54">
</div>

static 형태로 인스턴스를 반환하는 `unpaged()` 와 `ofSize()`가 있고, 현재 paging 정보가 포함되어 있는지 여부를 boolean으로 응답하는 `isPaged()`와 `isUnpaged()`가 default 메서드로 선언되어 있다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/8b588a2a-a22d-48f6-8de9-f70783a26e0e">
</div>

그 아래로 현재 페이지 번호, 페이지 사이즈, 오프셋, 정렬방식 등을 반환하는 함수들이 선언되어 있고, 이 때 정렬은 Streamable과 Serializable을 구현한 **Sort 타입**으로 반환되며, 정렬 기준과 정렬 방향(오름차순/내림차순) 등의 정보를 포함한다. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/a32c104c-89f6-48ac-9bfb-a17aec709b37">
</div>

마지막으로, 다음 페이지를 반환하는 `next()`, 이전 페이지를 반환하거나 현재 첫 페이지라면 첫 페이지를 반환하는 `previousOrFirst()`, 첫 페이지를 반환하는 `first()`, 특정 페이지 번호를 통해 바로 해당 페이지를 가져오는 `withPage()`, 이전 페이지가 존재하는지 여부를 확인하는 `hasPrevious()` 가 함수로 선언되어 있고, `Optional<>` 형태로 변환해주는 `toOptional()` 까지 default 메서드로 선언되어 있는 것을 확인할 수 있다.

<div style="width:80% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/dc37f11c-59ab-4345-80f7-dd588e0e2d52">
</div>

Pageable은 인터페이스 형태로 제공되기 때문에 사용하려면 이를 구현한 구현체를 확인해야 한다. 구현체 리스트를 보니 총 4개의 구현체가 제공되고 있고, 이 중 Unpaged는 위에서 `unpaged()` 함수에 반환되는 static한 enum 인스턴스이다.  
Pageable을 구현해놓은 추상 클래스가 **AbstractPageRequest**이고 이를 상속한 클래스들이 PageRequest와 QPageRequest이다. 이 중 QPageRequest는 QueryDSL을 위한 구현체이므로 이 글에서는 기본 구현체인 **PageRequest**에 대해 알아보자. 

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/87f17f72-b6a2-493e-9361-eb7aa3a73963">
</div>

## Pageable의 구현체 - PageRequest
먼저, PageRequst의 부모 클래스인 AbstractPageRequest의 소스코드를 확인해보자.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/c224f4b8-ed1e-43bc-8e30-526e1c2ff20e">
</div>

Pageable과 Serializable을 구현한 추상 클래스인 AbstractPageRequest은 private 필드로 **page와 size** 값을 갖는다. 이 때, page는 0보다 커야하고 size는 1보다 커야한다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/4280b94d-d8af-4307-8211-4e0eaddacca7">
</div>

위에서 정의된 두 개의 필드 page와 size를 이용해 Pageable에 선언되어있던 메서드들의 body가 작성되어 있는 것을 확인할 수 있다. 그 아래로 있는 `next()`, `previous()` 등의 함수들은 자식 클래스인 PageRequest에 구현되어 있으니 이제 PageRequest의 소스코드를 확인해보자.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/0e1e509c-0c96-46ee-a779-c81151394a92">
</div>

AbstractPageRequest를 상속한 자식 클래스 PageRequest는 private 필드로 **정렬 정보 Sort**를 갖는다. 그 아래로는 기본 생성자와 여러 가지 타입의 정적 팩토리 메서드들이 있다. 정렬 정보를 넣지 않으면 정렬하지 않음을 뜻하는 `Sort.unsorted()`가 적용된다. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/3242d977-79c0-41cc-ad99-9039dd68724f">
</div>

정적 팩토리 메서드에 페이지 사이즈만 입력하면 해당 사이즈만큼의 0번 페이지, 즉 첫 번째 페이지가 출력된다. `next()`, `previous()`, `first()` 메서드들은 정렬정보가 필요하기 때문에 부모 클래스가 아닌 자식 클래스에 구현되어 있는 것을 확인할 수 있다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/2427412c-cc03-44f8-af5b-0374e4d9ece4">
</div>

마지막으로, 페이지 번호를 넣어 해당하는 페이지를 바로 가져오는 `withPage()`와 새로운 정렬 기준을 적용할 수 있는 `withSort()` 메서드가 있고 `toString()`은 페이징 정보를 출력하도록 오버라이드 되어있다.

## 적용 예시와 동작 원리
이렇게 JPA에서 제공하는 Pageable을 실제 코드에 적용하면 아래와 같이 작성할 수 있다.

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

이렇게 간단히 pageable 객체를 Controller 부터 Repository까지 파라미터로 넘겨주는 방식으로 페이징된 데이터를 얻을 수 있다. 
요청을 보낼 때는 아래와 같이 QueryString 으로 URI 뒤에 페이징 설정(size, sort, page)을 보내주면 된다. 

```
GET /users?size=20&sort=age,DESC&page=0
```

<div style="width:70% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/8719eb8c-0356-4de7-bebf-eb5c696b6199">
</div>

여기서 size는 한 페이지당 데이터의 양이고, sort는 정렬 기준과 방식, page는 몇 번째 페이지를 받고 싶은지를 적어주면 된다. **0번째 페이지가 첫 번째 페이지**니 헷갈리지 않도록 주의해야 한다.
만약 이 값들 중 일부 혹은 전체를 적어주지 않으면 각 항목별 기본값으로 페이징된다. 사이즈는 20개가 기본이고, 페이지는 0번이 기본(즉, 첫 번째 페이지), 정렬은 하지 않음이 기본값이다.

그렇다면 어떻게 이 param 형태의 요청 데이터들이 별도의 작업없이 pageable 객체로 바로 매핑될 수 있는걸까? 
또, 기본으로 설정되어 있는 기본값들(size = 20, page = 0 등)을 커스텀하려면 어떻게 해야할까?

Spring의 ArgumentResolver의 구현체 중 하나인 [PageableHandlerMethodArgumentResolver](https://github.com/spring-projects/spring-data-commons/blob/main/src/main/java/org/springframework/data/web/PageableHandlerMethodArgumentResolver.java)를 
살펴보면 페이징 요청이 들어왔을 때의 동작을 확인할 수 있다.

<div style="width:80% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/dc1e184b-aae0-4708-a755-08a75ce6d8cb">
</div>
<br>
<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/1de55c38-3786-4ced-bc63-2946135161fc">
</div>

기본적으로 Spring의 ArgumentResolver는 `supportsParameter()`와 `resolveArgument()` 두 메서드로 구성되어 있고, 동작은 **supportsParameter 가 true인 경우 resolveArgument 에서 parameter를 원하는 형태로 바인딩**하도록 되어있다. 

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/ff4fc49a-48c9-4ba3-8826-4949536777c4">
</div>

이번엔 구현체 PageableHandlerMethodArgumentResolver의 동작부분 코드를 살펴보자.
`resolveArgument()`의 수행을 결정짓는 `supportsParameter()` 조건이 Pageable 타입검사인 것을 알 수 있다.
controller 메서드에 파라미터 타입이 Pageable 로 되어있는지를 확인하는 것이다.
즉, 파라미터의 타입이 Pageable 타입이면 supportsParameter에서 true를 반환하고 resolveArgument가 동작하게 된다. 

resolveArgument는 request를 받아 page, size, sort에 해당하는 값들을 꺼내 바인딩해주고 이를 Pageable 객체로 변환해 반환해준다.
여기서 구체적으로 어떻게 page, size, sort 에 해당하는 값들을 꺼내는지 알기 위해 87번 라인에 `getPageParameterName()`을 타고 들어가보자.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/3c74580a-88dd-4cba-b2e6-13389c61aa4a">
</div>

타고 들어가보면 `PageableHandlerMethodArgumentResolverSupport` 라는 추상 클래스가 나오는데 위에서 계속 보던 PageableHandlerMethodArgumentResolver의 부모 클래스이다. 
여기까지 들어와보니 비로소 퍼즐이 맞춰진다. 바인딩의 재료가 되는 것들이 전부 여기에 작성되어 있다.
QueryString의 이름이 되었던 `page`, `size` 와 같은 것들이 상수로 선언되어 있고, 아무것도 입력하지 않았을 때 default 값 역시 여기에 static 하게 선언되어 있는 것을 확인할 수 있다.

<div style="width:100% !important; margin:0 auto">
<img src="https://github.com/xrabcde/xrabcde.github.io/assets/56033755/f2519996-fc14-4aa2-a82a-90f7a75bf3db">
</div>

더 내려가 이 클래스의 240번대 라인을 보면 annotation으로 default 값들을 바인딩하는 `getDefaultFromAnnotationOr`
`Fallback()` 함수를 찾을 수 있다.

코드를 읽어보면 `@PageableDefault` 라는 어노테이션이 붙어있다면 그 값들을 읽어와서 페이징 디폴트 값들을 커스텀할 수 있게 해준다. 
`PageableDefault` 어노테이션은 아래와 같이 사용할 수 있다.

```kotlin
@GetMapping("/users")
fun findAll(@PageableDefault(size = 20, sort = ["updatedAt"], direction = Sort.Direction.DESC) pageable: Pageable): ResponseEntity<UsersResponse> {
    val usersResponse = userService.findAllUsers(pageable)
    return ResponseEntity.ok(usersResponse)
}
```

이 경우 요청 파라미터에 특별히 페이징 설정을 보내주지 않는다면 기본값으로 사이즈는 20개, 정렬은 업데이트 날짜 기준 내림차순으로 페이징되어 응답하게 된다. 
또, 페이지 번호는 커스텀하지 않았기 때문에 기본값인 0번 페이지 (첫 번째 페이지) 가 반환된다.
이 `@PageableDefault` 어노테이션을 사용하면 이렇게 페이징 기본값들을 설정할 수 있고, API 별로 다르게 세팅할 수도 있다.

## 결론
이번 글에서는 Spring Data JPA에서 제공하는 Pageable 인터페이스의 소스코드와 간단한 적용 예시, 그리고 내부 동작 원리까지 살펴봤다. 
Spring 에서는 이렇게 인터페이스와 어노테이션을 제공함으로써 개발자에게 간단히 페이징을 구현할 수 있도록 해준다.
하지만, 너무 간편하기 때문에 내부 동작이 어떻게 작동하는지는 직접 공부해보지 않으면 알기 어렵다. 
내부 동작과 소스코드를 공부해두면 페이징을 더 유연하게 적용할 수 있으니 프레임워크 내부 소스도 확인하는 습관을 가져보자.

## References
- [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/2.0.6.RELEASE/reference/html/)
- [Baeldung - Pagination and Sorting using Spring Data JPA](https://www.baeldung.com/spring-data-jpa-pagination-sorting)
- [Tecoble - Pageable을 이용한 Pagination을 처리하는 다양한 방법](https://tecoble.techcourse.co.kr/post/2021-08-15-pageable/)
