---
layout: post
title: Network/라우팅 프로토콜, 브로드캐스트, 멀티캐스트, 유니캐스트
date: 2021-07-14
excerpt: "컴퓨터 네트워크 스터디 #7 라우팅프로토콜(RIP, OSPF, BGP), DHCP, 브로드캐스트, 멀티캐스트, 유니캐스트"
tags: [csstudy, network]
comments: true
---

## 라우팅 프로토콜(RIP, OSPF, BGP ...)
### 라우터
- **Path Determination (경로결정)**과 **Switching (스위칭)**을 주로 하는 장비
- 경로결정 : 데이터 패킷이 목적지까지 가는 길을 검사하고 어떤 경로로 패킷이 가는 것이 적절한지 판단
- 스위칭 : 경로가 결정되면 그 방향으로 패킷을 보내는 것
- 라우팅 프로토콜 : 라우터가 목적지까지 가는 가장 좋은 방향을 결정할 때 사용하는 알고리즘, 라우팅 테이블을 만들어 관리

### 라우팅 프로토콜
- 패킷이 목적지까지 가는 방법을 결정해주는 프로토콜
- RIP, OSPF, BGP, IGRP 등이 있음
- 라우팅 테이블을 참조해 가장 좋은 길로 패킷을 전송
- **스태틱 (Static)** vs **다이나믹 (Dynamic)**
    - 스태틱(Static) : 정적으로 라우팅을 관리하는 것, 사람이 직접 경로를 결정해주고, 라우터는 입력받은 경로로만 패킷을 전송  
      ✅ 라우팅 테이블을 교환하지 않기 때문에 네트워크 대역폭 절약  
      ✅ 외부에 자신의 경로를 알리지 않기 때문에 보안에도 강함  
      ❌ 경로에 문제가 생길 경우 대처하기 어려움  
    - 다이나믹(Dynamic) : 자동으로 경로가 결정되는 프로토콜, 라우터가 판단하여 가장 효율적인 방법으로 패킷을 전송

  <div style="width:90% !important; margin:0 auto">
  <img src="/assets/img/routing1.png" alt="routing1.png">
  </div>

### RIP (Routing Information Protocol)
- 최소 Hop Count를 파악하여 라우팅하는 프로토콜
- Distance Vector (거리+방향으로 길을 찾아가는) 다이나믹 라우팅 프로토콜
- 최단거리(=라우팅되는 Hop 카운트가 가장 적은 경로)를 택하여 라우팅
    - 라우팅 테이블에 인접 라우터 정보를 저장하여 경로 결정
- 최대 Hop Count는 15이고 거리가 짧기 때문에 내부용(IGP)으로 많이 이용
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/routing2.png" alt="routing2.png">
</div>

- 직접 연결되어 있는 라우터는 Hop으로 계산하지 않고, 30초 주기로 default routing을 업데이트하여 인접 라우터로 정보 전송
- 4~6개 로드밸런싱 가능
- 주로 UDP 세그먼트에 캡슐화되어 사용
- 단순 Hop을 Count하여 경로를 결정하기 때문에 경로의 네트워크 속도는 판단하지 않음  
  ⇒ ❌ 비효율적인 경로로 패킷을 전달할 가능성 있음
- ❌ Distance Vector 알고리즘으로 네트워크 변화에 대처하는 시간(컨버전스 타임)이 느림

### OSPF (Open Shortest Path First)
- 최단 경로 우선 프로토콜
- 최저 COST(최소시간) 경로를 최적 라우팅 경로로 결정
- 대표적인 링크 상태 프로토콜로, SPF(최단거리 우선 알고리즘)을 통해 라우팅 테이블 생성
- 주로 내부 게이트웨이 프로토콜(IGP)로 대규모 기업망에서 사용
- Area라는 개념을 사용하여 전체 네트워크를 작은 영역으로 나눠 효율적으로 관리하는 방식
    - 각 Area는 Back Bone Area에 연결되어 있음
