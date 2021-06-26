---
layout: post
title: Java/디자인패턴 - 책임 연쇄 패턴
date: 2021-06-20
excerpt: "JAVA 디자인패턴 중 책임 연쇄 패턴에 대한 정리"
tags: [java, oop, design_pattern]
comments: true
---

## 1. 정의
책임 연쇄 패턴 (Chain of responsibility pattern) 이란 **명령 객체와 일련의 처리 객체**를 포함하는 디자인 패턴으로,
요청을 처리할 수 있는 처리객체를 Chain으로 만들어 **결합을 느슨하게 하기 위해** 사용한다.
실제로 굉장히 많이 쓰이는 패턴 중 하나로, 요청을 처리할 수 있는 객체가 여러 개이고 처리객체가 특정적이지 않을 경우 권장된다.
객체지향 개발에서 **어떤 조건에 따라 코드가 분기되는 이벤트에 활용**할 수 있다.

## 2. 동작방식
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/chaining.png" alt="chaining.png">
</div>

- Handler : 요청을 받아 처리객체들에게 전달하는 인터페이스. 
첫 번째 Chain에 대한 정보만 가지고 있으며, 그 이후의 Chain 들은 알지 못한다.
- Concrete Handlers : 요청을 처리하는 실제 처리 객체
- Client : 요청을 전달하는 클라이언트

핸들러는 이벤트의 처리 로직을 분리하여 행동을 독립적으로 관리한다.
처리 객체를 연결리스트와 같은 사슬 방식으로 연결한 후에 자신이 수행하지 못하는 요청이라면
다음 객체에 넘기며 책임을 넘기는 방식으로 작동한다.

## 3. 특징
### 장점
- **복합적인 처리 방식에서 결합을 느슨하게 유지할 수 있다.**
- 클라이언트는 처리 객체의 내부 구조를 알 필요가 없다.
- Chain 내의 처리 순서를 변경하거나 처리 객체의 추가/삭제가 용이하다.
- 새로운 요청에 대한 처리 객체 생성이 편리하다.

#### 단점
- 충분한 디버깅을 거치지 않았을 경우 Chain 내부에서 순환이 발생할 수 있다.
- 디버깅과 테스트가 쉽지 않다.

## 4. 활용
- 동전을 거슬러주는 프로그램
- 거리에 따른 지하철 요금을 계산하는 프로그램

