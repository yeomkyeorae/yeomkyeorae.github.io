---
title: "JavaScript ES6+ (7) Function etc."
excerpt: "javascript ES6 함수 기타 기능"
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



# function - etc.

## 1. name property

기존에는 기명 함수만 `name`이라는 이름으로 property를 제공하였지만, 익명 함수, arrow 함수에도 `name`을 제공하게 되었습니다.
```javascript
function print() {}
console.log(print.name);	// print

const fn = function() {}
console.log(fn.name);		// fn

const fn2 = () => {}
console.log(fn2.name);		// fn2

const obj = {
  fn1: function() {},
  fn2 () {},
  fn3: () => {}
}
console.log(obj.fn1.name, obj.fn2.name, obj.fn3.name);
// fn1, fn2, fn3
```

`class`의 경우를 한번 보겠습니다. 전자는 class로 구성했고, 후자는 function으로 구성했습니다. 결과부터 말하자면, class에 함수를 구성했을 경우에는 name이 도출됐지만, 생성자 함수에 함수를 프로퍼티로 할당했을 경우에는 name이 출력되지 않았습니다.
```javascript
class A {
  static fn1() {}
  fn2 () {}
}

const a = new A();
console.log(A.fn1.name, a.fn2.name);	// fn1 fn2
```
```javascript
function B() {}		// 생성자 함수로 사용. 함수는 일급 객체
B.fn1 = function() {}	// 그러므로 객체에 프로퍼티를 할당
B.prototype.fn2 = function() {}	// prototype에 할당하는 메소드들은 인스턴스에 상속이 된다.

const b = new B();
console.log(B.fn1.name, b.fn2.name);	// 아무 것도 안나옴
```
각각의 instance를 출력했을 때 그 내부를 보면 그 차이를 알 수 있습니다. 할당된 함수를 보면 class의 경우에는 name이 할당된 것을 알 수 있는데, 생성자 함수의 경우에는 name이 할당되지 않았습니다. 후자의 경우에는 함수 이름을 정해주기 위해서 프로퍼티 값을 참고할텐데 `fn2`로 할 것인지, `prototype.fn2`로 할 것인지... 그  기준을 정하는 부분이 구현이 되지 않은 것으로 보입니다😳
```javascript
console.log(a);
// A {}
//	__proto__:
//	constructor: class A
//	fn2: ƒ fn2()
//		arguments: (...)
//		caller: (...)
//		length: 0
//		name: "fn2"
//	...

console.log(b);
// B {}
//	__proto__:
//	fn2: ƒ ()
//		arguments: null
//		caller: null
//		length: 0
//		name: ""
//	...
```
bind로 함수를 새로 만들면 그 함수의 이름은 어떻게 될까요?? 확인해보고 이 부분을 마무리 짓도록 하겠습니다.
```javascript
const fn1 = function() {}
const fn2 = fn1.bind();
console.dir(fn2);

// bound fn()
// arguments: (...)
// caller: (...)
// length: 0
// name: "bound fn"
```
`bound` `bound 이전 함수명`으로 구성됩니다!😘
참고로 object의 `getter, setter` 함수는 `get 함수명`, `set 함수명`으로 이름이 정해집니다.

## 2. new.target
기존 ES5에서는 new 연산자를 강제하기 위해서 다음과 같은 방법을 사용했습니다.
```javascript
function Person(name) {
  if(this instanceof Person) {
    this.name = name;
  } else {
    throw new Error('you need new operator');
  }
}

let p1 = Person('error man');	// Error 발생
let p2 = new Person('babo');	// name에 'babo' 할당
let p3 = Person.call({}, 'super man'); 	// Error 발생. this가 {}이므로
let p4 = Person.call(p2, 'iron man'); // p2의 name에 'iron man' 할당
```
그러나 p4 변수의 사례처럼 new 연산자를 강제하고자 했지만 new를 사용하지 않고도, 위 Person 함수를 호출하는 악행이 벌어지고 말았다...😭 그래서 등장한 것이 `new.target`이다! 두둥

```javascript
function Person(name) {
  // new.target은 new 연산자가 붙은 함수 이름이 된다. 
  console.dir(new.target);	
  
  if(new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('you need new operator');
  }
}

let p2 = new Person('babo');
let p4 = Person.call(p2, 'iron man');	// Error 발생
					// new.target이 undefined
```
참고로 함수 내부에 arrow function이 있을 경우 내부에 `new.target`울 사용하면 `new.target`은 arrow function의 외부 함수를 가르킨다. 쉽게 예제를 보면 아래와 같다.
```javascript
function Person(name) {
  const fn = n => {	// arrow function
    this.name = n;
    console.log(new.target);	// 생성자 함수로 호출했을 경우 Person를 가리킨다.
  }
  fn(name);
}

const p1 = new Person('babo');	// Person
const p2 = Person('hulk');	// new 연산자를 안 썼으므로 에러 발생
```
`new.target`은 class에서 사용해 추상 클래스처럼 사용할 수 있게 해준다.
```javascript
class A {
  constructor() {
    if(new.target === A) {
      throw new Error('This is abstract class!');
    }
  }
}

class B extends A {
  constructor() {
  	super();
  }
}

const a = new A();	// 'This is abstract class!'
const b = new B();
```

## 3. 함수 선언문과 블록 스코프
기존 ES5에서는 블록 스코프 개념이 없어서, 함수도 마찬가지로 블록 스코프에 관계 없이 접근이 가능했습니다. 아래 예제를 보시죠.
```javascript
a();	// true
if(true) {
  a();	// true
  function a() { console.log(true); }
}
a();	// true
```
브라우저 환경 마다 다르지만 3번의 a함수 호출이 모두 가능해서 `true`가 3번 출력되었습니다. 하지만 `strict mode(반대는 sloppy mode: 브라우저 마다 다른 동작... 작동 예상이 어려움)`를 사용하면 함수 선언문도 역시 블록 스코프에 갇히게 할 수 있습니다.
```javascript
'use strict';
a();	// error
if(true) {
  a();	// true
  function a() { console.log(true); }
}
a();	// error
```
하지만 매번 `use strict`를 쓰면서 어떻게 동작할지 고려하는 것도 번거롭습니다. 이에 강의를 해주신 개발자님께서는 
> "arrow function을 사용하고, 객체에서는 메소드 축약형을 쓰고, 생성자 함수는 class를 구현하면, function 키워드는 function* 형태로 generator에서만 사용할 것이다"

라고 말씀하셨습니다. 이를 이해하기 위해서 좀 더 정진하도록 하겠습니다. 😍