- RIP는 30초마다 update 되어 정보를 전송시켜 주는 반면, OSPF는 링크 상태에 변화가 있을 시 즉각적으로 Flooding  
  ⇒ ✅컨버전스 타임이 매우 빠름
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/routing3.png" alt="routing3.png">
</div>  

- 과정
    - OSPF가 Hello 패킷을 보내 인접 라우터를 찾는다
    - OSPF 라우터 간 공유하는 라우터 ID를 통해 헬로 패킷을 전송, 이 때 브로드캐스트(`255.255.255.255`)가 아닌 멀티캐스트 주소(`224.0.0.5`)로 전송
    - 헬로 패킷 안에는 DR, BDR을 결정하기 위한 Priority 필드가 존재
    - OSPF 라우터는 헬로 패킷 송수신을 통해 Neighbor 관계를 유지
    - VLSM(Variable Length Subnet Mask)을 지원하기 때문에 IP 주소를 효율적으로 사용할 수 있고 라우팅 테이블을 줄일 수 있음

### BGP (Border Gateway Protocol)
- 외부 라우팅 프로토콜(EGP)로 AS(관리 도메인)와 AS간 사용되는 라우팅 프로토콜
- 정해진 정책에 의하여 최적 라우팅 경로 수립, 경로 벡터(Distance Vector) 방식의 라우팅 프로토콜  
  ⇒ IGP보다 컨버전스는 느리지만 대용량의 라우팅 정보 교환가능
- TCP 포트 179번을 통해 인접 라우터들과 Neighbor 관계를 성립, Neighbor 라우터 간에는 유니캐스트 라우팅 업데이트 실시

## DHCP
- IP를 동적으로 할당해주는 프로토콜
- DHCP를 통한 IP 할당 방법은 `임대` 개념을 사용하고 있음
    - 할당시간(임대시간) 동안 IP를 할당해주고, 시간이 지나면 IP할당시간을 연장하거나 반납하는 개념
- 과정
  <div style="width:90% !important; margin:0 auto">
  <img src="/assets/img/routing4.png" alt="routing4.png">
  </div>

    - DHCP Discover : 클라이언트가 브로드캐스팅 메시지를 통해 DHCP 서버를 찾는 과정
    - DHCP Offer : DHCP 서버가 클라이언트에게 서버의 위치를 알려주는 과정
    - DHCP Request : 클라이언트가 서버에게 IP를 요청하는 과정, 클라이언트가 사용할 네트워크 정보(IP주소, subnet mask, default gateway 등)를 요청  
    - DHCP ACK : 서버가 클라이언트에게 IP주소를 할당해주는 과정, DHCP ACK에는 서버정보, 서버할당시간(Lease Time), 서브넷 마스크, 라우터정보, DNS 정보 등이 포함
  ⇒ DHCP 과정을 통해 IP가 없는 클라이언트는 IP 및 네트워크 정보를 할당받아 인터넷을 사용할 수 있음
- DHCP 운용의 장단점
- 장점  
  ✅ COST 절약 : 사용자 중 PC를 켠 사용자만 IP가 할당되어 고정 IP에 비해 IP 절약 효과가 있음  
  ✅ 효율적인 네트워크 관리 : IP 방식에 비해 사용자 IP망 설계변경이 자유로움  
  사용자에게 DHCP IP를 할당할 경우 네트워크 정보가 바뀌더라도 DHCP 서버에만 네트워크 정보를 변경해주면 되므로 네트워크 정보 변경이 유연  
- 단점  
  ❌ DHCP 요구 단말은 초기 부팅 시 broadcast 트래픽(DHCP Discover 메시지)을 유발  
  ⇒ 한 개의 VLAN 의 설정범위에 있는 모든 단말에 전송되므로 네트워크의 성능 저하 발생가능  
  ❌ PC 전원을 OFF할 경우 Lease Time까지 IP가 다른 단말에 할당되지 못함  
  ⇒ IP주소 낭비가 발생  
  ❌ IP를 할당해주는 서버에 전적으로 의존  
  ⇒ 서버가 다운되면 IP를 받을 수 없으므로 인터넷을 사용할 수 없음  

