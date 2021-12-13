---
layout: post
title: Book/Effective Java 3E/Item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
date: 2021-12-13
excerpt: "이펙티브 자바 3판 - 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라"
tags: [book, effective_java, java]
comments: true
---

**싱글턴(singleton)**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.  
싱글턴을 만드는 방식은 두 가지가 있다.
두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 둔다.

### public static 멤버가 final 필드인 방식
```
public class Test {
    public static final Test INSTANCE = new Test();
    private Test() {..}
    
    public void method() {..}
}
```
이 방식에서 private 생성자는 public static final 필드인 INSTANCE를 초기화할 때 딱 한 번만 호출된다.
public이나 protected 생성자가 없으므로 Test 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.  
이 방식의 첫 번째 장점은 **해당 클래스가 싱글턴임이 API에 명백히 드러난다는 것**이다. 
public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
두 번째 장점은 바로 간결함이다.

### 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
```
public class Test {
    private static final Test INSTANCE = new Test();
    private Test() {..}
    public static Test getInstance() { return INSTANCE; }
    
    public void method() {..}
}
```
Test.getInstance는 항상 같은 객체의 참조를 반환하므로 INSTANCE가 아닌 다른 Test 인스턴스는 만들어지지 않는다.  
이 방식의 첫 번째 장점은 **API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점**이다. 
항상 같은 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 수정할 수도 있다.
두 번째 장점은 원한다면 **정적 팩터리를 제네릭 싱글턴 팩터리로 만들어 타입에 유연하게 대처할 수 있다는 것**이다.
세 번째 장점은 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점이다. 
이런 장점들을 굳이 사용하지 않는다면 위의 public 필드 방식이 더 좋다.

### 두 방식의 문제점
둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 추가적인 작업이 필요하다.
단순히, Serializable을 구현한다고 선언하기만 한다면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.
이를 방지하기 위해서는 **모든 인스턴스 필드에 transient(직렬화 제외) 키워드를 선언하고 readResolve 메서드를 제공해야 한다.**

```
// 싱글턴임을 보장해주는 메서드
private Object readResolve() {
    return INSTANCE;
}
```

### 원소가 하나인 열거 타입을 선언하는 방식
```
public enum Test {
    INSTANCE;
    
    public void method() {..}
}
```
싱글턴을 만드는 마지막 방법은 원소가 하나인 열거 타입을 선언하는 것이다.  
첫 번째 방식과 비슷하지만, **더 간결하고, 추가 노력없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나
리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.**
대부분 상황에서는 이 방법이 싱글턴을 만드는 가장 좋은 방법이다.
(단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.)
