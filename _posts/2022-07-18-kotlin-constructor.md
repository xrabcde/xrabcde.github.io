---
layout: post
title: Kotlin/코틀린에서 클래스를 초기화하는 다양한 방법
date: 2022-07-18
excerpt: "kotlin의 주 생성자와 초기화 블록, 부 생성자, 그리고 동반 객체의 팩토리 메소드"
tags: [kotlin, oop, constructor]
comments: true
---

## 개요
Java에서는 생성자를 하나 이상 선언할 수 있다.
코틀린도 비슷하지만 한 가지 다른 점은 **코틀린은 주 생성자와 부 생성자를 구분한다는 것**이다.
뿐만 아니라, 자바에는 없었던 **초기화 블록(`initializer block`)**과 **동반 객체(companion object)**라는 개념이 추가되었다. 

이 주/부 생성자와 초기화 블록을 잘 사용해서 코틀린에서 클래스를 초기화하는 방법에 대해 알아보고,
예시코드를 통해 **코틀린스럽게** 이것들을 활용하는 방법도 확인해보자.

## 주 생성자와 초기화 블록: 클래스 초기화
```kotlin
class Person(val name: String)
```
클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 **주 생성자(primary constructor)**라고 부른다.
주 생성자는 **생성자 파라미터를 지정**하고 그 **생성자 파라미터에 의해 초기화되는 프로퍼티를 정의**하는 두 가지 목적으로 쓰인다.
위의 코드를 Java로 작성하면 아래와 같다.

```java
public class JavaPerson {
    private String name;

    public JavaPerson(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
```
클래스 내부에 생성자 코드를 작성해서 인스턴스를 생성할 때 동작할 로직에 대해 명시적으로 작성해주어야 하는 자바와 달리 
코틀린에서는 클래스 이름 뒤에 오는 괄호에 파라미터만 명시해줘도 생성자가 되는 것이다.
하지만, 주생성자 코드는 괄호 안에 파라미터만 작성하기 때문에
파라미터의 값을 검증한다거나 하는 별도의 로직을 갖는 코드를 포함할 수 없다.

이럴 때 사용할 수 있는 것이 바로 **초기화 블록**이다.
대부분의 경우는 위의 코드들처럼 인자로 받은 값을 프로퍼티에 설정하기만 하는 생성자를 주로 사용하기 때문에
위처럼 초기화 블록없이 괄호 안에 주 생성자만 작성해주는 방식으로 충분하다.

```kotlin
class Person(val name: String) {
    init {
        require(name.length < 5) { "이름은 5자 이내로 입력해주세요" }
    }
}
```
하지만, 이처럼 넘겨준 파라미터에 대한 검증이 필요한 경우 초기화 블록을 활용하기 좋다. 
초기화 블록은 클래스의 객체가 만들어질 때 (인스턴스화될 때) 실행되며, 주 생성자와 함께 사용할 수 있다.
이 코드는 위에서 봤던 코드에서 이름에 대한 글자수 검증이 추가된 코드이다. 필요하다면 **클래스 안에 여러 초기화 블록을 선언할 수 있다.**

## 부 생성자: 상위 클래스를 다른 방식으로 초기화

Java와 달리, 코틀린에서는 생성자가 여러 개 있는 경우가 훨씬 적다.
자바에서 오버로드한 생성자가 필요한 상황 중 상당수는 코틀린의 **디폴트 파라미터 값**과 **이름붙인 인자 문법**을 사용해 해결할 수 있기 때문이다.

```kotlin
class Person(val name: String = "이름없음")
```
이 코드는 Person을 생성할 때 아무 값도 넘겨주지 않으면 `이름없음` 이라는 값을 name으로 갖는 Person 객체를 생성해주는 코드이다.
생성자 체이닝을 통해 기본값을 지정했던 Java와 달리, 코틀린에서는 이처럼 생성자 하나로 기본 인자를 지정해줄 수 있다.

```kotlin
val bada = Person(name = "바다")
```
이 코드는 Person을 생성하며 전달하는 인자의 이름을 명시해서 작성한 코드이다.
코틀린의 이름붙인 인자를 사용하면 Java의 빌더 패턴과 같은 것을 사용하지 않아도 어떤 인자인지 명시적으로 드러낼 수 있고,
파라미터가 여러 개인 경우 함수 호출의 가독성을 향상시킨다.

