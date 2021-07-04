---
layout: post
title: Network/HTTPS 동작 원리
date: 2021-07-02
excerpt: "컴퓨터 네트워크 스터디 #4 HTTPS 동작 원리"
tags: [csstudy, network]
comments: true
---

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/https1.png" alt="https1.png">
</div>

## 대칭키 암호화 방식 : 하나의 키로 암복호화 (ex. 자물쇠)
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/https2.png" alt="https2.png">
</div>

- 암, 복호화에 사용되는 키가 같다
- 송신자와 수신자가 대칭키를 공유한다
- 송신자는 데이터를 대칭키를 사용해서 암호화 후 전송
  -> 수신자는 수신한 데이터를 대칭키를 사용해서 복호화
  
## 공개키 암호화 방식(비대칭키) : 두 개의 키로 암복호화
- 암, 복호화할 때 사용되는 키가 다르다
- 공개키 암호화 방식은 아래 두 가지 방식이 있다

1) 데이터 암복호화 : 데이터를 공개키로 암호화 -> 개인키로 복호화
  <div style="width:100% !important; margin:0 auto">
  <img src="/assets/img/https3.png" alt="https3.png">
  </div>
  - 다수의 사람들이 공개키로 데이터를 암호화한 다음 개인키를 가지고 있는 사람에게 데이터를 안전하게 전송
  - ex. 편지함 (사람들은 편지함 투입구를 통해 누구나 편지를 넣을 수 있지만, 열쇠를 가진 사람만이 편지함을 열 수 있다)

2) 전자서명 : 데이터를 개인키로 암호화 -> 공개키로 복호화
  <div style="width:100% !important; margin:0 auto">
  <img src="/assets/img/https4.png" alt="https4.png">
  </div>

  - 복호화 키는 모두에게 공개하고 암호화 키는 개인만 갖게 하는 방식
  - 특정 개인이 **자신을 인증하는 용도** 로 사용
  - 암호화된 데이터가 공개키로 복호화해봤을 때 복호화가 된다면, 그 데이터는 개인키를 가진 사람이 보냈다는 것을 확신할 수 있음
  - ex. 편지봉투 (인장으로 봉인된 편지는 누구나 뜯어서 열어볼 수 있지만, 인장 확인을 통해 인장을 소유한 발신자가 보냈음을 증명할 수 있다)

---

## HTTP (HyperText Transfer Protocol)
- 서로 다른 시스템들 사이에서 통신을 주고받게 해주는 가장 기초적인 프로토콜
- 과정 : 클라이언트는 웹 서버의 80번 포트로 TCP 커넥션을 연다 -> HTTP 요청 메시지를 보낸다 -> 서버로부터 HTTP 응답 메시지를 받는다
-> TCP 커넥션을 닫는다  
=> 문제점 : 서버에서부터 브라우저로 전송되는 정보가 암호화되지 않음, 즉 **데이터가 쉽게 도난당할 수 있음**
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/https5.png" alt="https5.png">
</div>

## HTTPS (HyperText Transfer Protocol Secure)
- HTTP 프로토콜에 보안 기능을 추가한 것
- HTTPS는 HTTP 메시지를 **TCP로 보내기 전에 먼저 암호화하는 보안계층** 으로 보냄
- SSL을 사용함으로써 문제 해결
- 과정 : 클라이언트는 웹 서버의 443 포트로 TCP 커넥션을 연다 -> SSL 핸드 셰이크 
-> SSL 초기화가 완료되면, 클라이언트는 HTTP 요청 메시지를 SSL 계층에 보낸다 ->
SSL 계층에서 클라이언트의 HTTP 요청 메시지는 TCP로 보내지기 전에 암호화 된다

### SSL (Secure Sockets Layer) 핸드셰이크
- SSL은 컴퓨터 네트워크에 통신 보안을 제공하는 프로토콜(계층)
- TLS라는 이름은 SSL이 표준화되면서 바뀐 이름
- 암호화된 HTTP 메시지를 보낼 수 있게 되기 전에, 클라이언트와 서버는 SSL 핸드셰이크를 함  
  통신을 하는 브라우저와 웹 서버가 서로 암호화 통신을 시작할 수 있도록 신분을 확인하고, 필요한
  정보를 클라이언트와 서버가 주거니 받거니 하는 과정

  <div style="width:100% !important; margin:0 auto">
  <img src="/assets/img/https6.png" alt="https6.png">
  </div>

