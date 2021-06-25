---
layout: post
title: Book/모던자바인액션/ 8장. 컬렉션 API 개선
date: 2021-03-09
excerpt: "모던자바인액션 8장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

이 장에서는 자바 8, 자바 9에서 추가되어 프로그래밍을 더 편리하게 만들어 줄 새로운 컬렉션 API의 기능을 배운다.
## 컬렉션 팩토리
`Arrays.asList()` 팩토리 메서드로 리스트를 만들면 코드를 간단하게 줄일 수 있지만, 요소의 추가/삭제가 불가능하다.
```
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
friends.set(0, "Richard");
friends.add("Thibaut");
```
요소를 갱신하는 작업은 괜찮지만 요소를 추가하려 하면 `Unsupported OperationException`이 발생한다.
### 리스트 팩토리
`List.of` 팩토리 메서드를 이용해서 간단하게 리스트를 만들 수 있다.
```
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
friends.add("Chih-Chun");
```
하지만, 역시 요소를 추가하려하면 `Unsupported OperationException`이 발생한다.
컬렉션이 의도치 않게 변하는 것을 막기 위해 변경할 수 없는 리스트가 만들어졌기 때문이다.
### 집합 팩토리
`List.of`와 비슷한 방법으로 바꿀 수 없는 집합을 만들 수 있다.
```
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
`Set.of`에 중복된 요소를 넣으면 `IllegalArgumentException`이 발생한다. 집합은 중복 요소를 포함할 수 없기 때문이다.
### 맵 팩토리
맵을 만드려면 키와 값이 있어야 하기 때문에, 맵은 리스트나 집합보다는 조금 복잡하다.
자바 9에서 바꿀 수 없는 맵을 초기화하는 방법은 2가지가 있다.
```
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```
10개 이하의 키와 쌍을 갖는 작은 맵을 만들 때는 이 메소드가 유용하지만, 그 이상의 맵에서는
`Map.Entry<K,V>` 객체를 인수로 받으며 가변 인수로 구현된 `Map.ofEntries` 팩토리 메서드를 이용하는 것이 좋다.
```
Map<String, Integer> ageOfFriends = Map.ofEntries(entry("Raphael", 30),
                                                  entry("Olivia", 25),
                                                  entry("Thibaut", 26));                                                 
``` 

## 리스트와 집합 처리
자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다.
- removeIf: 프레디케이트를 만족하는 요소를 제거한다. List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다.
- replaceAll: 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다.
- sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

### removeIf 메서드
```
//기존 코드
for (Iterator<Transaction> iterator = transactions.iterator();iterator.hasNext();) {
    Transaction transaction = iterator.next();
    if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        iterator.remove();
    }
}
//removeIf를 통해 개선한 코드
transactions.removeIf(transaction ->
    Character.isDigit(transaction.getReferenceCode().charAt(0)));
```
### replaceAll 메서드
List 인터페이스의 `replaceAll`을 이용하면 리스트의 각 요소를 새로운 요소로 바꿀 수 있다.
```
referenceCodes.stream()
              .map(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1))
              .collect(Collectors.toList())
              .forEach(System.out::println);
```
하지만 이 코드는 새 문자열 컬렉션을 만든다. 우리가 원하는 것은 기존 컬렉션을 바꾸는 것이므로 replaceAll을 이용해 개선해보자.
```
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1));
```

## 맵 처리
### forEach 메서드
```
//기존 코드
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}
//개선한 코드
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```
### 정렬 메서드
`Entry.comparingByValue`와 `Entry.comparingByKey`를 이용하면 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.
```
Map<String, String> favoriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"),
                                    entry("Christina", "Matrix"),
                                    entry("Olivia", "James Bond"));
favoriteMovies.entrySet().stream()
              .sorted(Entry.comparingByKey())
              .forEachOrdered(System.out::println); //알파벳 순으로 처리
```
### getOrDefault 메서드
찾으려는 키가 존재하지 않을 때 NullPointerException이 발생하는 것을 방지하기 위해 `getOrDefault` 메서드를 사용할 수 있다.
`getOrDefault`는 요청 결과가 널이면 기본값을 반환한다.
```
favoriteMovies.getOrDefault("Olivia", "Matrix") //Olivia가 없으면 Matrix 출력
```
### 계산 패턴
맵에 키가 존재하는지 여부를 확인해야하는 경우가 있다. 이런 상황에서 다음의 세 가지 연산을 활용할 수 있다.
- `computeIfAbsent` : 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가한다.
키가 존재하지 않으면 값을 계산해 맵에 추가하고 키가 존재하면 기존 값을 반환한다.
- `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- `compute` : 제공된 키로 새 값을 계산하고 맵에 저장한다.

### 삭제 패턴
자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다.
```
favoriteMovies.remove(key, value);
```
### 교체 패턴
- `replaceAll` : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. List의 replaceAll과 유사하다.
- `Replace` : 키가 존재하면 맵의 값을 바꾼다.

### 합침
`putAll`을 사용하면 두 개의 맵을 합칠 수 있다. 하지만, 중복된 키에 유의해야 한다.
`forEach`와 `merge` 메서드를 이용하면 충돌을 해결할 수 있다.
```
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k,v) ->
        everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```
지정된 키와 연관된 값이 없거나 널이면 `merge`는 키를 널이 아닌 값과 연결한다.
아니면 `merge`는 연결된 값을 주어진 매핑 함수의 결과 값으로 대치하거나 결과가 널이면 항목을 제거한다.  
`merge`를 이용해 초기화 검사를 구현할 수도 있다.
```
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```
이 코드는 키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 값이 없거나 키에 널 값이 연관되어 있다면 이 값을 키와 연결한다.

## 개선된 ConcurrentHashMap
`ConcurrentHashMap`은 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다. 내부 자료구조의 특정 부분만 잠궈
동시 추가, 갱신 작업을 허용한다. 따라서, 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다.
### 리듀스와 검색
`ConcurrentHashMap`은 스트림에서 봤던 것과 비슷한 세 가지 연산을 지원한다.
- forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

다음처럼 키에 함수 받기, 값, Map.Entry, (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.
- 키, 값으로 연산(forEach, reduce, search)
- 키로 연산(forEachKey, reduceKeys, searchKeys)
- 값으로 연산(forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry.reduceEntries, searchEntries)

이 연산들은 `ConcurrentHashMap`의 상태를 잠그지 않고 연산을 수행하기 때문에, 이 연산들에 제공한 함수는
계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.  
또한, 이들 연산에 __병렬성 기준값__ 을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다.
기준값을 1로 지정하면 병렬성을 극대화하고, Long.MAX_VALUE로 지정하면 한 개의 스레드로 연산을 실행한다.
### 계수
`ConcurrentHashMap`은 맵의 매핑 개수를 반환하는 `mappingCount`를 제공한다.
기존 size 대신 새 코드에서는 int를 반환하는 `mappingCount`를 사용하는 것이 좋다.
그래야 매핑의 개수가 int의 범위를 넘어서는 상황에 대처할 수 있기 때문이다.
### 집합뷰
`ConcurrentHashMap`은 ConcurrentHashMap을 집합뷰로 반환하는 `KeySet`이라는 새 메서드를 제공한다.
맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다.
