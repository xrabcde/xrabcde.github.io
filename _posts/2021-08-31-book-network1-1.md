---
layout: post
title: Book/성공과 실패를 결정하는 1% 네트워크 원리/1장 - 1
date: 2021-08-31
excerpt: "성공과 실패를 결정하는 1% 네트워크 원리 - 1장 웹 브라우저가 메시지를 만든다 #1"
tags: [csstudy, network, book]
comments: true
---

### 1. URL 입력
- 모든 URL의 맨 앞에 (`http:`, `ftp:`, `file:`, `mailto:`)와 같은 액세스 방법을 나타내는 문자열이 붙음
- http : 액세스 대상이 웹 서버, HTTP 프로토콜 사용
- ftp : 엑세스 대상이 FTP 서버, FTP 프로토콜 사용

### 2. URL 해독

<div style="width:50% !important; margin:0 auto">
<img src="/assets/img/network1-1.png" alt="network1-1.png">
</div>

- URL의 요소 (ex. http://www.lab.cyber.co.kr/dir1/file1.html)

  `http:` + `//` + `웹 서버명` + `/` + `디렉토리명` + `/` + ... + `파일명`

### 3. 파일명 생략한 경우
- http://www.lab.cyber.co.kr/dir**/** : 끝에 디렉토리명 다음 파일명을 생략하고 `/` 로 끝난 경우, 미;리 서버측에 설정해둔 기본 파일명으로 엑세스 (index.html, default.htm)
- http://www.lab.cyber.co.kr**/** : 끝에 `/` 가 있으므로 `/` 라는 디렉토리의 기본 파일에 엑세스
- http://www.lab.cyber.co.kr : 끝에 `/` 까지 모두 생략된 경우, 루트 디렉토리 아래의 기본 파일에 엑세스
- http://www.lab.cyber.co.kr**/whatisthis** : 끝에 `/` 가 없으므로 파일명으로 보는 것이 맞지만 이런 경우 웹서버에 해당 파일이 있으면 파일명으로, 해당 디렉토리가 있으면 디렉토리로 간주

### 4. HTTP의 기본 개념
- HTTP 프로토콜 : 클라이언트와 서버가 주고받는 메시지의 내용이나 순서를 정한 것
- URI : 리퀘스트 메시지 안의 `무엇을` 에 해당, `/index.html`
- 메서드 : 리퀘스트 메시지 안의 `어떻게` 에 해당, `GET`, `POST`
    - **GET** : 웹 서버에 엑세스하여 페이지의 **데이터를 읽을 때** 사용
    - **POST** : **폼에 데이터를 사용**해서 웹 서버에 송신하는 경우 사용

    <div style="width:50% !important; margin:0 auto">
    <img src="/assets/img/network1-2.png" alt="network1-2.png">
    </div>

- Status code : 실행 결과를 나타내는 코드, `404 Not Found`

### 5. HTTP Request
- Request Line : 리퀘스트 메시지의 첫 번째 행 `GET /index.html HTTP/1.1` (메서드, URI, HTTP버전)
- Message Header : 리퀘스트 메시지의 두 번째 행부터 이어지는 내용(날짜, 데이터의 종류, 언어, 압축 형식, 버전, 데이터 유효기간 등 다수의 항목 포함)
- 메시지 본문 : 메시지 헤더 뒤 공백 행 다음 이어지는 내용, **POST일 때만 포함** (송신할 데이터)

### 6. HTTP Response

<div style="width:50% !important; margin:0 auto">
<img src="/assets/img/network1-3.png" alt="network1-3.png">
</div>

- Status Line : 실행 결과를 나타내는 Status code와 Status Message 포함
- Message Header

> 리퀘스트 메시지에 쓰는 URI는 하나뿐으로, 복수의 파일을 읽어올 때는 웹 서버에 별도의 리퀘스트 메시지를 보낸다.
