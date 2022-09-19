---
layout: post
title: Book/Kotlin In Action/3장. 함수 정의와 호출
date: 2022-09-11
excerpt: "코틀린인액션 #3 함수 정의와 호출"
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

## 개요
- 이번 장에서는 코틀린의 컬렉션에 대해 알아보고, 확장함수, 가변길이 인자, 중위 호출 등을 통해 컬렉션을 다양하게 활용할 수 있는 방법에 대해 살펴보자.

## 코틀린의 컬렉션
### 컬렉션 선언
- 코틀린은 자신만의 컬렉션 기능을 제공하지 않고 표준 자바 컬렉션을 활용하기 때문에, 자바 코드와 상호작용하기가 훨씬 더 쉽다.
- 코틀린에서는 다음과 같이 Set, List, Map을 선언한다.

```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

- 코틀린 컬렉션은 자바 컬렉션과 똑같은 클래스지만, 자바보다 더 많은 기능을 쓸 수 있다.

```kotlin
val strings = listOf("first", "second", "fourteenth")
println(strings.last()) // 리스트의 마지막 원소를 가져온다
>>> fourteenth

val numbers = setOf(1, 14, 2)
println(numbers.max()) // 수로 이루어진 컬렉션에서 최댓값을 찾는다
>>> 14
```

### 컬렉션 출력
- 리스트 `[1, 2, 3]` 을 `(1; 2; 3)` 과 같은 형태로 출력한다고 가정해보자.
- 컬렉션의 출력을 커스텀하게 만들고 싶은 경우 코틀린에서 제공하는 다양한 기능을 통해 더 유연하게 코드를 작성할 수 있다.

```kotlin
val list = listOf(1, 2, 3)
println(joinToString(list, "; ", "(", ")"))
>>> (1; 2; 3)

// 이름붙인 인자 활용
println(joinToString(list, separator=" ", prefix=" ", postfix="."))

// 디폴트 파라미터값 활용
println(joinTostring(list, postfix=";", prefix="# "))
```

## 확장함수, 확장 프로퍼티 : 메소드를 다른 클래스에 추가
- 어떤 클래스의 멤버 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수를 **`확장함수`**라고 한다.
- 다음 코드는 String 클래스에 확장함수를 선언한 예이다.

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1) // this 생략가능
println("Kotlin".lastChar())
>>> n
```

- 위의 코드에서 `String`이 수신객체 타입이고 `“Kotlin”`이 수신 객체이다.
- 클래스 안에 정의된 멤버 함수와 달리 확장함수 안에서는 클래스 내부에서만 사용할 수 있는 **private이나 protected 멤버를 사용할 수 없다.**
- 확장함수는 정적 메서드와 같은 특징을 가지므로, **확장함수를 하위 클래스에서 오버라이드 할 수는 없다.**

### 확장함수를 이용한 컬렉션 출력
- 확장함수를 이용해 위에서 커스텀했던 컬렉션 출력을 다시 작성해보자.
- `Collection<T>` 의 확장함수로 `joinToString`을 선언하면 `joinToString`을 마치 클래스의 멤버인 것처럼 호출할 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String=", ",
    prefix: String="",
    postfix: String=""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex())
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

val list = listOf(1, 2, 3)
println(list.joinToString(" "))
>>> 1 2 3
```

### 확장 프로퍼티
- 확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다.
- 이름은 프로퍼티이지만 상태를 저장할 방법이 없기 때문에 실제로 **확장 프로퍼티는 아무 상태도 가질 수 없다.**
- 하지만 접근자(get, set) 커스텀 등의 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있어 편한 경우들이 있다. 예를 들어,

```kotlin
val String.lastChar: Char
    get() = get(length - 1)

// 확장 프로퍼티를 사용하는 방법은 멤버 프로퍼티를 사용하는 방법과 같다
println("Kotlin".lastChar)
>>> n
```

## 코틀린에서 컬렉션 활용
### 가변 길이 인자 : 인자 개수가 달라질 수 있는 함수 정의
- `vararg` 키워드를 사용하면 호출 시 인자 개수가 달라질 수 있는 함수를 정의할 수 있다.
- 가장 대표적인 예시로 리스트를 생성하는 함수를 들 수 있다.

```kotlin
val list = listOf(2, 3, 5, 7, 11)

// 함수의 정의부분
fun listOf<T>(vararg values: T): List<T> { ... }
```

- 자바의 가변길이 인자와 유사하지만 문법이 조금 다르다. 타입 뒤에 `…` 를 붙이는 대신 파라미터 앞에 `vararg` 변경자를 붙인다.

### 중위 함수 호출
- **중위(infix) 함수** 호출 구문을 사용하면 인자가 하나뿐인 메소드를 간편하게 호출할 수 있다.
- mapOf에서 사용하는 `to` 라는 단어는 코틀린 키워드가 아닌, **중위 호출**이라는 특별한 방식으로 `to` 라는 일반 메서드를 호출한 것이다.

```kotlin
val map = mapOf(1 tp "one", 7 to "seven", 53 to "fifty-three")

