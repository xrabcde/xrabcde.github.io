---
layout: post
title: Network/흐름제어와 오류제어
date: 2021-07-08
excerpt: "컴퓨터 네트워크 스터디 #5 흐름제어와 오류제어"
tags: [csstudy, network]
comments: true
---

## 흐름제어 (Flow Control)
- 송신 ↔ 수신
- 수신측과 송신측의 **데이터 처리 속도 차이** 를 해결하기 위한 기법
- 송신측의 전송량이 수신측의 처리량보다 많은 경우, 전송된 패킷은 수신측의 큐를 넘어서 손실될 수 있기 때문에
**송신측의 패킷 전송량을 제어**

### 1. 정지-대기 (Stop and Wait)
- 매번 전송한 패킷에 대해 응답을 받아야만 그 다음 패킷을 전송
- 구조가 간단한 대신, 하나씩 주고받기 때문에 **비효율적**

### 2. 슬라이딩 윈도우 (Sliding Window)
- 수신측에서 설정한 윈도우 크기만큼 송신측에서 확인 응답없이 세그먼트를 전송할 수 있게 하여 데이터를 동적으로 조절하는 기법
- 윈도우 : 전송, 수신 스테이션 양쪽에서 만들어진 `버퍼`의 크기
- 윈도우 크기 = (가장 최근 ACK로 응답한 프레임의 수) - (이전에 ACK 프레임을 보낸 프레임의 수)
- Stop and Wait 기법의 비효율성을 개선한 기법
- ACK 프레임을 수신하지 않더라도 여러 개의 프레임을 연속적으로 전송가능

### 3. Piggyback 방식
- 수신측에서 수신된 데이터에 대한 확인 응답을 즉시 보내지 않고, 별도의 제어 프레임도 사용하지 않은 채,
전송할 데이터가 있는 경우에만, 그 데이터 프레임 안에 확인 필드를 덧붙여 전송하는 기법

## 오류 제어 (Error Control)
- 송신측에서 수신측에 전달된 프레임에서 발생한 오류를 감지하고 수정하기 위한 매커니즘
    - 패리티 검사 : 프레임에 포함된 `1` 비트 수가 짝수인지 홀수인지를 나타내는 단일비트를 프레임에 추가하여 오류 감지  
    => 짝수 개의 오류가 발생한다면 감지 불가
    - CRC (순환 중복 코드)
    - 체크섬
- 오류 검출(Error Detection)과 재전송(Retransmission)을 포함
- ARQ (Automatic Repeat Request) 기법을 사용하여 프레임이 손상되었거나 손실되었을 경우 재전송을 통해 오류 복구
- ARQ 기법은 흐름제어 기법과 관련된어 있는데 Stop and Wait 은 Stop and Wait ARQ로, Sliding Window는 GBn(Go-Back-N) ARQ
또는 SR(Selective-Reject) ARQ 형태로 구현
- ARQ : 신뢰성 있는 데이터 전달을 위해 재전송을 기반으로 한 에러제어 방식

### Stop and Wait ARQ
- 전송측은 수신측에서 보내준 ACK를 받을 때까지 프레임의 복사본을 유지
- 식별을 위해 데이터 프레임과 ACK 프레임은 각각 0,1 번호를 부여
- 수신측이 데이터를 받지 못했을 경우, NAK를 송신측에게 보내고 NAK을 받은 송신측은 데이터를 재전송
- 데이터나 ACK가 분실되어 일정 간격의 시간을 두고 타임아웃이 되면 송신측은 데이터를 재전송

### Go-Back-N ARQ
- 전송된 프레임이 손상되거나 분실된 경우, 확인된 마지막 프레임 이후로 모두 재전송하는 기법
- 슬라이딩 윈도우는 연속적인 프레임 전송 기법으로 전송 스테이션은 전송된 모든 프레임의 복사본을 가져야 하며,
ACK와 NAK 모두 각각 구별해야 함
    - ACK : 다음 프레임을 전송
    - NAK : 손상된 프레임 자체 번호를 반환
- 재전송 되는 경우
    - NAK 프레임을 받았을 경우
    - 전송 데이터 프레임의 분실
    - 지정된 타임아웃 내의 ACK 프레임의 분실

### Selective-Reject ARQ
- 전송된 프레임이 손상되거나 분실된 경우, 해당 프레임만 재전송하는 기법

<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/flowcontrol.png" alt="flowcontrol.png">
</div>

---

### 출처
- [참고링크1](https://woovictory.github.io/2018/12/28/Network-Erro-Flow-Control/)
- [참고링크2 - 오류제어](http://www.ktword.co.kr/abbr_view.php?nav=&m_temp1=1299&id=772)
- [참고링크3 - 흐름제어](http://www.ktword.co.kr/abbr_view.php?m_temp1=392)
