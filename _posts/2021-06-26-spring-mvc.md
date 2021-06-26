---
layout: post
title: Spring/스프링 MVC 흐름 - Dispatcher Servlet
date: 2021-06-26
excerpt: "Spring MVC Flow - Dispatcher Servlet"
tags: [spring, MVC]
comments: true
---
Spring MVC는 프론트 컨트롤러 패턴으로 구현되어 있다. 스프링 MVC의 핵심인 Dispatcher Servlet이
스프링 MVC의 프론트 컨트롤러 역할을 한다.
**Dispatcher Servlet에 요청이 들어와서 응답으로 반환될 때까지의 흐름**을 알아보자.

## Dispatcher Servlet Flow
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/springmvc1.jpg" alt="springmvc1.jpg">
</div>

1. **getHandler()**  
  Dispatcher Servlet에 요청이 들어오면 먼저 getHandler() 메서드를 통해 `HandlerMapping` 객체가
핸들러를 조회 후 찾은 Handler에 대한 `HandlerExecutionChain`을 반환한다.
2. **getHandlerAdapter()**  
  1.에서 찾은 `HandlerExecutionChain`을 이용해 `HandlerAdapter`를 찾아 반환한다.
3. **applyPreHandle()**  
  요청에 대해 컨트롤러를 실행하기 전 처리하고 싶은 로직을 `HandlerInterceptor`에서 `preHandle()`을
이용해 처리해준다. (ex. 인증과 인가) 이 때, `preHandle()`은 반환값으로 boolean을 사용하는데 만약 반환값이 `false`라면 
컨트롤러 로직을 실행하지 않고 `afterCompletion`을 실행한다.
4. **handle()**  
  조회를 통해 찾아낸 `HandlerAdapter`에서 해당 핸들러에 대해 `ArgumentResolver`를 처리하고 Handler를 실행한다.
이 때, Handler를 실행한다는 의미는 Service > Repository > DB 로 이어지는 비즈니스 로직을 실행한다는 뜻이다.
`HandlerAdapter`는 handle()의 반환값으로 `ModelAndView`를 반환한다.  
5. **applyPostHandle()**  
  `HandlerInterceptor`에서 컨트롤러 실행 후 하고 싶은 로직을 `postHandle()`로 처리한다. 
`preHandle()`과 달리 `postHandle()`은 반환값이 없고, 역순으로 처리된다.
6. **render()**  
  `ViewResolver`가 `resolveViewName()`을 통해 View의 이름으로 View 객체를 조회 후 반환한다. 
7. **view.render()**  
  찾은 View에 Model 데이터를 렌더링 후 요청의 응답값을 `DispatcherServlet`이 클라이언트에게 반환한다. 

## 결론
복잡한 Spring MVC의 동작 원리를 다 이해하는 것은 쉽지 않지만, 이렇게 핵심 동작방식을 알아두면
향후 개발과정에서 문제가 발생했을 때 **어떤 부분에서 문제가 발생했는지 쉽게 파악**하고, 문제를 해결할 수 있을 것이다. 

## 출처
- [참고링크1](https://github.com/binghe819/TIL/blob/master/Spring/MVC/Spring%20MVC%20flow.md)
