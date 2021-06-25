---
layout: post
title: Book/모던자바인액션/ 9장. 리팩토링, 테스팅, 디버깅
date: 2021-03-09
excerpt: "모던자바인액션 9장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

람다 표현식을 이용해 가독성과 유연성을 높이려면 기존 코드를 어떻게 리팩터링해야 하는지 알아보자.
또한, 람다 표현식으로 전략, 템플릿 메서드, 옵저버, 의무 체인, 팩토리 등의 객체지향 디자인 패턴을 어떻게 간소화할 수 있는지도 살펴보자.
## 가독성과 유연성을 개선하는 리팩터링
람다, 메서드 참조, 스트림 등의 기능을 이용해 더 가독성 좋은 코드로 __리팩터링__ 하는 방법을 살펴보자.
### 코드 가독성 개선
일반적으로 __코드 가독성이 좋다__ 는 것은 __어떤 코드를 다른 사람도 쉽게 이해할 수 있음__ 을 의미한다.
즉, 내가 구현한 코드를 다른 사람이 __쉽게 이해하고 유지보수 할 수 있게 만드는 것__ 을 의미한다.
코드의 가독성을 높이기 위해 람다, 메서드 참조, 스트림을 활용한 3가지 리팩터링 방법이 있다.
### 1. 익명 클래스를 람다 표현식으로 리팩터링하기
익명 클래스는 코드를 장황하게 만들고 쉽게 에러를 일으키기 때문에 더 간결하고 가독성 좋은 코드를 구현하기 위해 __람다 표현식__ 을 이용하는 것이 좋다.
하지만, 모든 익명 클래스가 다 람다 표현식으로 변환가능한 것은 아니다.  
첫째, 익명 클래스에서 사용한 __this와 super__ 는 람다 표현식에서 다른 의미를 갖는다.
익명 클래스에서 this는 익명클래스 자신을 가리키지만 람다에서는 람다를 감싸는 클래스를 가리킨다.  
둘째, 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다. 하지만 __람다 표현식으로는 변수를 가릴 수 없다.__  
마지막으로, 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
익명 클래스는 인스턴스화할 대 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라지기 때문이다.
하지만, IntelliJ 등의 IDE에서는 이를 __자동으로 해결해주는 리팩터링 기능__ 을 제공한다!
### 2. 람다 표현식을 메서드 참조로 리팩터링하기
람다 표현식은 쉽게 전달할 수 있는 짧은 코드이지만, __메서드 참조__ 를 이용하면 더 가독성을 높일 수 있다.
메서드 참조의 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.
```
//람다 표현식
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
                        .collect(groupingBy(dish -> {
                                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                                    else return CaloricLevel.FAT;
}));
//메서드 참조
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
                        .collect(groupingBy(Dish::getCaloricLevel));
```
또한, `comparing`과 `maxBy` 같은 정적 헬퍼 메서드를 활용하는 것도 좋다.
`sum`, `maximum`등 자주 사용하는 리듀싱 연산은 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 제공한다.
내장 컬렉터를 이용하면 더 명확한 코드를 작성할 수 있다.
### 3. 명령형 데이터 처리를 스트림으로 리팩터링하기
스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다.
```
//스트림을 사용하지 않은 코드
List<String> dishNames = new ArrayList<>();
for(Dish dish: menu) {
    if(dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}
//스트림을 사용한 코드
menu.parallelStream().filter(d -> d.getCalories() > 300)
                     .map(Dish::getName)
                     .collect(toList());
```
## 람다로 객체지향 디자인 패턴 리팩터링하기
__디자인 패턴__ 은 다양한 패턴을 유형별로 정리한 것으로 공통적인 소프트웨어 문제를 설계할 떄 재사용할 수 있는, 검증된 청사진을 제공한다.
디자인 패턴에 람다 표현식을 이용하면 문제를 더 쉽고 간단하게 해결할 수 있다.
### 1. 전략
__전략 패턴__ 은 __한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법__ 이다.
다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법을 사용하거나, 입력 형식을 설정하는 등의 시나리오에 적용할 수 있다.
전략패턴은 다음 세 부분으로 구성된다.
- 알고리즘을 나타내는 인터페이스 (Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현 (ConcreteStrategyA, ConcreteStrategyB 같은 구체적인 구현 클래스)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

```
//람다 사용 전
Validator numericValidator = new Validator(new IsNumberic());
boolean b1 = numericValidator.validate("aaaa");
Validator lowerCaseValidator = new Validator(new IsAllLowwerCase());
boolean b2 = lowerCaseValidator.validate("bbbb");

//람다 사용 후
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa");
Validator lowewrCaseValidator = new Validator((String s) -> s.matches("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb");
```
위처럼 람다를 이용하면 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있다. 람다 표현식은 코드조각(또는 전략) 을 캡슐화한다.
### 2. 템플릿 메서드
알고리즘의 개요를 제시한 다음, __알고리즘의 일부를 고칠 수 있는 유연함을 제공__ 해야 할 때는 __템플릿 메서드__ 디자인 패턴을 이용하는 것이 좋다.
```
//람다 사용 전
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
    abstract void makeCustomerHappy(Customer c);
}
//람다 사용 후
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}
new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
            System.out.println("Hello " + c.getName());
```
람다를 사용하면 onlineBanking 클래스를 상속받지 않고 직접 람다 표현식을 전달해 다양한 동작을 추가할 수 있다. 
### 3. 옵저버
어떤 이벤트가 발생했을 때 __주체__ 가 되는 한 객체가 __옵저버__ 라 불리는 다른 객체 리스트에게 자동으로 알림을 보내야 하는 상황에서 __옵저버 디자인 패턴__ 을 사용한다.
```
//람다 사용 전
class Feed implements Subject {
    private final List<Observer> observers = new ArrayList<>();
    public void registerObserver(Observer o) {
        this.observers.add(o);
    }
    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}
//람다 사용 후
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY! " + tweet);
    }
});
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("queen")) {
        System.out.println("Yet more news from London... " + tweet);
    }
});
```
하지만, 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다
기존의 클래스 구현방식을 고수하는 것이 바람직할 수도 있다.
### 4. 의무 체인
작업 처리 객체의 체인(동작 체인 등)을 만들 때는 __의무 체인 패턴__ 을 사용한다.
한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한
다음에 또 다른 객체로 전달하는 식이다.  
이 패턴은 __합수 체인(함수 조합)__ 과 비슷하다. 작업 처리 객체를 `Function<String, String>`, 
더 정확히 표현하자면 `UnaryOperator<String>` 형식의 인스턴스로 표현할 수 있다. 
### 5. 팩토리
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 __팩토리 디자인 패턴__ 을 사용한다.
```
//람다 사용 전
public class ProductFactory {
    public static Product createProduct(String name) {
        switch(name) {
            case "loan" : return new Loan();
            case "stock" : return new Stock();
            case "bond" : return new Bond();
            default : throw new RuntimeException("No such product " + name);
        }
    }
}
//람다 사용 후
Supplier<Product> loanSupplier = Loan::new; 
Loan loan = loanSupplier.get();

final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}
```
하지만, 팩토리 메서드 역시 생성자로 여러 인수를 전달하는 상황에서는 적용하기 어렵다.
예를 들어, 세 인수를 받는 생성자라면 TriFunction이라는 특별한 함수형 인터페이스를 사용해야 하고,
결국 다음과 같이 Map의 시그니처가 복잡해진다.
```
public interface TriFunction <T, U, V, R> {
    R apply(T t, U u, V v);
}
Map<String, TriFunction<Integer, Integer, String, Product>> map = new HashMap<>();
```
## 람다 테스팅
프로그램이 의도대로 동작하는지 확인하기 위해 __단위 테스트__ 를 해볼 수 있다.
```
@Test
public void testMoveRightBy() throws Exception {
    Point p1 = new Point(5, 5);
    Point p2 = p1.moveRightBy(10);
    assertEquals(15, p2.getX());
    assertEquals(5, p2.getY());
}
```
### 보이는 람다 표현식의 동작 테스팅
위의 테스트코드는 `moveRightBy`가 public이므로 문제없이 작동한다.
하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없다.
따라서 필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.
```
public class Point {
    public final static Comparator<Point> compareByXAndThenY = 
        comparing(Point::getX).thenComparing(Point::getY);
    ...
}

@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```
### 람다를 사용하는 메서드의 동작에 집중하라
람다의 목표는 __정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것__ 이다.
람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.
```
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
    return points.stream()
                 .map(p -> new Point(p.getX() + x, p.getY()))
                 .collect(toList());
}
```
예를 들어, 위 코드에서 `p -> new Point(p.getX() + x, p.getY());` 는 다음과 같이 테스트할 수 있다.
```
@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points = 
            Arrays.asList(new Point(5, 5), new Point(10, 5));
    List<Point> expectedPoints = 
            Arrays.asList(new Point(15, 5), new Point(20, 5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
    assertEquals(expectedPointes, newPoints);
}
```
## 디버깅
문제가 발생한 코드를 디버깅할 때 개발자는 다음 두 가지를 먼저 확인해야 한다.
- 스택 트레이스
- 로깅  

하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화한다.
### 스택 트레이스 확인
예외 발생으로 프로그램 실행이 갑자기 중단되었다면 먼저 어디에서 멈췄고 어떻게 멈추게 되었는지 살펴봐야 한다.
유감스럽게도 람다표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성된다.
```
Exception in thread "main" java.lang.NullPointerException
    at Debugging.lambda$main$0(Debugging.java:6)
    at Debugging$$Lambda$5/284720968.apply(Unknown Source)
```
람다 표현식 내부에서 에러가 발생하면 람다 표현식은 이름이 없으므로 컴파일러가 람다를 참조하는 이름을 만들어낸다.
따라서 람다 표현식과 관련한 스택 트레이스는 이해하기 어려울 수 있다는 점을 염두에 두자.
