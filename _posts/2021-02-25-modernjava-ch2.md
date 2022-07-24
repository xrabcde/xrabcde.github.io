---
layout: post
title: Book/모던자바인액션/ 2장. 동작 파라미터화 코드 전달하기
date: 2021-02-25
excerpt: "모던자바인액션 2장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

동작 파라미터화는 변화하는 요구사항에 유연하게 대응할 수 있게 코드를 구현하는 방법 중 하나로, 메서드 내부적으로 다양한 동작을
수행할 수 있도록 코드를 메서드 인수로 전달한다. 동작 파라미터화를 이용하면 다음과 같은 기능을 수행할 수 있다.
- 리스트의 모든 요소에 대해서 `어떤 동작`을 수행할 수 있음
- 리스트 관련 작업을 끝낸 다음에 `어떤 다른 동작`을 수행할 수 있음
- 에러가 발생하면 `정해진 어떤 다른 동작`을 수행할 수 있음

## 변화하는 요구사항에 대응하기
기존의 농장 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링하는 기능을 추가하는 상황을 가정하자.
### 첫 번째 시도 : 녹색 사과 필터링
사과 색을 정의하는 `enum Color { RED, GREEN }`이 존재한다고 가정하면
```java
public static List<Apple> filterGreenApples (List<Apple> inventory) {
    List<Apple> result = new ArrayList<>(); //사과 누적 리스트
    for (Apple apple : inventory) {
        if (GREEN.equals(apple.getColor()) { //녹색 사과만 선택
            result.add(apple);
        }
    }
    return result;
}
```
이 경우, 녹색이 아닌 다른 사과를 필터링하고 싶을 땐 새로운 메서드를 만들어야 한다는 문제가 있다.
즉, 다양한 변화에 적절하게 대응할 수 없다. __거의 비슷한 코드가 반복된다면 코드를 추상화한다__ 규칙을 이용해 코드를 개선시켜보자.
### 두 번째 시도 : 색을 파라미터화
색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응할 수 있다.
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
    return result;
}
```
```java
filterApplesByColor(inventory, GREEN); //녹색 사과만 선택

filterApplesByColor(inventory, RED); //빨간 사과만 선택
``` 
그러나 이 경우 역시, 색깔이 아닌 무게로 사과를 필터링하고 싶을 땐 또 새로운 메서드를 만들어야 한다는 문제가 있다.
### 세 번째 시도 : 가능한 모든 속성으로 필터링
```java
public static List<Apple> filterApple (List<Apple> inventory, Color color,
                                        int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (flag && apple.getColor().equals(color) ||
            (!flag && apple.getWeight() > weight)) {
...
```
와 같은 코드는 모든 속성을 필터링할 수는 있지만 형편없는 데다가 요구사항이 바뀔 때마다 유연하게 대응할 수도 없다.
이제 이러한 상황에서 `동작 파라미터화`를 이용해서 유연성을 얻는 방법을 알아보자.

## 동작 파라미터화
색상, 무게 등의 선택 조건을 사과의 어떤 속성에 기초해서 `boolean`값을 반환하는 방법이 있다. 
참 또는 거짓을 반환하는 함수를 __프레디케이트__ 라고 한다. 선택조건을 결정하는 인터페이스를 정의하자.
```java
public interface ApplePredicate {
    boolean test (Apple apple);
}
```
```java
public class AppleHeavyWeightPredicate implements ApplePredicate { //무거운 사과만 선택
    public boolean test (Apple apple) {
        return apple.getWeight() > 150;
    }
}
public class AppleGreenColorPredicate implements ApplePredicate { //녹색 사과만 선택
    public boolean test (Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```
이와 같이 각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음, 런타임에 알고리즘을 선택하는 기법을 __전략 디자인 패턴__ 이라고 한다.  
`ApplePredicate`가 알고리즘 패밀리고 `AppleHeavyWeightPredicate`와 `AppleGreenColorPredicate`가 전략이 되는 것이다.
이제 `filterApples` 메서드가 `ApplePredicate` 객체를 인수로 받도록 코드를 수정해보자.
### 네 번째 시도 : 추상적 조건으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) { //프레디케이트 객체로 사과 검사 조건을 캡슐화했다.
            result.add(apple);
        }
    }
    return result;
}
```
```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public boolean test (Apple apple) {
        return RED.equals(apple.getColor()) && apple.getWeight() > 150;
    }
}
```
`filterApples` 메서드의 동작을 파라미터화하였더니 첫 번째 코드에 비해 더 유연하고 가독성 좋은 코드가 되었다.
이처럼 동작 파라미터화는 __한 메서드가 다른 동작을 수행하도록 재활용__ 할 수 있고, 이는 유연한 API를 만들 때 중요한 역할을 한다.  
지금까지 동작을 추상화해서 변화하는 요구사항에 대응가능한 코드를 작성하는 방법을 알아보았다. 
이번에는 여러 클래스를 구현해서 인스턴스화하는 과정을 좀 더 개선해보자.

## 복잡한 과정 간소화
자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 `익명 클래스`라는 기법을 제공한다. 익명 클래스란 말 그대로 이름이 없는 클래스이고,
이를 이용하면 즉석에서 필요한 구현을 만들어서 사용할 수 있다는 장점이 있다.
### 다섯 번째 시도 : 익명 클래스 사용
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
}); //filterApples 메서드의 동작을 직접 파라미터화했다.
```
하지만 익명클래스는 여전히 부족한 점이 있다.
- 여전히 많은 공간을 차지한다는 것
- 많은 프로그래머가 익명 클래스 사용에 익숙하지 않다는 것  

익명 클래스로 코드를 조금 개선해볼 수는 잇지만 여전히 만족스럽지 않다. (결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않는다.)
### 여섯 번째 시도 : 람다 표현식 사용
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
`람다 표현식`을 이용하면 코드가 훨씬 간결해지고 가독성이 좋아진다.
### 일곱 번째 시도 : 리스트 형식으로 추상화
```java
public interface Predicate<T> {
    boolean test(T t);
}
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```
코드를 리스트 형식으로 추상화하면 사과뿐 아니라 바나나, 오렌지, 정수, 문자열 등 다양한 타입에 대해 메소드가 적용가능해진다.
```java
filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
filter(numbers, (Integer i) -> i % 2 == 0);
```
