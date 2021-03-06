---
layout: post
title: Book/모던자바인액션/ 7장. 병렬 데이터 처리와 성능
date: 2021-03-09
excerpt: "모던자바인액션 7장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

이 장에서는 스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 알아보자.
## 병렬 스트림
컬렉션에 `parallelStream`을 호출하면 병렬 스트림이 생성된다. 이를 이용하면 아주 간단하게 요소를 병렬로 처리할 수 있다.
다음은 숫자 n을 인수로 받아 1부터 n까지의 모든 숫자의 합계를 반환하는 메서드를 구현한 코드이다.
```
//기존 코드
public long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1) //무한 자연수 스트림 생성
                 .limit(n) //n개 이하로 제한
                 .reduce(0L, Long::sum); //모든 숫자를 더하는 리듀싱 연산
//병렬로 구현한 코드
public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
                 .limit(n)
                 .parallel() //스트림을 병렬 스트림으로 변환
                 .reduce(0L, Long::sum);
```
반대로 `sequential`로 병렬 스트림을 순차 스트림으로 바꿀 수도 있다. 병렬 스트림은 다음과 같은 과정으로 작동한다.
1. 스트림을 여러 청크로 분할
2. 각 청크를 병렬로 리듀싱 연산 수행
3. 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과 도출

## 스트림 성능 측정
하지만, 병렬화를 사용한다고 항상 성능이 좋아지는 것은 아니다. 그렇다면 성능에 영향을 주는 요인은 무엇이 있을까?
### 박싱 언박싱
```
public long iterativeSum() {
    long result = 0;
    for (long i = 1L; i <= N; i++) {
        result += i;
    }
    return result;
}
```
전통적인 `for 루프`를 사용해 반복하는 방법은 기본값을 박싱하거나 언박싱할 필요가 없으므로 앞에서 본 코드보다 더 빠르다.
### 분할이 어려운 경우
```
public long parallelSum() {
    return Stream.iterate(1L, i -> i + 1)
                 .limit(N)
                 .parallel()
                 .reduce(0L, Long::sum);
}
```
위 코드와 같이 병렬화 박싱을 적용한 코드는 순차코드보다 5배정도 느리다. 리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가
준비되지 않았으므로 분할할 수 없고, 결국 순차처리 방식과 크게 다른점이 없어 스레드 할당 오버헤드만 증가하게 되는 것이다.

## 병렬 스트림을 효과적으로 사용하기
- 확신이 서지 않으면 직접 측정하라
- 박싱을 주의하라
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다 (ex. 순서에 의존하는 연산)
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다
- 스트림을 구성하는 자료구조가 적절한지 확인하라 (ex. LinkedList보다 ArrayList가 효율적이다)
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다
- 최종 연산의 병합 과정 비용을 살펴보라
