---
layout: post
title: GitHub/Jekyll 깃헙블로그에 GitHub 댓글기능 추가하기 (with Utterances)
date: 2021-06-26
excerpt: "Jekyll github.io에 Utterances를 이용해 댓글 붙이기"
tags: [github, blog]
comments: true
---

## 1. 상황
[Jekyll 테마](https://taylantatli.github.io/Moon/) 를 fork 해서 GitHub 블로그를 사용하고 있는데
이 테마에선 기본 댓글 플랫폼으로 `Disqus`를 적용하고 있다. 하지만, `Disqus`를 이용해 댓글을 작성하려면
Disqus를 가입해야 하고 무거운 데다가 광고도 많이 붙는다. 
그래서 fork 받은 테마에 댓글 부분이 다 구현되어 있었지만 포스팅을 할 때 항상 `comments: false`로 하여
댓글 관련 기능은 아예 이용하지 않고 있었다.  
그러던 중, 다른 사람의 GitHub 블로그를 보다가 GitHub 계정으로 댓글을 달 수 있다는 사실을 알았다. 
검색해보니 `Utterances`라는 댓글 플랫폼을 이용하는 것이었다.  

`Utterances`는 우선 광고가 없고 가볍다고 하며, 최대 장점은 **따로 가입할 필요없이 GitHub 계정으로 댓글을 달 수 있다는 것**이다.
내 블로그를 보는 사람은 대부분 개발을 공부하는 사람들이기 때문에 GitHub 계정으로 댓글을 달 수 있다는 점이 굉장히 큰 장점으로 느껴졌다.
뿐만 아니라, GitHub 레포에 Issue를 통해 댓글이 달리는 방식이기 때문에 메일로 댓글 알림도 받을 수 있고,
마크다운 문법을 사용할 수 있다.

## 2. 과정
먼저, 댓글 Issue가 올라올 public 저장소를 정하거나 새로 생성한다.
나의 경우는 [댓글 이슈가 올라올 전용 Repository](https://github.com/xrabcde/xrabcde.github.io-comments) 를 새로 만들었다.

그 다음, GitHub App에서 Utterance를 설치한다. [Utterance 링크](https://github.com/apps/utterances) 에 들어가 설치하면 된다.
<div style="width:600px !important; margin:0 auto">
<img src="/assets/img/comments1.png" alt="comments1.png">
</div>

나의 경우, 이미 설치했기 때문에 Configure라고 나오는데 Install을 눌러 설치해주면 된다.

Install을 누르면 저장소를 선택하는 화면이 나온다.
<div style="width:600px !important; margin:0 auto">
<img src="/assets/img/comments2.png" alt="comments2.png">
</div>

앞에서 만든 Repository를 선택하고 Install을 눌러 설치를 완료한다. 설치가 완료되면 저장소를 설정할 수 있는 설정 페이지로 이동한다.
<div style="width:600px !important; margin:0 auto">
<img src="/assets/img/comments3.png" alt="comments3.png">
</div>

<div style="width:600px !important; margin:0 auto">
<img src="/assets/img/comments4.png" alt="comments4.png">
</div>

repo에 `계정명/저장소이름` 을 입력하고 블로그 포스트와 이슈 매핑 방법에 대한 설정을 해주었다.
이슈가 생성될 때 블로그 글 경로가 이슈의 제목으로 생성되는 설정이다.

이슈 라벨은 굳이 설정할 필요는 없지만 구분을 위해 `comments` 를 붙이도록 설정했다.
<div style="width:600px !important; margin:0 auto">
<img src="/assets/img/comments5.png" alt="comments5.png">
</div>

설정을 마치면 html 스크립트가 나온다.
```
<script src="https://utteranc.es/client.js"
        repo="xrabcde/xrabcde.github.io-comments"
        issue-term="pathname"
        label="comments"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```
이 스크립트를 `post.html` 과 같은 스크립트에 추가해주면 된다.
내가 사용하고 있는 Jekyll 테마는 기본적으로 Disqus를 사용하고 있었으므로 [disqus 관련 코드들은 모두 지우고 이 코드를 추가해줬다.](https://github.com/xrabcde/xrabcde.github.io/commit/b4aef9df34ce44ca330f86d5f158423c281deef9)
<div style="width:700px !important; margin:0 auto">
<img src="/assets/img/comments7.png" alt="comments7.png">
</div>

이제 GitHub 저장소에 push 하면 댓글이 하단에 추가된 것을 확인할 수 있다.
`page.comments`에 따른 설정도 코드에 있기 때문에 이전과 같이 `comments: false`로 포스팅하면 댓글기능을 사용하지 않을 수도 있다.

## 3. 결론
<div style="width:700px !important; margin:0 auto">
<img src="/assets/img/comments6.png" alt="comments6.png">
</div>

이제 블로그 하단에 GitHub 계정으로 댓글을 달 수 있는 부분이 추가되었고 
작성한 댓글은 댓글 Repository에 Issue로 잘 생성되고 있다!


