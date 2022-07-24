---
layout: post
title: Github/mac에서 여러 개의 github 계정 사용하기
date: 2022-01-25
excerpt: "한 맥북에서 두 가지의 github 계정으로 여러 프로젝트 동시에 진행하기"
tags: [github, mac, terminal]
comments: true
---

한 맥북에서 github 회사계정과 개인계정으로 여러 프로젝트를 동시에 진행해야 할 때 ssh-key 설정을 이용해 여러 github 계정을 등록할 수 있다.

## 개인 계정으로 ssh-key 생성
- 먼저 개인 계정의 ssh-key를 생성한다.
```bash
$ ssh-keygen -t rsa -b 4096 -C 'xrabcde@gmail.com'
```
- `Enter file in which to save the key (/Users/hwang/.ssh/id_rsa):` 가 나오면 아무것도 입력하지 않고 그냥 엔터를 쳐 기본 파일명(`id_rsa`)으로 ssh-key를 생성한다.
- 생성한 ssh-key를 등록해준다.
```bash
$ ssh-add -K ~/.ssh/id_rsa
```
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/github_multi1.png" alt="github_multi1.png">
</div>

- 개인 github 계정에 로그인 후 ssh 키를 추가한다.
<div style="width:50% !important; margin:0 auto">
<img src="/assets/img/github_multi2.png" alt="github_multi2.png">
</div>
- New SSH key 버튼을 클릭해 키 추가
<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/github_multi3.png" alt="github_multi3.png">
</div>
- Title에는 본인이 키를 식별할 수 있도록 적절히 텍스트를 입력하고 Key 부분에는 위에서 생성한 ssh key값을 복사해서 입력
```bash
$ pbcopy < ~/.ssh/id_rsa.pub //id_rsa.pub가 클립보드에 복사됨
```

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/github_multi4.png" alt="github_multi4.png">
</div>

- ❗️주의 : 키를 붙여넣을 때, 공백이나 줄바꿈이 들어가면 제대로 추가되지 않으니 주의

## 회사 계정으로 ssh-key 생성
- 그 다음 이 과정을 반복하여 회사 계정의 ssh-key를 생성해주는데 이번에는 파일 이름을 지정해주어야 하므로  
`Enter file in which to save the key (/Users/hwang/.ssh/id_rsa):` 에 본인이 구분할 수 있는 `id_rsa_work` 과 같은 이름을 입력한다.
```bash
$ ssh-keygen -t rsa -b 4096 -C 'bada.jo@kakaoenterprise.com'
```
<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/github_multi5.png" alt="github_multi5.png">
</div>

- 마찬가지로 이번에 생성한 ssh-key도 등록해준다.
```bash
$ ssh-add -K ~/.ssh/id_rsa_work
$ pbcopy < ~/.ssh/id_rsa_work.pub //id_rsa_work.pub가 클립보드에 복사됨
```

- 회사 github 계정에 로그인 후  ssh 키를 추가한다.

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/github_multi6.png" alt="github_multi6.png">
</div>

- `~/.ssh` 에서 ls 명령어를 통해 지금까지 생성한 ssh-key 들을 확인할 수 있다.
    <div style="width:100% !important; margin:0 auto">
    <img src="/assets/img/github_multi7.png" alt="github_multi7.png">
    </div>
    - `id_rsa` : 개인계정-비공개-ssh
    - `id_rsa.pub` : 개인계정-공개-ssh
    - `id_rsa_work` : 회사계정-비공개-ssh
    - `id_rsa_work.pub` : 회사계정-공개-ssh
- `ssh-add -l` 명령어를 통해 등록된 key 목록을 확인할 수 있다.

## .ssh/config 생성
- 이제 두 가지 계정을 편하게 사용하기 위해 SSH 설정파일을 작성해야 한다.
```bash
$ cd ~/.ssh
$ vi config
```

- vi 에디터로 config 파일을 열어 아래와 같이 작성한다.
```
Host work
    HostName github.com
    User bada-jo
    IdentityFile ~/.ssh/id_rsa_work
Host xrabcde
    HostName github.com
    User xrabcde
    IdentityFile ~/.ssh/id_rsa
```

## 프로젝트 세팅
- ssh 설정을 이용하려면 git 프로젝트를 클론받을 때 HTTPS가 아닌 SSH 링크를 사용해야 한다.
    <div style="width:70% !important; margin:0 auto">
    <img src="/assets/img/github_multi8.png" alt="github_multi8.png">
    </div>
- ssh config 설정 이후 프로젝트 클론 받는 경우 : git@`xrabcde` 와 같이 위에서 지정한 Host 네임을 입력해주어야 한다.
```bash
$ git clone git@xrabcde:xrabcde/xrabcde.github.io.git
```

- 이미 이전에 클론 받은 프로젝트의 경우 : remote url을 확인한 후 원하는 url로 변경해준다.
```bash
$ git remote -v
$ git remote set-url origin git@work:C**/repo-name.git
```

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/github_multi9.png" alt="github_multi9.png">
</div>