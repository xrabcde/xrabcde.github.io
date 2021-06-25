---
layout: post
title: Spring/스프링 입문 강의 정리
date: 2021-05-04
excerpt: "코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술"
tags: [spring, inflearn]
comments: true
---

## 스프링 웹개발 기초
### 1. 정적 컨텐츠
- 파일 그대로 내려주는 거

### 2. 템플릿 엔진
- 템플릿 엔진으로 결과를 만들어 결과.html을 내려주는 거

### 3. API
- 객체를 json 스타일로 반환

## DI의 3가지 방법
### 1. 필드 주입
### 2. SETTER 주입
- setter가 public하게 열려있게 됨

### 3. 생성자 주입
- 처음 조립되는 시점에만 호출되고 그 뒤는 변경할 수 없음

## 의존관계 설정
### 1. 스프링 빈을 이용한 의존관계 설정
-  `@Controller`, `@Service`, `@Repository` 들을 자동으로 스캔해 빈 등록

### 2. 자바 코드로 직접 빈 등록
- 상황에 따라 구현 클래스를 변경해야 하는 경우에 유리
- `@Controller` 는 어노테이션 붙여줘야 함

## 개방 폐쇄 원칙 (OCP)
- 확장에는 열려있고, 수정에는 닫혀있다.
- 스프링의 DI를 사용하면 **"기존 코드를 전혀 손대지 않고, 설정만으로 구현 클래스를 변경"**할 수 있다.

## 통합테스트
- `@SpringBootTest` : 스프링 컨테이너와 테스트를 함께 실행
- `@Transactional` : 테스트 시작 전 트랜잭션 시작, 테스트 완료 후 항상 롤백  
  => db에 데이터가 남지 않아 **반복 테스트 가능!**

## JPA
- 반복 코드 해결, SQL도 직접 만들어 실행해줌
- SQL과 데이터 중심의 설계에서 **객체 중심의 설계**로 전환  
  => 개발 생산성 증가!
- 서비스 계층에서 `@Transactional` : 해당 클래스의 메서드를 실행할 때 트랜잭션 시작, 메서드가 정상 종료되면 트랜잭션 커밋, 런타임 예외가 발생하면 롤백
- JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.

## 스프링 데이터 JPA
- JPA를 편리하게 사용하도록 도와주는 기술
- 스프링 데이터 JPA가 `SpringDataJpaMemberRepository`를 스프링 빈으로 자동 등록
- 스프링 데이터 JPA 제공 기능
    - 인터페이스를 통한 기본적인 CRUD
    - `findByName()`, `findByEmail()` 처럼 메서드 이름만으로 조회 기능
    - 페이징 기능

## AOP : Aspect Oriented Programming
- 공통관심사항(cross-cutting concern)과 핵심관심사항(core concerrn)을 분리
    - ex) 시간을 측정하는 공통 관심 사항 분리  
      => 변경 필요 시 이 로직만 변경하면 됨!
- 핵심관심사항(서비스)을 깔끔하게 유지할 수 있다.
- AOP로 사용할 클래스에는 `@Aspect` 를 붙여준다.
- 메서드 앞에는 `@Around()`를 붙여준다. ex) @Around("execution(* hello.hellospring..*(..))")

