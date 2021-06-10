---
title: "JavaScript ES6+ (5) Enhanced object functionalities"
excerpt: "javascript ES6 향상된 객체 기능들"
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



이번 글에서는 object의 향상된 기능들에 대해서 다뤄보겠습니다.

## 1. shorthand property

프로퍼티 축약. object의 key와 value에 할당하는 변수명이 동일한 경우, value 부분을 생략할 수 있는 방법이다.
```javascript
const price = 10000;
const mileage = 500;
const product = {
  price,
  mileage
};
```
위와 같이 선언하면 위에서 선언한 price, mileage 변수가 동일한 이름으로 product의 key가 되고, value는 그 값이 된다.

> 참고) destructing assignment
```javascript
const animals = {
  tiger: 'uh-heung',
  rabbit: 'kkang-chong',
  dog: 'daeng-daeng'
};
const { tiger, rabbit, dog } = animals;
```
위와 같이 객체의 key값에 해당하는 값을 각 변수로 할당할 수 있다.


## 2. concise methods
객체 프로퍼티에 함수를 할당할 때 축약할 수 있는 기능이다. 예제를 보면,
```javascript
const obj = {
  num: 100,
  getNum1() { return this.num },
  getNum2: function() { return this.num }
}

console.dir(obj.getNum1);
console.dir(obj.getNum2);
```
`getNum1` 이름으로 축약형, `getNum2` 이름으로 축약형이 아닌 문법을 사용하였다. 그리고 `console.dir`로 두 함수의 차이점을 살펴보자.
> ```javascript
> // getNum1
> arguments: [Exception: TypeError: ...]
> caller: [Exception: TypeError: ...]
> length: 0
> name: "getNum2"
> __proto__: ƒ ()
```

축약형은 `arguments`와 `caller`에 에러가 발생했다. 외부 스코프에서 함수가 종료된 후에 접근하고자 해서 생긴 에러라고 한다.

> ```javascript
// getNum2()
arguments: null
caller: null
length: 0
name: "getNum1"
prototype: {constructor: ƒ}
__proto__: ƒ ()
```

축약형이 `arguments`와 `caller`에 에러가 생긴 것과 달리 명시적으로 에러가 발생했음을 알려주지 않았다. 그리고 기존 문법은 `__proto__`와 `prototype`이 존재한다. 하지만 축약형은 `prototype`이 사라지고 `__proto__`만이 존재한다. 이것이 무엇을 의미할까?

기존 문법으로는 생성자 함수를 사용할 수 있었음을 알 수 있다. `getNum2`가 갖고 있는 `prototype`을 `__proto__`에 연결시켜 주는 역할을 하였다.
```javascript
const a = new obj.getNum2();	// 생성됨. a에 __proto__ 내용 존재.
const b = new obj.getNum1();	// not a constructor 에러.
```
위 예제처럼 축약형은 생성자 함수로서 기능하지 못한다. 대신 함수 본연으로서의 기능을 하면서 더 가벼워진 장점이 존재한다.

`concise methods`를 사용함으로써 `super` 명령어("상위에 있는 애를 호출하라!!!")로 상위 클래스에 접근이 가능하다. 

```javascript
const person = {
  sayToHello() {
    return 'Hello';
  }
};
const brother = {
  sayToHello() {
    return 'Gracias, ' + super.sayToHello();
  }
};

Object.setPrototypeOf(brother, person);
```
위에서 `setPrototypeOf`는 `brother를 인스턴스로 하면서 person을 생성자로 하라는 의미`이다. 쉽게 말하면 `person`이 상위 `brother`가 하위가 된다. `brother`를 출력하면 아래와 같다.
```javascript
{sayToHello: ƒ}
sayToHello: ƒ sayToHello()	// Gracias, Hello
__proto__:
	sayToHello: ƒ sayToHello()	// Hello
	__proto__: Object
```
`brother.__proto__.sayToHello()`를 실행하면 super를 활용해 상위 객체의 `sayToHello`를 실행하는 것과 같다!😉

위와 동일한 형식의 코드는 아래와 같이 작성할 수 있다.

```javascript
const person = function() {}
person.prototype.sayToHello = function() { return 'Hello' }

const brother = new person();
brother.greeting = function() {
  return 'Gracias, ' + this.__proto__.sayToHello();
}
```

## 3. computed property
computed property는 객체 선언시 프로퍼티 키 값에 대괄호 표기로 접근 가능한 것을 말한다. 이 대괄호 내에는 값이나 식을 넣어 조합이 가능하다. 기존에는 이미 선언한 객체에 키-값 조합을 한 개 추가하는 용도로 대괄호 표기법을 사용했지만, 이제 선언시에도 가능하게 되었다.
```javascript
// 기존
const className = ' Class';
const obj1 = {};
obj1['Super' + className] = 'Super';

// 선언시에도 대괄호를 사용할 수 있게 되었다.
const obj2 = {
  ['Super' + className]: 'Super'
};
```
클로저와 즉시 실행 함수를 활용한 예제로 이 부분은 마무리한다😉
```javascript
const count = (function(c) {
  return function() {
    return c++;
  }
})(0)

const obj = {
  [`num_${count()}`]: count(),
  [`num_${count()}`]: count(),
  [`num_${count()}`]: count()
};
console.log(obj);
// num_0: 1
// num_2: 3
// num_4: 5               
```

## 4. property enumeration order
이번 내용은 object의 key를 반복문으로 저장하면 그 순서는 어떻게 될까? 먼저 코드를 보자.
```javascript
const obj = {
  b: 1,
  4: 2,
  c: 3,
  2: 4,
  1: 5,
  a: 6
};
const keys = [];
for (const key in obj) keys.push(key);

console.log(keys);
// ["1", "2", "4", "b", "c", "a"]

console.log(Object.keys(obj));
// ["1", "2", "4", "b", "c", "a"]

console.log(Object.getOwnPropertyNames(obj));
// ["1", "2", "4", "b", "c", "a"]
```
다양한 방식으로 배열 형식의 key 모음을 출력했을 때 결과는 먼저 숫자가 오름차순으로 정렬되고, 문자열은 저장된 순서대로 출력이 된다.

다음 예를 보자.
```javascript
const obj = {
  [Symbol('2')]: true,
  '02': true,
  '10': true,
  '01': true,
  '2': true,
  [Symbol('1')]: true
};
const keys = [];
for (const key in obj) keys.push(key);

console.log(keys);
// ["2", "10", "02", "01"]
```
key가 모두 문자열인데, 순서가 바뀌었다. 문자열 앞에 0이 없는 문자열은 숫자로 인식한다. 그래서 '2'는 2로 '10'은 10으로 인식하게 되었다. '02'와 '01'은 입력된 순서로 출력되었다. 여기서 `Symbol`은 열거 대상에서 제외돼서 출력되지 않는다. 하지만 `Reflect.ownKeys()`를 사용하면 `Symbol`도 확인할 수 있다.
```javascript
console.log(Reflect.ownKeys(obj));
// ["2", "10", "02", "01", Symbol(2), Symbol(1)]
```
심볼은 마지막 순서로 위치한다.