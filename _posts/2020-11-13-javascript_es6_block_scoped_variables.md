---
title: "JavaScript ES6+ (2) Block scoped variables"
excerpt: "javascript ES6 블록스코프 변수"
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



 ## 1. let

 `let`은 `var`와 비슷한 개념이지만 변수 범위가 블록 스코프에 한정돼 있고, `TDZ`가 있다. 또한 재할당이 가능하다.
 ```javascript
let num = 10;
num = 20;
console.log(num)	// 20
 ```

### 반복문에서의 함수 실행시 var, let 차이
먼저 var의 경우를 보겠습니다. 아래 예제처럼 반복문으로 `funcs`에 반복문 인자 `i`를 포함하여 함수를 추가하였을 때, 함수를 실행하면 결과는 어떤 값을 출력할까??
```javascript
var funcs = [];
for(var i=0; i < 5; i++) {
  funcs.push(function() {
    console.log(i);
  });
}
funcs.forEach(function(f) {
  f();	// 5, 5, 5, 5, 5
});
```
`0, 1, 2, 3, 4`? 아니면 `4, 4, 4, 4, 4`? 정답은 `5, 5, 5, 5, 5`이다. 반복문이 다 돌고 `i`이 5로 할당되고 나서야 함수가 실행되는데, 그제서야 실행 컨텍스트가 열리고, 변수를 호이스팅하게 된다. 자신에게 없는 변수를 외부에서 찾게 되는데 아직 var로 선언한 `i`가 살아있으므로 5를 출력하게 되는 것이다. 
반복문이 도는 상황에서의 변수 값을 출력하고자 한다면 즉시 실행 함수로 변수를 넘겨 주어 함수를 만드는 방법이 있다. 아래와 같다(클로저와 관련돼 있다).
```javascript
var funcs = [];
for(var i=0; i < 5; i++) {
  funcs.push((function(value) {
    return function() {
      console.log(value);
    }
  })(i))
}
funcs.forEach(function(f) {
  f();	// 0, 1, 2, 3, 4
});
```
이게 무슨 짓인가 싶다. 그냥 `let`을 쓰면 쉽게 해결할 수 있다.

```javascript
var funcs = [];
for(let i=0; i < 5; i++) {
  funcs.push(function() {
    console.log(i);
  });
}
funcs.forEach(function(f) {
  f();	// 0, 1, 2, 3, 4
});
```
반복문에서 `var`가 아닌 `let`을 사용하면 각각의 `i` 마다 블록 스코프가 별도로 생성돼서 블록 스코프 안에 개별적인 `i`가 존재한다고 생각하면 된다(?). 잘 이해가 되지는 않는다.

## 2. const
`const`는 constant(상수)의 약자이다. 변수 선언시 할당이 이루어져야 하며, let과 달리 재할당이 불가능하다. 상수이므로.

### 참조형 데이터의 경우
`const` 변수에 할당한 값 자체를 바꿀 수 없지만 할당한 데이터가 참조형일 경우에는 참조형 데이터 내부의 값을 바꿀 수 있다. 예를 들어,
```javascript
const obj = {
  num1: 10,
  num2: 20
};

obj = 100;	// 재할당 에러
obj.num1 = 50;
```
다음과 같이 `obj` 변수를 `const`에 할당했을 때, `obj = 100`처럼 `obj`에 다른 값을 재할당할 수 없다. 하지만 `obj.num1 =50`처럼 `obj`에 할당한 객체 내부의 값을 바꿀 수 있다. 왜냐하면 `obj`가 바라보는 것은 할당된 객체의 `주소`이고, 할당된 객체 자체는 상수가 아니기 때문이다.

### Object.freeze와 deepcopy
때에 따라서는 `const`에 할당된 참조형 데이터 내부 값을 상수로 활용하고 싶을 수도 있다. 그때 활용할 수 있는 것이 `Object.freeze`이다.
```javascript
const nums = {
  num1: 20,
  num2: 50,
  nums: [1, 2, 3]
};
Object.freeze(nums);

nums.num1 = 100;	// 바뀌지 않음
nums.nums = 500;	// 바뀌지 않음
nums.nums[1] = 200;	// [1, 200, 3] ... 어라 바뀌네?
```
하지만 이것도 한계가 있다. 객체 프로퍼티에 할당된 것이 역시 참조형 데이터일 경우에는 꽁꽁 얼리지 못한다. 한파가 제대로 닥칠 수 없다. 객체를 모두 얼리려면 결국 재귀적으로 프로퍼티가 참조형 데이터일 경우 다시 `Object.freeze`를 실행 반복해야 한다. 이를 deepFreezing이라고 한다. 
얼리기보다 복사를 한다면? `Object.assign`을 활용할 수 있지만 이는 얕은 복사(depth 1)에 해당한다. 깊은 복사(deepcopy)는 `불변 객체(immutable)`와 관련 있으므로 추후에 정리해 보도록 해야겠다.

## 3. for문에서의 주의사항
암기가 필요할 수 있는 부분이다. `for ... in` 문에서 `const`를 사용할 수 있지만, 일반적인 for문에서는 const를 사용할 수 없다. 이 경우에는 `let`을 사용하자.
```javascript
const nums = {
  num1: 10,
  num2: 20
}
for(const num in nums) {	// 에러가 발생하지 않음
  console.log(num)
};	

for(const i=0; i < 10; i++ {	// 에러 발생
  console.log(i)
};
```
전자는 개별 블록 스코프에 `const` 변수를 활용하는 것, 후자는 `const` 변수에 재할당하는 로직이라고 이해하자.

## 4. let, const 공통사항
`var`이 재선언이 가능한 것에 비해 `let, const`는 재선언이 되지 않는다.
```javascript
var a = 10;
var a = 20;

let b = 10;
let b = 20; 	// error

const c = 10;
const c = 20;	// error
```
`let`은 값 자체의 변경이 불가피한 경우에 사용하고, 그 외의 경우는 웬만하면 `const`를 사용하자!

### 전역변수이자 전역객체인 var
기이한 `var`의 행태를 보고 마치도록 하겠습니다.
```javascript
var a = 100;
console.log(window.a);	// 100
delete a;		// false

window.b = 100;
console.log(b)		// 100
console.log(window.b)	// 100
delete b		// true: 전역 객체의 프로퍼티로 접근해 삭제
```
`delete`는 객체의 프로퍼티를 지우라는 함수이다. 따라서 변수에 접근해 삭제할 수 없다.`var`로 선언한 경우, 전역변수이자 전역객체로 선언되기 때문에 삭제되지 않는다. `let`으로 선언한 경우에는 전역 프로퍼티로 선언되지 않기 때문에 window로 접근할 수 없다. 따라서 `delete`로 변수를 삭제할 수 없다.
결론은 그냥 `var`를 쓰지 말라는 것이다!