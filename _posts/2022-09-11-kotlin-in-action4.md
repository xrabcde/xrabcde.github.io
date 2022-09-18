---
layout: post
title: Book/Kotlin In Action/4장. 클래스, 객체, 인터페이스
date: 2022-09-11
excerpt: "코틀린인액션 #4 클래스, 객체, 인터페이스"
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

## 개요
- 이번 장에서는 코틀린의 인터페이스와 생성자, 다양한 클래스들(data 클래스, object 클래스)에 대해 알아본다.

## 인터페이스와 접근 제어자
### 코틀린 인터페이스
- 코틀린 인터페이스 안에는 추상 메서드뿐 아니라 구현이 있는 메서드도 정의할 수 있다
- 다만 인터페이스에는 **아무런 상태(필드)도 들어갈 수 없다**

```kotlin
interface Clickable {
    fun click() // 추상 메서드
}

// 이 인터페이스를 구현하는 모든 클래스는 click()에 대한 구현을 제공해야 한다
```

- 코틀린에서는 클래스 이름 뒤에 : 을 붙이는 것으로 상속과 구현을 모두 처리한다 (자바의 `extends`, `implements` 키워드)

```kotlin
class Button : Clickable {
    override fun click() = println("I was clicked")
}

>>> Button().click()
I was clicked
```

- 자바와 마찬가지로 **인터페이스는 원하는 만큼 구현**할 수 있지만, **클래스는 오직 하나만 확장**할 수 있다
- 자바의 `@Override` 어노테이션과 비슷한 `override` 변경자는 상위 클래스나 상위 인터페이스에 있는 프로퍼티나 메서드를 오버라이드한다는 표시다
  - 하지만, 자바와 달리 **코틀린에서는 override 변경자를 생략할 수 없다**
  - `override` 변경자는 **실수로 상위 클래스의 메서드를 오버라이드하는 경우를 방지**해준다
- 인터페이스 메서드에 디폴트 구현을 제공하고 싶을 때, 메서드 앞에 `default`를 붙여야하는 자바와 달리 코틀린에서는 그냥 **메서드 본문을 시그니처 뒤에 추가**하면 된다

```kotlin
interface Clickable {
    fun click() // 일반 메서드
    fun showOff() = println("I'm clickable!") // 디폴트 구현이 있는 메서드
}

// 이 인터페이스를 구현하는 모든 클래스는 click에 대한 구현을 제공해야 하지만, 
// showOff는 새로운 동작을 정의할 수도 있고, 생략할 수도 있다

interface Focusable {
    fun showOff() = println("I'm focusable!")
}
```

