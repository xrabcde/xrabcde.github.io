---
layout: post
title: Network/응용계층 프로토콜
date: 2021-06-30
excerpt: "컴퓨터 네트워크 스터디 #3 응용계층 프로토콜"
tags: [csstudy, network]
comments: true
---

## 프로토콜이란?
네트워크 통신을 위한 규약

## 응용 계층의 역할
클라이언트의 요청을 전달하기 위해 서버가 이해할 수 있는 메시지로 변환하고 전송계층에 전달하는 역할  
클라이언트 측에서 서버 측으로 데이터를 보내기 위해서는 응용계층의 프로토콜을 사용해야 함  
응용 계층은 웹이나 이메일과 같은 서비스를 제공하는 계층  
응용 계층은 사용자가 직접 사용하며 체감할 수 있는 서비스를 제공한다  
네트워크 계층 모델 중 전송 이하의 계층들은 데이터 전송을 담당하고 있으므로 이들 데이터 전송 관련 계층을 제외한 모든 영역이 응용계층의 범주라고 보면 된다  
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/application-protocol1.png" alt="application-protocol1.png">
</div>

---

## 사용자가 직접 사용하는 프로토콜
### HTTP (HyperText Transfer Protocol)
**클라이언트(웹 브라우저)와 웹 서버 사이에 데이터를 주고받기 위해 사용되는 네트워크 프로토콜**
WWW 상에서 정보를 주고 받을 수 있는 프로토콜  
TCP와 UDP를 사용하며, 80번 포트 사용  
클라이언트와 서버 사이에 이루어지는 요청/응답 프로토콜  
HTTP를 통해 전달되는 자료는 http: 로 시작하는 URL로 조회가능  

### POP3 (Post Office Protocol version 3), SMTP (Simple Mail Transfer Protocol), IMAP
**메일을 송수신**하고 보관한다  
- **POP3 : 메일을 수신**하는 데 사용하는 프로토콜, 멀리 떨어져있는 메일서버에 지정된 사용자 ID로 접속해서, 메일박스 내에 도착한 메일을 클라이언트에게 가져오기 위해 사용되는 비교적 간단한 프로토콜
- **SMTP : 메일을 송신**하는 데 사용하는 프로토콜, 인터넷에서 이메일을 보내고 받기 위해 이용되는 프로토콜, TCP 포트번호 25번 사용  
  상대서버를 지시하기 위해 DNS의 MX 레코드가 사용됨, RFC2821에 따라 규정됨  
  텍스트 기반 프로토콜로서 요구/응답 메시지 뿐 아니라 모든 문자가 아스키로 되어 있어야 함  
- IMAP : 원격파일 서버와 유사하게 작동, POP3와 달리 메일을 받아올 때 서버에서 메일을 삭제하지 않고 보관하는 프로토콜  
  다른 컴퓨터 환경에서 항상 같은 메일 내용을 메일 서버로부터 받아올 수 있는 장점이 있음  
  POP3에 비해 메일서버와의 통신 트래픽이 높은 단점을 가짐  

### SMP, AFP
LAN 안에서 파일을 공유한다

### FTP (File Transfer Protocol)
서버를 통해 파일을 주고 받는다  
**컴퓨터 간 파일을 전송**하는데 사용되는 프로토콜  
사용자인증, 데이터의 전환, 디렉터리 검색 등과 같은 많은 기능 및 선택사항 제공  
클라이언트/서버 관계를 이루며 동작, 표준 RFC959에 따라 규정됨  
**데이터 전달 : 20번 포트, 제어정보전달 : 21번포트**

### Telnet, SSH
원격에서 서버를 제어한다  
- Telnet : 인터넷이나 로컬 영역 네트워크 연결에 쓰이는 네트워크 프로토콜, IETF STD 8로 표준화  
  보안문제로 사용이 감소하고 있으며, 원격제어를 위해 SSH로 대체되기도 함
- SSH : 네트워크 상의 다른 컴퓨터에 로그인하거나 원격 시스템에서 명령을 실행하고 다른 시스템으로 파일을 복사할 수 있도록 해주는 프로토콜  
  기존 RSH, Rlogin, Telnet을 대체하기 위해 설계됨  
  강력한 인증방법 제공 및 안정성 우수, 22번 포트 사용

---

## 사용자가 간접적으로 사용하는 프로토콜
### DNS (Domain Name System)
**도메인명과 IP 주소의 정보를 서로 변환할 때 사용**하는 프로토콜  
호스트에 대한 정보를 분산시켜서 관리하는 데이터베이스  
IP주소와 Host 이름이 서로 연결되어 구조화된 역 트리구조를 갖는 계층적이고, 분산된, 클라이언트/서버 구조의 데이터베이스 시스템

