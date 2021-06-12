---
layout: post
title: 우테코/LV2 학습로그 정리
date: 2021-05-18
excerpt: "우아한테크코스 레벨 2기간 동안의 학습로그"
tags: [woowacourse, study_log]
comments: false
---

## 1. 체스 - step1,2
### REST API
- [REST API 제대로 알고 사용하기](https://meetup.toast.com/posts/92)
- `태그` : Spring

### Spring 데이터베이스 초기설정
- [Spring 데이터베이스 Schema 및 Data 초기설정하기](https://wan-blog.tistory.com/52)
- `태그` : Spring, MySQL, DB

### DTO
- 각 팀의 score를 dto를 사용해 전달하고 js에서 score 메세지를 생성하도록 했다.
- 에러 상황에 대해 js에서 alert를 띄우기 위해 ResponseDto를 만들어 status code와 에러메시지를 전달했다.

### JdbcTemplate
- JdbcTemplate를 사용해 DB의 정보에 접근했다.
- query, queryForList 등의 메서드를 학습했다.
- [JDBC 학습테스트](https://github.com/next-step/spring-learning-test/tree/jdbc)
- `태그` : Spring, DB

## 2. 체스 - step3
### 계층관계에 맞춰 API URI 설계하기
- URI 설계 시 슬래시(/)는 계층관계를 나타내는 데 사용
- 마지막 문자로 슬래시(/)를 포함하지 않는다.
  ```
  http://restapi.example.com/rooms/{roomId}/move
  http://restapi.example.com/rooms/{roomId}/board
  ```
- `태그` : Spring, REST API, HTTP
- [참고링크](https://meetup.toast.com/posts/92)

### DTO
- Data Transfer Object, 계층간 데이터 교환을 위해 사용하는 객체
- 클라이언트 요청에 포함된 데이터를 담아 서버측에 전달, 서버 측의 응답 데이터를 담아 클라이언트에 전달하는 계층간 전달자 역할
- `태그` : Spring Boot, Layer
- [참고링크](https://woowacourse.github.io/javable/post/2021-04-25-dto-layer-scope/)

## 3. 지하철1(노선도관리) - step1,2
### Acceptance Test
- 개발된 시스템이 고객의 요구사항과 일치하는지 확인하기 위해 고객의 입장에서 수행하는 테스트
- 초점이 기술쪽인 단위테스트와는 달리 초점이 비즈니스 쪽이다
- 목적
    - 확신 : 시스템이나 시스템의 일부 또는 특정한 비기능적인 특성에 대한 확신을 얻는 것
    - 배포가능성 평가 : 결함을 찾는 것이 아니라 시스템을 배포하거나 사용할 준비가 되었는지 평가
- `태그` : Test, TDD, ATDD

### BatchUpdate
- 구간정보를 한 번에 DB에 insert하기 위해 사용
  ```
  //내부적으로 batchList를 batchSize만큼 처리
  jdbcTemplate.batchUpdate(INSERT_SQL, batchList, batchSize, (ps, arg) -> {})
  jdbcTemplate.batchUpdate(INSERT_SQL, sections, sections.size(), (ps, arg) -> {})
  ```
- `태그` : Spring boot, JDBC, batchUpdate
- [참고링크](https://woowabros.github.io/experience/2020/09/23/hibernate-batch.html?fbclid=IwAR0mKmOVyJuLYF8N3uRkelNSdFxkK8Mw0DGg4HFC64T2p0XtvqZ-2y1Tusw)

### CustomException
- 커스텀 예외를 사용함으로써 얻을 수 있는 이점
    - 이름으로도 정보 전달이 가능
    - 상세한 예외 정보 제공
    - 예외에 대한 응집도 향상
    - 예외 발생 후 처리가 용이
    - 예외 생성 비용 절감
- 중복되는 역/노선의 이름을 검증하고 찾는 역/노선/구간이 없을 경우를 검증하기 위해 사용
- `태그` : Spring boot, Exception
- [참고링크](https://woowacourse.github.io/javable/post/2020-08-17-custom-exception/)

## 4. 지하철1(노선도관리) - step3
### Web Application Layer
![img](https://github.com/binghe819/TIL/raw/master/Network/Layer/image/web_layer.png)
- **Controller** : 웹 어플리케이션 최상위 영역, **외부 요청**인 사용자의 입력을 처리하고, 올바른 **응답을 사용자에게 반환**하는 역할.
- **Service** : Controller와 Dao의 중간 영역, Dao가 DB에서 받아온 **데이터를 전달받아 가공**, Domain Model을 묶어서 이 소프트웨어에서 사용 가능한 핵심 작업 집합을 설정하는 역할  
    == 이 소프트웨어가 수행해야 하는 작업을 명시  
    == 이 소프트웨어에게 내릴 수 있는 명령을 명시
- **Repository** : 가장 낮은 계층으로 사용되는 데이터 스토리지 계층과 통신하는 역할, **데이터베이스에 접근**하는 영역 (ex. DAO)
- 특정 계층에 속하는 요소들은 **동일한 혹은 그 아래에 있는 계층**에 속하는 요소를 사용할 수 있다.
- `태그` : Network, Web, Layer
- [참고링크1](https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/), [참고링크2](https://lifelife7777.tistory.com/100), [참고링크3](https://umbum.dev/1066), [참고링크4](https://github.com/binghe819/TIL/blob/master/Network/Layer/WebLayer.md)

### ControllerAdvice
- @Controller나 @RestController에서 발생한 예외를 한 곳에서 관리하고 처리할 수 있게 도와주는 어노테이션
- Controller에서 발생한 예외를 catch하여 처리하는 클래스
- 메서드 위에 @ExceptionHandler를 선언하여 등록한 예외를 catch
- `태그` : Spring, Annotation, Custom Exception, Exception Handle

## 5. 지하철2(로그인/경로조회) - step1,2
### Dispatcher Servlet
- Servlet Container로부터 들어오는 요청을 관제하는 컨트롤러
- 일반적으로 Servlet Container는 DispatcherServlet만 등록해놓고 Dispatcher Servlet이 HandlerMapping을 통해 적절한 Controller로 매핑
- `태그` : Spring, Spring MVC, Servlet

### Argument Resolver
- Controller에 들어오는 파라미터를 가공 (ex. 암호화된 내용 복호화)
- 파라미터를 추가하거나 수정해야 하는 경우 사용
    - Controller에서 파라미터를 가공/추가/수정할 수 있기 때문에 없어도 문제는 없다.
- 장점
    - 중복을 최소화해 깔끔한 코드 작성
    - **Controller의 파라미터에 대한 공통 기능 제공**
- `태그` : Spring, Spring MVC, Resolver

### Spring MVC 동작과정
![img](https://github.com/binghe819/TIL/raw/master/Spring/MVC/image/mvc-flow.png)
- Client Request 요청
- Dispatcher Servlet에서 해당 요청 처리
- Client Request에 대한 Handler Mapping
    1. RequestMapping에 대한 매칭 (RequestMappingHandlerAdapter)
    2. Interceptor 처리
    3. **Argument Resolver** 처리
    4. Message Converter 처리
- Controller Method invoke
- `태그` : Spring, Spring MVC
- [참고링크](https://github.com/binghe819/TIL/blob/master/Spring/MVC/Spring%20MVC%20flow.md)

## 6. 지하철2(로그인/경로조회) - step3

## 7. 지하철3(협업미션)
### 클래스에 abstract 혹은 final
- final 클래스 : 클래스에 final을 사용하게 되면 최종상태가 되어 더이상 상속 불가능
  final 로 선언되면 상속받을 수 없기 때문에 당연히 내부의 모든 메서드는 overriding 될 수 없음
- 클래스를 설계하는 단계에서 해당 클래스는 상속을 위한 클래스다! 혹은 상속을 고려하지 않는 클래스다! 라는 의도를 전달하는 것이 목적  
  => 많은 인원으로 프로젝트를 진행하거나, 코드가 장기간 유지보수 되는 경우
- [참고링크](https://velog.io/@ednadev/%EC%9E%90%EB%B0%94-%EC%B6%94%EC%83%81-%ED%81%B4%EB%9E%98%EC%8A%A4abstract-class-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A9%94%EC%84%9C%EB%93%9C%EC%99%80-final)

### 책임 연쇄 패턴
- 명령 객체와 일련의 처리 객체를 포함하는 디자인 패턴
- 체이닝 되어 있는 거리 요금 정책을 추상화하기 위해 사용
- 각 체인이 다음 계산을 위한 체인을 가짐
- 요청을 보내는 객체와 이를 처리하는 객체간의 결합도를 느슨하게 하기 위한 방법
- [참고링크](https://k0102575.github.io/articles/2020-02/chain-of-responsibility-pattern)  

=> 디자인패턴과 abstract/final 모두 **불필요한 커뮤니케이션 없이 코드를 통해 어떠한 목적을 이루고자 했는가를 나타내는 방법!!**

### Transactional
- **2개 이상의 쿼리를 하나의 커넥션으로 묶어** DB에 전송하고, 이 과정에서 에러가 발생할 경우 자동으로 롤백 처리
- 데이터의 추가 변경 또는 삭제가 일어나는 C, U, D 메서드에 @Transactional을 추가하여 트랜잭션 설정
- 단일 작업인 경우, 생략 가능
- [참고링크](https://mangkyu.tistory.com/50)
- [참고링크](https://velog.io/@kdhyo/JavaTransactional-Annotation-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90-26her30h)

### Exception 전략
- 통일된 예외 객체를 만들고, ControllerAdvice의 유지보수를 위해 exception을 한단계 추상화한 SubwayException을 만들었음
- SubwayException에 status 필드를 추가해 각 exception 별로 status를 가질 수 있도록 함
- ControllerAdvice에서는 SubwayException만 핸들링해주면 된다
- [참고링크](https://cheese10yun.github.io/spring-guide-exception/)