- 만약, 위처럼 showOff 라는 동일한 디폴트 메서드를 정의한 인터페이스를 한 클래스에서 함께 구현한다면 어느 쪽 showOff 메서드가 선택될까?
    - 어느 쪽도 선택되지 않는다 > **컴파일 오류 발생**
    - 이런 경우는, 디폴트 메서드라 하더라도 **두 메서드를 아우르는 구현을 하위 클래스에 직접 구현해야 한다**

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")
    override fun showOff() {
        // 상위 타입의 이름을 꺾쇠 괄호(<>) 사이에 넣어서 super를 지정하면
        // 어떤 상위 타입의 멤버 메서드를 호출할지 지정할 수 있다
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### 코틀린의 접근 제어자 : 기본적으로 final, public
- 상속을 금지시키기 위해 명시적으로 `final`을 작성해야 하는 자바와 달리, 코틀린의 클래스와 메서드는 기본적으로 `final`이다
- 어떤 클래스의 **상속을 허용하려면 클래스 앞에 `open` 변경자를 붙여야 한다**

```kotlin
open class RichButton : Clickable { // 열려있는 클래스 (다른 클래스가 이 클래스 상속가능)
    fun disable() {} // final 함수 (하위 클래스가 이 메소드 오버라이드 불가능)
    open fun animate() {} // 열려있는 함수 (하위 클래스가 이 메소드 오버라이드 가능)
    override fun click() {} // 열려있는 함수를 오버라이드한 함수
}
```

- 오버라이드하는 메서드의 구현을 하위 클래스에서 오버라이드하지 못하게 하려면 `final`을 명시해야 한다

```kotlin
open class RichButton : Clickable {
    final override fun click() {} // final이 없는 override 메서드는 기본적으로 열려있다
}
```

- 자바처럼 코틀린에서도 클래스를 `abstract`로 선언할 수 있다. `abstract`로 선언한 추상 클래스는 인스턴스화할 수 없다
- 인터페이스의 경우 `final`, `open`, `abstract`를 사용하지 않는다
    - 인터페이스 멤버는 항상 열려있으며 `final`로 변경할 수 없다
    - 인터페이스 멤버에 본문이 없으면 자동으로 추상 멤버

- 코틀린에는 기본적으로 자바와 같은 `public`, `protected`, `private` 변경자가 있지만, 기본 가시성 변경자는 `public`이다. 아무 변경자도 없는 경우 모두 `public` 이다
- 코틀린에는 자바의 패키지 전용 가시성 역할을 하는 `internal`이라는 새로운 가시성 변경자가 있다
    - `internal`은 모듈 내부에서만 볼 수 있음 이라는 뜻이다

<table>
  <tr>
    <td><center> <b>변경자</b> </center></td>
    <td><center> <b>클래스 멤버</b> </center></td>
    <td><center> <b>최상위 선언</b> </center></td>
  </tr>
  <tr>
    <td><center> public(기본 가시성임) </center></td>
    <td><center> 모든 곳에서 볼 수 있다 </center></td>
    <td><center> 모든 곳에서 볼 수 있다 </center></td>
  </tr>
  <tr>
    <td><center> internal </center></td>
    <td><center> 같은 모듈 안에서만 볼 수 있다 </center></td>
    <td><center> 같은 모듈 안에서만 볼 수 있다 </center></td>
  </tr> 
  <tr>
    <td><center> protected </center></td>
    <td><center> 하위 클래스 안에서만 볼 수 있다 </center></td>
    <td><center> 최상위 선언에 적용불가 </center></td>
  </tr>
  <tr>
    <td><center> private </center></td>
    <td><center> 같은 클래스 안에서만 볼 수 있다 </center></td>
    <td><center> 같은 파일 안에서만 볼 수 있다 </center></td>
  </tr>
</table>

- 코틀린에서는 최상위 선언(클래스, 함수, 프로퍼티)에 대해 `private` 가시성을 허용한다
- 자바와 달리 코틀린의 `protected`는 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 접근가능하다

## 코틀린의 다양한 클래스 초기화 방법
이 장의 내용은 이전에
[코틀린에서 클래스를 초기화하는 다양한 방법](https://xrabcde.github.io/kotlin-constructor/) 글을 통해 한 번 정리했기 때문에
위 글에서 다루지 않은 내용들만 적어보려고 한다.

### 인터페이스에 선언된 프로퍼티 구현
- 코틀린에서는 인터페이스에 추상 프로퍼티 선언을 넣을 수 있다

```kotlin
interface User {
    val nickname: String
}
```

- 인터페이스에는 뒷받침하는 필드나 게터 등의 정보가 없기 때문에 User를 구현하는 클래스가 nickname의 값을 얻을 수 있는 방법을 제공해야 한다

```kotlin
// 1. 주 생성자에 있는 프로퍼티
class PrivateUser(override val nickname: String) : User

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // 2. 커스텀 게터
}

class KakaoUser(val accountId: Int) : User {
    override val nickname = getKakaoName(accountId) // 3. 프로퍼티 초기화 식
}
```

- 코틀린에서는 인터페이스에 선언된 프로퍼티를 여러가지 방법으로 구현할 수 있다
  1. `주 생성자` 안에 프로퍼티를 직접 선언 : override 표시
  2. `커스텀 게터`로 nickname 프로퍼티 설정 : **호출될 때마다** 이메일 주소에서 별명을 계산해 반환
  3. `초기화 식`으로 nickname 값 초기화 : getKakaoName 함수를 호출해 nickname 초기화

### 접근자(get, set) 활용하기
- 코틀린에서 getter 와 setter 함수에서 해당 필드에 접근할 경우, `field`라는 특별한 식별자를 사용한다

```kotlin
class User(val name: String) {
    var address: String = "undefined"
        set(value: String) {
            println("""
                Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}

>>> val user = User("Bada")
>>> user.address = "Pangyo"
Address was changed for Bada:
"undefined" -> "Pangyo".
```

- 위의 코드는 `user.address = “new value”` 를 실행할 때 address의 커스텀 세터가 호출되어 추가 로직(값을 변경하는 동시에 변경 이력을 로그로 남김)을 실행하는 코드이다

- 접근자(get, set)의 가시성은 기본적으로는 프로퍼티의 가시성과 같지만, 원한다면 가시성 변경자를 추가해 변경할 수 있다

```kotlin
class LengthCounter {
    var counter: Int = 0
        private set
    fun addWord(word: String) {
        counter += word.length
    }
}
```

- 위의 코드는 추가된 단어의 길이를 합산하는 코드이다. 전체 길이를 저장하는 프로퍼티(counter)는 `public`으로 외부에 공개되어 있지만, 외부에서 단어 길이의 합을 마음대로 변경하지 못하도록 `set` 접근자를 `private`으로 변경했다.
- 이제 이 클래스 내부에서만 길이를 변경할 수 있다

## data 클래스와 object 클래스
### toString(), equals(), hashCode() : 모든 클래스가 정의해야 하는 메서드
- 자바에서는 클래스가 `equals`, `hashCode`, toString 등의 메서드를 구현해야 한다. 코틀린에서는 이런 작업을 보이지 않는 곳에서 할 수 있도록 해주어 코드를 깔끔하게 유지할 수 있다.
- `toString()` : 인스턴스의 문자열 표현을 얻을 방법을 제공하는 메서드, 주로 디버깅과 로깅 시 사용
- `equals()` : 객체의 동등성 보장(내부에 동일한 데이터를 포함하는 경우, 동등한 객체로 간주)
- `hashCode()` : HashSet과 같은 컬렉션에서 원소 비교를 할 때 비용을 줄이기 위해 객체의 해시 코드를 비교, equals()만 구현하고 hashCode()를 구현하지 않을 경우, 이런 상황에서 같은 객체임을 기대하지만 다른 결과가 나오는 오류가 발생

```kotlin
class User(val name: String, val age: Int) {
    override fun hashCode() : Int = name.hashCode() * 31 + age
}
```

- **이 모든 overrride 메서드를 자동으로 생성해주기 위해 코틀린에서 제공하는 키워드**에 대해 알아보자

### 데이터 클래스
- 코틀린에서는 `data`라는 변경자를 클래스 앞에 붙이면 필요한 메서드를 컴파일러가 자동 생성하고, 이를 `데이터 클래스`라고 부른다

```kotlin
data class User(val name: String, val age: Int)
```

- 이제 이 코드 한 줄은 위에서 override 했던 3가지 메서드들(toString, equals, hashCode)을 모두 포함하는 코드이다

### object 키워드 - 객체 선언
- 코틀린에서 object 키워드는 다양한 상황에 쓰이지만, 모두 **클래스를 정의하면서 동시에 인스턴스를 생성**한다는 공통점이 있다. object 키워드를 사용하는 여러 상황을 살펴보자.
  1. `객체 선언`은 싱글턴을 정의하는 방법 중 하나
  2. `동반 객체`는 인스턴스 메서드는 아니지만 어떤 클래스와 관련 있는 메서드와 팩토리 메서드를 담을 때 씀
  3. `객체 식`은 자바의 무명 내부 클래스 대신 쓰임

- 싱글턴을 구현하기 위해 생성자를 private으로 제한하고 정적인 필드에 그 클래스의 유일한 객체를 저장하는 자바와 달리, 코틀린은 **객체 선언 기능**을 통해 싱글턴을 언어에서 기본 지원한다. 객체 선언은 **클래스 선언과 그 클래스에 속한 단일 인스턴스의 선언을 합친 선언**이다

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployees) {
        ... }
    }
}
```

- `object` 키워드로 시작하는 객체 선언은 **클래스를 정의하고 인스턴스를 만들어서 변수에 저장하는 모든 작업을 한 문장으로 처리**한다
- 클래스와 마찬가지로 프로퍼티, 메서드, 초기화 블록 등이 들어갈 수 있지만 **생성자는 쓸 수 없다** (싱글턴 객체는 생성자 호출 없이 즉시 만들어지기 때문에 생성자 정의 필요없음)
- 객체 선언도 클래스나 인터페이스를 상속할 수 있다

### object 키워드 - 동반 객체
- [코틀린에서 클래스를 초기화하는 다양한 방법](https://xrabcde.github.io/kotlin-constructor/)에서 살펴봤던 동반 객체는 일반 객체처럼 사용할수도 있기 때문에

1) 이름을 붙이거나 : 동반객체는 대부분의 경우 클래스 이름을 통해 먼저 참조하기 때문에 네이밍에 공을 들일 필요가 없다. 특별히 이름을 지정하지 않으면 동반 객체 이름은 `Companion`이 된다

```kotlin
class Person(val name: String) {
	companion object Loader {
		fun fromJSON(jsonText: String) : Person = ...
	}
}

