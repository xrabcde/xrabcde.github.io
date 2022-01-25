---
layout: post
title: DB/MySQL DB 모든 테이블 TRUNCATE 하기
date: 2022-01-24
excerpt: "MySQL 모든 테이블의 schema는 유지하면서 데이터만 날리는 방법"
tags: [db, mysql, truncate]
comments: true
---

# 개요
프로젝트 운영중에 DB의 schema가 크게 변경되면서 모든 테이블의 데이터를 날려야 할 일이 생겼다.
모든 테이블의 schema는 유지하면서 데이터만 싹 날리기 위해 `TRUNCATE DATABASE`가 가능하면 좋겠지만 
아쉽게도 TRUNCATE는 TABLE에만 가능하고, 외래키 의존성 때문에 TRUNCATE의 순서도 주의해서 작성해주어야 하는 번거로움이 있다.
이를 **외래키 검사 옵션을 해제해주는 방법**으로 간단히 해결할 수 있다.

# 과정
1. 먼저 데이터베이스의 외래키 검사 옵션을 해제해준다.
```
SET foreign_key_checks = 0;
````

2. DB의 전체 테이블에 대한 TRUNCATE를 실행한다.
나의 경우, flyway를 사용하고 있었기 때문에 flyway_schema_table을 제외한 나머지 테이블을 TRUNCATE 해주는 명령문을 직접 작성했다.
(만약, 모든 테이블에 대해서 TRUNCATE할 경우라면 CONCAT을 이용하는 방법도 있다.)
```
TRUNCATE TABLE map;
TRUNCATE TABLE member;
TRUNCATE TABLE preset;
TRUNCATE TABLE reservation;
TRUNCATE TABLE space;
````

3. 해제했던 외래키 검사 옵션을 다시 복원한다.
```
SET foreign_key_checks = 1;
````
