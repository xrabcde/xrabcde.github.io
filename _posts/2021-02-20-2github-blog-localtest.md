---
layout: post
title: GitHub/ 블로그 로컬환경에서 테스트하기
date: 2021-02-20
excerpt: "fork한 github.io의 로컬 작업 환경 구축하기"
tags: [github, blog]
comments: true
---

처음 깃헙블로그를 만들면서 로컬환경을 구축하는 과정에서 생각보다 과정이 복잡하고 오래 걸려서 나중에 헷갈릴 때 다시 보기 위해 기록한다. 

## 1. 상황
jekyll theme을 내 GitHub repository에 fork 한 후, 내 pc에 clone 했다.
```
> git clone https://github.com/xrabcde/xrabcde.github.io.git
```
config와 index, about 등 개인화 설정이 필요한 부분을 수정한 뒤 반영한 내용들을 확인해보고 싶은데
Github에 계속 push한 뒤 기다렸다가 새로고침해서 보는 과정이 너무 불편했다.
결론적으로는 `깃헙블로그 로컬` 등의 키워드로 구글링해가며 로컬환경 구축에 __성공!__ 했다.

## 2. 과정
먼저, <https://rubyinstaller.org/downloads/>에서 `=>` 표시되어있는 __Ruby+Devkit 2.7.2-1 (x64)__ 를 설치했다.
그 다음 윈도우 검색창에서 `Start command prompt with Ruby`를 실행해서 인코딩을 부여해주는 명령인 `chcp 65001`을 입력했다.
```
> chcp 65001
```
그리고 내 pc내 저장소를 clone 해두었던 위치로 이동하고 gem 명령을 통해 Jekyll 라이브러리를 설치한 뒤, 초기화 설정을 진행했다.
```
> xrabcde.github.io> gem install bundler jekyll minima jekyll-feed tzinfo-data rdiscount
...
> xrabcde.github.io> jekyll new xrabcde.github.io
```
그리고 나서 지킬 서버를 구동하면 끝!
```
> xrabcde.github.io> bundle exec jekyll serve
```
이라고 대부분의 블로그에서 알려주었지만 왜인지 나는 계속 오류가 났다.
열심히 구글링해 본 결과, 오류는 `bigdecimal`의 버전이 맞지 않아서 발생하는 것으로 보였고
이를 맞춰주기 위해 `Gemfile`에 `gem 'bigdecimal', '1.3.5'`를 추가했다.
```
source "https://rubygems.org"
...
gem 'jekyll-feed'
gem 'bigdecimal', '1.3.5'
```
여기까지 하고 다시 `bundle exec jekyll serve` 명령어를 통해 지킬 서버를 구동했더니 다행히 잘 작동이 되었다. 
하지만, 블로그의 홈화면까지만 로컬로 구동이 되고 버튼을 눌러 다른 화면으로 이동 시
xrabcde.github.io 주소로 변경이 되어서 로컬에서 실시간으로 바꾸는 내용들이 반영이 되지 않았다.  
이번에는 `_config.yml`의 `url`부분을 수정해주어야 했다. 로컬에서 작업내용을 실시간으로 확인하기 위해서는 
`url` 부분을 __http://localhost:4000__ 으로 수정한 뒤 지킬서버를 구동시켜야 한다.
```
url:                http://localhost:4000
#url:                https://xrabcde.github.io
```
url 수정까지 완료한 뒤 `bundle exec jekyll serve` 명령을 통해 다시 지킬 서버를 구동했더니 실시간으로 반영이 잘 되는 것을 확인할 수 있었다!

## 3. 결론
글로 적으니 굉장히 빨리 이 문제들을 해결한 것 같이 보이지만 블로그를 처음 세팅하는 것부터해서 거의 하루종일이 걸렸다.. 
앞으로 블로그를 꾸준히 잘 하려면 로컬 서버를 띄우는 방법을 잘 숙지해놓고 있어야할 것 같아서 첫 블로그 글로 이 내용을 정리해보았다.  
위의 모든 과정은 모두 초기에 한 번만 해주면 되는 것이고, 앞으로는 `_config.yml`에 주석처리 해놓은 url을 바꿔가면서 사용하면 될 것 같다.
- 로컬 작업 시
  - url : http://localhost:4000 으로 변경 후
  - `bundle exec jekyll serve` 실행
- GitHub에 push 할 때는
  - url : https://xrabcde.github.io 로 다시 변경하기!
