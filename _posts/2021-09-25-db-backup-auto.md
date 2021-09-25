---
layout: post
title: DB/MySQL DB 백업 자동화하기
date: 2021-09-25
excerpt: "mysqldump와 crontab을 이용해 MySQL DB 백업을 자동화하는 방법"
tags: [db, mysql, backup]
comments: true
---

# 개요
- [DB 백업](https://xrabcde.github.io/db-backup/) 글에서 이어지는 글입니다.
- 이 글은 백업 방법이 아닌 mysqldump를 이용한 백업을 **crontab을 통해 자동화시키는 과정**에 대해 다룹니다.

# 과정
## 1. DB 자동 백업 쉘 스크립트 작성

```bash
$ vi backup.sh

#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR=/home/ubuntu/backup_auto/
mysqldump -u root -p 1234 zzimkkong > $BACKUP_DIR"backup_"$DATE.sql

find $BACKUP_DIR -ctime +3 -exec rm -f {} \;
```

- DATE 변수에 현재 날짜 저장 (date 뒤에 띄어쓰기가 있는 것에 주의)
- BACKUP_DIR 변수에는 백업 파일들이 저장될 디렉토리 경로 저장
- mysqldump 명령을 이용하여 백업 파일 생성
- find 명령을 이용하여 3일이 지난 백업파일은 삭제
    - `-ctime n` 옵션은 파일의 퍼미션을 마지막으로 변경시킨 날짜를 기준으로 파일을 찾아낸다.

    ```bash
    +n : n일 또는 n일 이전에 퍼미션이 변경된 파일
    -n : 오늘 부터 n일 전 사이에 퍼미션이 변경된 파일
    n : 정확히 n일 전에 퍼미션이 변경된 파일
    ```

    - `-exec command {} \;` 옵션은 찾아낸 파일들을 대상으로 특정 명령을 수행한다.
- 쉘 스크립트 파일에 실행 권한 주기

```bash
$ chmod 554 backup.sh
```

## 2. crontab 을 이용하여 스크립트 자동 실행
- 예약된 작업 리스트 확인

```bash
$ crontab -l
```

- 예약된 작업 수정

```bash
$ crontab -e

0 20 * * * /home/ubuntu/backup_auto/script/backup.sh >> /home/ubuntu/backup_auto/crontab_log/backup.log 2>&1
```

- 위 크론탭 명령에서 로그 파일이 잘 생성되도록, /home/ubuntu/backup_auto/crontab_log라는 디렉토리 만들어놓기
  (**디렉토리를 미리 만들어놓지 않으면 에러 발생함!!**)
- 작업 명령 이용 방법

```bash
* * * * * 수행할 명령어
┬ ┬ ┬ ┬ ┬
│ │ │ │ └───────── 요일 (0 - 6) (0:일요일, 1:월요일, 2:화요일, …, 6:토요일)
│ │ │ └───────── 월 (1 - 12)
│ │ └───────── 일 (1 - 31)
│ └───────── 시 (0 - 23)
└───────── 분 (0 - 59)
```

- 현재 저희 DB에는 매일 20시(UTC 기준) 정각마다 명령을 수행하도록 설정해놓은 상태입니다.
- **20시(UTC 기준, KST 기준으로 새벽 5시)로 해놓은 이유?**

  DB 부하가 가장 적은 시간으로 백업시간을 정해놓는 것이 좋을 것 같다고 생각했고 저희 서비스는 새벽 시간대에 트래픽이 가장 적기 때문에 새벽 5시로 설정했습니다. 추가적으로 UTC 기준으로 작성한 이유는 EC2 서버 시간이 UTC 기준이기 때문이고 crontab을 위해서 EC2 서버 시간을 바꾸기는 리스크가 있을 것 같아서 그냥 명령수행 시간을 UTC 기준으로 작성해 놓았습니다.

- cron 재시작

```bash
$ sudo service cron restart
```

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/backup_auto1.png" alt="backup_auto1.png">
</div>

## 3. 백업 파일 확인
- 이렇게 쉘 스크립트 작성 후 crontab으로 해당 스크립트를 자동 실행되도록 설정하면 아래와 같이 backup sql 파일들이 자동 생성되는 것을 확인할 수 있습니다.
- 참고로 3일이 경과한 sql 파일들은 삭제하도록 작성했습니다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/backup_auto2.png" alt="backup_auto2.png">
</div>

끝!

## 참고
- [MySQL 자동 백업](https://cjh5414.github.io/mysql-backup-automatically/)
- [Linux crontab 이용하여 작업 명령 예약하기](https://cjh5414.github.io/linux-crontab/)
