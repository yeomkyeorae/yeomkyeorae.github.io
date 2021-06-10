---
title: "정규표현식(regular expression) 정리(1)"
excerpt: "정규표현식 정리(1)"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - regular expression
tags:
  - re
---

이 글은 [손에 잡히는 10분 정규 표현식](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=196607460)에 나오는 내용을 정리한 것입니다. 



# 1. 문자 하나 찾기

## (1) 문자 그대로 찾기

일치하는 문자열을 찾고자 할 때는 단순히 그 문자열을 정규표현식에 입력하면 된다.
```javascript
const arr = ['Hello', 'Gracias', '안녕'];
const re = new RegExp(/Hello/);
const filtered = arr.filter(el => re.test(el));

console.log(filtered);
// 결과
// ['Hello']
```

## (2) 모든 문자 찾기
정규표현식에서 `마침표(.)`는 아무 문자 하나와 일치한다. 마침표 자리에 아무 문자가 와도 일치한다고 인식된다.
```javascript
const arr = ['apple', 'app.js', 'index.html'];
const re = new RegExp(/app./);
const filtered = arr.filter(el => re.test(el));

console.log(filtered);
// 결과
// ['apple', 'app.js']
```

## (3) 특수 문자 찾기
`마침표(.)`는 아무 문자 하나를 의미하는데 그렇다면 마침표 그 자체만을 표시하려면 어떻게 해야할까? 이때는 역슬래시(\\) 문자를 앞에 붙여 `\.`로 표현하면 된다. 여기서 역슬래시는 `메타 문자`이다. 메타 문자란 문자 그대로 사용되지 않고 특수한 의미를 지니는 문자를 말한다.
```javascript
const arr = ['apple', 'app.js', 'index.html'];
const re = new RegExp(/app\./);
const filtered = arr.filter(el => re.test(el));

console.log(filtered);
// 결과
// ['app.js']
```
메타 문자는 추후에 더 자세히 소개될 것이다.

# 2. 문자 집합으로 찾기
## (1) 여러 문자 중 하나와 일치시키기
메타 문자인 대괄호([])를 사용해서 문자 집합을 표현할 수 있다. 검색하는 문자열의 영역과 대괄호 안에 포함된 문자 집합들 중 문자 하나만 일치하면 정규표현식에 의해 검색될 수 있다. 예를 들어 보자.
```javascript
const arr = ['abc', 'bbc', 'cbc'];
const re = new RegExp(/[ab]bc/);
const filtered = arr.filter(el => re.test(el));

console.log(filtered);
// 결과
// ['abc', 'bbc']
```
위에서 정규표현식을 `/[ab]bc/`라고 정의했다. 대괄호가 정의된 위치는 문자열의 첫번째 인덱스에 해당하는 문자를 가리키고, 그 위치에 `a` 또는 `b`가 오면 test에 의해 참으로 인식된다.

## (2) 문자 집합 범위 사용하기
문자 집합을 나타내는 대괄호 표기 내부에는 하이픈(-)을 사용하여 문자의 범위를 나타낼 수 있다. 대표적인 것으로 영문 대소문자, 숫자가 있다. 참고로 하이픈은 대괄호 내부에서만 메타 문자로서 활용된다.
- [A-Z]: 영어 대문자 A에서부터 Z까지
- [a-z]: 영어 소문자 a에서부터 z까지
- [0-9]: 숫자 0에서부터 9까지

```javascript
const arr = ['computer', 'comedy', 'rome', 'bomb'];
const re = new RegExp(/[a-c]om/);
const filtered = arr.filter(el => re.test(el));

console.log(filtered);
// 결과
// ['computer', 'comedy', 'bomb']
```
위에서 정규표현식을`/[a-c]om`라고 정의했다. 대괄호가 위치한 인덱스의 문자가 `a-c` 즉, a 또는 b 또는 c가 오면 test에 의해 참이 된다.

## (3) 제외하고 찾기
정규표현식에는 정해진 문자를 제외할 수도 있다. `캐럿(^)` 문자를 사용하면 해당 문자를 제외하고 문자열을 검색할 수 있다. 예를 들어 이전 예제에서 정규표현식 에서 대괄호 부분에 캐럿(^)을 붙이면 `/[^a-c]om/`이 되는데, `a-c`을 제외한 다른 전체 문자를 검색한다. 그래서 그 결과를 실행하면 `['rome']`만 검출이 된다.