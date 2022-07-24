---
layout: post
title: Network/서브도메인과 함께 TLS(letsencrypt) 인증서 발급하기
date: 2022-01-30
excerpt: "서브도메인과 함께 TLS(letsencrypt) 인증서 발급하기"
tags: [dns, network, nginx, letsencrypt]
comments: true
---

# 개요
HTTPS 연결을 위해 보통 무료로 사용가능한 letsencrypt를 많이 사용하는데
이 글에서는 letsencrypt에서 지원하는 **서브도메인까지 묶어서 인증서를 발급하고 사용하는 방법**에 대해 정리하려고 한다.

# 과정
### 처음으로 도메인을 등록하는 경우
letsencrypt를 설치하고 처음 도메인을 등록할 때 보통 아래와 같은 명령어를 사용한다.
(나의 경우, Docker를 이용해 Reverse proxy를 구성했지만 Docker를 사용하지 않아도 무방하다)
```bash
$ docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d 'zzimkkong.com' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

여기서 `-d`를 이용하면 체인형식으로 서브도메인을 연결할 수 있다.  
`-d 'example.com' -d '*.example.com'`
```bash
$ docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d 'zzimkkong.com' -d '*.zzimkkong.com' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/letsencrypt_subdomain1.png" alt="letsencrypt_subdomain1.png">
</div>

서브도메인을 `-d`로 연결하여 인증서를 발급하는 경우 위와 같이 TXT를 생성하라는 안내가 두 번 나온다. (하나가 먼저 나오고 엔터를 치면 한 개가 더 나오므로 한 개씩 한 개씩 TXT 레코드를 추가한다.)  
❗️ 주의 : 두 가지를 모두 TXT 레코드로 추가해주어야 인증서가 정상적으로 생성된다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/letsencrypt_subdomain2.png" alt="letsencrypt_subdomain2.png">
</div>

`Press Enter to Continue`를 하기 전에 [DNS레코드 생성 확인 사이트](https://toolbox.googleapps.com/apps/dig/#TXT/)에서 TXT 레코드가 잘 생성되었는지를 반드시 확인해야 한다.
두 가지 레코드를 동시에 생성해서 그런지는 몰라도 레코드가 생성되기까지 최소 20분에서 길게는 1시간까지 걸렸다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/letsencrypt_subdomain3.png" alt="letsencrypt_subdomain3.png">
</div>

TXT 레코드가 잘 등록이 되었다면 위의 사이트에서 이렇게 두 개의 TXT 값들이 조회된다.
값이 잘 조회되는 것을 꼭 확인한 뒤, 다음 단계를 진행한다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/letsencrypt_subdomain4.png" alt="letsencrypt_subdomain4.png">
</div>

`Successfully received certificate`가 뜨면 인증서가 잘 발급된 것이다.

### 이미 등록한 체인에 도메인을 추가 또는 삭제하는 경우
이미 도메인을 등록한 상황이라면 인증서를 굳이 재발급하지 않고도 체인에 도메인을 추가하거나 삭제할 수 있다.  
처음 인증서를 발급할 때와 비슷하게 명령어에 `-d`로 연결하여 도메인 정보를 입력해준다.

``` bash
$ certbot --cert-name zzimkkong.com -d zzimkkong.com -d www.zzimkkong.com -d api.zzimkkong.com
```

여기서 `--cert-name` 뒤에 나오는 `zzimkkong.com`은 체인명이고, 그 뒤에 나오는 `-d zzimkkong.com`이 도메인명이다.  
도메인을 추가 또는 삭제하는 경우 삭제할 도메인은 제거하고 추가할 도메인은 추가해서 **이 체인에 등록될 도메인을 모두 적어줘야 한다.** 즉, 현재 등록되어 있는 도메인이 zzimkkong.com, www.zzimkkong.com인 상황에서 api.zzimkkong.com을 추가하고 싶다면 위처럼 세 가지 도메인을 모두 적어줘야 한다는 것이다.