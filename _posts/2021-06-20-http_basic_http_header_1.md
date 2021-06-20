---
title: "HTTP 기본 지식(7) HTTP 헤더1"
excerpt: "HTTP Header"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - HTTP
tags:
  - HTTP


---

이 글은 김영한님의 인프런 강의인 [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard) 강의를 요약한 것입니다.



## 1. HTTP 헤더 개요



- header-field의 구성: field-name ":" OWS field-value OWS (OWS: 띄어쓰기 허용)

  - 예: Host: www.google.com (대소문자 구분하지 않음)

- 용도: HTTP 전송에 필요한 모든 부가 정보

  - 예: 메시지 body의 내용, 메시지 body의 크기, 압축, 인증, 클라이언트 정보, 캐시 정보 등

- 필요시 임의 헤더 추가가 가능하다

- 과거의 Header 구분 방식 - RFC2616(1999년)

- 현재 HTTP 표준 - **RFC7230~7235**(2014년)

  ```http
  HTTP/1.1 200 OK
  Content-Type: text/html;charset=UTF-8
  Content-Length: 1212
  
  <html>
  	<body>...</body>
  </html?
  ```

  - 위에서 Content-Type이 있는 부분이 표현 헤더. 아래 html 부분은 메시지 body 예시.
  - 메시지 body(payload)를 통해 표현(representation) 데이터를 전달한다.
  - 표현 데이터는 요청 또는 응답에서 실제로 전달하는 데이터
  - 표현 헤더는 표현 데이터를 해석할 정보를 제공한다.

  

## 2. 표현(Representation) 헤더



### 표헌 헤더

표현 헤더는 전송, 응답 둘 모두에 사용된다.

- **Content-Type**: 표현 데이터의 형식
  - 미디어 타입, 문자 인코딩
  - 예) text/html; charset=utf-8, application/json, image/png
- **Content-Encoding**: 표현 데이터의 압축 방식
  - 표현 데이터를 압축하기 위해 사용
  - 데이터를 전달하는 측에서 압축 후 인코딩 헤더를 추가
  - 데이터를 받는 측에서는 인코딩 헤더 정보를 기반으로 압축을 해제한다
  - 예) gzip, deflate, identity(압축하지 않음)
- **Content-Language**: 표현 데이터의 자연 언어
  - 표현 데이터의 언어를 표현
  - 예) ko, en, en-US
- **Content-Length**: 표현 데이터의 길이
  - Byte 단위
  - Transfer-Encoding을 사용하면 Content-Length를 사용하면 안된다.



## 3. 컨텐츠 협상(Contents Negotiation)

클라이언트가 선호나는 표현 형식을 요청하는 것. 요청 시에만 사용한다.

예를 들어 한국어 브라우저를 사용하는 경우, 다중 언어를 지원하는 사이트에 접속했을 때, 클라이언트에서 컨텐츠 협상 관련 헤더(Accept-Langauge: ko-KR)를 추가하여 요청하면, 서버에서 지원하는 기본 언어는 영어지만 이 헤더를 보고 한국어 관련 페이지를 내려줄 수 있다.

- **Accept**: 클라이언트가 선호하는 미디어 타입 전달
- **Accept-Charset**: 클라이언트가 선호하는 문자 인코딩
- **Accept-Encoding**: 클라이언트가 선호하는 압축 인코딩
- **Accept-Language**: 클라이언트가 선호하는 언어



### 협상과 우선순위1

컨텐츠 협상에서는 선호하는 표현 형식에 대한 우선순위를 지정할 수 있으며, **Quality Values(q)** 값을 사용한다. 0~1 사이의 값을 가지며 클수록 높은 우선순위를 갖는다. 생략할 경우 1의 값을 갖는다. **예) Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7**



### 협상과 우선순위2

구체적으로 작성한 것이 우선한다.

