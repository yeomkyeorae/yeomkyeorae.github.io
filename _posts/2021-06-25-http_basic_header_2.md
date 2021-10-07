---
title: "HTTP 기본 지식(8) HTTP 헤더2 - 캐시와 조건부 요청"
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



## 1. 캐시 기본 동작



### 캐시가 없을 때

데이터가 변경되지 않았음에도 불구하고 요청할 때 마다 데이터를 매번 다운로드 받아야 한다. 그러므로 네트워크는 느려지고, 브라우저 로딩 속도도 느려지게 된다.



### 캐시 적용

처음 요청을 하면 아래와 같이 `cache-control` header에 cache가 유효한 시간을 명시할 수 있다. 응답받은 데이터를 브라우저에서 캐시로 저장한다. 이후 요청할 때는 캐시에 있는지 확인하고 있으면 그 캐시를 사용하고, 없으면 다시 서버에 요청한다.

```http
HTTP/1.1 200 OK
Content-Type: image/jpeg
cache-control: max-age=60
Content-Length: 12345

ajfksdjfiewji234k2j39f32jlf9...
```

당연하게도 60초가 지나면 저장된 캐시는 사용할 수 없으므로, 서버에 다시 요청해 데이터를 조회한다. 서버에 조회하는 데이터가 변하지 않았음에도 캐시 유효 시간이 만료되었다고 다시 서버에 요청하는 것이 효율적일까? 이에 대한 방안을 아래에서 알아보자.



## 2. 검증 헤더와 조건부 요청1



캐시 유효 시간 초과 후(60초 경과) 서버에 다시 데이터를 요청하면 두 가지 경우 중 하나이다.

1. 서버에서 기존 데이터를 변경함(노란색 이미지 -> 초록색 이미지)
2. 서버에서 기존 데이터를 변경하지 않음(노란색 이미지 유지)

2번의 경우에 브라우저 캐시 데이터와 서버의 데이터가 변경되지 않았다는 사실을 입증하면 캐시의 데이터를 재사용할 수 있다. 여기에서 사용하는 것이 검증 헤더이다. 아래와 같이 `Last-Modified` header를 추가해 마지막에 수정된 날짜를 응답으로 내려준다.

```http
HTTP/1.1 200 OK
Content-Type: image/jpeg
cache-control: max-age=60
Last-Modified: 2021년 6월 25일 22:55:00
Content-Length: 12345

ajfksdjfiewji234k2j39f32jlf9...
```

이 경우에 브라우저 캐시는 캐시 데이터 뿐만 아니라 데이터의 `최종 수정일`에 대한 정보를 같이 저장한다. 캐시가 만료되고, 클라이언트는 서버에 다시 데이터를 요청하게 되는데, 이때 브라우저 캐시에 저장된 `최종 수정일`을 기반으로 `if-modified-since` header를 추가해 요청한다.

```http
GET /star.jpg
if-modified-since: 2021년 6월 25일 22:55:00
```

서버는 `if-modified-since` header를 활용해 서버에 저장된 데이터의 `최종 수정일`을 비교한다. 동일할 경우, 서버는 응답으로 304 `Not Modified`를 내려주게 된다. 이 경우, 서버는 헤더만 전송하고 메세지 body를 전송하지 않기 때문에 네트워크 부하를 줄일 수 있다.

```http
HTTP/1.1 304 Not Modified
Content-Type: image/jpeg
cache-control: max-age=60
Content-Length: 12345
```

 브라우저 캐시는 저장된 데이터의 캐시 유효 시간을 갱신해 데이터를 재활용할 수 있다.



## 3. 검증 헤더와 조건부 요청2



### Last-Modified, If-Modified-Since의 단점

- 1초 미만 단위로 캐시 조정이 불가능하다
- 날짜 기반의 로직을 사용한다
  - 데이터를 수정해서 날짜가 변경되었지만, 수정 결과가 이전과 같은 경우에 서버에서 날짜 변경돼 데이터가 변경됐다고 인지할 수 있다.
- 서버에서 별도의 캐시 로직을 관리할 수 없다.
  - 스페이스, 주석과 같이 데이터에 큰 영향이 없는 변경이 있을 경우에도 캐시를 유지하고 싶을 수 있다



### ETag(Entity Tag), If-None-Match

