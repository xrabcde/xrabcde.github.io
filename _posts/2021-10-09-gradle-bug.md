---
layout: post
title: Gradle/Window 터미널에서 gradle build 로그가 깨지는 현상 해결하기
date: 2021-10-09
excerpt: "Window Git Bash 에서 ./gradlew build 로그가 깨지는 현상 해결하기"
tags: [gradle, window, intellij, gitbash, terminal]
comments: true
---

## 1. 상황
Window에서 IntelliJ에 Git Bash Terminal을 사용중이었는데 `./gradlew clean build`를 할 때마다 매번 로그가 깨져서 나왔다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/gradlebug1.png" alt="gradlebug1.png">
</div>

대략... 이런식으로...  
무시하고 그냥 쓰려고 했는데 도저히 디버깅이 힘들어서 해결해보았다. 참고로 한글이 깨지는 현상은 아니라서 `window intellij 깨짐` 뭐 이런
키워드로 구글링해서 나오는 **UTF-8 Encoding** 같은 설정으로는 절대 해결이 되지 않는다. 

## 2. 과정
일단 `window terminal gradle output`와 같은 키워드로 검색을 하다가 [microsoft 레포에 있는 이슈](https://github.com/microsoft/terminal/issues/7091)를 하나 찾았다.
읽어보니 나와 똑같은 문제를 겪고 있었고 마땅한 해결책은 없어 보였다. 그래서 해당 이슈에 링크되어있는 [gradle 레포에 있는 이슈](https://github.com/gradle/gradle/issues/13279)를 들어가보았다.
많은 사람들이 나와 같은 이슈로 화를 내고 있었고(ㅋㅋㅋㅋ) 작년 5월에 오픈된 이슈인데 저번달까지도 토론이 이어지고 있었다.  
결론만 적어보자면 [해당 이슈](https://github.com/gradle/gradle/issues/13279)에 제일 마지막에 있는 코멘트를 참고해서 해결할 수 있었다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/gradlebug2.png" alt="gradlebug2.png">
</div>

[해당 코멘트](https://github.com/gradle/gradle/issues/13279#issuecomment-925523692)를 참고해 Git Bash에 
`BASH_ALIASES[./gradlew]='TERM=cygwin ./gradlew'` 이 명령어를 입력해보았다.

<div style="width:60% !important; margin:0 auto">
<img src="/assets/img/gradlebug3.png" alt="gradlebug3.png">
</div>

그리고 나서 `./gradlew clean build`를 실행해봤더니 로그가 깨지지 않고 잘 나오는 것을 확인할 수 있었다!!

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/gradlebug4.png" alt="gradlebug4.png">
</div>

## 3. 결론
이렇게 Git Bash에 명령어를 입력하는 것으로 gradle build 로그가 깨지는 것을 해결할 수 있으나 설정을 완전히 바꿔주는 것이 아니므로
IntelliJ를 껐다가 켜면 다시 그 전과 같이 로그가 깨져 보이게 된다. 
이 현상을 완전히 해결하려면 [위 코멘트](https://github.com/gradle/gradle/issues/13279#issuecomment-925523692)에 나와있던대로
`~./bashrc` 파일을 수정해주어야 한다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/gradlebug5.png" alt="gradlebug5.png">
</div>

`sudo vi ~/.bashrc` 명령어를 입력해서 ~/.bashrc 파일에 아까 입력했던 명령어를 추가하고 저장해준다. 그리고 나서 
변경해준 내용을 적용하기 위해 `source ~/.bashrc` 명령어를 입력하면 된다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/gradlebug6.png" alt="gradlebug6.png">
</div>

이제 IntelliJ를 껐다가 켜도 해당 설정이 적용되어 로그가 깨지지 않고 잘 나오는 것을 확인할 수 있다!

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/gradlebug7.png" alt="gradlebug7.png">
</div>

---

### 참고
- [microsoft 이슈](https://github.com/microsoft/terminal/issues/7091)
- [gradle 이슈](https://github.com/gradle/gradle/issues/13279)
