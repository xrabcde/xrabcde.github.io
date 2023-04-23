---
layout: post
title: JPA GenerationType과 MySQL auto_increment 간의 관계
date: 2023-04-23
excerpt: "JPA에서 GenerationType 설정과 MySQL auto_increment 설정에 따른 PK 채번 방식에 대해 알아보자"
tags: [jpa, spring, database, db, mysql]
comments: true
---

## 개요
회사에서 2.0 서비스를 출시하며 구 데이터를 마이그레이션하여 운영을 유지해야하는 상황이 있었다. 
테이블 간 연관관계가 있었기 때문에 id 값은 기존 값을 그대로 마이그레이션 해오도록 했는데
문제는 이후에 새로 생성된 데이터들과 마이그레이션된 데이터 사이에서 PK 충돌이 일어나기 시작한 것이다. 
이 문제를 해결하는 과정에서 공부했던 JPA의 GenerationType과 MySQL auto_increment 설정 간의 관계에 대해 정리해보려고 한다.

## GenerationType 종류
먼저 JPA의 기본키 생성 전략에 대해 알아보자. GenerationType은 총 4가지가 있다. 

<div style="width:90% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233832032-e4da18a0-d396-4215-b639-5cee97ceec4b.png">
</div>

정의되어 있는 내용을 읽어보면 다음과 같다.

### 1 . TABLE :
Indicates that the persistence provider must assign primary keys for the entity using an underlying database table to ensure uniqueness.  
PK의 고유성을 보장하기 위해 데이터베이스 테이블을 사용한다. 즉, 키 생성 전용 테이블을 만들어 데이터베이스 시퀀스를 흉내내는 방식이다. 운영에서는 잘 사용하지 않는다.

### 2 . SEQUENCE : 
Indicates that the persistence provider must assign primary keys for the entity using a database sequence.  
데이터베이스의 시퀀스를 사용하여 PK를 채번한다. 시퀀스를 사용하는 Oracle, DB2, H2 등의 DB에서 사용하고, 만약 시퀀스를 사용하지 않는 DB 벤더를 사용할 경우 시퀀스 테이블을 생성해주어야 한다.

### 3 . IDENTITY : 
Indicates that the persistence provider must assign primary keys for the entity using a database identity column.  
데이터베이스의 ID 열을 사용하여 기본 키를 할당한다. 즉, 기본키 생성 책임을 DB에게 위임하는 방식으로 MySQL의 경우는 auto_increment로 동작한다.

### 4 . AUTO : 
Indicates that the persistence provider should pick an appropriate strategy for the particular database. The AUTO generation strategy may expect a database resource 
to exist, or it may attempt to create one. A vendor may provide documentation on how to create such resources in the event that it does not support schema
generation or cannot create the schema resource at runtime.  
DB 벤더에 따라 자동으로 전략을 선택하는 방식으로, 아무것도 설정하지 않았을 때의 기본값이다.

## 문제 상황
기존의 코드에서는 네 가지 방식 중 AUTO 전략을 사용하여 모든 객체를 생성하고 있었다.
위 설명에 따라 세 가지 방식 중 가장 적합한 방식이 자동으로 선택되었고, 로그를 보니 그 방식이 SEQUENCE 였던 것 같다. 

<div style="width:40% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233834689-628cd1d0-c62d-4eb8-aa33-4ce917ab870c.png">
</div>

MySQL DB를 사용하고 있었기 때문에 Sequence를 관리할 테이블인 hibernate_sequence 가 생성되었고 이 테이블에 다음 PK값을 저장하는 식으로 동작하고 있었다. 
이 방식은 다음에 채번할 PK 값을 시퀀스 테이블에서 가져와야 하기 때문에 객체를 저장하기 전에 한 번의 조회쿼리가 나가고, 
시퀀스 테이블의 next_val을 업데이트 해주기 위한 업데이트 쿼리와 객체를 저장하는 생성 쿼리까지 총 3번의 쿼리가 나가게 된다.

```
Hibernate: select next_val as id_val from hibernate_sequence for update
Hibernate: update hibernate_sequence set next_val= ? where next_val=?
Hibernate: insert into ex_entity (name, id) values (?, ?)
```

그리고 해당 데이터베이스의 테이블 개수와 관련없이 모든 테이블의 PK 값을 한 시퀀스 테이블에서 관리하기 때문에 모든 테이블이 PK 순서를 공유하게 된다. 
이 말이 무슨 뜻이냐면 만약 사용자 정보를 저장하는 User 테이블과 그 사람들의 할 일을 저장하는 Task 테이블이 있다고 할 때, 테이블에 저장되는 id는 다음 그림과 같이 되는 것이다.

<div style="width:100% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233835595-fd58815c-7c43-462d-9b90-7f108e897e26.png">
</div>

이런 식으로 여러 개의 테이블에 대해 한 시퀀스 테이블에서 관리하기 때문에 그림과 같이 id가 2, 3인 user는 존재하지 않게 되는 것이다.
지금까지 문제는 성능상의 문제나 불편함 정도에 그칠 수 있지만 다음 문제는 아예 기능이 동작하지 않게 되기 때문에 이 전략을 바꾸게 된 결정적인 계기가 되었다. 

