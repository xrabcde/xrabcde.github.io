---
layout: post
title: Book/모던자바인액션/ 11장. null 대신 Optional 클래스
date: 2021-03-10
excerpt: "모던자바인액션 11장 내용 공부"
tags: [book, modern_java_in_action]
comments: false
---

자바로 프로그램을 개발하면서 한 번도 `NullPointerException`을 겪지 못한 사람은 없을 것이다.
Optional을 사용하면 이러한 상황들을 방지할 수 있다.
## 값이 없는 상황을 어떻게 처리할까?
### 보수적인 자세로 NullPointerException 줄이기
예기치 않은 NullPointerException을 피하려면 어떻게 해야할까?
다음은 null 확인 코드를 추가해서 NullPointerException을 줄이려는 코드다.
```
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```
위의 메서드에서는 모든 변수가 null인지 의심하므로 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다.
이를 반복하다보면 코드의 구조가 엉망이 되고 가독성도 떨어진다. 다음 코드는 다른 방법으로 이 문제를 해결하는 코드다.
```
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
``` 
위 코드에서는 `null` 변수가 있으면 즉시 `Unknwon`을 반환하는 방법으로 중첩 if 블록을 없앴다.
하지만, 이 예제도 그렇게 좋은 코드는 아니다. 메서드에 네 개의 출구가 생겼기 때문에 유지보수가 어려워진다.
### null 때문에 발생하는 문제
자바에서 null 참조를 사용하면서 발생할 수 있는 이론적, 실용적 문제를 확인하자.
- __에러의 근원이다__ : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
- __코드를 어지럽힌다__ : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.
- __아무 의미가 없다__ : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적저하지 않다.
- __자바 철학에 위배된다__ : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터이다.
- __형식 시스템에 구멍을 만든다__ : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다.

## Optional 클래스 소개
`Optional`은 선택형값을 캡슐화하는 클래스다. 예를 들어 어떤 사람이 차를 소유하고 있지 않다면 Person 클래스의 car 변수는 null을
가져야 할 것이다. 하지만, 새로운 Optional을 이용할 수 있으므로 null을 할당하는 것이 아니라 변수형을 `Optional<Car>`로 설정할 수 있다.  
값이 있으면 Optional 클래스는 값을 감싼다. 반면, 값이 없으면 `Optional.empty` 메서드로 Optional을 반환한다.
`Optional.empty`는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드다.  
null 참조와 Optional.empty()는 의미상으론 비슷하지만 실제로는 차이점이 많다.
null을 참조하려 하면 `NullPointerException`이 발생하지만 `Optional.empty()`는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.
## Optional 적용 패턴
이제 Optional을 실제로 어떻게 활용할 수 있는지 알아보자.
### Optional 객체 만들기
1. 빈 Optional
정적 팩토리 메서드 Optional.empty로 빈 Optional 객체를 얻을 수 있다.  
Optional<Car> optCar = Optoinal.empty();
2. null이 아닌 값으로 Optional 만들기
또는 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
이제 car가 null이라면 즉시 `NullPointerException`이 발생한다.  
Optional<Car> optCar = Optional.of(car);
3. null값으로 Optional 만들기
마지막으로 정적 팩토리 메서드 `Optional.ofNullable`로 null값을 지정할 수 있는 Optional을 만들 수 있다.
car가 null이면 빈 Optional 객체가 반환된다.  
Optional<Car> optCar = Optional.ofNullable(car);

### 맵으로 Optional의 값을 추출하고 변환하기
get 메서드를 이용해서 Optional의 값을 가져올 수 있지만, 만약 Optional이 비어있으면 예외가 발생한다.
이런 유형의 패턴에 사용할 수 있도록 Optional은 map을 지원한다.
```
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```
Optional은 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾸고, 비어있으면 아무 일도 일어나지 않는다.
### flatMap으로 Optional 객체 연결
map을 사용하는 방법을 배웠으므로 map을 이용해 코드를 재구현할 수 있다.
```
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar)
                                 .map(Car::getInstance)
                                 .map(Insurance::getName);
```
하지만, 안타깝게도 위 코드는 컴파일 되지 않는다. 
변수 optPeople의 형식은 Optional<People>이므로 map을 호출할 수 있지만, getCar는 Optional<Car> 형식의 객체를 반환한다.
즉, map 연산의 결과는 Optional<Optional<Car>> 형식이 되는 것이다.  
스트림의 `flatMap`을 이용하면 이러한 문제를 해결할 수 있다. flatMap은 함수를 인수로 받아서 다른 스트림을 반환하는 메서드이다.
```
String name = person.flatMap(Person::getCar)
                    .flatMap(Car::getInsurance)
                    .map(Insurance::getName)
                    .orElse("Unknown");
```
