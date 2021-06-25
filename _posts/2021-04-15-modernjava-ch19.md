---
layout: post
title: Book/모던자바인액션/ 19장. 함수형 프로그래밍 기법
date: 2021-04-15
excerpt: "모던자바인액션 19장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---
## 일급 함수
일반값처럼 취급할 수 있는 함수를 **일급 함수**라고 한다. 함수형 프로그래밍 언어에선 함수를 일반값처럼
사용해서 인수로 전달하거나, 결과로 반환받거나, 자료구조에 저장할 수 있다.

### 고차원 함수
1. 하나 이상의 함수를 인자로 받거나
2. 함수를 결과로 반환하거나

둘 중 하나라도 조건이 맞으면 **고차원 함수**라고 부른다. 함수를 다룰 수 있는 또 다른 함수

### 커링
함수를 모듈화하고 코드를 재사용할 수 있는 기법 중 하나.
1. 인자를 여러 개 받는 함수를 분리하여, 인자를 하나씩만 받는 함수의 체인으로 만드는 방법
2. 다중 인수를 갖는 함수를 단일 인수를 갖는 함수들의 함수열로 바꾸는 방법

```
//기존 코드
static double converter(double x, double y, double z) {
    return x * y + z;
}

//커링 적용
static DoubleUnaryOperator curriedConverator(double y, double z) {
    return (double x) -> x * y + z;
}

//변환기(함수)를 반환하는 함수를 재사용
DoubleUnaryOperator convertCtoF = curriedConverter(9.0 / 5, 32);
DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);
DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);

convertCtoF.applyAsDouble(24);
convertUSDtoGBP.applyAsDouble(100);
convertKmtoMi.applyAsDouble(20);
```

## 영속 자료구조
자료구조를 갱신하면 그것을 참조하는 다른 곳에서 모두 영향을 받으므로 여러 부작용을 발생시킨다.
**결과 자료구조를 바꾸지 말라** 하지만, 자료구조 갱신 시마다 자료구조를 통째로 복사하는 건 낭비다.

### 트리 예제
```
zs = xs.append(ys);
```
다른 자료구조와 공통부분을 공유하여 메모리를 절약할 수 있다.