1.to("one") // "to" 메서드를 일반적인 방식으로 호출
1 to "one" // "to" 메서드를 중위 호출 방식으로 호출
```

- **인자가 하나뿐인 일반 메서드나 확장 함수에 중위 호출을 사용**할 수 있다.
- 중위 호출을 사용하게 허용하고 싶으면 `infix` 변경자를 함수 선언 앞에 추가해야 한다.

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

### 구조 분해 선언
- 구조 분해 선언(destructuring declaration)을 사용하면 복합적인 **값을 분해해서 여러 변수에 나눠 담을 수 있다.**

```kotlin
val (number, name) = 1 to "one"
```

- 루프에서도 구조 분해 선언을 활용할 수 있다.
- 아래 코드는 `withIndex`를 사용해 컬렉션 원소의 인덱스와 값을 변수에 담아 루프를 도는 코드이다.

```kotlin
for ((index, value) in colletion.withIndex()) {
    println("$index: $element")
}
```

## 문자열과 정규식
- 코틀린의 문자열은 자바 문자열과 같다. 특별한 변환도 필요없고 별도의 래퍼(wrapper)도 생기지 않는다.
- 코틀린은 다양한 확장함수를 통해 표준 자바 문자열을 더 즐겁게 다루게 해주고, 더 명확하기 때문에 프로그래머의 실수를 줄여준다.
- 자바에서 `split()`의 구분 문자열은 정규식이기 때문에 `split(”.”)`은 **모든 문자열을 구분한 빈배열을 반환한다.**
- 코틀린에서는 여러 가지 다른 조합의 파라미터를 받는 **split 확장함수를 제공함으로써 이런 혼동을 야기하는 상황을 줄여준다.**

```kotlin
println("12.345-6.A".split("\\.|-".toRegex())) // 명시적으로 정규식을 만듦
>>> [12, 345, 6, A]

println("12.345-6.A".split(".", "-")) // 여러 구분 문자열을 지정함
>>> [12, 345, 6, A]
```

- 뿐만 아니라, 코틀린 표준 라이브러리에는 어떤 문자열에서 구분 문자열이 맨 나중에 나타난 곳 뒤의 부분 문자열을 반환하는 함수가 있다.
- 아래 파일 경로를 파싱하는 코드를 읽어보면 훨씬 이해하기 쉽다.

```kotlin
val path = "/Users/bada/kotlin-book/chapter.adoc"

val directory = path.substringBeforeLast("/")
val fullName = path.substringAfterLast("/")
val fileName = fullName.substringBeforeLast(".")
val extension = fullName.substringAfterLast(".")
```

- 코틀린의 3중 따옴표 문자열을 사용하면 역슬래시(`\`)를 포함한 어떤 문자도 이스케이프할 필요가 없다.

```kotlin
val regex = """(.+)/(.+)\.(.+)""".toRegex()
```

## 코틀린에서 중복코드를 줄이는 법 : 로컬함수와 확장
- 흔히 발생하는 코드 중복을 로컬함수를 통해 어떻게 제거하는지 살펴보자.
- 아래 코드는 사용자를 데이터베이스에 저장하는 코드인데 각 필드를 검증하는 부분에서 중복이 계속 발생하게 된다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException()
    }
    if (user.address.isEmpty()) {
        throw IllegalArgumentException()
    }
    // user를 데이터베이스에 저장한다
}
saveUser(User(1, "", ""))
>>> IllegalArgumentException!
```

- 검증 코드를 로컬 함수로 분리하면 중복을 없애는 동시에 코드 구조를 깔끔하게 유지할 수 있고 필요하면 다른 필드에 대한 검증도 쉽게 추가할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun saveUser(user: User) {
    fun validate(value: String) {
        if (value.isEmpty()) throw IllegalArgumentException()
    }
    validate(user.name) // 로컬 함수를 호출해 각 필드 검증
    validate(user.address)
    // user를 데이터베이스에 저장한다
}
```

- 검증 로직을 User 클래스의 확장 함수로 작성하면 이 예제를 더 개선할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() { // 검증 로직을 확장함수로 작성
    fun validate(value: String) {
        if (value.isEmpty()) throw IllegalArgumentException()
    }
    validate(name) // User의 프로퍼티 직접 사용가능
    validate(address)
}

fun saveUser(user: User) {
    user.validateBeforeSave()
    // user를 데이터베이스에 저장한다
}
```

## References
- [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)

### 이 시리즈의 다음글
- [코틀린인액션 #4 클래스, 객체, 인터페이스](https://xrabcde.github.io/kotlin-in-action4/)