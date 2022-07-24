---
layout: post
title: DB/Redis의 자료구조 - String과 Hash
date: 2022-06-25
excerpt: "Redis의 자료구조 중 String과 Hash 비교 및 다양한 Redis 툴 소개"
tags: [redis, database, data_structure, string, hash]
comments: true
---

## Redis란?
- `REmote DIctionary Server`의 약자로 key:value 형태의 Dictionary 저장소
- In-Memory Key-Value NoSQL
- Single Thread : Race Condition을 방지하고, Atomic을 보장
- Open Source
- 다양한 자료구조 지원 : Strings, Set, List, Hashes 등
- 다양한 기능 제공 : Replication, 트랜잭션 등

## Redis의 자료구조
- Strings : 기본적인 Key-Value 구조
- Lists : String element의 모음, 순서는 삽입된 순서를 유지하며 기본적인 자료구조로 Linked List 사용
- Sets : 유일한 값들의 모임인 자료구조, 순서는 유지되지 않음
- Sorted Sets : Sets 에 score라는 값을 추가로 두어 해당 값을 기준으로 순서를 유지
- Hashes : Key-Value 안에 Key-Value가 들어가있는 구조

Redis는 기본적으로 위와 같이 다양한 자료구조를 지원한다. 이 중 **Strings와 Hashes**에 대해 자세히 알아보자.

### Strings
일반적으로 사용되는 Key-Value의 형식을 가지는 자료구조이다.
key와 value의 관계는 1:1이고 key는 중복될 수 없다.
이미 존재하는 Key를 하나 더 추가하려고 하는 경우 기존에 존재하던 value가 덮어씌워진다.  
* 기본 명령어 : get, set, del, exists

### Hashes
내부에 또 다른 Key-Value를 저장할 수 있는 자료구조이다.
Hashes는 key 하나에 여러 개의 field와 value로 구성된다.
Key-Field-Value 형식으로 key는 중복될 수 없고 key 안에 여러 개의 Field-Value를 가질 수 있다.  
* 기본 명령어 : hget, hgetall, hset, hdel, hexists

### 저장하기
```
1. String : 
SET key value // key에 value를 저장

2. Hash :
HSET key field value // key에 field와 value를 쌍으로 저장
```

### 읽기
```
1. String :
GET key value // key에 해당하는 value를 반환

2. Hash :
HGETALL key // key에 저장되어있는 field와 value를 모두 반환
HGET key field // key와 field에 해당하는 value를 반환
```

### 지우기
```
1. String :
DEL key // key를 삭제

2. Hash :
HDEL key field // key에서 field를 삭제
```

### 존재여부 확인
```
1. String :
EXISTS key // key가 존재하면 1, 존재하지 않는다면 0

2. Hash :
HEXISTS key field // key에 field가 존재하면 1, 존재하지 않는다면 0
```

## 기타 유용한 Redis 툴
1. [Redis command를 테스트 해볼 수 있는 사이트](https://redis.io/commands/hset/) : Redis를 세팅하다 보면 다양한 명령어들을 테스트해보고 싶어지는 상황이 생기는데, 이럴 때 Redis 서버에 직접 들어가지 않아도 위의 사이트에서 커맨드를 쳐보며 테스트할 수 있다.
<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/redis1.png" alt="redis1.png">
</div>

2. [Redis GUI툴 - Another Redis Desktop Manager](https://github.com/qishibo/AnotherRedisDesktopManager) : Redis에 들어가있는 데이터들을 편하게 확인할 수 있는 무료 GUI툴이다.
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/redis2.png" alt="redis2.png">
</div>
왼쪽 상단에 있는 `New Connection` 버튼을 눌러 Redis 서버의 IP주소와 Port번호를 입력해주면 된다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/redis3.png" alt="redis3.png">
</div>
연결이 잘 되면 위와 같이 해당 Redis에 저장되어있는 데이터들을 Key와 Value 형태로 한 눈에 볼 수 있다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/redis4.png" alt="redis4.png">
</div>
뿐만 아니라, Connection 정보 오른쪽에 있는 버튼을 통해 CLI로 바로 접속해서 명령어를 입력할 수도 있다.

설치 방법은 여러 가지가 있다.
깃헙 레포 Release에서 dmg 파일을 다운받아 설치하거나 앱스토어에서 유료로 구매하는 방법이 있는데 나의 경우는 brew로 설치했다.

```bash
$ brew install --cask another-redis-desktop-manager
```

*여기서 맥 보안설정 관련 팝업(확인되지 않은 개발자의 소프트웨어)이 뜨는 경우가 있는데 `설정 > 보안 및 개인 정보 보호`에서 `확인 없이 열기`를 클릭해주면 된다.*
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/redis5.png" alt="redis5.png">
</div>

## References
- [우아한 레디스 by 강대명님](https://www.youtube.com/watch?v=mPB2CZiAkKM&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9CTech)
- [Redis 공식사이트](https://redis.io/commands/hset/)
- [Another Redis Desktop Manager](https://github.com/qishibo/AnotherRedisDesktopManager)
