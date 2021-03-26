---
layout: post
title: Book/모던자바인액션/ 13장. 디폴트 메서드
date: 2021-03-23
excerpt: "모던자바인액션 13장 내용 공부"
tags: [book, modern_java_in_action]
comments: false
---

인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 아니면 부모 클래스의 구현을 상속받아야 한다.
때문에 인터페이스를 바꾸고 싶을 때는 이전에 해당 인터페이스를 구현했던 모든 클래스도 고쳐야 한다는 문제가 발생한다.  
자바 8에서는 이 문제를 해결하기 위해 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법을 제공한다.
첫 번째는 인터페이스 내부의 __정적 메서드__ 를 사용하는 것이고, 두 번째는 인터페이스의 기본 구현을 제공할 수 있도록 
__디폴트 메서드__ 기능을 사용하는 것이다.

## 변화하는 API
API를 바꾸는 것이 왜 어려운지 예제를 통해 알아보자.
```
// Resizable 인터페이스의 초기 버전
public interface Resizable extends Drawable {
	int getWidth();
	int getHeight();
	void setWidth(int width);
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
}
```
```
// 사용자의 구현
public class Ellipse implementes Resizable {
	...
}

public class Game {
	public static void main(String...args) {
		List<Resizable> resizableShapes = 
			Arrays.asList(new Square(), new Rectangle(), new Ellipse());
		Utils.paint(resizableShapes);
	}
}
```

```
// setRelativeSize 메서드를 추가한 새로운 버전의 Resizable 
public interface Resizable extends Drawable {
	int getWidth();
	int getHeight();
	void setWidth(int width);
	void setHeight(int height);
	void setAbsoluteSize(int width, int height);
	void setRelativeSize(int wFactor, int hFactor);
}
```
### 사용자가 겪는 문제
- Resizable을 구현하는 모든 클래스는 setRelativeSize 메서드를 새로 구현해야 한다.
    - 인터페이스에 새로운 메서드를 추가하면 **바이너리 호환성**은 유지된다. (새로 추가된 메서드를 호출하지만 않으면 구현없이도 기존 클래스 구현이 잘 동작함)
- Ellipse를 포함하는 전체 어플리케이션을 재빌드하면 컴파일 에러가 발생한다.

## 디폴트 메서드란 무엇인가?
- 호환성을 유지하면서 API를 바꿀 수 있도록 해주는 기능
- 이름 그대로 인터페이스 자체에서 구현

```
public interface Sized {
	int size();
	default boolean isEmpty() { //디폴트 메서드
		return size() == 0;
	}
}
```
- 인터페이스를 구현하는 모든 클래스는 디폴트 메서드의 구현도 상속받음
- 호환성을 유지하며 라이브러리를 고칠 수 있음

```
default void setRelativeSize(int wFactor, int hFactor) {
	setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
}
```

### 추상 클래스와 인터페이스
1. 클래스는 하나의 추상 클래스만 상속받을 수 있지만 **인터페이스는 여러 개 구현가능**.
2. 추상 클래스는 인스턴스 변수를 가질 수 있지만, 인터페이스는 **인스턴스 변수를 가질 수 없음**.


## 디폴트 메서드 활용 패턴
### 1. 선택형 메서드
잘 사용하지 않는 메서드에 기본 구현을 제공할 수 있으므로 인터페이스를 구현하는 클래스에서 빈 구현을 제공할 필요가 없다.
→ **불필요한 코드를 줄일 수 있음**
```
interface Iterator<T> {
	boolean hasNext();
	T next();
	default void remove() {
		throw new UnsupportedOperationException();
	}
}
```

### 2. 동작 다중 상속
다중 상속을 이용해 기존 코드를 재사용할 수 있다.  
클래스는 하나만 상속 가능하지만, 인터페이스는 여러 개 구현 가능 → 자바 8에서는 인터페이스가 구현을 포함할 수 있으므로
**여러 인터페이스에서 동작을 상속가능!**

