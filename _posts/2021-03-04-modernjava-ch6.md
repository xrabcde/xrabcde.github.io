---
layout: post
title: Book/모던자바인액션/ 6장. 스트림으로 데이터 수집
date: 2021-03-04
excerpt: "모던자바인액션 6장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

명령형 프로그래밍이 아닌 __함수형 프로그래밍__ 을 이용하면 더 편리하고 명확한 프로그래밍을 할 수 있다.
## 리듀싱과 요약
reducing 대신 아래의 컬렉터들을 사용하면 프로그래밍적 편의성과 코드의 가독성을 개선할 수 있다.
- `counting` : 개수를 카운트
```
//기존 코드
long howManyDishes = menu.stream().collect(Collectors.counting());
//개선된 코드
long howMabyDishes = menu.stream().count();
```
- `maxBy`, `minBy` : 스트림의 최댓값, 최솟값 계산
```
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```
menu가 비어있다면 아무 요리도 반환하지 않기 때문에 `Optional<Dish>`를 사용한다.
- `summingInt` : 객체를 int로 매핑하는 함수를 인수로 받아 합을 계산
```
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
- `averagingInt` : 객체를 int로 매핑하는 함수를 인수로 받아 평균을 계산
```
double avgCalories = menu.stream().collect(averagingInt(Dish::GetCalories));
```
- `summarizingInt` : 요소 수, 합계, 평균, 최댓값, 최솟값 등을 계산
```
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```
- `joining` : 내부적으로 StringBuilder를 이용해 문자열을 하나로 만듦
```
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
//리스트를 콤마로 구분해 출력
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

### collect와 reduce의 차이
collect와 reduce는 동일한 기능을 구현할 수 있다. 하지만, collect는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면
reduce는 두 값을 하나로 도출하는 불변형 연산이라는 점에서 차이가 있다.
여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 리듀싱 연산을 병렬로 수행할 수 없다는 점도 다르다.

## 그룹화
그룹화를 명령형으로 구현하면 까다롭고, 에러도 많이 발생한다. 하지만, 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.
```
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
```
스트림의 각 요리에서 Dish, Type와 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달했다.
이 함수를 기준으로 스트림이 그룹화되므로 이를 __분류 함수__ 라고 부른다.
### 그룹화된 요소 조작
Map 형태로 되어있는 코드에 filtering을 적용해보자.
```
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
                    .collect(groupingBy(Dish::getType,
                            filtering(dish -> dish.getCalories() > 500, toList())));                 
```
`filtering`은 Collectors 클래스의 또 다른 정적 팩토리 메서드로 프레디케이트를 인수로 받는다.

## 분할
분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 두 개의 그룹으로 분류된다. (참 or 거짓)
```
Map<Boolean, List<Dish>> partitionedMenu =
            menu.stream().collect(partitioningBy(Dish::isVegetarian));
//결과
{false=[pork, beef, chicken, prawns, salmon],
true=[french fries, rice, season fruit, pizza]}
//참값의 키로 맵에서 모든 채식 요리를 얻을 수 있다.
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```
### 분할의 장점
분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.
이를 활용하면, 참인 그룹과 거짓인 그룹 각각에서 최댓값과 최솟값을 찾을 수 있다.
### 숫자를 소수와 비소수로 분할하기
`isPrime` 메서드와 `partitioningBy` 컬렉터로 숫자를 소수와 비소수로 분류할 수 있다.
```
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
                .collect(partitioningBy(candidate -> isPrime(candidate)));
}
```

## Collector 인터페이스
지금까지 살펴본 것과 같은 리듀싱 연산을 직접 만들 수도 있다. Collector 인터페이스를 직접 구현해서 더 효율적인 컬렉터를 만드는 방법을 알아보자.
```
//Collector 인터페이스
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```
- T : 수집될 스트림 항목의 제네릭 형식
- A : 수집 과정에서 중간 결과를 누적하는 객체의 형식
- R : 수집 연산 결과 객체의 형식

- `supplier` : 새로운 결과 컨테이너 만들기. 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수이다.
- `accumulator` : 결과 컨테이너에 요소 추가하기. 리듀싱 연산을 수행하는 함수를 반환한다.
- `finisher` : 최종 변환값을 결과 컨테이너로 적용하기. 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출한 함수를 반환해야 한다.
- `combiner` : 두 결과 컨테이너 병합. 리듀싱 연산에서 사용할 함수를 반환한다. 스트림의 서로 다른 서브파트를 병렬로 처리할 떄 누적자가 이 결과를 어떻게 처리할지 정의한다.
- `characteristics` : 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환한다. 다음 세 항목을 포함하는 열거형이다.
    - UNORDERED : 리듀싱 결과는 스트림 요소의 방분 순서나 누적 순서에 영향을 받지 않는다.
    - CONCURRENT : 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 스트림의 병렬 리듀싱을 수행할 수 있다.
    - IDENTITY_FINISH : 단순히 identity를 적용할 뿐이므로 생략가능하다. 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.
    
