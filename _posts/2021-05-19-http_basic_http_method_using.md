---
title: "HTTP 기본 지식(5) HTTP 메소드 활용"
excerpt: "HTTP 메소드 활용"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - HTTP
tags:
  - HTTP

---

이 글은 김영한님의 인프런 강의인 [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard) 강의를 요약한 것입니다.



## 1. 클라이언트에서 서버로의 데이터 전송



#### 1. 정적 데이터 조회
이미지, 정적 테스트 문서 등을 조회
#### 2. 동적 데이터 조회
주로 검색, 게시판 목록에서 (검색어를 활용한)정렬 필터 사용 시
#### 3. HTML FORM 데이터 전송
HTML은 `GET`, `POST`만 지원한다.
##### POST 전송 - 저장
```html
<form action="/save" method="post">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <button type="submit">전송</button>
</form>
```
위의 html을 활용해 url로 요청하면 브라우저는 자동으로 HTTP 메시지를 아래와 생성한다.
>
>POST /save HTTP/1.1
>
>Host: localhost:3000
>
>Content-Type: application/x-www-form-urlencoded
>
>username=yeom&age=17

##### POST 전송 - multipart/form-data
파일을 전송할 때는 binary 형식으로 데이터를 전송하는 `multipart/form-data`을 사용한다. `multipart/form-data`는 다른 종류의 여러 파일과 폼의 내용을 함께 전송이 가능하다.

```html
<form action="/save" method="post" enctype="multipart/form-data">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <input type="file" name="file1" />
  <button type="submit">전송</button>
</form>
```
위에서는 `form` 태그에 `enctype` 속성을 추가해 `multipart/form-data`로 데이터를 보낼 것임을 명시하였다. 아래에서 `boundary`는 파싱하는 경계를 의미한다.
> POST /save HTTP/1.1
> 
> Host: localhost:3000
> 
> Content-Type: multipart/form-data; boundary=----XXX
>
> ----XXX
> 
> Content-Disposition: form-data; name="username"
>
> yeom
> 
> ----XXX
> 
> Content-Disposition: form-data; name="age"
>
> 17
> 
> ----XXX
> 
> Content-Disposition: form-data; name="file1"; filename="http.png"
> 
> Content-Type: image/png
>
> sajldfkjaiwjasdjkdfjlkjlasjdkfjalsdf
> 
> ----XXX

#### 4. HTML API 데이터 전송
- 서버에서 서버로
- 앱 클라이언트
- 웹 클라이언트(Form 전송 대신 자바스크립트를 통한 통신에 사용)
- Content-Type: appliation/json을 주로 사용한다.
- GET: query parameter로 데이터 전송.
- POST, PUT, PATCH: message body를 통해 데이터 전송.



## 2. HTTP API 설계 예시



#### 회원 관리 시스템
API 설계 - POST 기반 등록
- 회원 목록: /members -> `GET`
- 회원 등록: /members -> `POST`
- 회원 조회: /members/{id} -> `GET`
- 회원 수정: /members/{id} -> `PATCH`, `PUT`, `POST`
- 회원 삭제: /members/{id} -> `DELETE`

POST 기반 등록의 경우 클라이언트는 등록될 리소스의 정확한 URI를 알지 못한다. 그래서 서버가 새로이 등록된 리소스의 URI를 생성해 응답해 준다.
>
> HTTP/1.1 201 Created
> 
> Location: /members/100

컬렉션(Collection)?🧐
- 서버가 관리하는 리소스 디렉토리로서 서버가 리소스의 URI를 생성하고 관리.
- 위의 예제에서 컬렉션은 `/members`이다.

#### 파일 관리 시스템
API 설계 - PUT 기반 등록
- 파일 목록: /files -> `GET`
- 파일 조회: /files/{filename} -> `GET`
- 파일 등록: /files/{filename} -> `PUT`
- 파일 삭제: /files/{filename} -> `DELETE`
- 파일 대량 등록: /files -> `POST`

PUT 기반 등록의 경우 클라이언트가 리소스의 URI를 알고 있어야 한다. (예: `PUT /files/star.jpg`)

스토어(Store)?🧐
- 클라이언트가 관리하는 리소스 저장로서 클라이언트가 리소스의 URI를 관리.
- 위의 예제에서 스토어는 `/files`

#### HTML FORM 사용
HTML FORM은 GET, POST만 지원. 따라서 `컨트롤 URI`를 사용해야할 경우가 있음.

컨트럴 URI?🧐
- GET, POST만 지원하므로 제약이 존재.
- 제약을 해결하기 위해 동사로 된 리소스의 경로를 사용한다.
- POST의 /new, /edit, /delete가 컨트롤 URI(동사 형태).
- HTTP 메서드로 해결하기 애매한 경우 사용(HTTP API 포함)

#### 참고하기 좋은 URI 설계 개념
[REST API naming conventions](https://restfulapi.net/resource-naming/)
- 문서(document, 단일 개념으로서 파일 1개, 객체 인스턴스, 데이터베이스 1row 등)
- 컬렉션(collection)
- 스토어(store)
- 컨트롤러(controller), 컨트롤 URI