### DHCP
LAN 내의 컴퓨터에게 IP 주소를 할당할 때 사용, 빠른 통신을 위해 UDP 사용  
- RARP : Reverse ARP (MAC주소로 IP주소를 가져옴)  
- BOOTP : IP주소와 추가적인 정보를 같이 가져옴, 정적인 주소를 가지고 있음
  ⇒ DHCP

### SSL/TLS
통신 데이터를 암호화하여 주요 정보를 안전하게 주고 받을 때 사용

### NTP
네트워크에 연결된 장비들의 시스템 시간을 동기화할 때 사용

### LDAP
네트워크에 연결된 자원의 통합관리에 필요한 디렉터리 서비스를 제공할 때 사용

---

## HTTP 흐름
- 클라이언트가 데이터를 요청할 때는 GET이라고 하는 HTTP method, 파일 이름, 버전 등을 서버에 전송. 서버는 요청을 정상적으로 처리했다는 OK 를 반환하고 index.html을 클라이언트에게 보냄
- HTTP/1.1 버전은 HTTP/1.0 버전에서 keepalive가 추가됨
- keepalive : 연결을 한 번 수립하면 데이터 교환을 마칠 때까지 유지하고, 데이터 교환을 모두 끝내면 연결을 끊는 구조  
  요청을 순서대로 처리한다는 특징이 있음  
  과정 : 연결 수립 → 1 요청 → 1 응답 → 2 요청 → 2 응답 → 연결 끊기  
  순서대로 응답을 반환하기 때문에, 이전 요청을 처리하는 시간이 길어지면 다음 요청에 대한 처리가 늦어져 콘텐츠 표시가 늦어진다는 단점  
- HTTP/2 버전은 요청을 순서대로 응답하지 않아도 되기 때문에 콘텐츠를 빠르게 표시할 수 있음  
  과정 : 연결 수립 → 1 요청 → 2 요청 → 3 요청 → 2 응답 → 3 응답 → 1 응답 → 연결 끊기

## DNS 흐름
- 기본적으로 컴퓨터(서버)에는 IP 주소가 있어 인터넷을 통해 웹 서버에 접속하여 웹 사이트를 볼 수 있다. 예를 들어, 웹 브라우저의 주소 창에 URL을 직접 입력하면 웹 사이트가 보인다. 컴퓨터(서버)에 접속하려면 IP 주소를 입력해야 하는데, 우리는 보통 URL을 입력한다. 이는 DNS가 URL을 IP주소로 변환해주는 서비스를 제공하기 때문이다.
- `https://www.google.com` 과 같은 주소를 사용하여 숫자로만 되어 있는 IP 주소로 접속하도록 돕는 것을 DNS의 이름해석(Name Resolution) 이라고 한다. 사용자가 url에 접속하면 DNS 서버가 이 웹 사이트 서버의 IP 주소를 알려주는 것을 이름해석이라고 함
- `https://www.google.com` 과 같이 컴퓨터나 네트워크를 식별하기 위해 붙여진 이름을 **도메인 이름**이라하고, 도메인 이름 앞에 있는 `www` 는 **호스트 이름(서버 이름)**이라고 한다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/application-protocol2.png" alt="application-protocol2.png">
</div>
   
1. 클라이언트가 DNS recursor server로 요청을 보냄
2. DNS recursor server는 DNS 질의가 풀릴 때까지 여러 네임 서버로 질의를 날림
3. DNS 서버는 해당 요청에 해당하는 도메인 이름의 IP 주소를 알려줌
4. 컴퓨터는 IP 주소로 해당 웹 서버에 접속
   요청한 첫 번째 DNS 서버가 도메인의 IP 주소를 모르는 경우, 첫 번째 DNS 서버가 다른 DNS 서버에 질의한다. 그리고 다른 DNS 서버로부터 받은 IP 주소를 다시 클라이언트에게 전달해준다.
   DNS 서버는 전 세계에 흩어져 있고 모두 계층적으로 연결되어 있다.

## 메일 흐름 (SMTP와 POP3)
[https://velog.io/@gndan4/네트워크-응용-계층](https://velog.io/@gndan4/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%9D%91%EC%9A%A9-%EA%B3%84%EC%B8%B5)
[https://coding-nyan.tistory.com/132?category=881363](https://coding-nyan.tistory.com/132?category=881363)

---

### 참고링크
[참고링크1](https://yoeubi.github.io/network/Frontend-TCP-IP-2)  
[참고링크2](https://math-coding.tistory.com/144)  
[참고링크3](http://www.jidum.com/jidums/view.do?jidumId=415)  
[참고링크4](http://tech.kobeta.com/wp-content/uploads/2016/10/22619.pdf)  
[참고링크5](https://velog.io/@gndan4/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%9D%91%EC%9A%A9-%EA%B3%84%EC%B8%B5)