[지하철 노선도 미션](https://github.com/xrabcde/atdd-subway-fare/tree/step1)에서 
체이닝 되어있는 거리 요금 정책을 구현하기 위해 책임 연쇄 패턴을 적용해보았다.

> **< 거리별 요금 정책 >**  
> ~ 10km : 기본 요금 (1,250원)  
> 10 ~ 50km : 100원 / 5km  
> 50km ~ : 100원 / 8km

먼저, 처리 객체 Chain을 형성할 하나의 틀 역할을 하는 인터페이스 DistanceChain을 만든다.
거리 정책에 따라 3개의 클래스가 존재하지만 모두 이 인터페이스를 구현하여 다른 방법으로 요금을 계산하게 된다.

```
public interface DistanceChain {
    int calculate(int distance);
}
```

그리고 정책에 따른 계산 로직을 포함하는 3개의 클래스를 각각 DistanceChain을 이용해 만든다.

```
// ~ 10km : 기본 요금(1,250원)
public class DefaultDistance implements DistanceChain {
    private static final int BASIC_FARE = 1250; // 기본요금
    
    private final DistanceChain chain; // 다음 정책을 포함하는 클래스
    private final int threshold; // 정책 사이의 임계값(10)
    
    // 생성자를 통해 다음 Chain과 임계값을 주입
    public DefaultDistance(DistanceChain chain, int threshold) {
        this.chain = chain;
        this.threshold = threshold;
    }
    
    @Override
    public int calculate(int distance) {
        if (distance <= threshold) { // 계산하려는 거리가 임계값보다 작으면 기본요금
            return BASIC_FARE;
        }
        // 계산하려는 거리가 임계값을 넘어서면 다음 체인으로 계산 책임을 넘김
        return BASIC_FARE + this.chain.calculate(distance - threshold);
    }
}
```

```
// 10 ~ 50km : 100원 / 5km
public class SecondDistance implements DistanceChain {
    private static final int UNIT = 5; // 추가요금 단위 (5km)
    private static final int UNIT_FARE = 100; // 단위별 추가요금 (100원)
    
    private final DistanceChain chain; // 다음 정책을 포함하는 클래스
    private final int threshold; // 정책 사이의 임계값(50)
    
    public SecondDistance(DistanceChain chain, int threshold) {
        this.chain = chain;
        this.threshold = threshold;
    }
    
    @Override
    public int calculate(int distance) {
        // 계산하려는 거리가 임계값보다 크면 임계값만큼을 재귀를 통해 계산한 뒤
        // 다음 체인으로 계산 책임을 넘김 
        if (distance > threshold) {
            return calculate(threshold) + this.chain.calculate(distance - threshold);
        }
        // 계산하려는 거리가 임계값 이내이면 해당 정책의 계산 로직을 이용해 요금 계산
        return (int) ((Math.floor((distance - 1.) / UNIT) + 1) * UNIT_FARE);
    }
}
```

```
// 50km ~ : 100원 / 8km
public class ThirdDistance implements DistanceChain {
    private static final int UNIT = 8; // 추가요금 단위 (8km)
    private static final int UNIT_FARE = 100; // 단위별 추가요금 (100원)
    
    @Override
    public int calculate(int distance) {
        // 마지막 체인이므로 임계값과 비교 없이 바로 해당 정책의 계산 로직을 이용해 요금 계산
        return (int) ((Math.floor((distance - 1.) / UNIT) + 1) * UNIT_FARE);
    }
}
```

그리고 이 정책 클래스들을 체이닝 시켜주고 요금계산 요청을 받는 매니저(?) 역할을 하는 클래스를 하나 만들었다.

```
public class ChainOfDistance {
    // 이 클래스에서 거리 요금 정책의 임계값들을 모두 갖고 있음
    private static final int FIRST_THRESHOLD = 10;
    private static final int SECOND_THRESHOLD = 50;
    
    private final DistanceChain firstChain;
    
    public ChainOfDistance() {
        // 생성자를 통해 체이닝 구현
        // 맨 마지막 정책을 먼저 생성한 뒤, 이전 체인을 생성하며 주입시켜줌
        DistanceChain thirdChain = new ThirdDistance();
        DistanceChain secondChain = new SecondDistance(thirdChain, SECOND_THRESHOLD - FIRST_THRESHOLD);
        this.firstChain = new DefaultDistance(secondChain, FIRST_THRESHOLD);
    }
    
    public int calculate(int distance) {
        return firstChain.calculate(distance);
    }
}
```

이렇게 하면 이제 거리에 따른 요금을 계산할 때, 다음과 같이 ChainOfDistance의 calculate() 메서드만 호출하면 된다.

```
// subway/path/domain/fare/Fare.class
public int calculate(int extraFare, int distance) {
    // Fare 내부에서 ChainOfDistance 와 직접 의존성을 갖고 있음
    ChainOfDistance chainOfDistance = new ChainOfDistance();
    int fare = chainOfDistance.calculate(distance) + extraFare;
    ...
}
```

하지만, 이렇게 구현하고 리뷰요청을 보냈더니 리뷰어께서 체이닝은 잘 구현이 되었지만 [OCP를 지키는 구조로 리팩터링해보는 것을 제안](https://github.com/woowacourse/atdd-subway-fare/pull/28#discussion_r644097108)하셨다.
지금의 구조는 Fare 클래스 내부에서 ChainOfDistance를 직접 생성하고 있기 때문에 어떠한 방식으로 체이닝 되었는지가 드러나게 된다.
따라서, 책임연쇄패턴의 핵심인 **처리 객체 간 결합도를 느슨하게 하기** 와 **체이닝 구조를 외부에 드러내지 않기** 를 지키도록 리팩터링 해보았다.  

먼저, 기존의 Fare에서 ChainOfDistance 클래스에 직접 의존성을 갖고 있던 구조를 
**DistanceChain 인터페이스 의존성을 가지고 있는 구조**로 수정했다.
Spring을 사용하고 있으므로 @Bean 등록을 통해 체이닝 작업을 Config에서 하도록 수정했다. 

```
@Configuration
public class DistanceChainConfig {
    private static final int FIRST_THRESHOLD = 10;
    private static final int SECOND_THRESHOLD = 50;

    @Bean
    public DistanceChain defaultChain() {
        DistanceChain thirdChain = new ThirdDistance();
        DistanceChain secondChain = new SecondDistance(thirdChain, SECOND_THRESHOLD - FIRST_THRESHOLD);
        return new DefaultDistance(secondChain, FIRST_THRESHOLD);
    }
}
```

이렇게 하면 기존에 모든 Chain 객체들을 생성하고 체이닝 작업을 해주었던 ChainOfDistance 클래스가 필요없어지고
Fare 클래스에서 ChainOfDistance 클래스가 아닌 **DistanceChain 인터페이스에 의존성을 갖도록 구조가 바뀐다.**

```
public class Fare {
    private final DistanceChain defaultChain;
    private final AgeStrategy ageStrategy;

    public Fare(DistanceChain defaultChain, AgeStrategy ageStrategy) {
        // Fare 내부에서 인터페이스인 DistanceChain 의존성을 갖고, 생성자를 통해 주입받음
        this.defaultChain = defaultChain;
        this.ageStrategy = ageStrategy;
    }

    public int calculate(int extraFare, int distance) {
        int fare = defaultChain.calculate(distance) + extraFare;
        return ageStrategy.calculate(fare);
    }
}
```

첫 번째로 설계했던 구조는 정책이 변경될 시 모든 정책에 대한 정보와 체이닝 작업을 담당하는 **ChainOfDistance 클래스만 변경해주면 된다**는 장점이 있었지만,
책임 연쇄 패턴의 핵심인 **처리 객체 간 결합도를 느슨하게 하기**와 **체이닝 구조를 외부에 모르게 하기**가 지켜지지 않아 아쉬운 구조이다.
리뷰어가 조언해주신 대로 리팩터링했더니 이 두 가지 핵심을 지키면서 OCP를 잘 지키는 구조가 되었다.  

이 과정에서 리뷰어와 각자가 생각하는 책임연쇄패턴의 장점에 대해 긴 대화를 했다.
디자인 패턴은 기존에 존재했던 문제의 해결 방법을 패턴화하여 해결한다는 점도 중요하지만, 
상황에 맞는 패턴을 적용함으로써 개발과정에서 커뮤니케이션에 드는 비용을 줄이는 것도 중요하다.
나의 경우는 이번 패턴을 적용하면서 디자인 패턴의 목적 중 **커뮤니케이션 비용을 낮추는 목적**에는 위반되었다고 할 수 있다 ㅎㅎ..  
앞으로 디자인 패턴을 적용할 때는 이러한 목적도 고려해야겠다.

## 출처
- [참고링크1](https://always-intern.tistory.com/1)
- [참고링크2](https://k0102575.github.io/articles/2020-02/chain-of-responsibility-pattern)
