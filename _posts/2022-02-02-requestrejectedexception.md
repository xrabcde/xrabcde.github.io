---
layout: post
title: Spring/RequestRejectedException 핸들링
date: 2022-02-02
excerpt: "Spring Security - RequestRejectedException 핸들링하기"
tags: [spring, exception, spring_security]
comments: true
---

# 개요
서버 모니터링과 장애 알람을 위해 slack-appender를 사용해서 500에러 발생에 대한 슬랙 알림을 받고 있었는데
어느 순간부터 간헐적으로 아래와 같은 알림이 DEV와 PROD에서 모두 발생했다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/requestrejectedexception1.png" alt="requestrejectedexception1.png">
</div>
<br>

```
[ERROR] [http-nio-8081-exec-10] [[dispatcherServlet]:?] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception

org.springframework.security.web.firewall.RequestRejectedException: The request was rejected because the URL contained a potentially malicious String "//"
    at org.springframework.security.web.firewall.StrictHttpFirewall.rejectedBlocklistedUrls(StrictHttpFirewall.java:456)
    at org.springframework.security.web.firewall.StrictHttpFirewall.getFirewalledRequest(StrictHttpFirewall.java:429)
    at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:196)
    at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:183)
    ...
```

읽어보면 요청 URL에 malicious한 문자열(`//`, `%2e` 등)이 포함되어 요청이 거부되었다는 뜻인데,
spring security에서 XSS 공격을 방지하기 위해 해당 문자열이 포함된 요청을 거부하도록 막아놓은 것이다.
`package org.springframework.security.web.firewall.StrictHttpFirewall`을 확인해보니 XSS를 유발할 수 있어 블랙리스트로 지정된 문자열들을 확인할 수 있었다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/requestrejectedexception2.png" alt="requestrejectedexception2.png">
</div>

나의 경우, 이 예외에 대한 로그와 알림이 오지 않도록 하는 것이 목적이었기 때문에
RequestRejectedException 발생 시 500을 400(BAD_REQUEST)로 핸들링해주는 방법을 적용했다.

# 과정
RequestRejectedException으로 구글링했을 때 이 에러를 해결하는 방법으로 Spring Security에서 지정한 
블랙리스트 문자열들을 통과시키는 방법들이 많이 나왔는데, 
나는 이 문자열들을 통과시키는 것이 목적이 아니라 예외 상태코드를 내려서 로그와 알림의 대상에서 벗어나도록 하는 것이었다.

그래서 찾아보니 Spring Security 레포에서 Leonard 라는 분이 이 예외를 커스텀하게 handle 할 수 있는
[RequestRejectedExceptionHandler라는 인터페이스를 구현해놓은 PR](https://github.com/spring-projects/spring-security/pull/7052)이 있었다.
이 PR과 [Spring 공식문서](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/firewall/RequestRejectedHandler.html)를 참고해 사용하던 ControllerAdvice에 아래와 같이 RequestRejectedException handle 코드를 추가했다.

```
@RestControllerAdvice
public class ControllerAdvice implements RequestRejectedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, RequestRejectedException requestRejectedException) throws IOException, ServletException {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST);
    }

    ...
}
```

그리고 요청 URL에 `//`을 넣어 테스트해본 결과 아래와 같이 400으로 잘 오는 것을 확인할 수 있었다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/requestrejectedexception3.png" alt="requestrejectedexception3.png">
</div>

## References
- [Spring Seuciry Repo](https://github.com/spring-projects/spring-security/pull/7052)
- [Spring 공식문서 - RequestRejectedHandler](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/firewall/RequestRejectedHandler.html)