>>> person = Person.Loader.fromJSON("{name: 'bada'}")
>>> person.name
bada
>>> person2 = Person.fromJSON("{name: 'ara'}")
>>> person2.name
ara
```

2) 인터페이스를 구현하거나 : 인터페이스를 구현하는 동반 객체는 객체를 둘러싼 클래스 이름으로 바로 참조할 수 있다

```kotlin
class Person(val name: String) {
	companion object : JSONFactory<Person> {
		override fun fromJSON(jsonText: String) : Person = ...
	}
}

// JSON을 다시 역직렬화에 원소로 만들어내는 추상 팩토리
fun loadFromJSON<T>(ffactory: JSONFactory<T>): T {
	...
}

>>> loadFromJSON(Person) // Person 객체를 바로 넘길 수 있음
```

3) 확장 함수와 프로퍼티를 정의할 수 있다 : 확장 함수를 사용하면 객체의 관심사를 좀 더 명확히 분리할 수 있다

```kotlin
// 비즈니스 로직 모듈
class Pserson(val firstName: String, val lastName: String) {
	companion object { // 비어있는 동반 객체 선언
	}
}

// 클라이언트/서버 통신 모듈
fun Person.Companion.fromJSON(json: String): Person {
	... // 확장 함수 선언
}

>>> val p = Person.fromJSON(json)
// 마치 동반 객체 안에서 fromJSON 함수를 정의한 것처럼 호출할 수 있다
```

- 동반 객체에 대한 확장 함수를 작성하려면 원래 클래스에 **빈 객체라도 동반 객체가 꼭 선언되어 있어야 한다**

### object 키워드 - 객체 식
- 싱글턴 객체를 정의할 때 쓰는 `object` 키워드는 **무명 객체를 정의할 때도 사용**할 수 있다. 무명 객체는 자바의 무명 내부 클래스를 대신한다

```kotlin
// 무명 객체로 이벤트 리스너 구현
window.addMouseListener(
    object: MouseAdapter () { // MouseAdapter를 확장하는 무명 객체 선언
        override fun mouseClicked(e: MouseEvent) { // MouseAdapter의 메서드 오버라이드
            // ...
        }
        override fun mouseEntered(e: MouseEvent) { // MouseAdapter의 메서드 오버라이드
            // ...
        }
    }
}

// 무명 객체는 변수에 대입해 이름을 붙일 수 있음
val listener = object: MouseAdpater() {
    override fun mouseClicked(e: MouseEvent) { ... }
    override fun mouseEntered(e: MouseEvent) { ... }
}
```

- 사용한 구문은 객체 선언과 유사하지만, 객체 이름이 빠졌다. **객체 식**은 클래스를 정의하고 그 클래스에 속한 인스턴스를 생성하지만, 그 클래스나 인스턴스에 이름을 붙이진 않는다 (무명 객체는 보통 함수를 호출하면서 인자로 넘기기 때문에 클래스와 인스턴스 모두 이름이 필요하지 않음)
- 코틀린 무명 클래스는 **여러 인터페이스를 구현하거나 클래스를 확장할 수 있다**
- 객체 선언과 달리 무명 객체는 싱글턴이 아니기 때문에 **객체 식이 쓰일 때마다 새로운 인스턴스가 생성**된다
- 객체 식 안의 코드는 그 식이 포함된 함수의 변수에 접근할 수 있고, 자바와 달리 final이 아닌 변수도 사용할 수 있다. 즉, **객체 식 안에서 그 변수의 값을 변경할 수도 있는 것이다**

## References
- [코틀린인액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791161750712)