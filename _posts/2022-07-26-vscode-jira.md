---
layout: post
title: VScode/VScode에 Jira 연동하기
date: 2022-07-26
excerpt: "VScode 플러그인 중 Jira and Bitbucket의 소개와 사용방법"
tags: [vscode, jira, plugin]
comments: true
---

## 개요
회사에서 Jira와 VScode를 사용하고 있는데 이 두 개를 연동해놓으면
IDE에서 티켓들을 생성하고 조회할 수 있다고 해서 한 번 연동해보기로 했다.
그 과정에서 생각보다 Jira Plugin에 대해 잘 정리해놓은 글이 없어서 이번에 내가 연동해본 과정을 글로 정리해보려고 한다.

## 과정
<div style="width:60% !important; margin:0 auto">
<img src="/assets/img/vscode-jira1.png" alt="vscode-jira1.png">
</div>

먼저, VScode Extensions에 Jira를 검색하면 이 정도의 플러그인들이 나오는데
그 중에서 제일 첫 번째에 있는 `Jira and Bitbucket`을 설치한다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/vscode-jira2.png" alt="vscode-jira2.png">
</div>
<br>
<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/vscode-jira3.png" alt="vscode-jira3.png">
</div>

Install을 누르면 이렇게 Bitbucket과 Jira 중 사용할 것들을 선택하라는 화면이 나온다.
나는 Jira만 연동하려고 Bitbucket의 체크는 해제하고 Jira만 체크한 뒤 Next를 눌렀다.

<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/vscode-jira4.png" alt="vscode-jira4.png">
</div>

그러면 이제 Jira Cloud와 Custom Jira 중 선택하라는 화면이 나온다.
나의 경우, 회사에서 Custom Jira를 사용하고 있기 때문에 Custom Jira를 선택했다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/vscode-jira5.png" alt="vscode-jira5.png">
</div>

Custom Jira를 사용하기 위한 URL과 로그인 정보를 입력하라는 창이 나온다.
회사에서 사용하는 Jira의 주소와 계정정보를 입력하고 SAVE SITE를 누른다.

<div style="width:100% !important; margin:0 auto">
<img src="/assets/img/vscode-jira6.png" alt="vscode-jira6.png">
</div>

이렇게 ready to get started 안내가 나오면 이제 Jira가 연동된 것이다!

### Jira 이슈 조회하기

<div style="width:60% !important; margin:0 auto">
<img src="/assets/img/vscode-jira7.png" alt="vscode-jira7.png">
</div>

VScode에 Jira가 연동이 되면 왼쪽 아이콘리스트 제일 하단에 Atlassian 아이콘이 추가된 것을 확인할 수 있다.
아이콘을 눌러 이슈목록을 확인해보자.

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/vscode-jira8.png" alt="vscode-jira8.png">
</div>

Atlassian 아이콘을 눌러 해당 탭으로 이동하면 내가 연동해두었던 Jira에서 아까 입력한 나의 계정에 할당된 티켓들만 볼 수 있다.

<div style="width:110% !important; margin:0 auto">
<img src="/assets/img/vscode-jira9.png" alt="vscode-jira9.png">
</div>

또 이슈를 클릭하면 그 이슈에 대한 상세 정보들을 확인할 수 있고 티켓의 상태(Status), 담당자(Assignee), 작성자(Reporter), 라벨(Labels) 등을 확인하고 수정할 수 있다.
심지어, 이 이슈 조회화면에서 우측 상단의 `Start Work` 버튼을 누르면 그 이슈에 해당하는 작업을 하기 위한 브랜치를 바로 딸 수도 있다.

<div style="width:80% !important; margin:0 auto">
<img src="/assets/img/vscode-jira10.png" alt="vscode-jira10.png">
</div>

`Start Work` 버튼을 눌렀을 때의 화면이다. 브랜치를 세팅하기 위한 다양한 설정들을 직접 선택한 후 브랜치를 딸 수 있다.
브랜치를 따기 위한 기준 브랜치(예시사진에서는 develop), 작업할 브랜치명(따로 수정하지 않으면 기본적으로 이슈번호로 작성하도록 추천해줌) 등을 확인한 뒤 하단의 `START` 버튼을 누르면 로컬 VScode에서 작업할 브랜치가 생성되고, Github remote repository에 자동으로 push 된다.

### Jira 이슈 생성하기
이번엔 Jira 플러그인을 통해서 티켓을 생성하는 방법을 알아보자.

<div style="width:70% !important; margin:0 auto">
<img src="/assets/img/vscode-jira11.png" alt="vscode-jira11.png">
</div>

Atlassian 아이콘을 눌러 나오는 Jira 탭에서 제일 위의 Create issue 버튼을 클릭한다.

<div style="width:90% !important; margin:0 auto">
<img src="/assets/img/vscode-jira12.png" alt="vscode-jira12.png">
</div>

Epic Name, Project, Issue Type, Summary, Description 등을 작성한 뒤 하단의 Submit 버튼을 누르면 Jira에 아까 연동해놓은 계정으로 작성자가 등록된 이슈가 생성된다.