그렇다면 부 생성자는 어떤 경우에 사용하는 것이 좋을까?

생성자가 여럿 필요한 가장 일반적인 상황은 프레임워크 클래스를 확장해야 하는데 여러 가지 방법으로 인스턴스를
초기화할 수 있게 다양한 생성자를 지원해야 하는 경우다. 
예를 들어, 자바에서 선언된 생성자가 2개인 View 클래스가 있다고 할 때
그 클래스를 코틀린으로는 다음과 비슷하게 정의할 수 있다.

```kotlin
class Person {
    constructor(name: String) {
        // 코드
    }

    constructor(name: String, age: Int) {
        // 코드
    }
}
```
위의 코드는 주 생성자를 선언하지 않고 부 생성자만 2가지 선언한 코드이다. 
부 생성자는 constructor 키워드로 시작하고 필요에 따라 얼마든지 많이 선언해도 된다.

클래스 인스턴스를 생성할 때 **파라미터 목록이 다른 생성 방법이 여러 개 존재하는 경우**에는
부 생성자를 여러 개 둘 수밖에 없는데, 이 경우는 **동반 객체**의 팩토리 메소드를 활용할 수 있다.

## 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소
코틀린은 자바 static 키워드를 지원하지 않기 때문에 클래스 안에 정적인 멤버가 없다.
그 대신 코틀린에서는 패키지 수준의 최상위 함수와 객체 선언을 활용한다.
대부분의 경우 최상위 함수를 더 권장하지만, 최상위 함수는 private인 클래스의 비공개 멤버에 접근할 수 없다.

이럴 때 활용할 수 있는 것이 바로 **동반 객체의 팩토리 메소드**이다.
동반 객체의 팩토리 메소드는 클래스에 중첩해 선언한 객체 안에 멤버 함수를 정의하는 것을 말한다.

```kotlin
class Person {
    companion object {
        fun move() {
            println("Companion object called")
        }
    }
}
```
클래스 안에 객체를 정의할 때 앞에 `companion`이라는 특별한 표시를 붙이면 그 클래스의 동반 객체로 만들 수 있다.
이 때 객체의 이름을 따로 지정할 필요는 없고, 동반 객체의 멤버를 사용하는 구문은
자바의 정적 메소드 호출이나 정적 필드 사용 구문과 같아진다.

동반 객체는 최상위 함수와 달리 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있기 때문에
**private 생성자를 호출하기 좋은 위치**이며, 팩토리 패턴을 구현하기 가장 적합한 위치다.

이제 부 생성자가 2개 있는 클래스를 다시 동반 객체 안에서 팩토리 클래스를 정의하는 방식으로 변경해보자.
```kotlin
class User {
    val nickname: String

    constructor(email: String) {
        nickname = email.subStringBefore('@')
    }

    constructor(facebookAccountId: Int) {
        nickname = getFacebookName(facebookAccountId)
    }
}
```

위와 같이 클래스 인스턴스를 생성할 때 **파라미터 목록이 다른 생성 방법이 여러 개 존재하는 경우**,
클래스의 인스턴스를 생성하는 팩토리 메소드를 통해 더 유용한 방법으로 코드를 작성할 수 있다.

```kotlin
class User private constructor(val nickname: String) {
    companion object {
        fun newSubscribingUser(email: String) = User(email.subStringBefore('@'))
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```
이 코드를 사용할 때는 클래스 이름을 사용해 그 클래스에 속한 동반 객체의 메소드를 호출할 수 있다.

```kotlin
val subscribingUser = User.newSubscribingUser("bada@bada.com")
val facebookUser = User.newFacebookUser(4)
println(subscribingUser.nickname)
>>> bada
```

팩토리 메소드는 매우 유용하다.
- 위의 예제처럼 **목적에 따라 메소드의 이름을 정할 수 있고,**
- 그 팩토리 메소드가 선언된 클래스의 **하위 클래스 객체를 반환할 수도 있고,**
- **생성할 필요가 없는 객체를 생성하지 않을 수도 있다.**
  - 예를 들어, 이메일 주소별로 유일한 User 인스턴스를 만드는 경우 팩토리 메소드가 이미 존재하는 인스턴스에 해당하는 이메일 주소를 전달받으면 새 인스턴스를 만들지 않고 캐시에 있는 기존 인스턴스를 반환할 수 있다.

