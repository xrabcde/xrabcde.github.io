---
layout: post
title: Book/Kotlin In Action/5장. 람다로 프로그래밍
date: 2022-10-13
excerpt: "코틀린인액션 #5 람다로 프로그래밍"
tags: [book, kotlin, kotlin_in_action]
comments: true
---

> 이 글은 [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)을 읽고 정리한 글입니다.

<div style="width:40% !important; margin:0 auto">
<img src="/assets/img/kotlin_in_action.png" alt="kotlin_in_action.png">
</div>

### 이 시리즈의 이전글
- [코틀린인액션 #1 코틀린이란 무엇이며, 왜 필요한가?](https://xrabcde.github.io/kotlin-in-action1/)
- [코틀린인액션 #2 코틀린 기초](https://xrabcde.github.io/kotlin-in-action2/)
- [코틀린인액션 #3 함수 정의와 호출](https://xrabcde.github.io/kotlin-in-action3/)
- [코틀린인액션 #4 클래스, 객체, 인터페이스](https://xrabcde.github.io/kotlin-in-action4/)

## 개요
- 람다는 기본적으로 **다른 함수에 넘길 수 있는 작은 코드 조각**을 뜻한다. 람다를 사용하면 쉽게 공통 코드 구조를 라이브러리 함수로 뽑아낼 수 있다. 이번 장에서는 코틀린 표준 라이브러리에서 람다를 사용하는 방법에 대해 알아보고, 자바 라이브러리와 함께 사용하는 방법까지 살펴본 뒤, 마지막으로 수신 객체 지정 람다에 대해 알아보자.

## 람다 식과 멤버 참조
### 람다를 이용한 컬렉션 처리
- 함수형 언어에서는 함수를 값처럼 다루기 때문에 함수를 직접 다른 함수에 전달할 수 있다. 람다 식을 이용하면 별도의 함수 선언 없이, **코드 블록을 직접 함수의 인자로 전달할 수 있다.**
- 쓸 수 있는 컬렉션 라이브러리가 적어 직접 구현해야했던 자바와 달리, **코틀린에서는 대부분의 기능을 제공하기 때문에 컬렉션을 더 편리하게 다룰 수 있다.**

```kotlin
data class Person(val name: String, val age: Int)
val people = listOf(Person("Ara", 29), Person("Bada", 31))
```

- 이름과 나이를 갖는 Person 객체들 중에 나이가 가장 많은 사람을 찾아야 할 때, 컬렉션 라이브러리가 없다면 다음과 같이 작성해야 한다.

```kotlin
// 컬렉션 라이브러리 없이 직접 구현한 코드
fun findMaxByAge(people: List<Person>): Person {
    var maxAge = 0
    var maxPerson: Person? = null
    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            maxPerson = person
        }
    }
    return maxPerson
}

// 컬렉션 라이브러리를 활용한 코드
people.maxBy { it.age }
people.maxby(Person::age) // 멤버 참조로 대치한 코드
```

- `maxBy` 는 비교에 사용할 값을 인자로 받아 최댓값을 반환하는 함수이다. 이런 식으로 단지 함수나 프로퍼티를 반환하는 역할을 수행하는 람다는 멤버 참조로 대치할 수 있다.

### 람다 식의 문법과 멤버 참조
- 람다는 값처럼 여기저기 전달할 수 있는 동작의 모음이므로, 람다를 따로 선언해서 변수에 저장할 수도 있지만 대부분 바로 함수의 인자에 전달한다.

```kotlin
{ x: Int, y: Int -> x + y }
```

- 코틀린 람다 식은 항상 중괄호 `{ }` 로 둘러싸여 있고, 인자 목록 주변에 괄호없이 화살표 `→` 로 인자 목록과 람다 본문을 구분한다.

```kotlin
people.maxBy({ p: Person -> p.age }) // 생략없이 작성
people.maxBy { it.age } // 생략가능한 부분을 전부 생략한 코드
```

- 위에서 사용했던 `maxBy` 함수를 생략없이 작성하면 위의 코드에서 첫번째 줄이 된다. 코틀린에는 함수 호출 시 **맨 뒤에 있는 인자가 람다 식이라면 그 람다를 괄호 밖으로 빼낼 수 있다**. 이 코드에서는 람다가 유일한 인자이자, 마지막 인자이므로 괄호를 생략할 수 있는 것이다.
- 뿐만 아니라, 로컬 변수처럼 컴파일러는 람다 파라미터의 타입도 추론 가능하므로, 타입도 생략이 가능하다. 람다의 파라미터가 하나뿐이고 컴파일러가 추론가능한 타입이라면, 디폴트 값인 `it` 을 사용해 화살표도 생략이 가능하다.

- 람다 안에서 파이널 변수만 접근가능했던 자바와 달리, **코틀린 람다 안에서는 파이널이 아닌 변수에도 접근할 수 있고, 그 변수를 변경할 수도 있다**.
- `Person::age` 와 같이 `::` 을 사용하는 식을 **멤버 참조**라고 부른다. 멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수이다.

```kotlin
// 멤버 참조의 동작을 풀어서 작성한 코드
val getAge = { person: Person -> person.age }
```

## 함수형 스타일로 컬렉션 다루기
### filter와 map
- 대부분의 컬렉션 연산은 `filter`와 `map`을 통해 표현할 수 있다.

```kotlin
val list = listOf(1, 2, 3, 4)
list.filter { it % 2 == 0 }
```

- `filter`는 **컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨 true를 반환하는 원소만 모은다.** 하지만, `filter`는 원소를 변환할 수는 없다. 변환하려면 `map` 함수를 사용해야 한다.

```kotlin
val list = listOf(1, 2, 3, 4)
list.map { it * it }
```

- `map`은 **주어진 람다를 컬렉션의 각 원소에 적용한 결과를 모아 새 컬렉션을 만든다.** 결과는 원본과 개수는 같지만, 변환된 원소들을 담은 새로운 컬렉션이다.
- 이 두 가지 연산은 연쇄시킬 수도 있다.

```kotlin
people.filter { it.age > 30 }.map(Person::name)
```

### all, any, count, find
- `all`, `any`, `count`, `find`는 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산들이다.
- `all`은 모든 원소가 조건을 만족하는지 여부를 반환하고, `any`는 만족하는 원소가 하나라도 있는지를 반환한다. `count`는 조건을 만족하는 원소의 개수를 반환하며, `find`는 조건을 만족하는 첫 번째 원소를 반환한다.
- `count`를 사용하지 않고 `filter().size`를 사용하면 조건을 만족하는 원소들이 담긴 중간 컬렉션이 생기므로, `count`를 사용하는 것이 훨씬 더 효율적이다.

### groupBy, flatMap, flatten
- `groupBy`는 컬렉션의 모든 원소를 **어떤 특성에 따라 여러 그룹으로 나누어주는 함수이다.** 원소를 구분하는 특성이 인자로 주어지면 그에 따라 나누어진 그룹 리스트를 반환한다.
- `flatMap`은 인자로 주어진 람다를 **컬렉션의 모든 객체에 적용하고 적용한 결과 리스트들을 하나로 모아 반환한다.** `flatten`은 같은 동작을 하지만, 특별히 반환해야 할 내용 없이 펼치기만 할 경우 사용할 수 있다.

## 시퀀스: 지연(lazy) 컬렉션 연산
### 시퀀스의 연산 순서
- 앞에서 살펴보았던 `map`이나 `filter` 같은 함수는 결과 컬렉션을 즉시 생성하는 연산이다. 즉, 이들을 연쇄하면 매 단계마다 중간 계산결과를 임시 컬렉션에 담는다는 뜻이다.
- **시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.**

```kotlin
// 시퀀스를 사용하지 않은 코드
people.map(Person::name).filter { it.startsWith("A") }

// 시퀀스를 사용한 코드
people.asSequence()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()
```

- 시퀀스를 사용하지 않은 첫 번째 코드는 `map`과 `filter` 연산마다 한번씩, 2개의 리스트가 만들어진다.
- 이를 더 효율적으로 만들기 위해서는 두 번째 코드처럼 시퀀스를 사용하면 된다. **시퀀스는 중간 결과를 저장하는 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 눈에 띄게 좋아진다.**
- `Sequence` 인터페이스 안의 `Iterator`를 통해 시퀀스로부터 원소 값을 얻을 수 있다. 시퀀스의 원소는 필요할 때 계산되기 때문에, 중간처리 결과를 저장하지 않고 연산을 연쇄적으로 적용해 효율적인 계산이 가능한 것이다.
- 어떤 컬렉션이든 `asSequence`를 호출하면 시퀀스로 바꿀 수 있고, 다시 리스트로 만들 때는 `toList`를 사용한다.

```kotlin
// 시퀀스를 사용하지 않은 코드
listOf(1, 2, 3, 4).map { it * it }.find { it > 3 }

// 시퀀스를 사용한 코드
listOf(1, 2, 3, 4).asSequence()
    .map { it * it }.find { it > 3 }
```

- `map` 연산을 모두 수행하고 얻은 결과 컬렉션에 다시 `find` 연산을 하는 첫번째 코드와 달리, **시퀀스 연산은 원소 하나하나에 대해 연쇄적으로 연산을 수행한다.**
- 즉, 원소 `1`을 먼저 제곱하고 그 값이 3보다 큰지 확인 > 원소 `2`를 제곱하고 그 값이 3보다 큰지 확인 > …
- 이 때, 원소 `2`에서 `find` 에서 찾고자 하는 값을 얻었으므로 원소 `3`, `4`는 연산을 하지 않고 그 즉시 결과를 반환하게 된다.

## 수신 객체 지정 람다 사용
### with와 apply
- `with`와 `apply`는 자바의 람다에는 없는 코틀린 람다의 독특한 기능이다. 이 두 함수는 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있게 해준다. 이를 **수신 객체 지정 람다**라고 부른다.
- `with`는 어떤 객체의 이름을 반복하지 않고도 그 객체에 대한 다양한 연산을 수행할 수 있게 해준다.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("Finish!")
    return result.toString()
}
```

- 위의 코드는 result에 대해 다른 여러 메서드를 호출하며 매번 result를 반복 사용한다.

```kotlin
// with를 활용헤 개선한 코드
fun alphabet() =  with(StringBuilder()) { // 메서드를 호출하려는 수신 객체 지정
    for (letter in 'A'..'Z') {
        this.append(letter) // this는 앞에서 지정한 수신 객체
    }
    append("Finish!") // this 생략도 가능
    toString() // 람다 본문의 마지막 식의 값을 반환
}
```

- `with`는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 `this`를 사용해 수신 객체에 접근할 수 있고, 수신 객체의 메서드를 바로 호출할 수도 있다.
- `with`는 특별한 구문처럼 보이지만 사실 파라미터가 2개 있는 함수이다. 단지, 마지막 파라미터가 람다이기 때문에 람다를 괄호 밖으로 빼내는 관례를 사용함에 따라 특별한 구문처럼 보이는 것이다.

- `apply` 함수는 거의 `with`와 같지만, 항상 자신에게 전달된 객체를 반환한다는 차이가 있다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("Finish!")
}.toString()
```

- `apply`는 확장함수로 정의되어 있기 때문에, `apply`의 수신 객체가 전달받은 람다의 수신 객체가 되고, 실행 결과는 `StringBuilder` 객체이다. 따라서, `with`와 달리 결과 객체에 `toString`을 호출해 `String` 객체를 얻을 수 있다.
- 이런 `apply` 함수는 **객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우** 유용하다. 자바에서는 보통 별도의 `Builder` 객체가 이런 역할을 담당했다.

## References
- [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)