아예 신규 서비스를 새로 구축하는 상황에서는 이 전략을 사용해도 크게 문제가 없다. 
나의 경우 기존에 있던 서비스를 2.0 으로 고도화를 하는 과제를 하고 있었고 이 과정에서 DB를 분리+이동하게 되어 DB는 새로 구축한 상황이었다. 
그렇기 때문에 실제 운영에 오픈하기 전까지 테스트 phase들과 QA 과정에서는 아무 문제 없이 잘 동작을 했지만 
운영에 오픈을 하기 위해 기존에 있던 1.0 데이터들을 마이그레이션하며 문제가 발생했다. 

마이그레이션 대상 테이블은 여러 개였고 user_id와 같이 서로 연관관계가 있었기 때문에 임의로 조작하지 않고 id값까지 그대로 마이그레이션 해야했다.
이 과정에서 시퀀스 테이블과 다른 테이블들의 싱크가 달라져버린 것이다. 
JPA에서는 시퀀스 테이블을 참조해 기본키 생성을 시도하기 때문에 DB에 객체 생성 요청을 하는데 DB에서는 이미 있는 PK로 생성 요청이 들어오니 아래와 같은 sql 에러가 난다.

```
org.springframework.dao.DataIntegrityViolationException: could not execute statement; 
SQL [n/a]; constraint [PRIMARY]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
```

이 문제를 해결하려면 시퀀스 테이블의 next_val 값을 임의로 변경해주면 된다.
하지만, 위에서 말했듯이 이 시퀀스 테이블은 모든 테이블의 PK 전략을 담당하기 때문에 모든 테이블의 id 값 중 가장 큰 값을 찾아 그 다음 값으로 설정해줘야 했다. 
결국, PK 생성 전략을 DB의 auto_increment 설정을 따르는 IDENTITY로 바꾸기로 했다.
그렇다면 이미 기존 데이터가 있는 상태에서 이 설정을 변경하려면 어떻게 해야하지? 라는 고민에 다시 부딪히게 된다. 

## 해결 과정
이 고민을 해결하려면 우선 JPA의 GenerationType과 DB의 auto_increment 설정에 대해 공부해야 한다. 
GenerationType에 대해서는 위에서 정리했지만 한 번 더 짚자면, 
기존의 생성방식이던 GenerationType.SEQUENCE 에서는 DB의 auto_increment 설정이 아닌 시퀀스 테이블이라는 테이블을 생성해
다음 PK값을 저장하는 방식으로 기본키를 생성했고, 그렇기 때문에 일단 모든 테이블의 auto_increment 설정은 꺼져있는 상태이다.

바꾸려고 하는 방식인 GenerationType.IDENTITY 는 DB의 auto_increment 설정에 따라 기본키가 채번되기 때문에 
테이블마다 auto_increment 설정이 켜져있어야 하고 테이블마다 기본키 순서를 다르게 가져갈 수 있다. 

먼저, 현재 테이블의 기본키 상태를 확인해보자.

```
show table status where name = '이름';
```

<div style="width:35% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233837069-a3071044-0d8e-44ee-9e19-4c671f8acf9c.png">
</div>

auto_increment 가 위와 같이 null 이라고 나오면 설정이 꺼져있는 것이다. 

테이블에 auto_increment 설정을 추가해주려면 auto_increment 하고 싶은 PK 칼럼을 업데이트 해주면 된다.
만약 ddl-auto 설정이 update 로 되어있다면 GenerationType을 IDENTITY로 바꾸고 어플리케이션을 다시 띄우면
자동으로 auto_increment 설정이 추가되겠지만 운영에서는 보통 그럴일이 없으니 
이렇게 수동으로 주의해서 칼럼 업데이트를 해줘야 한다.

```
alter table 테이블이름 modify column id bigint auto_increment not null;
```

그렇다면 기존에 이미 데이터들이 잔뜩 있는데 이제와서 auto_increment 설정을 켠다고 DB가 알아서 자동으로 다음값부터 PK를 채번할까? 궁금해질 수 있다.

<div style="width:35% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233837374-537a88f8-a736-4385-b579-a6f4006b65bb.png">
</div>

다시 auto_increment 설정 상태를 확인해보면 똑똑한 DB가 알아서 다음에 채번될 기본키의 값으로 auto_increment 설정값을 넣어준 것을 확인할 수 있다.
물론 이 값을 임의로 변경해줄 수도 있다. 

```
alter table 테이블이름 auto_increment = 123;
```

<div style="width:35% !important; margin:0 auto">
<img src="https://user-images.githubusercontent.com/56033755/233837461-8873ddc8-0d75-4730-86b4-60d5be17e9bf.png">
</div>

이렇게 하면 이제 4 ~ 122는 건너뛰고 123부터 다음 값이 저장될 것이다. 
임의로 특정 값을 건너뛰어 채번을 해야하는 경우엔 이 방법을 사용할 수 있지만, 이미 더 큰 값의 id가 있는 상황에서 이전 번호로 설정은 (당연히) 불가능하다.
만약 이전 값으로 다시 기본키 값을 줄이고 싶다면 해당 데이터를 삭제한 뒤 줄여야 한다. 
(즉, 123번 데이터가 생성되었다면 해당 데이터를 지운 뒤 auto_increment 값을 다시 4로 내려주면 가능하다.)