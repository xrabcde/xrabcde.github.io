---
layout: post
title: 우테코/LV1 학습로그 정리
date: 2021-04-21
excerpt: "우아한테크코스 레벨 1기간 동안의 학습로그"
tags: [woowacourse, study_log]
comments: false
---

# 로또
## [JCF] Map - 2
### 내용
- 각 등수별 당첨 개수를 저장하기 위한 자료형이 필요해 사용하게 되었음
- 출력 시 5등부터 순서대로 출력하기 위해 key 순서대로 정렬해주는 자료형인 `TreeMap` 사용
- 총 당첨금액 계산 시 Map Iteration을 위해 `EntrySet` 사용

### 링크
- https://ithub.tistory.com/34
- https://stove99.tistory.com/96

## [OOP] Enum - 4
### 내용
- 각 등수별 당첨 금액과 몇 등인지를 저장하기 위해 enum 사용
- enum에 어디까지 책임을 줘야하는지 결정하는 부분이 어려웠음

### 링크
- https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html

# 블랙잭
## [OOP] 상속 - 4
### 내용
- Player와 Dealer가 모두 게임의 참가자라는 공통적 속성이 있기 때문에 상속을 활용해 보았음
- `Dealer is Player`는 맞지만, `Player is Dealer`는 아니기 때문에 Dealer가 Player를 상속받도록 설계
- 추상화와 상속 중 어떤 방법을 이용할지 고민했음

### 링크
- [`bce1e80`](https://github.com/woowacourse/java-blackjack/pull/121/commits/bce1e804585ad8a6bccdbeda086a08facb61eeec)

## [설계] TDD - 2
### 내용
- 도메인을 설계하기 전 TDD를 통해 도메인의 역할을 확실히 나눌 수 있었음
- controller와 view에 대한 테스트는 안 해도 되는걸까?
- 블랙잭과 같이 복잡한 로직을 갖는 경우, 작은 단위로 테스트 해볼 수 있어 좋았음
- 생성자 테스트에 assertThatCode와 doesNotThrowAnyException()을 이용

### 링크
- [`2b370b3`](https://github.com/woowacourse/java-blackjack/pull/121/commits/2b370b37f16b2efb954fc9f03e8b1d1a4b790938)
- [`e8efd98`](https://github.com/woowacourse/java-blackjack/pull/121/commits/e8efd986a4135bff212abeb8140fe8e609ed0915)

## [설계] MVC - 5
### 내용
- domain, view, controller로 구조를 잡음
- controller의 역할을 어떻게 나누면 구조를 더 개선할 수 있을지 고민됨

### 링크
- [`3f05220`](https://github.com/woowacourse/java-blackjack/pull/121/commits/3f0522065e5cbd4499b8f0b4ab9f1d51784c23e1)
- [`3baa0e6`](https://github.com/woowacourse/java-blackjack/pull/121/commits/3baa0e6ad7e21e782965a36ac6c870a5612d3044)
