---
layout: post
title: Book/모던자바인액션/ 3장. 람다 표현식
date: 2021-02-25
excerpt: "모던자바인액션 3장 내용 공부"
tags: [book, modern_java_in_action]
comments: false
---

2장에서 `동작 파라미터화`를 이용하면 더 유연하고 재사용가능한 코드를 구현할 수 있다는 것을 학습했다.
3장에서는 더 깔끔한 코드를 위해 자바8의 `람다 표현식`에 대해 배워보자.
## 람다란 무엇인가?
__람다 표현식__ 은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다. 람다의 특징을 살펴보자.
- __익명__ : 보통의 메서드와 달리 이름이 없는 __익명__ 으로 표현된다.
- __함수__ : 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 __함수__ 라고 부른다.
- __전달__ : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- __간결성__ : 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.  

람다를 이용하면 동작 파라미터 형식의 코드를 더 쉽게 구현할 수 있으므로 코드가 간결하고 유연해진다.
```
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
```
이 코드에 람다를 적용하면 다음과 같이 바꿀 수 있다.
```
Comparator<Apple> byWeight =
        (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
람다는 세 부분으로 이루어진다.
- __파라미터 리스트__ : Comparator의 compare 메서드 파라미터 `(Apple a1, Apple a2)`
- __화살표__ : 람다의 파라미터 리스트와 바디를 구분 `->`
- __람다 바디__ : 람다의 반환값에 해당하는 표현식 `a1.getWeight().compareTo(a2.getWeight()`  

```
(parameters) -> expression 
(parameters) -> { statements; }
```

## 어디에, 어떻게 람다를 사용하는가?
### 함수형 인터페이스
2장에서 필터 메서드를 파라미터화하기 위해 사용했던 `Predicate<T>`가 바로 __함수형 인터페이스__ 이다.
함수형 인터페이스는 __정확히 하나의 추상 메서드를 지정하는 인터페이스__ 로, Comparator, Runnable 등이 있다.
```
public interface Predicate<T> {
    boolean test (T t);
}
public interface Comparator<T> {
    int compare (T o1, T o2);
}
public interface Runnable {
    void run();
}
``` 
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 __전체 표현식을 함수형 인터페이스의 인스턴스로 취급__ 할 수 있다. 
### 함수 디스크립터
람다 표현식의 시그니처를 서술하는 메서드를 __함수 디스크립터__ 라고 한다. `() -> void` 표기는 파라미터 리스트가 없으며 void를 반환하는 함수를 의미한다.

## 람다 활용 : 실행 어라운드 패턴
실제 자원을 처리하는 코드를 `설정`과 `정리` 두 과정이 둘러싸는 형태를 __실행 어라운드 패턴__ 이라고 부른다.
```
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine(); //실제 필요한 작업을 하는 행
    }
}
```
### 1단계 : 동작 파라미터화를 기억하라
위의 코드에서 한 번에 한 줄만 읽을 수 있는 문제를 어떻게 개선할 수 있을끼? 람다를 이용해서 processFile의 동작을 파라미터화해보자.
```
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
### 2단계 : 함수형 인터페이스를 이용해서 동작 전달
이번에는 `BufferedReader -> String`과 `IOException`을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어서 동작을 전달해보자.
```
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```
### 3단계 : 동작 실행
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있다.
```
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br); //BufferedReader 객체 처리
    }
}
```
### 4단계 : 람다 전달
이제 람다를 이용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.
```
//한 행을 처리하는 코드
String oneLine = processFile((BufferedReader br) -> br.readLine());
//두 행을 처리하는 코드
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
## 함수형 인터페이스 사용
다양한 함수형 인터페이스들을 알아보자.
### Predicate
`java.util.function.Predicate<T>` 인터페이스는 test라는 추상 메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환한다.
따로 정의할 필요없이 불리언 표현식이 필요한 상황에서 바로 사용할 수 있다.
### Consumer
`java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.
T 형식의 객체를 인수로 받아 어떤 동작을 수행하고 싶을 때 사용할 수 있다.
### Function
`java.util.function.Function<T, R>` 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다.
입력을 출력으로 매핑하는 람다를 정의할 때 활용할 수 있다.
### 기본형 특화
위의 인터페이스들 외에도 특화된 형식의 함수형 인터페이스도 있다. 제네릭 파라미터에는 참조형(`Byte`, `Integer`, `Object`, `List`)만 사용할 수 있다.
__박싱__ 은 기본형을 참조형으로 변환하는 기능이고, __언박싱__ 은 참조형을 기본형으로 변환하는 기능이다. 또한, 이들이 자동으로 이루어지는 __오토박싱__ 도 있다.  
자바 8에서는 이러한 오토방식 동작을 피할 수 있도록 IntPredicate과 같은 함수형 인터페이스를 제공한다.
```
public interface IntPredicate {
    boolean test(int t);
}
IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); //참(박싱 없음)
Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); //거짓(박싱)
```
## 형식 검사, 형식 추론, 제약
### 형식 검사
```
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```
어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 __대상 형식__ 이라고 부른다. 위의 예제에서는 다음과 같은 순서로 형식 확인 과정이 진행된다.
1. filter 메서드의 선언 확인
2. filter 메서드는 두 번째 파라미터로 `Predicate<Apple>` 형식을 기대
3. `Predicate<Apple>`은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스
4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 함  