- Accept: text/*, text/plain, text/plain;format=flowed
- text/plain;format=flowed, text/plain, text/* 순으로 더 우선한다.



## 4. 전송 방식

1. **단순 전송**

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/html;charset=UTF-8
   Content-Lenght: 3434
   
   <html>
   	<body>...</body>
   </html>
   ```

   

2. **압축 전송**(Content-Encoding 사용)

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/html;charset=UTF-8
   Content-Encoding: gzip
   Content-Lenght: 642
   
   fkljefkj34j29dflk3jkfwfkasjdlfk
   ```

   

3. **분할 전송**(Tranfer-Encoding 사용): chunk로 쪼개서 전송. 쪼개진 응답이 오는대로 표현할 수 있는 장점이 있음. 분할해서 오기 때문에 length를 파악하기 어려워 Content-Length를 사용하지 않음. 

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/plain
   Transfer-Encoding: chunked
   
   5
   Hello
   5
   World
   0
   \r\n
   ```

   

4. **범위 전송**(Range, Content-Range): 데이터의 정해진 범위를 요청해서 받을 수 있다. 중간에 요청이 끊겼을 때 어디서부터 데이터를 받아야 할지에 대한 범위를 알 수 있을 경우 유용하다.

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/plain
   Content-Range: bytes 1001-2000 / 2000
   
   asdfkljkewjfi32529fj93fjsd9f3f
   ```

   

## 5. 일반 정보

1. **From**: 유저 에이전트의 이메일 정보
   - 잘 사용하지 않음.
   - 요청에서 사용
2. **Referer**: 이전 웹 페이지 주소, refererr의 오타
   - 현재 요청된 페이지의 이전 웹페이지의 주소(어디에서 검색해서 들어왔나)
   - 유입 경로 분석에 사용한다
   - 요청에서 사용
3. **User-Agent**: 유저 에이전트 어플리케이션의 정보
   - user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36
   - 사용자 통계 정보 파악
   - 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
   - 요청에서 사용
4. **Server**: 요청을 처리하는 origin 서버의 소프트웨어 정보
   - Server: Apache/2.2.22 (Debian)
   - Server: nginx
   - 응답에서 사용
5. **Date**: 메시지가 발생한 날짜 및 시간
   - Date: Sun, 20 Jun 2021 02:44:31 GMT
   - 응답에서 사용



## 6. 특별 정보

1. **Host**: 요청한 호스트의 정보(도메인)
   - 하나의 서버가 여러 도메인을 처리해야 할 때
   - 하나의 IP 주소가 여러 도메인에 적용돼 있을 때
   - 필수 값!
     - **가상 호스트**로 하나의 서버에서 여러 도메인을 한번에 처리해 여러 개의 어플리케이션을 구동할 수 있다.
     - 200.200.200.2 IP를 갖는 서버에서 `aaa.com`, `bbb.com`, `ccc.com` 에 해당하는 어플리케이션을 구동하는 경우에
     - HOST가 없을 경우 클라이언트는 IP로만 통신을 해야하는데 어느 어플리케이션에 요청해야 할지 모른다.
   - 요청에서 사용
2. **Location**: 페이지 리다이렉션
   - 웹 브라우저는 300대 응답 결과에 따라 Location 헤더가 있으면 Location 위치로 자동 이동한다.
3. **Allow**: 허용 가능한 HTTP 메서드
   - 405 (Method Not Allowed)에서 응답에 포함해야 한다.
   - Allow: GET, HEAD, PUT

4. **Retry-After**: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
   - 503 (Service Unavailable): 서비스가 언제까지 이용 불가인지 알려 줄 수 있다
   - Retry-After: Sun, 20 Jun 2021 02:44:31 GMT (날짜 표기)
   - Retry-After: 3600 (초 단위 표시)
5. **Authorization**: 클라이언트 인증 정보를 서버에 전달
   - Authorization: Basic xxxxxxx
6. **WWW-Authenticate**: 리소스 접근시 필요한 인증 방법을 정의
   - 401 Unauthorized 응답과 함께 사용
   - WWW-Authenticate: Newauth realm="apps", type=1, title="Login to apps", Basic realm="simple"



## 7. 쿠키

HTTP는 Stateless한 프로토콜이므로, 클라이언트와 서버는 서로 상태를 유지하지 않는다. 따라서 클라이언트에서 로그인을 했을지라도 쿠키와 같은 수단을 사용하지 않으면 서버는 현재 클라이언트가 로그인 상태인지 알 수 없다.



### 쿠키 Header 종류

- Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
  - 예) set-cookie: **sessionId=abcd1234**; **expires**=Sun, 20 Jun 2021 00:00:00 GMT; **path**=/; **domain**=.google.com; **Secure**
- Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달



### 쿠키의 특성

1. 사용 목적
   - 사용자 로그인 세션 관리(세션 ID 저장하고 서버에 저장한 것과 일치하는지를 확인해 인증)
   - 광고 정보 트레킹
2. 쿠키 정보는 항상 서버에 전송된다.
   - 따라서 네트워크 트래픽을 추가적으로 발생시킴
   - 최소한의 정보만 사용하고(세션 ID, 인증 토큰), 보안에 민감한 데이터는 저장하지 말아야 한다.
   - 서버에 전송하지 않고, 웹 브라우저에 데이터를 저장하고자 한다면 웹 스토리지(`localStorage`, `sessionStorage`)를 사용할 수 있다.



### 쿠키 사용 예시

1. 클라이언트에서 서버에 요청

   ```http
   POST /login HTTP/1.1
   user=염겨레
   ```

2. 서버에서 응답

   ```http
   HTTP/1.1 200 OK
   Set-Cookie: user=염겨레
   
   염겨레님이 로그인했습니다
   ```

3. 브라우저는 서버에서 응답으로 온 쿠키를 쿠키 저장소에 저장(user=염겨레)

4. 로그인 이후 클라이언트가 welcome 페이지에 접근. 브라우저는 쿠키 저장소를 무조건 참조해 서버에 전달. 쿠키는 모든 요청에 자동으로 포함된다.

   ```http
   GET /welcome HTTP/1.1
   Cookie: user=염겨레
   ```



### 쿠키 생명 주기

expires, max-age

- Set-Cookie: **expires**=Sun, 20 Jun 2021 00:00:00 GMT
  - 만료 시간이 되면 쿠키가 삭제
- Set-Cookie: **max-age**=3600
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지 유지된다.
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지



### 쿠키 도메인

명시한 도메인의 경우에만 쿠키를 저장하고 전송할 수 있도록 함

- 명시한 경우: 명시한 문서 기준 도메인과 서브 도메인도 포함
- 예) domain=example.org의 경우, example.org와 dev.exmple.org도 쿠키에 접근
- 생략한 경우: 현재 문서 기준 도메인만 적용.
  - example.org에서 쿠키를 생성해 domain 지정을 생략했다면, dev.example.org에서는 쿠키에 접근할 수 없다



### 쿠키 Path

경로를 포함한 하위 경로 페이지만 쿠키에 접근할 수 있다. 일반적으로 path=/ 루트로 지정한다.

- 예) path=/home
- /home 접근 가능
- /home/hello 접근 가능
- /hello 접근 불가



### 쿠키 보안

Secure, HttpOnly, SameSite

- Secure
  - https의 경우에만 쿠키를 전송한다(http X)
- HttpOnly
  - XSS 공격 방지
  - 자바스크립트에서 접근 불가(document.cookie)
  - HTTP 전송에만 쿠키를 사용
- SameSite
  - XSRF 공격 방지
  - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키를 전송한다

