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

## HTTP (HyperText Transfer Protocol)
- 서로 다른 시스템들 사이에서 통신을 주고받게 해주는 가장 기초적인 프로토콜
- 웹 서핑을 할 때 서버에서 브라우저로 데이터를 전송해주는 용도로 사용
  ⇒ 문제점 : 서버에서부터 브라우저로 전송되는 정보가 암호화되지 않음, 즉 데이터가 쉽게 도난당할 수 있음

## HTTPS (HyperText Transfer Protocol Secure)
- HTTP 프로토콜에 보안 기능을 추가한 것
- 암호화되지 않은 HTTP 메시지를 TCP를 통해 보내는 대신, HTTPS는 HTTP 메시지를 TCP로 보내기 전에 먼저 암호화하는 보안 계층으로 보냄
- SSL (보안 소켓 계층 : 서버와 브라우저 사이에 안전하게 암호화된 연결을 만들 수 있게 도와주고, 서버 브라우저가 민감한 정보를 주고 받을 때 도난당하는 것을 막아줌) 을 사용함으로써 문제 해결
- HTTPS는 TCP위에 놓인 보안 계층 위의 HTTP

### SSL 핸드셰이크

- 암호화된 HTTP 메시지를 보낼 수 있게 되기 전에, 클라이언트와 서버는 SSL 핸드셰이크를 함
  통신을 하는 브라우저와 웹 서버가 서로 암호화 통신을 시작할 수 있도록 신분을 확인하고, 필요한 정보를 클라이언트와 서버가 주거니 받거니 하는 과정

  <div style="width:100% !important; margin:0 auto">
  <img src="/assets/img/https2.png" alt="https2.png">
  </div>
  
    1. 프로토콜 버전 번호 교환
    2. 양쪽이 알고 있는 암호 선택
    3. 양쪽의 신원을 인증
    4. 채널을 암호화하기 위한 임시 세션 키 생성

### TLS
- TLS는 SSL의 개선버전으로, 최신 인증서는 TLS를 사용하지만 편의적으로 SSL 인증서라고 부름
- 탑재된 보안 기술
    - 하나의 키(key)로 암호화/복호화를 수행하는 대칭키 암호화 방식
    - 한 쌍의 키 페어(key pair)로 암호화/복호화를 수행하는 비대칭키 암호화 방식
    - 통신 대상을 서로가 확인하는 신분 확인(authentication)
    - 믿을 수 있는 SSL 인증서를 위한 디지털 서명(digital signature)
    - 디지털 서명을 해주는 인증 기관(CA: Certificate Authority)
    - 공개키를 안전하게 전달하고 공유하기 위한 프로토콜
    - 암호화된 메시지의 변조 여부를 확인하는 메시지 무결성 알고리즘

### 대칭키 암호화 방식 : 자물쇠 (하나의 키로 암복호화)
### 공개키 암호화 방식(비대칭키) : 공개키로 암호화, 개인키로 복호화

## HTTPS 적용하였다면 100% 안전한가?
- HTTPS는 웹에서 보안을 적용하기 위한 가장 기본적인 단계, 이것으로 모든 보안성이 완벽하게 지켜졌다고는 할 수 없음
- 루트 권한을 탈취당했다면 모든 데이터를 열람할 수 있는 권한이 넘어갈 수 있음