### 같은 람다, 다른 함수형 인터페이스
대상 형식이라는 특징 때문에 같은 람다 표현식이라도 다른 함수형 인터페이스로 사용될 수 있다.
즉, __하나의 람다 표현식을 다양한 함수형 인터페이스에 사용__ 할 수 있다.
```
Comparator<Apple> c1 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
### 형식 추론
자바 컴파일러는 대상형식을 이용해서 람다 표현식과 관련된 함수형 인터페이스와 시그니처를 추론할 수 있다. 
때문에 람다 문법에서 이를 생략해서 코드를 더 단순하게 만들 수 있다.
```
//형식을 추론하지 않음
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
//형식을 추론함
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

### 지역 변수 사용
람다식에서 지역 변수는 `final`이 붙은 변경불가능한 지역변수만 사용해야 한다.

## 메서드 참조
__메서드 참조__ 는 특정 람다 표현식을 축약한 것이라고 생각하면 된다. 메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.
```
//기존 코드
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
//메서드 참조를 활용한 코드
inventory.sort(comparing(Apple::getWeight));
```
### 특징
람다 표현식 대신 명시적으로 메서드명을 참조함으로써 코드의 __가독성을 높일 수 있다__ 실제로 메서드를 호출하는 것은 아니므로 괄호는 필요 없다.
메소드 참조는 세 가지 유형으로 구분할 수 있다.
1. __정적 메서드 참조__ : 예를 들어 Integer의 parseInt 메서드는 Integer::parseInt로 표현할 수 있다.
2. __다양한 형식의 인스턴스 메서드 참조__ : 예를 들어 String의 length 메서드는 String::length로 표현할 수 있다.
3. __기존 객체의 인스턴스 메서드 참조__ : 예를 들어 Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고,
Transaction 객체에는 getValue 메서드가 있다면, 이를 expensiveTransaction::getValue라고 표현할 수 있다.

### 생성자 참조
`ClassName::new`처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.
```
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get(); //Supplier의 get 메서드를 호출해서 새로운 Apple 객체를 만들 수 있다.
```
## 람다, 메서드 참조 활용하기
지금까지 학습한 내용을 예제 코드에 적용해보자.
### 1단계 : 코드 전달
```
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());
```
### 2단계 : 익명 클래스 사용
한 번만 사용하는 `Comparator`는 위 코드보단 __익명 클래스__ 를 이용하는 것이 좋다.
```
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```
### 3단계 : 람다 표현식 사용
하지만, 여전히 코드가 가독성이 좋지 않다. __람다 표현식__ 을 이용해 더 간결하게 코드를 개선해보자.
```
inventory.sort((Apple a1, Apple a2) ->
                a1.getWeight().compareTo(a2.getWeight())
);
```
자바 컴파일러가 람다의 파라미터 형식을 추론할 수 있다고 학습했으므로 이 코드는 더 간결해질 수 있다.
```
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```
이 코드는 `comparing` 메서드를 이용하면 더 가독성을 높일 수 있다.
```
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
//더 간소화한 코드
inventory.sort(comparing(apple -> apple.getWeight()));
```
### 4단계 : 메서드 참조 사용
마지막으로, __메서드 참조__ 를 이용해 람다 표현식의 인수를 더 깔끔하게 전달해보자.
```
inventory.sort(comparing(Apple::getWeight));
```
이로써 코드가 간결해지고 의미도 명확해졌다!
