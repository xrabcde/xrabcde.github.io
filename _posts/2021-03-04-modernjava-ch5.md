---
layout: post
title: Book/모던자바인액션/ 5장. 스트림 활용
date: 2021-03-04
excerpt: "모던자바인액션 5장 내용 공부"
tags: [book, modern_java_in_action]
comments: false
---

이번에는 4장에서 살펴본 스트림 API가 지원하는 다양한 연산을 살펴보자.
## 필터링
### 프레디케이트로 필터링
스트림의 `filter` 메서드는 __프레디케이트__ 를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.
```
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian)
                                .collect(toList());
```
### 고유 요소 필터링
스트림의 `distinct` 메서드는 고유 요소로 이루어진 스트림을 반환한다. 즉, __중복을 제거__ 할 때 사용할 수 있다.
```
numbers.stream().filter(i -> i % 2 == 0)
        .distinct().forEach(System.out::println);
```
## 슬라이싱
### 프레디케이트로 슬라이싱
`filter` 연산을 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하게 된다.
하지만, 만약 이미 오름차순으로 정렬된 리스트라면 반복 작업을 줄일 수 있지 않을까? 이런 경우에 `takeWhile`을 사용할 수 있다.
```
List<Dish> slicedMenu1 = specialMenu.stream()
                            .takeWhile(dish -> dish.getCalories() < 320)
                            .collect(toList());
```
그렇다면 조건에 해당하지 않는 나머지 요소를 선택하려면 어떻게 할 수 있을까? `dropWhile`을 사용해보자.
```
List<Dish> slicedMenu2 = specialMenu.stream()
                            .dropWhile(dish -> dish.getCalories() < 320)
                            .collect(toList());
```
`dropWhile`은 `takeWhile`과 정반대의 작업을 수행한다.
처음으로 거짓이 되는 지점까지의 요소를 버리고, 그 지점에서 작업을 중단하고 남은 요소를 반환한다.
### 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 `limit(n)` 메서드를 지원한다.
```
List<Dish> dishes = specialMenu.stream()
                        .filter(dish -> dish.getCalories() > 300)
                        .limit(3)
                        .collect(toList());
```
위의 코드는 프레디케이트와 일치하는 처음 세 요소를 선택한 뒤 즉시 반환한다.
정렬되지 않은 스트림에도 `limit`을 사용할 수 있다. (`limit`의 결과도 정렬되지 않은 채 반환된다.)
### 요소 건너뛰기
스트림은 처음 n개 요소를 제외한 스트림을 반환하는 `skip(n)` 메서드를 지원한다.
만약 n개 이하의 요소를 갖는 스트림에 `skip(n)`을 호출하면 빈 스트림이 반환된다.
`limit(n)`과 `skip(n)`은 상호 보완적인 연산을 수행한다.
```
List<Dish> dishes = menu.stream()
                        .filter(d -> d.getCalories() > 300)
                        .skip(2)
                        .collect(toList());
```
## 매핑
### 스트림의 각 요소에 함수 적용하기
스트림은 함수를 인수로 받는 `map` 메서드를 지원한다. 
인수로 제공된 함수는 스트림의 각 요소에 적용된다.
```
List<String> dishNames = menu.stream()
                            .map(Dish::getName)
                            .collect(toList());
```
```
//단어 리스트가 주어졌을 때 각 단어의 글자수를 반환하는 코드
List<Integer< wordLengths = words.stream()
                                .map(String::length)
                                .collect(toList());
```
### 스트림 평면화
`map`을 응용해서 리스트에서 고유 문자로 이루어진 리스트를 반환해보자.
리스트의 각 단어를 문자로 매핑한 다음에 `distinct`로 중복 문자를 제거하면 해결되지 않을까?
```
words.stream().map(word -> word.split(""))
              .distinct().collect(toList());
```
하지만 이 코드에서 `map`으로 전달한 람다는 각 단어의 String[]을 반환하기 때문에 map메소드가 반환한 형식은 `Stream<String[]>`이다.
`flatMap`이라는 메서드를 이용하면 이 코드를 `Stream<String>`을 반환하도록 수정할 수 있다.  
#### map과 Arrays.stream 활용
우선 배열 스트림 대신 문자열 스트림이 필요하다.
```
String[] arrayOfWords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfWords);
words.stream().map(word -> word.split("")) //각 단어를 개별 문자열 배열로 변환
              .map(Arrays::stream) //각 배열을 별도의 스트림으로 생성
              .distinct()
              .collect(toList());
```
하지만, 이 코드는 엄밀히 따지면 `List<Stream<String>>`가 만들어지면서 문제가 해결되지 않았다.
#### flatMap 사용
```
List<String> uniqueCharacters = words.stream()
                                .map(word -> word.split("")) //각 단어를 개별 문자를 포함하는 배열로 변환
                                .flatMap(Arrays::stream) //생성된 스트림을 하나의 스트림으로 평면화
                                .distinct()
                                .collect(toList());
```
`flatMap`은 `map(Arrays::Stream)`과 달리 하나의 평면화된 스트림을 반환한다.
즉, `flatMap`은 스트림의 각 값을 다른 스트림으로 만든 뒤, 모든 스트림을 하나의 스트림으로 연결한다.
## 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 스트림 API로는 `allMatch`, `anyMatch`, `noneMatch`, `findFirst`, `findAny` 등이 있다.
### 프레디케이트가 적어도 한 요소와 일치하는지 확인
`anyMatch`를 이용하면 적어도 한 요소와 일치하는지 확인할 수 있다., `anyMatch`는 boolean을 반환한다.
```
if(menu.stream().anyMatch(Dish::isVegetarian)) {
    System.out.println("The menu is (somewhat) veegetarian friendly!!");
}
```
### 프레디케이트가 모든 요소와 일치하는지 검사
`allMatch`를 이용하면 모든 요소와 일치하는지 확인할 수 있다. 역시, boolean을 반환한다.
```
boolean isHealthy = menu.stream()
                        .allMatch(dish -> dish.getCalories() < 1000);
```
### 프레디케이트와 일치하는 요소가 없는지 확인
`noneMatch`는 `allMatch`와 반대 연산을 수행한다. 즉, 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다.
```
boolean isHealthy = menu.steram()
                        .noneMatch(d -> d.getCalories() >= 1000);
```
### 요소 검색
`findAny`는 현재 스트림에서 임의의 요소를 반환한다.
```
Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian)
                          .findAny();
```
#### Optional이란?
`Optional<T>`는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다.
만약, `findAny`를 통해 null이 반환된다면 에러를 일으킬 수 있으므로 `Optional<T>`를 사용하는 것이 좋다.
### 첫 번째 요소 찾기
논리적인 아이템 순서가 정해져 있는 일부 스트림에서는 첫 번째 요소를 반환할 때 `findFirst`를 사용할 수 있다.  
- `findFirst` vs `findAny`  
  findAny도 임의의 요소 한 개를 반환한다는 점은 같지만, 스트림의 병렬성 때문에 순서가 상관없다면 제약이 적은 findAny를 사용한다.

## 리듀싱
### 요소의 합
`reduce`를 이용하면 반복된 패턴을 추상화할 수 있다. 다음 예시 코드를 살펴보자
```
//스트림의 모든 요소를 더하기
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
//스트림의 모든 요소를 곱하기
int product = numbers.stream().reduce(1, (a, b) -> a * b);

//메서드 참조를 이용한다면?
int sum = numbers.stream().reduce(0, Integer::sum);
```
### 최댓값과 최솟값
최댓값과 최솟값을 찾을 때도 `reduce`를 활용할 수 있다. reduce는 두 인수를 받는다.
1. 초깃값
2. 스트림의 두 요소를 합쳐 하나의 값으로 만드는 데 사용할 람다  

이 때 `2.`에 두 요소에서 최댓값을 반환하는 람다를 넣어주면 최댓값을 구할 수 있다.
```
//최댓값 구하기
Optional<Integer> max = numbers.stream().reduce(Integer::max);
//최솟값 구하기
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```
