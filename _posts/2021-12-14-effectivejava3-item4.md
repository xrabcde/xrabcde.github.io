---
layout: post
title: Book/Effective Java 3E/Item4. 인스턴스화를 막으려거든 private 생성자를 사용하라
date: 2021-12-14
excerpt: "이펙티브 자바 3판 - 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라"
tags: [book, effective_java, java]
comments: true
---

정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 것이 아니다. 
하지만, 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자(매개변수를 받지 않는 public 생성자)를 만들어주기 때문에
사용자는 이 생성자가 자동 생성된 것인지 구분할 수 없다.  
인스턴스화를 막는 방법은 아주 간단하다.
**컴파일러가 자동으로 기본 생성자를 만들 수 없도록 private 생성자를 추가해주면 된다.**
```
public class UtilityClass {
    //기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    private UtilityClass() {
        throw new AssertionError();
    }
    ...
}
```
이 코드는 어떤 환경에서도 클래스가 인스턴스화되는 것을 막아준다.
명시적 생성자가 private이니 클래스 바깥에서는 접근할 수 없다.  
이 방식은 **상속을 불가능하게 하는 효과도 있다.**
모든 생성자는 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 하위 클래스가 상위 클래스의 생성자에 접근할 길이 막혀버린다.

