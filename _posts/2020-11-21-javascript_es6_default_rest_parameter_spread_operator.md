---
title: "JavaScript ES6+ (4) default parameter, rest parameter, spread operator"
excerpt: "javascript ES6 default parameter, rest parameter, spread operator"
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



## 1. default parameter

기존에는 함수 매개변수의 default value를 함수 내부에서 처리했다. 😳
```javascript
const f = function(num1, num2, num3) {
  num1 = num1 ? num1: 10;
  num2 = num2 ? num2: 20;
  // ...? 이게 무슨 짓이야
}
```
하지만 num1에 의도적으로 0을 넘긴다면? false로 인식해 num1은 10이 된다! 결과적으로 변수를 활용하는데 여간 신경 쓰이는 일이 아닐 수 없었다. 하지만 default parameter를 사용하면 이를 손쉽게 해결할 수 있다. 😘

```javascript
const f = function(a=10, b=20, c=30, d=40, e=50, f=60) {
	console.log(a, b, c, d, e, f);
}
f(5, 0, '', false, null);
// 5, 0, '', false, null, undefined
```
함수를 선언할 때 매개변수에 값을 할당하면 default value로서 기능한다. 이는 매개변수로 전달된 값이 `undefined` 혹은 누락됐을 경우에만 작동한다.

default parameter는 식을 대입할 수도 있고, 함수를 호출할 수도 있다. 대신 매개변수의 순서가 중요하게 된다.
```javascript
const f = function(x=10, y=x+10, z=getNumber()) {
	console.log(x, y);
}
f(20);	//20, 30
```
`function(y=x+10, x=10)`로 함수를 선언하게 되면 `reference error`가 발생하게 된다. default parameter로 선언하게 되면 `let` 변수를 생성한 것과 동일한데 `y` 변수는 `x`를 참조해야할 주소를 모르기 때문이다. TDZ에 걸리게 되고 error가 발생한다. 아래 코드를 보면 더 이해가 쉽다.
```javascript
const f = function(y=x+10, x=10, z=getNumber()) {
  let _y = x + 10;	// x는 TDZ에 걸리게 된다!! 
  let _x = 10
  //	...
}
```
주의할 점은 하나 더 있다. 매개변수에서 할당하는 변수명과 할당받는 변수명이 동일하면 에러가 발생한다. 이것도 TDZ와 관련이 있다.
```javascript
let x = 10;
let y = 20;
const f = function(xx = x, y = y) {
  console.log(xx, y);
}
//위의 코드에서 let으로 y를 선언하고, f함수에서 매개변수 y의 기본값으로 y를 사용하고 있다. 이는 아래 코드처럼 수행하는 것이다!!

let y;
(let ) y = y;
//호이스팅으로 y가 선언됐지만 주소 매칭이 되지 않은 y를 다시 y에 할당하고자 했으므로 TDZ에 걸려 referece에러가 발생된다!!
```
마지막으로 `arguments`에 대한 사항을 보겠다. `arguments`는 유사배열객체로서 index, value로 이루어진 배열처럼 각각을 property와 value로 만든 객체이다. 객체이므로 배열 메소드를 쓸 수 없어...
`const arg = Array.protoType.slice.call(arguments)`처럼 사용하기도 했다고 한다. `arguments`는 default parameter의 영향을 받을까? 답은 받지 않는다이다. 예제를 보자.
```javascript
const f = function(num1=10, num2=20, num3=30) {
  console.log(arguments);
}
f(1, 2, 3);
//Arguments(3) [1, 2, 3, callee: (...), Symbol(Symbol.iterator): ƒ]

f();
//Arguments [callee: (...), Symbol(Symbol.iterator): ƒ]
```
`arguments`라는 놈이 있다는 사실만 기억하고 넘어가도록 하자.

## 2. rest parameter
arguments를 대체할 훌륭한 녀석이 있다. 그 놈은 바로 `rest parameter`다. 말 그대로 나머지 매개변수의 정보를 갖고 있는 배열이다. `...매개변수명`으로 표현하는데 예제를 보면 바로 이해할 수 있다.
```javascript
const f = function(x, y, ...z) {
  console.log(z);
}
f(10, 20, true, false, undefined, null, 100);
// [true, false, undefined, null, 100]
```
첫 번째 인자와 두 번째 인자인 x, y를 제외한 매개변수들을 z에 배열로 담겨져 있는 모습을 볼 수 있다. 주의할 점은 매개변수 마지막에 `rest parameter`가 와야한다는 점이다. `function(...z, x, y)`는 NoNo다. 또한 객체의 setter에서는 사용할 수 없다.
```javascript
const obj = {
  _num: '',
  get num() { return this._num; },
  set num(value) { this._num = value; }
}
```
위에서 setter의 역할을 하는 set에는 1개의 parameter만 허용하기 때문이다. 객체 자체는 하나의 property에 하나의 value만 허용한다!

## 3. spread operator
`spread operator`는 직역하면 펼치기 연산자다. ...을 사용하면 rest parameter와 유사하다. 예시부터 보자.
```javascript
const pokemon1 = ['pikachu', 'pigeon'];
const pokemon2 = ['mew'];
const newPokemon = [...pokemon1, 'Charmeleon', ...pokemon2];
console.log(newPokemon);
// ["pikachu", "pigeon", "Charmeleon", "mew"]
```
위의 `newPokemon`을 구성함에 있어서 rest parameter처럼 ...을 사용하면 안에 요소들을 하나씩 펼쳐주어 새로운 배열을 구성하도록 할 수 있다.
`Math.max`는 배열이 아닌 여러 매개변수들을 넣어주어 최댓값을 구하는데, `spread operator`를 활용하면 배열의 최댓값을 구할 수 있다.
```javascript
const nums = [1, 5, 4, 3, 2];
console.log(Math.max.apply(null, nums);	//기존
console.log(Math.max(...nums));		//spread operator
```
`spread operator`의 특징은 `iterable`한 모든 것(String, Set, Array 등)을 펼칠 수 있다는 점, 새로운 배열을 생성하지만 얕은 복사라는 점 등이 있다. 얕은 복사에 대한 예제를 보고 이 장을 마무리하고자 한다😘
```javascript
const zoo = [{
    name: 'king',	
    species: "lion",
    age: 3
}, {
    name: "kirin",
    species: "giraffe",
    age: 5
}];

const newZoo = [...zoo];
newZoo[0].species = "rabbit";

console.log(zoo);
console.log(newZoo);
```
결과를 보면 기존 동물원과 새로운 동물원의 왕은 토끼가 된 것을 알 수 있다
🦁🦒 🔜 🐰🦒