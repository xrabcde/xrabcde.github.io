---
layout: post
title: Network/nginx에 여러 서버, 여러 도메인 연동하기(with letsencrypt)
date: 2022-01-30
excerpt: "한 인스턴스에 두 WAS를 동시에 띄워보자 (부제 : 극한의 다이어트)"
tags: [dns, network, nginx, letsencrypt]
comments: true
---

# 개요
우아한테크코스에서 하던 회의실 예약 프로젝트 `찜꽁`이 우테코 기간동안은 ~~인스턴스를 자그마치 52개나 사용하며~~
부족함없는 행복한 개발을 하다가 수료 후 갑자기 인스턴스를 2개로 단축해야하는 (;;;;) 상황이 생겼다 ㅋㅋㅋㅋㅋ

때문에 그 전에는 모든 환경마다 인스턴스를 따로 따로 구성해서 사용하던 것을 이제 한 인스턴스에 몰아넣어보기로 했다.
인프라를 정리하기 위해 불필요한 DB Replication과 Redis 등 제거할 수 있는 애들은 먼저 정리를 했고,
이 글에서는 **한 인스턴스에서 포트번호를 다르게 해서 개발용(DEV) 환경과 운영용(PROD) 환경을 동시에 구축한 과정**을 정리하려고 한다.
(근데 이제 nginx와 DNS와 letsencrypt 삽질을 곁들인..)

# 과정
### application properties 설정
먼저 우리 팀은 인스턴스 2개를 하나는 DB용, 하나는 WAS용으로 사용하기로 하고, 8080포트에 운영용 환경을 먼저 구축해놓은 상태였다.

```
# application-prod.properties
server.port=8080

spring.datasource.url=jdbc:mysql://{db의 private IP}:3306/zzimkkong
```

여기에 8081포트로 DEV용 WAS를 하나 더 띄우면서 DB도 운영용과 개발용을 분리하기 위해 `application-dev.properties`에 다음과 같은 설정을 추가했다.

```
# application-dev.properties
server.port=8081

spring.datasource.url=jdbc:mysql://{db의 private IP}:3306/zzimkkong_dev
```

이렇게 하면 운영용과 개발용 모두 한 인스턴스에 있는 DB를 사용하지만 운영용에서는 `zzimkkong`이라는 database를 개발용에서는 `zzimkkong_dev`라는 database를 사용하게 된다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/nginx_subdomain1.png" alt="nginx_subdomain1.png">
</div>

### reverse proxy 구성하기
여기까지는 아주 순조롭게 진행할 수 있었지만, 문제는 reverse proxy를 구성하면서 발생한다.
프론트에게 dev용 요청 url과 prod용 요청 url을 각각 알려주어야 하는데 
그 당시 docker로 구성한 reverse proxy는 80포트로 들어오는 모든 요청을 8080포트로 보내주고 있었기 때문에 이걸 도메인에 따라 다른 포트로 가도록 설정해주어야 했다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/nginx_subdomain2.jpg" alt="nginx_subdomain2.jpg">
</div>

그림으로 나타내면 대충 위와 같은 구조가 된다. 클라이언트는 80포트로 요청을 보내게 되고 이를 SSL 인증서를 사용해 443 포트로 리다이렉트 한 뒤 도메인에 따라 DEV인지 PROD인지 확인해서 DEV라면 8081포트로 PROD라면 8080포트로 요청이 가야한다.

그래서 일단 귀찮은 Docker를 버리고 nginx를 직접 설치하는 방법으로 이를 간단히 해결해보기로 했다. 먼저 [서브도메인과 함께 TLS(letsencrypt) 인증서 발급하기](https://xrabcde.github.io/letsencrypt-subdomain/) 글을 통해 두 가지 도메인에 대한 DNS 설정과 인증서를 모두 발급해놓은 상태에서 nginx 설치부터 시작했다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/nginx_subdomain3.png" alt="nginx_subdomain3.png">
</div>

```
$ sudo apt-get install nginx
```

nginx 설치 후 기존의 설정 파일을 삭제해준다. 여기서부터 등장하는 `sites-enabled`와 `sites-available`을 혼동하지 않도록 주의하자!! ~~(나도 알고싶지 않았...ㄷ...)~~
두 가지 default 파일을 모두 삭제한다.

```
$ rm /etc/nginx/sites-enabled/default
$ rm /etc/nginx/sites-available/default
```

그 다음 `sites-enabled`에 두 개의 새로운 설정 파일을 작성한다. 나의 경우, dev용과 prod용이 필요했기 때문에 `dev.conf`와 `prod.conf` 두 개를 작성했다.

```
$ vi /etc/nginx/sites-enabled/dev.conf
$ vi /etc/nginx/sites-enabled/prod.conf
```

```
# dev.conf

server {
        listen 80;
        listen [::]:80;

        server_name api.zzimkkong.com;
        return 301 https://api.zzimkkong.com$request_uri;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name api.zzimkkong.com;

        ssl_certificate /etc/letsencrypt/live/api.zzimkkong.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/api.zzimkkong.com/privkey.pem;

        location / {
                proxy_pass http://127.0.0.1:8081;
        }
}
```

```
# prod.conf

server {
        listen 80;
        listen [::]:80;

        server_name k8s.zzimkkong.com;
        return 301 https://k8s.zzimkkong.com$request_uri;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name k8s.zzimkkong.com;

        ssl_certificate /etc/letsencrypt/live/zzimkkong.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/zzimkkong.com/privkey.pem;

        location / {
                proxy_pass http://127.0.0.1:8080;
        }
}
```

두 파일에서 확인해줘야하는 부분은 **ssl key의 경로, proxy_pass의 포트번호, server_name과 redirect 주소**이다. 이제 이렇게 작성한 파일을 `sites-available`과 심볼릭 링크로 엮어준다.

```
$ ln -s /etc/nginx/sites-enabled/dev.conf /etc/nginx/sites-available/
$ ln -s /etc/nginx/sites-enabled/prod.conf /etc/nginx/sites-available/
```

그러면 `sites-available`에도 `dev.conf`와 `prod.conf`라는 파일이 생성된다.

<div style="width:40% !important; margin:0 auto">
<img src="/assets/img/nginx_subdomain4.png" alt="nginx_subdomain4.png">
</div>

그 다음 nginx를 restart 해주면 설정이 완료된다!

```
# nginx 관련 명령어

$ sudo service nginx restart # nginx 재시작
$ sudo service nginx status # nginx 상태확인
```

# 결론
~~막상 이렇게 인스턴스 2개로도 잘 돌아가는 걸 알고 나니까 그동안 얼마나 낭비를 했었는ㅈ...~~  
포비 감사합니다! :)