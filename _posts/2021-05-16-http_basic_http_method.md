---
title: "HTTP 기본 지식(4) HTTP 메서드"
excerpt: "HTTP 메서드"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - HTTP
tags:
  - HTTP


---

이 글은 김영한님의 인프런 강의인 [모든 개발자를 위한 HTTP 웹 기본 지식](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard) 강의를 요약한 것입니다.



## 1. HTTP API



HTTP API를 설계할 때는 `리소스(Resource)`를 중심으로 설계한다. 따라서 URI에는 리소스 단위만 포함하고, 리소스에 대한 행위는 `메소드(조회, 등록, 수정, 삭제 등)`로 분리해 URI에 포함시키지 않는다. 리소스만으로 설계가 불가능한 경우 행위의 내용(동사)가 URI가 포함될 수도 있다(`컨트롤 URI`). 리소스는 복수형으로 사용이 권장됨.
- `회원` 목록 조회: /members
- `회원` 단일 조회: /members/{id}
- `회원` 등록: /members
- `회원` 수정: /members/{id}
- `회원` 삭제: /members/{id}



## 2. GET, POST



#### 주요 메서드
- GET: 리소스`(Representation)` 조회
- POST: 요청 데이터 처리, 등록에 사용
- PUT: 리소스를 대체, 해당 리소스가 없을 경우 새로 생성
- PATCH: 리소스의 부분을 변경
- DELETE: 리소스 삭제
- HEAD, OPTIONS, CONNECT, TRACE, etc.

#### GET
리소스를 조회하는 역할. 서버에 전달하고자 하는 데이터는 `query(query parameter, query string)`를 통해서 전달한다. 메세지 body를 전달할 수 있지만 지원하지 않는 곳이 많아 권장하지는 않음.

1. GET 요청 예시
>
> GET /members/1 HTTP/1.1
> Host: localhost:3000

2. GET 응답 예시
>
> HTTP/1.1 200 OK
> Content-Type: application/json
> Content-Length: 31
>
> {
> "username": "my",
> "age": "17"
> }

#### POST
요청 데이터를 처리하는 역할. 메시지 body를 통해 서버로 요청 데이터를 전달한다. 주로 전달된 데이터로 신규 리소스를 등록하거나, 프로세스 처리에 사용한다. 스펙에는 다음과 같이 POST 메서드를 정의한다.`대상 리소스가 리소스의 고유한 의미 체계에 따라 요청에 포함된 표헌을 처리하도록 요청한다.😂` 결론은 POST 메서드는 요청 데이터를 어떻게 처리할지 정해진 것이 없어서 리소스 마다 어떻게 처리할지 스스로 정의해야 한다(JSON 데이터를 활용해 조회하는 경우 GET이 아닌 POST를 사용할 수도 있다).

1. POST 요청 예시
>
> POST /members HTTP/1.1
> Content-Type: application/json
>
> {
> "username": "giraffe",
> "age": "27"
> }

2. 신규 리소스 생성 후 POST 응답 예시
신규 리소스의 식별자를 생성해 응답에 포함시킨다.
>
> HTTP/1.1 201 Created
> Content-Type: application/json
> Content-Length: 31
> `Location: /members/17` 
>
> {
> "username": "giraffe",
> "age": "27"
> }



## 3. PUT, PATCH, DELETE



#### PUT
PUT 요청은 리소스가 있으면 `완전히 대체`하고, 리소스가 없으면 새로 생성한다(덮어쓰기). POST와의 차이점은 PUT은 요청하는 클라이언트가 리소스의 정확한 위치(예: `회원의 id`)를 알고, URI에 그 정보를 포함시킨다는 점이다. 

아래 요청 결과 회원 id 100에 해당하는 리소스가 있는 경우, 아래 메시지 body로 리소스가 완전히 대체된다. 100에 해당하는 리소스가 없는 경우, 리소스를 새로 생성한다.

> PUT /members/100 HTTP/1.1
> Content-Type: application/json
>
> {
> "username": "lion",
> "age": 12
> }

주의할 점은 PUT은 리소스를 완전히 대체하므로 리소스에 누락된 부분이 있을 경우, 누락된 그 자체로 리소스가 변경될 수 있다는 것이다. 예를 들어, 아래 요청에서 메시지 body에 age만 포함돼 있을 경우, 회원 id 100에 해당하는 리소스가 있다면 username은 삭제되고 age만 대체된다. 또는 age 정보만을 갖는 리소스가 새로 생성된다. 따라서 리소스의 부분 변경을 요청할 경우에는 PATCH를 사용한다.

> PUT /members/100 HTTP/1.1
> Content-Type: application/json
>
> {
> "age": 12
> }


#### PATCH
리소스의 부분을 변경한다. 아래 요청 결과 username은 변경 없이 age만 30으로 변경된다.
> PATCH /members/17 HTTP/1.1
> Content-type: application/json
>
> {
> "age": 30
> }

#### DELETE
리소스를 제거한다.
>DELETE /members/17 HTTP/1.1
>Host: localhost:3000



## 4. HTTP 메서드 속성



#### 안전(Safe)
호출해도 리소스를 변경하지 않는 속성. 로그가 쌓여서 장애가 발생한 것에 대해서는 고려하지 않고 리소스만을 고려.

#### 멱등(Idempotent)
호출 횟수에 상관없이 그 결과가 항상 똑같은 속성. f(f(x)) = f(x). 외부 요인으로 리소스가 중간에 변경되서 호출 결과가 달라지는 것은 고려하지 않는다.

- GET: 여러 번 조회해도 조회된 결과는 항상 같으므로 멱등.
- PUT: 결과를 대체하므로, 요청에 대한 결과는 항상 같으므로 멱등.
- DELETE: 같은 요청을 여러 번 해도 해당 리소스는 삭제된 상태이므로 멱등.
- POST: 여러 번 호출할 경우 리소스가 중복해서 생성될 수 있으므로 멱등이 아니다(결제 요청을 두 번하면 두번 결제될 수 있음).

멱등 속성은 `자동 복구 메커니즘`에 활용될 수 있다. 서버가 timeout으로 응답을 주지 못했을 때, 클라이언트가 동일한 요청을 다시 해도 되는지 판단할 수 있는 근거가 된다.

#### 캐시 가능(Cacheable)
응답 결과에 대한 리소스를 캐시해서 사용해도 되는지에 대한 속성. GET, HEAD, POST, PATCH 메서드가 캐시 가능하지만, 캐시 키를 고려하는 부분이 POST, PATCH는 까다로워 실제로 `GET`, `HEAD` 정도만 캐시로 사용한다. 