`ETag`를 사용하면 캐시용 데이터에 임의의 고유한 버전의 이름을 달 수 있다(예: ETag: "v1.0", ETag: "abcdedfg"). 데이터가 변경되면 이름을 바꿀 수 있다. 단순히 `ETag`가 같으면 캐시를 유지하고, 다르면 서버가 다시 클라이언트로 데이터를 내려준다. `ETag`를 사용하면 캐시 제어 로직을 서버에서 완전히 관리할 수 있다.



클라이언트에서 데이터를 처음 요청할 경우 서버는 클라이언트에게 `ETag` header를 포함해 응답을 내려준다.

```http
HTTP/1.1 200 OK
Content-Type: image/jpeg
cache-control: max-age=60
ETag: "abcdefg"
Content-Length: 12345

ajfksdjfiewji234k2j39f32jlf9...
```

 그러면 브라우저는 브라우저 캐시에 해당 데이터와 `ETag`를 저장한다. 캐시 유효 시간이 경과하고, 클라이언트에서 데이터를 요청하면 요청 시에 `If-None-Match` header에 `ETag` 값을 포함해 서버에 전송하고, 서버는 이를 비교한다.

```http
GET /star.jpg
If-None-Match: "abcdefg"
```

변경되지 않았을 경우에는 `304 Modified` 응답을 주고, 캐시를 재사용할 수 있도록 한다.



## 4. 캐시와 조건부 요청 헤더



### Cache-Control

캐시 지시어

- `Cache-Control: max-age`
  - 캐시 유효 시간, 초 단위
- `Cache-Control: no-cache`
  - 데이터는 캐시해도 되지만, 항상 `origin 서버`에 검증하고 사용
- `Cache-Control: no-store`
  - 민감한 데이터가 있으므로 저장하면 안됨. 메모리에서만 잠깐 사용



### Pragma

캐시 제어(HTTP 1.0 하위 호환으로 현재는 잘 사용하지 않음)



### Expires

캐시 만료일 지정(하위 호환).

- 예 - expires: Sat, 25 Jun 2021 23:50:00 GMT
- 캐시 만료일을 날짜로 지정
- 현재는 Cache-Control: max-age 권장.



## 5. 프록시 캐시



미국에 origin 서버가 있을 때 한국에서 클라이언트로 origin 서버와 통신하면 시간이 오래 걸린다. 따라서 한국에 `프록시 캐시`를 두고, origin 서버에 직접 데이터를 요청하는 것이 아니라 프록시 캐시에 데이터가 존재하는 여부를 파악한다음 빠르게 데이터를 가져올 수 있도록 한다. 위에서 설명한 브라우저 캐시와 작동 방식은 유사하다. 이때 브라우저 캐시를 `private 캐시`, 프록시 캐시를 `public 캐시`라고 한다.



### Cache-Control

- Cache-Control: public
  - 응답이 public 캐시에 저장되어도 됨
- Cache-Control: private
  - 응답이 현재 사용자만을 위한 것. 데이터가 private 캐시에 저장되어야 함(기본 값)
- Cache-Control: s-maxage
  - 프록시 캐시에만 적용되는 max-age 값
- Age: 60 (HTTP 헤더)
  - origin 서버에서 응답하고 프록시 캐시에 머무는 시간(초)



## 6. 캐시 무효화



### 확실한 캐시 무효화 응답

브라우저가 임의로 데이터 또는 페이지를 캐싱하는 경우가 있으므로, 무조건적으로 캐싱을 방지해야 한다면 아래 header를 다 넣어 주어야 한다.

- `Cache-Control: no-cache, no-store, must-revalidate`(캐시 만료 후 최초 조회시 origin 서버에 검증해야 함, origin 서버에 접근할 수 있다면 오류를 발생해야 함. 504 Gateway Timeout)
- `Pragma: no-cache`
  - HTTP 1.0 하위 호완을 위해



### no-cache vs must-revalidate

캐시 만료 후 클라이언트가 서버에 데이터를 요청할 때 프록시 캐시가 네트워크 장애로 origin 서버에 접근할 수 없는 경우에 서버 설정에 따라서 프록시 캐시에 저장된 데이터를 보여줄 수도 있다(오류 보다는 데이터를 보여주는 것이 낫다는 판단하에). 하지만 `must-revalidate`를 설정하면 origin 서버에 접근할 수 없는 경우에 항상 오류가 발생하기 때문에 캐시 데이터를 클라이언트로 내려주는 것을 방지할 수 있다.