## 브로드캐스트, 멀티캐스트, 유니캐스트
### 비교
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/routing5.png" alt="routing5.png">
</div>

- 브로드캐스트 (Broadcast) : 1대 다수(불특정 다수)
- 멀티캐스트 (Multicast) : 1대 다수(특정 집단)
- 유니캐스트 (Unicast) : 1대 1(특정 단일)

### 브로드캐스트
<div style="width:30% !important; margin:0 auto">
<img src="/assets/img/routing6.png" alt="routing6.png">
</div>
- 로컬 네트워크에 연결되어 있는 모든 시스템에게 프레임을 보내는 방식
- 브로드캐스트용 주소가 미리 정해져있고, 수신 받는 시스템은 이 주소가 오면 패킷을 자신의 CPU로 전송하고 CPU가 패킷을 처리하는 방식
- ❌ 모든 시스템에게 패킷이 전송되므로 트래픽 증가
- ❌ CPU도 패킷을 처리해야 하므로 성능 저하
- 통신하고자 하는 시스템의 MAC 주소를 알지 못하는 경우, 네트워크에 있는 모든 시스템에게 알리는 경우, 라우터끼리 정보를 교환하거나 새로운 라우터를 찾는 경우

### 멀티캐스트
<div style="width:25% !important; margin:0 auto">
<img src="/assets/img/routing7.png" alt="routing7.png">
</div>
- 네트워크에 연결되어 있는 시스템 중 일부(특정 그룹)에게만 정보를 전송하는 방법
- 송신자는 여러 수신자에게 한 번에 메시지가 전송되도록 하여 데이터의 중복 전송으로 인한 네트워크 자원 낭비를 최소화
- 일반적으로 TCP/IP 상의 인터넷 응용 프로그램은 데이터의 송신자가 이를 수신할 수신자의 인터넷 주소를 전송 패킷의 헤더에 표시해 패킷 전송하지만, 멀티캐스트 전송을 위해서는 헤더에 수신자의 주소 대신 **수신자들이 참여하고 있는 그룹 주소를 표시**하여 패킷 전송
- ❌ 라우터가 멀티캐스트를 지원해야만 사용가능하다는 단점
- ❌ UDP를 사용하여 전송함으로 신뢰성을 보장받지는 못함
- 멀티캐스트 그룹 단위로 묶어 그 그룹의 Host 들은 동시에 데이터를 받을 수 있음
- 클라이언트에서 멀티캐스트를 사용하는 Application을 시작하면 멀티캐스트 IP 주소와 멀티캐스트 MAC 주소를 라우터에 등록함으로 멀티캐스트 그룹에 등록됨
- 하나의 클라이언트에서 여러 멀티캐스트 주소를 수용할 수 있음, 즉 여러 가지 멀티캐스트 데이터를 동시에 받을 수 있음
- 서버가 멀티캐스트 주소로 데이터를 전송 중에 있을 때 중간에 클라이언트가 끼어들어도 처음부터 데이터를 받을 수 없고 중간부터 데이터를 받게 됨

### 유니캐스트
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/routing8.png" alt="routing8.png">
</div>
- 정보를 전송하기 위한 프레임에 자신의 MAC주소와 목적지의 MAC주소를 첨부하여 전송하는 방식
- 같은 네트워크에 있는 모든 시스템들은 그 MAC주소를 받아서 자신의 MAC 주소와 비교 후 자신의 MAC 주소와 같지 않다면 프레임을 버리고 같다면 프레임을 받아서 처리하게 됨
- 가장 많이 사용하는 방식으로 한 개의 목적지 MAC 주소를 사용하고 CPU 성능에 문제를 주지 않는 방식

---
