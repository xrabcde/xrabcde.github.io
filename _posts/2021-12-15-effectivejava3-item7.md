---
layout: post
title: Book/Effective Java 3E/Item7. 다 쓴 객체 참조를 해제하라
date: 2021-12-15
excerpt: "이펙티브 자바 3판 - 아이템 7. 다 쓴 객체 참조를 해제하라"
tags: [book, effective_java, java]
comments: true
---

### 메모리 누수

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }
    
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위의 코드는 특별한 문제는 없어보이지만, 꼭꼭 숨어있는 **메모리 누수**의 위험성이 존재한다.
이 코드에서는 스택이 커졌다가 줄어들 때 스택에서 꺼낸 객체들을 가비지 컬렉터가 회수하지 않는다.
프로그램에서 그 객체들을 더 이상 사용하지 않더라도 말이다.
그 이유는 바로 이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.
elements 배열의 활성 영역(인덱스가 size보다 작은 원소들) 밖의 참조들이 모두 여기 해당한다.

가비지 컬렉션 언어에서는 메모리 누수를 찾기가 아주 까다롭고,
객체 참조 하나를 살려두면 그 객체가 참조하는 모든 객체를 회수하지 못하기 때문에
단 몇 개의 객체가 매우 많은 객체를 회수하지 못하게 함으로써 성능에 악영향을 줄 수 있다.

### 메모리 누수 해결

```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; //다 쓴 참조 해제
    return result;
}
```
이를 해결하는 방법은 간단하다. **해당 참조를 다 썼을 때 null 처리(참조 해제)**를 하면 된다.
예시 코드에서는 스택에서 꺼내질 때 각 원소의 참조가 필요없어지기 때문에 `pop()`에 참조 해제 코드를 추가했다.
또한, null 처리를 해주면 null 처리한 참조를 실수로 사용하려고 하는 경우에 대해서도 예외처리가 된다.

하지만, 모든 경우에 이런 식으로 코드를 작성하는 것은 프로그램을 지저분하게 만들 수 있다.
**객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.**
다 쓴 참조를 해제할 때는 null 처리 대신 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이 바람직하다.
그렇다면 null 처리를 해야하는 예외적인 경우는 언제일까?
자기 메모리를 직접 관리하는 Stack의 경우, 객체 자체가 아닌 객체 참조를 담는 elements 배열로
저장소 풀을 만들어 원소들을 관리한다.
가비지 컬렉터는 배열의 활성 영역과 비활성 영역을 구분할 능력이 없기 때문에 
개발자가 명시적으로 비활성 영역이 되는 순간 null 처리해서 해당 객체를 사용하지 않음을 가비지 컬렉터에 알려야 한다.
일반적으로 **자기 메모리를 직접 관리하는 클래스라면 개발자는 항상 메모리 누수에 주의해야 한다.**

### 캐시
**캐시 역시 메모리 누수를 일으키는 주범**이다. 
객체 참조를 캐시에 넣어두고 그 객체를 다 쓴 뒤에도 그냥 놔둘 경우 메모리 누수가 발생한다.
키를 참조하는 동안만 엔트리가 살아있는 캐시가 필요한 상황이라면 
WeakHashMap을 사용해 캐시를 만들어 다 쓴 엔트리는 즉시 자동으로 제거되도록 하자.

### 리스너 또는 콜백
메모리 누수의 세 번째 주범은 바로 **리스너 혹은 콜백**이라 부르는 것이다.
클라이언트가 콜백을 등록만 하고 명확히 해제하지 않는다면, 콜백은 계속 쌓여간다.
이럴 때 콜백을 약한 참조로 저장하면 (ex. WeakHashMap에 키로 저장) 가비지 컬렉터가 즉시 수거해간다.

> 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 
> 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다.
> 그래서 이런 예방법을 익혀두는 것이 매우 중요하다.

