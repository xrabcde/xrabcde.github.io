---
layout: post
title: Book/Kotlin In Action/2장. 코틀린 기초
date: 2022-08-20
excerpt: "코틀린인액션 #2 코틀린 기초"
tags: [book, kotlin, kotlin_in_action]
comments: true
---

> 이 글은 [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)을 읽고 정리한 글입니다.

<div style="width:40% !important; margin:0 auto">
<img src="/assets/img/kotlin_in_action.png" alt="kotlin_in_action.png">
</div>

## 개요
- 이번 장에서는 코틀린에서 **변수, 함수, 클래스** 등을 어떻게 선언하는지 살펴보고 프로퍼티라는 개념에 대해 알아본다. 뿐만 아니라, **코틀린의 제어 구조와 예외 처리**에 대해서도 살펴보자.

## 함수, 변수, 클래스와 프로퍼티
### 함수와 변수
```kotlin
// 블록이 본문인 함수
fun max(a: Int, b: Int) : Int {
    return if (a > b) a else b
}
```
- 위 코드를 통해 알아볼 수 있는 코틀린의 몇 가지 특성
  - 코틀린에서는 함수를 선언할 때 `fun` 키워드를 쓴다
  - 파라미터 이름 앞이 아닌 **뒤에 파라미터의 타입을 쓴다**
  - 줄 끝에 **세미콜론(`;`)을 붙이지 않아도 된다**

- 코틀린에서 위의 코드처럼 식이 본문인 함수는 아래와 같이도 작성이 가능하다.
- 컴파일러가 **타입추론**을 통해 반환 타입을 결정하기 때문에 반환 타입도 생략이 가능하다.

```kotlin
// 식이 본문인 함수
fun max(a: Int, b: Int) = if (a > b) a else b
```

```kotlin
val name = "bada"
val answer: Int
answer = 42

println("Hello, $name!")
```
- 변수를 선언할 때 자바와 달리 타입 생략이 가능하기 때문에 변수 이름 앞이 아닌 **뒤에 타입을 명시하거나 생략한다.**
- `val` : value, 변경 불가능한 참조를 저장하는 변수, **immutable**, 자바의 `final`
- `var` : variable, 변경 가능한 참조, **mutable**
- 문자열 템플릿 : 문자열 리터럴의 필요한 곳에 `$` 와 함께 변수명을 작성할 수 있다.

### 클래스와 프로퍼티
```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public string getName() {
        return name;
    }
}
```
- 자바에서는 데이터를 필드에 저장하며, 멤버 필드의 가시성은 보통 `private` 이다.
- 뿐만 아니라, `getter` 메서드를 명시적으로 작성해 외부에서 그 데이터를 접근할 수 있는 통로를 만들어준다.

```kotlin
class Person(val name: String)
```

- 이 코드는 위의 자바코드와 같은 동작을 하는 코틀린 코드이다.
- 기본적으로 **프로퍼티를 선언함과 동시에 생성자, 접근자가 모두 선언이 된다.** (자세한 내용은 4장에서 다룸)
  - 읽기 전용 프로퍼티의 경우, getter만 선언이 되고
  - 변경가능한 프로퍼티의 경우, getter와 setter를 모두 선언한다

## enum, when, while, for
### 선택 표현과 처리: enum, when
```kotlin
enum class Color (val r: Int, val g: Int, val b: Int) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    INDIGO(75, 0, 130),
    VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}
```
- enum도 클래스와 마찬가지로 생성자와 프로퍼티를 선언한다. 각 enum 상수를 정의할 때는 프로퍼티 값을 지정해야 한다.
- enum 클래스 안에서 메서드를 정의하고 싶을 때는 반드시 enum 상수 목록과 메서드 사이에 세미콜론(`;`)을 넣어야 한다.

```kotlin
fun getWarmth(color: Color) = // 함수의 반환 값으로 when 식을 직접 사용
    when (color) { // 색이 특정 enum 상수와 같을 때 그 상수에 대응하는 온도를 반환
        Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
        Color.GREEN -> "neutral"
        Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
    }

fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) { // when 식의 인자로 아무 객체나 사용가능
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.YELLOW, Color.BLUE) -> Color.GREEN
        setOf(Color.BLUE, Color.VIOLET) -> Color.INDIGO
        else -> throw Exception("Dirty color") // 매치되는 분기 조건이 없는 경우
    }
```
- `when`은 자바의 `switch`와 유사하지만 훨씬 더 강력하다.
- `if`와 마찬가지로 `when`도 값을 만들어내는 식이기 때문에, 위처럼 식이 본문인 함수에 반환값으로 `when`을 바로 사용할 수 있다.
- `switch`와 달리 각 분기의 끝에 `break`를 넣지 않아도 되고, 한 분기 안에서 콤마(`,`)를 이용해 여러 개의 매치 패턴을 사용할 수도 있다.
- 뿐만 아니라, 분기 조건으로 임의의 객체를 허용하고, 분기 조건에 해당하는 패턴이 없을 경우는 `else`에 기본값을 지정할 수 있다.

### 이터레이션: while, for
```kotlin
while ("조건") { // 조건이 참인 동안 본문을 반복 실행
    /* ... */
}

do { // 무조건 본문을 한 번 실행한 뒤, 조건이 참인 동안 본문을 반복 실행
    /* ... */
} while ("조건")
```
- 코틀린의 `while`, `do-while` 은 자바와 크게 다르지 않다.

```kotlin
for (c in list) { // 리스트를 순회하며 이터레이션한다
    /* ... */
}

for ((index, element) in list.withIndex()) { // 인덱스와 함께 컬렉션을 이터레이션한다
    println("$index: $element")
}
```
- 코틀린에서는 이터레이션을 할 때 `for .. in` 루프를 자주 쓴다.
- 이 때 순회를 하는 컬렉션 원소로는 숫자 타입 뿐 아니라 문자 타입도 적용이 가능하다.
- `withIndex()`를 사용하면 인덱스와 함께 컬렉션을 이터레이션할 수 있다.

## 코틀린의 예외 처리
```kotlin
if (percentage !in 0..100) {
    throw IllegalArgumentException("A percentage must be between 0 and 100")
}
```
- 코틀린의 예외 처리는 자바나 다른 언어의 예외 처리와 비슷하다.
- 함수 안에서 오류가 발생하면 예외를 던질 수 있고, 함수를 호출하는 쪽에서 그 예외를 잡아 처리할 수 있다.
- 예외 인스턴스를 만들 때, 다른 클래스와 마찬가지로 `new`를 붙일 필요가 없다.

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine()) // 예외가 발생하지 않은 경우 실행
    } catch (e: NumberFormatException) {
        null // 예외가 발생하면 null 값 사용
    }
}
```
- 코틀린의 `try` 키워드는 `if` 나 `when` 과 마찬가지로 식이기 때문에, `try` 의 값을 변수에 바로 대입할 수 있다. `if` 와 달리, `try` 의 본문은 반드시 중괄호 `{}` 로 둘러싸야 한다.

## References
- [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)