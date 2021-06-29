---
layout: post
title: Network/웹 동작 방식
date: 2021-06-28
excerpt: "컴퓨터 네트워크 스터디 #1 웹 동작 방식"
tags: [csstudy, network]
comments: true
---

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/web_process1.jpg" alt="web_process1.jpg">
</div>


1. Client로부터 요청이 들어오면 먼저 DNS 서버를 통해 해당 도메인 네임에 해당하는 IP 주소를 찾는다.
2. Application Layer에서 HTTP 프로토콜을 사용하여 HTTP 요청 메시지를 생성한다.
3. Transport Layer에서 전송하고자 하는 데이터를 패킷으로 분리한다.
4. 분리한 패킷에 TCP 헤더를 붙인다.
   이 때, TCP 헤더에는 `나와 상대방의 port 주소`가 들어있고, 패킷에 TCP 헤더를 붙인 것을 `세그먼트`라고
5. Network Layer에서 세그먼트에 IP 헤더를 붙인다.
   이 때, IP 헤더에는 `나와 상대방의 IP 주소`가 들어있고, 세그먼트에 IP 헤더를 붙인 것을 `데이터그램`이라고 한다.
6. 이 과정에서, `라우터`가 라우팅 테이블을 통해 다음 경로를 탐색하고 설정한다.
   이 때, ARP 프로토콜을 이용해 다음 라우터의 MAC 주소를 찾는다.
7. Data-Link Layer에서 데이터그램에 MAC 헤더를 붙인다. 데이터그램에 MAC 헤더를 붙인 것을 `프레임`이라고 한다.
