---
title: "HTTP 기본 지식(2) URI와 웹 브라우저 요청 흐름"
excerpt: "URI, 기본 웹 브라우저 요청 흐름"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - HTTP
tags:
  - HTTP


---

이 글은 김영한님의 인프런 강의인 [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard) 강의를 요약한 것입니다.



## 1. URI(Uniform Resource Identifier)



#### URI, URL, URN
"URI는 위치(Locator), 이름(Name) 또는 둘 다 추가로 분류될 수 있다" 이 문구로 보았을 때 URI는 URL(Locator)과 URN(Name)을 포함하는 개념으로 볼 수 있다. URN은 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화되지 않아 잘 사용하지 않는다. [관련 스펙](https://www.ietf.org/rfc/rfc3986.txt)

URL의 예시)
>
> URL은 리소스가 있는 위치를 지정
> foo://example.com:8042/over/there?name=ferret#nose


URN의 예시)
>
> URN은 리소스에 이름을 부여
> urn:example:animal:ferret:nose


#### URI의 뜻
- Uniform: 리소스를 식별하는 통일된 방식
- Resource: 자원, URI로 식별하는 있는 모든 것
- Identifier: 다른 항목과 구분하는데 필요한 정보

#### URL 분석
`https://www.google.com/search?q=hello&hl=ko` URL이 있다고 해보자. 이 URL은 아래와 같은 구조를 갖는다.

>
>scheme://[userinfo@]host[:port][/path][?query][#fragment]

1. `scheme`: 어떤 방식으로 자원에 접근할 것인가에 대한 약속/규칙(http, https, ftp)
2.`userinfo`: URL에 사용자 정보를 포함해 인증할 때(사용하지 않음)
3. `host`: 호스트명(도메인명 또는 IP 주소를 직접 입력)
4. `port`: 포트(생략 가능, 생략 시 http는 80, https는 443)
5. `path`: 리소스 경로. 계층적 구조로 되어 있다. (/home/file1.jpg, /members)
6. `query`: key=value 형태. ?로 시작하며 &로 추가가 가능하다(?height=180&weight=80). query parameter 또는 query string(모두 문자열 형태로 넘어가므로)으로 불림.
7. `fragment`: html 내부 북마크 등에 사용. 잘 사용하지 않음



## 2. 웹 브라우저 요청 흐름



먼저 `https://www.google.com/search?q=hello&hl=ko`와 같은 URL을 주소 창에 입력하면,

1. 웹 브라우저는 port 정보(https이므로 443)와 호스트명인 `www.google.com`을 DNS에 조회해 IP 주소를 알아낸다.
2. 아래와 같은 HTTP 메시지를 생성한다.
>
> GET /search?q=hello&hl=ko HTTP/1.1
> Host: www.google.com

3. 소켓 라이브러리를 통해 TCP/IP로 HTTP 메시지를 전달한다. (TCP/IP 연결 및 데이터 전달)
4. TCP/IP 계층에서 패킷을 생성하고, 전달받은 HTTP 메시지를 포함해 전송한다.
5. TCP/IP 패킷을 버리고(?) 구글 서버는 HTTP 메시지를 받아 처리하고, HTTP 응답 메시지 생성해 웹 브라우저로 보낸다.
>
> HTTP/1.1 200 OK
> Content-Type: text/html;charset=UTF-8
> Content-Length: 3423
> ```html
> <html>
> <body>...</body>
> </html>

6. 웹 브라우저는 전달받은 HTTP 응답 메시지로 HTML 렌더링을 한다.