### 예시 코드
어떤 모양은 회전이 불가능하지만, 크기조절이 가능하고,  
어떤 모양은 회전가능하고 움직일 수 있지만 크기 조절이 불가능하다고 할 때
어떻게 최대한 기존 코드를 재사용해서 이 기능을 구현할 수 있을까?
```
public interface Rotatable {
	void setRotationAngle(int angleInDegrees);
	int getRotationAngle();
	default void rotateBy(int angleInDegrees) { //디폴트 메서드
		setRotationAngle((getRotationAngle() + angleInDegrees) % 360);
	}
}
```
Rotatable 인터페이스를 구현하는 클래스는 setRotationAngle과 getRotationAngle의 구현을 제공해야하지만,
rotateBy는 기본 구현이 제공되므로 구현을 제공하지 않아도 됨.
```
public interface Movable {
	int getX();
	int getY();
	void setX(int x);
	void setY(int y);

	default void moveVertically(int distance) {
		setY(getY() + distance);
	}
}
```
```
public interface Resizable {
	...
}
```


위 세 인터페이스를 조합해 만든 움직일 수 있고, 회전가능하며, 크기조절 가능한 Monster 클래스 
-> Rotatable, Moveable, Resizable의 디폴트 메서드를 **자동 상속 받음**

```
public class Monster implements Rotatable, Moveable, Resizable {
	...
}

Monster m = new Monster();
m.rotateBy(180); //Rotatable의 rotateBy 호출
m.moveVertically(10); //Moveable의 moveVertically 호출
```

움직일 수 있고, 회전가능하지만, 크기조절이 불가능한 Sun 클래스
```
public class Sun implements Moveable, Rotatable {
	...
}
```
- 디폴트 메서드를 수정하면 해당 인터페이스를 구현하는 모든 클래스에 자동 반영됨

## 해석 규칙
어떤 클래스가 같은 디폴트 메서드 시그니처를 포함하는 두 인터페이스를 구현하는 상황이라면 어떻게 될까?
이를 해결할 수 있는 규칙에 대해 알아보자.
```
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B extends A {
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B, A {
	public static void main(String... args) {
		new C().hello(); //무엇이 출력될까?
	}
}
```
### 알아야 할 세 가지 해결 규칙
1. **클래스가 항상 이긴다**.
클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 **서브 인터페이스가 이긴다**.
상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다.
즉, B가 A를 상속받는다면, B가 A를 이긴다.
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는
**클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출**해야 한다.
   
### 디폴트 메서드를 제공하는 서브인터페이스가 이긴다
```
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B extends A {
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B, A {
	public static void main(String... args) {
		new C().hello(); //Hello from B
	}
}
```
```
public class D implements A {  }
public class C extends D implements B, A {
	public static void main(String... args) {
		new C().hello(); //무엇이 출력될까?
	}
}
```
클래스나 슈퍼클래스에 **메서드에 대한 정의가 없을 때**는 디폴트 메서드를 정의하는 서브인터페이스가 선택된다.
```
public class D implements A {
	void hello() {
		System.out.println("Hello from D");
	}
}

public class C extends D implements B, A {
	public static void main(String... args) {
		new C().hello(); //무엇이 출력될까?
	}
}
```
D가 명시적으로 A의 hello를 오버라이드

### 충돌 그리고 명시적인 문제 해결
```
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B {
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B, A { }
```
⇒ `Error: class C inherits unrelated defaults for hello() from types B and A.`

⇒ 개발자가 직접 클래스 C에서 사용하려는 메서드를 명시적으로 오버라이드 해야 함 `X.super.m()`
```
public class C implements B, A {
	void hello() {
		B.super.hello(); //명시적으로 인터페이스 B의 메서드를 선택
	}
} 
```
### 다이아몬드 문제
```
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}
public interface B extends A { }
public interface C extends A { }
public class D implements B, C {
	public static void main(String... args) {
		new D().hello(); //무엇이 출력될까?
	}
} 
```
UML 다이어그램이 다이아몬드 형태인 시나리오를 다이아몬드 문제라고 부름.

1. A만 디폴트 메서드를 정의하고 있는 상황 ? ⇒ `Hello from A`
2. B에서도 hello를 정의한다면? ⇒ `Hello from B`
3. B와 C에서 모두 hello를 정의한다면? ⇒ `충돌! 명시적으로 호출해야 함`
