---
layout: post
title: Network/데이터링크계층/MAC주소, 패킷스위칭, 프레이밍
date: 2021-07-17
excerpt: "컴퓨터 네트워크 스터디 #9 Data Link layer - MAC주소, 패킷 스위칭, 프레임(프레이밍)"
tags: [csstudy, network]
comments: true
---

## MAC주소
- 컴퓨터간 데이터를 전송하기 위해 있는 컴퓨터의 **물리적 주소**
    - IP주소간 통신은 사실, 각 라우터에서 일어나는 MAC주소와 MAC주소 통신의 연속과정이다
- 절대 변하지 않는 기계의 고유 주소번호

### IP주소 vs MAC주소
- IP주소
    - ex. 집주소(**바뀔 수 있음**, NOT portable)
    - depends on network to which one attaches
    - Network-layer address
    - 네트워크(서브넷)을 찾아가는 주소
    - Used to get datagram to destination network
- MAC주소
    - ex. 주민등록번호(**바뀌지 않음**, portable)
    - can move LAN card from one LAN to another
    - Data Link layer address
    - Used to get datagram from one interface to another physically-connected interface (same network)


- 하드웨어 주소, 물리적 주소, 이더넷 주소
    - NIC(랜카드)를 만들 때 카드 내에 있는 ROM에 아주 기본적인 OS 관련 정보가 들어가는데 여기에 MAC주소가 포함되어 있음
    - ROM : 통신 모듈에 주소를 부여하는 곳
- 컴퓨터는 자신의 IP와 MAC주소는 알아도 상대방의 MAC주소는 모름

### MAC주소 구성
<div style="width:60% !important; margin:0 auto">
<img src="/assets/img/switching1.png" alt="switching1.png">
</div>

- 맥주소는 6byte(=48bit)로 구성되어 있고, 이를 16진수로 표기
- 8자리마다 `-` 이나 `:` 이나 `.` 으로 구분
- 노란 부분 : OUI (Organizational Unique Identifier), 생성자를 나타내는 코드, 해당 장비를 만들어낸 회사를 나타내는 것
- 파란 부분 : 고유 번호를 부여하기 위한 시리얼 넘버

## 패킷 스위치
### Circuit Switching (회선 교환) *designed for voice*
- 실제 데이터 전송이 시작되기 전에 경로가 결정됨
- 두 엔드 포인트 간의 전체 통신 길이에 대해 대역폭과 기타 리소스가 고정, 세션이 종료될 때만 해제
- ✅ 끝에서 끝까지 보장된 QoS를 제공할 수 있으므로 음성 및 비디오와 같은 실시간 응용프로그램에 적합
- ✅ 소스에서 보낸 메시지의 순서가 이동중에 변경되지 않음 (순서보장)
- ✅ 원본 메시지를 다시 생성하기 위해 대상에서 처리하는 작업이 줄어듦
- ❌ Must have switching capacity and reserve channel capacity to establish connection
- ❌ Must have intelligence to work out routing
- ❌ Inefficient : Channel capacity is dedicated for duration of connection (둘이 설정해놓고 대화를 안 하면 낭비)
- ❌ Set up (connection) takes time

### Packet Switching (패킷 교환) *designed for data*
- 데이터를 패킷(packet)이라고 하는 **작은 데이터 덩어리로 분할하여 전송**
- 송신측과 수신측 사이에서 각 패킷은 통신 링크와 패킷 스위치를 거침
- 네트워크를 공유해서 사용하기 때문에 큰 크기의 파일 전송은 길고 안정적이지 않아 작은 단위로 나눔 ⇒ 받는 쪽에서 reassemble
- received ⇒ stored briefly (buffered) ⇒ passed to the next node
- 특징
    - **Store-and-Forward (저장 후 전달)**  
      스위치가 출력 링크로 패킷의 첫 비트를 전송하기 전에 전체 패킷을 받아야 하는 것을 뜻함  
    - Queueing delay, loss (큐잉 지연, 패킷 손실)  
      각 패킷 스위치는 접속된 여러 링크를 가지고 있고, 각 링크에 대해 패킷 스위치는 출력 버퍼(큐)를 가지고 있으며, 그 링크로 송신하려는 패킷을 저장하고 있음  
      도착하는 패킷은 한 링크로 전송될 필요가 있는데 그 링크가 다른 패킷을 전송하고 있다면 도착하는 패킷은 출력 버퍼에서 대기해야 함  
      따라서, store-and-forward(저장 후 전달) 지연뿐만 아니라, 패킷은 출력 버퍼에서 queueing delay(큐잉 지연)을 겪게 됨  
- 장단점
    - ✅ line을 공유하기 때문에 낭비를 줄이고 효율적
    - ✅ 다른 data rate 끼리도 패킷을 주고받을 수 있음
    - ✅ 패킷 교환이 더 간단하고, 효율적이며, 회선 교환보다 구현 비용이 적음
    - ❌ 가변적이고 예측할 수 없는 종단 간의 지연 (주로 불규칙하고 예측할 수 없는 큐잉 지연에서 발생) 때문에 실시간 서비스에 적합하지 않음

1. Datagram
    - Each packet is treated independently 
    - Packets may arrive out of order
    - Packets may go missing
    - ✅ No call setup phase : 보낼 패킷이 적을 때 더 빠름
    - ✅ More flexible
    - ✅ More reliable
    - 패킷이 적을 때 적합
2. Virtual Circuit
    - Preplanned route is established before any packets are sent
    - Not a dedicated path, but a logical connection
    - ✅ provide sequencing and error control
    - ✅ forwarded more quickly
    - 패킷이 많을 때 적합

<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/switching2.png" alt="switching2.png">
</div>

## 프레임(프레이밍)
### 데이터링크 계층
- OSI 레이어 중 계층 하단에서 2번째에 위치
- 1계층인 물리 계층의 역할은 Node to Node 바로 옆 node 끼리 전기적 신호를 보내는 것이라면, 2계층은 Node to Node는 동일하지만 물리계층에선 하지 않는 **오류 검출과 신뢰성을 더하는 역할**
- 데이터링크 계층의 기능 4가지
    - 흐름 제어 (Flow Control)
    - 에러 검출 및 정정 (Err Detection & Correction)
    - 반이중 통신, 전이중 통신 (Half Duplex & Full Duplex)
    - **프레이밍 (Framing)**

### 프레이밍
- 프레임(frame) : 2계층의 데이터단위 PDU(Protocol Data Unit)
- 프레이밍(프레임화, framing) : 데이터그램을 캡슐화해서 프레임 단위로 만들고 헤더와 트레일러 추가
    - 헤더 : 목적지, 출발지 주소와 데이터 내용을 정의
    - 트레일러 : 비트 에러를 감지

<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/switching3.png" alt="switching3.png">
</div>

- network 계층에서 내려온 datagram은 datalink 계층에서 frame 단위로 쪼개진 다음 전송된다.

---

### 출처
- [참고링크1 - MAC주소](https://jhnyang.tistory.com/404)
- [참고링크2 - 패킷 스위치](https://hwan.dev/Network/Packet_exchange/)
- [참고링크3 - 패킷 스위치](https://ko.strephonsays.com/circuit-switching-and-vs-packet-switching-648)
- [참고링크4 - 프레임(프레이밍)](https://as-backup.tistory.com/4)