## 예시코드를 통해 리팩터링해보기
이제 위에서 학습한 개념들을 어떻게 잘 사용할 수 있을지 예시코드를 통해 확인해보자.

자동로또와 수동로또 두 종류를 가지는 로또라는 객체가 있다고 가정할 때, 로또 객체를 생성할 때  
1) **별도의 인자를 넘겨주지 않는다면** 내부적으로 랜덤함수를 돌려 **자동로또를 생성**해주고  
2) **숫자들을 인자로 넘겨준다면** 해당 숫자들로 **수동로또를 생성**해주는 코드를 작성하고 싶었다.

Java로 가정하면 이런 코드를 작성하고 싶었던 것이다.

```java
public class JavaLotto {
    private List<Integer> numbers;

    public JavaLotto() {
        this.numbers = generateLotto();
    }

    public JavaLotto(List<Integer> numbers) {
        this.numbers = numbers;
    }
}
```

그래서 처음에 내가 작성했던 코드는 이렇다.

```kotlin
class Lotto() {
    var numbers: List<Int>

    init {
        numbers = generateLotto()
    }

    constructor(input: List<String>) : this() {
        numbers = validateNumbers(input)
    }
}
```

이 코드는 초기화블록(`init {}`)과 부 생성자를 적절하게 사용하지 못한 코드이다.  
이 Lotto 객체의 인스턴스를 생성하는 흐름을 생각해보면, 
1. **numbers = null**인 상태로 Lotto 인스턴스 생성
2. numbers에 값을 채워넣기 위해 `generateLotto()` 동작

이 되는데, 이 코드를 위에서 학습했던 코틀린의 다양한 초기화 방법들을 활용해 개선해보자.

```kotlin
class Lotto(numbers: List<Int> = generateLotto()) {
    val numbers: Set<Int>

    init {
        this.numbers = convertToSet(numbers)
        validateNumbers(this.numbers)
    }
}
```

먼저, **디폴트 파라미터**를 이용해 주 생성자에 인자로 아무것도 들어오지 않으면
자동로또를 생성해주는 `generateLotto()` 함수를 호출하도록 수정했다.

이렇게 수정하니 로또 객체를 생성할 때  
1) 별도의 인자를 넘겨주지 않는다면 > `generateLotto()` 실행해 자동로또를 생성하고  
2) 숫자들을 인자로 넘겨준다면 > 해당 숫자들로 수동로또를 생성해주는 코드를  
**별도의 부 생성자 없이 주 생성자와 초기화 블록으로 해결**할 수 있었다.

이제 동반 객체의 팩토리 메소드를 활용하여 한 번 더 코드를 개선해보자.

## 개선한 코드
```kotlin
class Lotto private constructor(val numbers: Set<Int>) {
    init {
        validateSize()
        validateRange()
    }

    companion object {
        fun auto() = Lotto(generateLotto())
        fun manual(numbers: List<Int>) = Lotto(numbers.toSet())
    }
}
```

이렇게 자동로또와 수동로또를 생성해주는 로직을 모두 동반 객체의 팩토리 메소드로 작성해주면,  
1) 동반 객체의 팩토리 메서드에서는 **다양한 입력값에 대한 변환의 역할**만 담당하게 되고,  
2) 주 생성자는 **할당의 역할만 하게 되어 private으로 둘 수가 있다.**  
3) 또, 초기화 블록은 **들어온 값에 대한 검증의 역할**만 담당하게 된다.

이렇게 동반 객체의 팩토리 메소드를 활용해 인스턴스 생성코드를 작성해주면
메소드의 이름을 통해 **생성의 목적을 명시**해줄 수가 있고,
파라미터가 다른 생성 방법이 추가되는 경우에 대해 **확장에 유리**해진다.

이 예시와 같이 파라미터 목록이 다른 생성 방법이 여러 개 존재하는 경우, 인자에 대한 기본값을 제공하기 위해 부 생성자를 여러 개 만들지 말고 **디폴트 파라미터 값**과 **동반 객체의 팩토리 메소드**를 잘 활용하면 더 코틀린스럽게 클래스를 초기화할 수 있다!

## References
- [코틀린 인 액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791161750712&orderClick=LAG&Kc=)
