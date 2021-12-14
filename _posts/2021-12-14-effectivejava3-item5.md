---
layout: post
title: Book/Effective Java 3E/Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
date: 2021-12-14
excerpt: "이펙티브 자바 3판 - 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
tags: [book, effective_java, java]
comments: true
---

많은 클래스가 하나 이상의 자원에 의존한다.
이를 구현할 수 있는 방법은 세 가지가 있다.

### 정적 유틸리티를 사용하는 방법
```
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {} //객체 생성 방지
    
    public static boolean isValid(String word) {...}
    public static List<String> suggestions(String typo) {...}
}
```
이 방식은 정적 유틸리티를 잘못 사용한 예시다.

### 싱글턴을 사용하는 방법
```
public class SpellChecker {
    private final Lexicon dictinoary = ...;
    
    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```
이 방식은 싱글턴을 잘못 사용한 예시다.  

위의 두 방식 모두 dictionary를 단 하나만 사용한다고 가정하기 때문에 유연하지도 않고 테스트하기 어렵다.
또한, 언어별로 다른 사전을 사용하거나 특수 어휘용 사전을 두는 경우에 대해서도 대응하기 어렵다.
이처럼 **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

### 의존 객체 주입을 사용하는 방법
의존 객체 주입이란 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**이다.
이 방법은 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다는 조건을 만족한다.
```
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker (Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {...}
    public List<String> suggestions(String typo) {...}
}
```
이 방식의 또 다른 장점은 바로 **불변을 보장한다는 것**이다. 
따라서, 같은 자원을 사용하려는 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있다.  
이 패턴을 조금 더 변형하면, 생성자에 자원 팩터리를 넘겨주는 방식으로 사용할 수도 있다.
팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다.
즉, 팩터리 메서드 패턴을 구현한 것이다.
이 방식을 사용해 클라이언트느 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다.  
의존 객체 주입이 유연성과 테스트 용이성을 개선해주긴 하지만, 의존객체 주입 프레임워크(Spring 등)를 사용하지 않으면 코드를 어지럽게 만들기도 한다.
이런 프레임워크는 의존 객체를 직접 주입하도록 설계된 API를 알맞게 응용해 사용하는 것이다.

> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
> 이 자원들을 클래스가 직접 만들게 해서도 안 된다.
> 필요한 자원을 생성자에 넘겨주는 의존객체 주입 기법을 사용하면 클래스의 유연성, 재사용성 테스트 용이성을 개선해준다.
