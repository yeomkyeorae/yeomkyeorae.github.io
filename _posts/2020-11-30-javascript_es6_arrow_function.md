---
title: "JavaScript ES6+ (6) Arrow function"
excerpt: "javascript ES6 화살표 함수"
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



## 1. Arrow function

### (1) 기본 사용 예제
Arrow function을 사용하면 `=>` 표기를 사용해 함수를 축약할 수 있습니다.
```javascript
let a = function() {
  return new Date();
};

let b = function(num) {
  return num * num;
};

let c = function(num1, num2) {
  console.log(num1, num2);
};

let d = function(x) {
  return {
    x: x
  };
};

let e = function(a) {
  return function(b) {
    return a + b;
  };
};
```
위의 함수들을 Arrow function을 사용하면 아래와 같이 축약할 수 있습니다.
```javascript
let a = () => new Date();

let b = num => num * num;

let c = (num1, num2) => {
  console.log(num1, num2);
};

let d = x => ({x});

let e = a => b => a + b;
// 사용: const num = e(1)(2);
```
내용을 정리하면 아래와 같습니다.
> 
- `(매개변수) => {본문}`와 같은 형식으로 작성한다.
- 매개변수가 하나인 경우`()` 괄호를 생략 가능하다.
- 매개변수가 없을 경우에는 `()` 괄호가 필수이다. `_`나 `$`를 사용해 매개변수를 사용하지 않을 것임을 명시적으로 보여줄 수 있지만, 되도록 `()`를 사용하는 것이 좋다.
- 본문이 return만 할 경우 `{}` 괄호와 return 문이 생략 가능하다.
- return 하는 것이 객체`{}`일 경우에는 괄호를 사용해야 한다.


### (2) 실행 컨텍스트 생성시의 this 바인딩을 하지 않는 arrow functin
예제를 먼저 보겠습니다.
```javascript
const obj = {
  a() {
    console.log(this);		// {a: f}
    
    const b = function() {	
      console.log(this);	// window
    };
    
    b();
  }
};
obj.a();
```
`obj`의 `a 메소드` 내부의 `b 함수`는 `this`를 바인딩해서 전역객체인 `window`를 할당하였습니다. 그렇다면 `b 함수`를 arrow function으로 선언하면 어떻게 될까요? 
그 경우에는 아래와 같이 `this`를 바인딩 하지 않고 외부 스코프에서 `this`를 찾아 `a 메소드` 내부의 첫 번째 줄의 `this`와 같은 결과를 도출하게 됩니다(블록 스코프에서 처럼). 이 경우처럼 arrow function을 사용하게 되면 함수 내부에서 `this`를 사용하기 위해 `this`를 인자로 넘겨주거나 명시적으로 `call 메소드`를 사용하지 않아도 됩니다.

```javascript
const obj = {
  a() {
    console.log(this);		// {a: f}
    
    const b = () => {	
      console.log(this);	// {a: f}
    };
    
    b();
  }
};
obj.a();
```
arrow function은 함수 스코프인지 블록 스코프인지 확인하기 위해서 arrow function 내부에서 `var` 변수를 선언하고 외부에서 접근해 볼 수 있습니다. 함수 스코프라면 console.log로 출력했을 때 에러가 날 것이고, 블록 스코프일 경우 `var`는 외부에서 선언한 것처럼 작동하기 때문에 정상적으로 출력이 될 것입니다.
```javascript
const a = () => {
  var b = 100;
};
console.log(b);		// reference error
```
하지만 console.log로 출력했을 때 `reference error`가 발생하게 되고, 위의 사례들을 종합했을 때 arrow function은 함수 스코프를 생성하지만 실행 컨텍스트시 this를 바인딩하지 않음을 알 수 있습니다.

### (3) 활용 예제
```javascript
var total = 0;
const obj = {
  nums: [70, 80, 90, 100],
  getNums: function() {
    this.total = 0;	// this: obj
    this.nums.forEach(function(v) {	// => arrow function으로
      this.total += v;	// this: window
    });
  }
};
obj.getNums();
console.log(total);	// 340
console.log(obj.total);	// 0
```
위 예제에서는 `forEach` 내부에서 사용한 `this`가 obj일 것이라고 예상했지만 그렇게 하기 위해서는 `this` 인자를 따로 넘겨주어야 한다. 하지만 이를 쉽게 해결하는 방법은 `forEach` 내부 함수를 arrow function으로 활용하면 `this`를 바인딩하지 않아 내부 `this`를 외부스코프에서 찾아 `obj`로 인식하게 된다.

### (4) 명시적으로 this를 바인딩할 수 없음.
`this`를 바인딩하는 `call`과 같은 메소드로 `this`를 바꿀 수 없지만, 메소드 자체를 사용할 수 없는 것은 아니다. 아래 예제와 같다.
```javascript
const sum = (...arg) => {
  console.log(this);	// window
  return arg.reduce((p, c) => p + c);
}

sum.call({}, 1, 2, 3, 4, 5);	// 15
```
위에서 `this`를 `{}`로 바인딩하고자 했지만 출력한 `this`는 `window`가 도출되었다. 하지만 인자로 준 `1, 2, 3, 4, 5`는 정상적으로 넘어가 작동하였다.

### (5) 생성자 함수로 사용할 수 없음.
```javascript
function a() {		// 기존 함수 선언
	console.log(this);
}

const b = () => {	// arrow function
  console.log(this);
}

console.dir(a);
console.dir(b);
```
위 예제를 수행했을 때 `console.dir`를 출력했을 때 나타나는 두 함수의 차이점은 arrow function은 `prototype`이 없다는 점이다. 이로 알 수 있는 점은 arrow function은 `const c = new b();`와 같이 생성자 함수로 사용할 수 없다는 것이다. 쉽게 말하면 메소드는 메소드로만 작동하고, arrow function은 함수로서만 작동하는 것이다. 마지막 예제를 보고 마치도록 하겠다.
```javascript
const obj = {
  name: 'Son',
  a() {
    return this.name;
  },
  b: () => {
    return this.name;
  }
};

console.log(obj.a());	// "Son"
console.log(obj.b());	// ""
```
둘 다 메소드 형식으로 호출하기는 했지만, arrow function으로 선언한 `b 함수`의 경우에는 `this`에 `obj`가 바인딩되지 않고 `window`의 `name`을 출력하였다. `b 함수`는 함수로서 작동한 것이다.

강의를 보고 정리하기는 하였지만, 정리하면서도 이해가 되지 않는 부분들이 아직 많은 것 같다. 정리한 부분에 잘못 이해한 것이 있기도 한 것 같다. 다시 정리해봐야겠다. 끝😭