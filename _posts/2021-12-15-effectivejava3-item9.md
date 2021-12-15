---
layout: post
title: Book/Effective Java 3E/Item9. try-finally보다는 try-with-resources를 사용하라
date: 2021-12-15
excerpt: "이펙티브 자바 3판 - 아이템 9. try-finally보다는 try-with-resources를 사용하라"
tags: [book, effective_java, java]
comments: true
---

자바 라이브러리에는 `InputStream`, `OutputStream`, `java.sql.Connection` 과 같은
close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.
자원을 닫는 두 가지 방법 `try-finally`와 `try-with-resources`를 비교해보자.

### try-finally
전통적으로 자원을 닫는 방법으로 `try-finally`를 사용해왔다.
예외가 발생하거나 메서드에서 반환되는 경우를 포함해서 말이다.
```
static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

이 코드에서 만약, 자원이 둘 이상이라면 어떻게 될까?

```
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
			    out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

위의 두 코드는 바로 예외가 try 블록과 finally 블록에서 모두 발생할 수 있다는 것이다.
try를 하던 중 예외가 발생하더라도 close 메서드를 호출할 때 예외가 발생하면
**두 번째 예외만 남기 때문에 디버깅을 몹시 어려워진다.**

### try-with-resources
`try-finally`가 가지는 문제들을 자바 7부터 제공하는 `try-with-resources`에서 모두 해결해주었다.
(이 구조를 사용하려면 **해당 자원이 `AutoCloseable` 인터페이스를 구현해야 한다.**)
```
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		    out.write(buf, 0, n);
	}
}
```

이 코드는 위의 코드를 `try-with-resources`를 활용해서 다시 작성한 코드이다.
이 방법의 코드가 **훨씬 짧고 읽기 쉬울뿐 아니라 문제를 진단하기에도 훨씬 좋다.**
아까와 마찬가지로 두 가지 예외가 발생한다면 close에서 발생한 두 번째 예외는 숨겨지고
첫 번째 예외가 기록된다.

```
static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}
```
`try-finally`와 마찬가지로 `try-with-resources`에서도 catch 절을 쓸 수 있다.
catch 절 덕분에 **try 문을 중첩하지 않고도 다수의 예외를 처리할 수 있다.**
위의 코드는 파일을 열거나 데이터를 읽지 못했을 때 예외를 던지는 대신 기본값을 반환하도록 예외처리를 해준 코드이다.

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 대신, try-with-resources를 사용하자.
> 코드가 더 짧고 분명해질뿐 아니라, 만들어지는 예외 정보도 훨씬 유용하다.
> 무엇보다도, try-with-resources는 정확하고 쉽게 자원을 회수할 수 있다.