1. Client Hello : 클라이언트가 먼저 서버에 접속해서 말을 건다
    - 브라우저가 지원하는 **암호화 방식 모음** (cipher suite)
    - 브라우저가 순간적으로 생성한 임의의 **난수** (숫자)
    - 이전에 SSL 핸드셰이킹을 했다면, 기존의 세션을 재활용하기 위해 세션 키(Session ID)를 전송
    - 기타 확장 정보
2. Server Hello : 서버 또한 위의 인사에 응답하면서, 다음 정보를 클라이언트에 제공한다
    - 브라우저의 암호화 방식 정보 중에서, 서버가 지원하고 **선택한 암호화 방식** (cipher suite)
    - 서버의 공개키가 담긴 **SSL 인증서**, 인증서는 CA의 비밀키로 암호화되어 발급된 상태
    - 서버가 순간적으로 생성한 임의의 **난수** (숫자)
    - 클라이언트 인증서 요청 (선택 사항)
3. 인증서 검증 및 대칭키 생성 : 브라우저는 서버의 SSL 인증서가 믿을만한지 확인
    - 클라이언트는 서버로부터 받은 인증서(전자서명)가 CA에 의해 발급된 것인지 확인하기 위해 **클라이언트 측에 저장된 CA의 공개키로 인증서 검증** (정상적으로 복호화 된다면 CA가 발급한 것)
    - 클라이언트는 클라이언트 측에서 생성한 난수와 서버 측에서 생성한 난수를 사용해서 `pre master secret`이라는 대칭키를 만듬
    - `pre master secret` : 클라이언트와 서버의 암호화 통신에 사용될 대칭키
4. `pre master sescret`을 서버에게 전송
    - 클라이언트는 `pre master secret`을 서버의 공개키로 암호화해서 서버에게 전달
    - 서버로부터 받은 SSL 인증서에 서버의 공개키가 들어있음
5. `pre master secret` 수신 및 복호화
    - 서버는 자신의 개인키로 `pre master secret`을 복호화해서 알아냄
6. `master secret` 및 `session key` (대칭키) 생성
    - 서버와 클라이언트는 일련의 과정을 거쳐 `pre master secret`을 `master secret`으로 만든다
    - 이것을 사용하여 방금 브라우저와 만들어진 연결에 고유한 값을 부여하기 위한 `session key` 생성 ⇒ `session key`는 대칭키 암호화에 사용할 키, 이것으로 브라우저와 서버 사이제 주고받는 데이터를 암복호화
7. `session key`를 통한 대칭키 암호화 방식 통신
    - SSL 핸드셰이크를 종료하고, HTTPS 통신 시작
    - 데이터를 상대방에게 전송하기 전에 `session key`를 사용하여 대칭키 방식으로 암호화해서 전송
    - 데이터를 받은 상대방은 동일 `session key`를 사용하여 데이터 복호화
8. 세션 종료 및 `session key` 폐기
    - 데이터 전송이 끝나면 SSL 통신이 끝났음을 서로에게 알려줌
    - HTTPS 통신이 완료되는 시점에 서로에게 공유된 `session key` 폐기
    
### Q. 그냥 공개키 암호화방식으로 암호화 통신을 하면 되지 않을까?
- 공개키 암호화 방식은 많은 컴퓨터 파워를 사용한다는 단점
- 공개키 암호화 방식을 사용하면 많은 접속이 물리는 서버는 매우 큰 비용을 지불해야 함

### Q. 그냥 대칭키 암호화방식으로 암호화 통신을 하면 되지 않을까?
- 대칭키 암호화 방식을 사용하려면, 먼저 송신자와 수신자가 안전하게 대칭키를 공유해야 함
- 암호화 되지 않은 채로 대칭키를 공유하기 위해 네트워크를 통해 전송하게 되면 키가 탈취될 수 있으므로 위험

=> 속도는 느리지만 데이터를 안전하게 주고 받을 수 있는 **공개키 방식으로 대칭키를 안전하게 공유** 하고  
   공유된 대칭키를 이용해 **대칭키 방식으로 데이터를 안전하게 주고 받는다**

---

### 출처
- [참고링크1](https://jujubebat.github.io/cs/HTTPS/)
- [참고링크2](https://brunch.co.kr/@sangjinkang/38)
- [HTTP 완벽 가이드](https://book.naver.com/bookdb/book_detail.nhn?bid=8509980)
