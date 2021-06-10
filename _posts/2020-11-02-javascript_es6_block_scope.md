---
title: "JavaScript ES6+ (1) Block scope"
excerpt: "javascript ES6 블록스코프"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - javascript
tags:
  - javascript
  - ES6
---

이 글은 정재남 개발자님의 인프런 강의인 <a href="https://www.inflearn.com/course/ecmascript-6-flow" target="_blank">JavaScript ES6+ 제대로 알아보기 초급편</a>을 정리한 내용입니다.



## 1. 블록 스코프(block scope)

- 함수 스코프: 함수에 의해서 생기는 변수의 유효 범위. 기존 `var` 해당.
- 블록 스코프: 블록 `{}`에 의해서 생기는 변수의 유효 범위. `let`, `const` 해당.

아래 코드를 보면 `var` 변수는 `함수 스코프`를 가지므로 함수 내부에 선언한 `num` 변수를 함수 바깥에서 접근할 수 없음을 알 수 있다.
```javascript
(function() {
  var num = 100;
  (function() {
    var num = 200;
	console.log(num);	// 200
  })();
  console.log(num);		// 100
})();
console.log(num);		// num is not defined
```
`블록 스코프`를 갖는 `let`으로 대체해 위 코드와 동일하게 작성한 코드는 아래와 같다.
```javascript
{
  let num = 100;
  {
    let num = 200;
    console.log(num);	// 200
  }
  console.log(num);	// 100
}
console.log(num);	// num is not defined
```

## 2. TDZ(Temporal Dead Zone)
`TDZ`는 ECMAscript 정식 명칭은 아니지만 번역하면 임시사각지대라고 한다. DMZ 같은 느낌인가? 아래 코드를 먼저 보자.
```javascript
if(true) {
  let num = 100;
  if(true) {
    console.log(num);	// 주목!!!!!
    const num = 200;
  }
  console.log(num);
}
console.log(num);        
```
주목이라고 주석 처리된 곳의 num의 출력은 어떻게 나올까?

`가정 1) 호이스팅이 된다면?` 바로 아래 const로 선언된 num 변수가 호이스팅되고 reference 에러가 발생한다. 
`가정 2: 호이스팅이 되지 않는다면?` 바깥에서 num에 해당하는 변수를 찾아 출력해 100을 출력한다.

정답은 호이스팅이 적용돼 reference 에러가 발생하는 것이다. 변수 `let`이나 `const`에 대해서 변수가 선언된 위치에 오기 전에 변수를 호출할 수 없는 지역을 `TDZ`라고 한다. 기존 `var`는 호이스팅으로 변수를 끌어올리고 `undefined`를 할당하는 반면, `let, const`는 호이스팅으로 변수를 끌어올리고 할당하는 과정은 빠져 reference 에러가 발생하는 것이다.


## 3. this의 이해
예시를 먼저 보자.
```javascript
let value = 0;
let obj = {
  value: 10,
  setValue: function() {
    this.value = 20;		// this: obj
    (function() {
      this.value = 50;		// this: window or global
    })();
  }
};

obj.setValue();

console.log(value);		// 50
console.log(obj.value);		// 20
```

`obj` 내부에 setValue라는 이름의 함수를 정의하고, `obj.setValue()`로 메소드로 호출했다. 이때 함수 내부에 첫번째 `this`는 `obj`를 가리킨다. 그 이유는 메소드로 호출했을 때의 바로 앞의 객체를 가리키기 때문이다. 하지만 두번째 `this`는 window를 가리킨다. 함수로 바로 실행했을 때는 전역 객체를 가리키기 때문이다.

첫번째 `this`와 두번째 `this`를 같게 하기 위해서는 다음과 같은 방법들이 있다.
1) this를 변수에 할당하기
```javascript
setValue: function() {
  this.value = 20;
  let self = this;
  (function() {
    self.value = 50;
  })();
}
```
2) call이나 bind로 변수 할당하기
```javascript
setValue: function() {
  this.value = 20;
  (function() {
    this.value = 50;
  }).call(this);
}
```
3) 블록 스코프로 감싸기
`this`는 블록 스코프에 영향을 받지 않으므로!
```javascript
setValue: function() {
  this.value = 20;
  {
    this.value = 50;
  }
}
```