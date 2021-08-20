---
layout: post
title: Git/Git 브랜치 전략
date: 2021-08-20
excerpt: "Git Flow vs GitHub Flow"
tags: [git, github, branch]
comments: true
---

## Git Flow
- 기본적으로 5종류의 branch를 사용
- feature > develop > release > hotfix > master
- ✅ 명령어가 명료하게 나와있음
- ✅ 거의 모든 에디터와 IDE에 플러그인으로 이미 존재하고 있음
- ❌ branch 구조가 복잡하여 관리에 어려움
- ❌ 쓰이지 않거나 애매한 위치의 branch가 생길 수 있음

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/gitbranch1.png" alt="gitbranch1.png">
</div>


## GitHub Flow
- release branch가 명확하게 구분되지 않은 시스템에서 사용
- Git Flow와 달리 원격 서버에 자신이 하고 있는 일들을 지속적으로 push해서 팀원들이 수월하게 확인가능하도록 함
- 병합을 위한 준비가 완료되었을 경우, pull request를 생성
- 기능에 대한 리뷰가 끝난 후 master로 병합 (master로 병합되면 즉시 배포작업 수행)
- ✅ branch 구성 전략이 단순함
- ✅ 코드 리뷰를 자연스럽게 사용할 수 있음
- ✅ CI/CD가 필수가 됨
- ✅ Github 사이트에서 제공해주는 기능을 모두 사용해 작업을 진행하게 도와줌
- ❌ CI/CD가 이루어지지 않은 환경에서는 수동으로 해야 함
- ❌ 프로젝트의 규모가 커지면 관리에 어려움 발생가능

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/gitbranch2.png" alt="gitbranch2.png">
</div>
