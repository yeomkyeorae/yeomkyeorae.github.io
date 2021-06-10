---
title: "JavaScript ES6+ 중급(3) iterable, iterator"
excerpt: "javascript ES6 iterable, iterator"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - javascript
tags:
  - javascript
  - ES6


---

 이 글은 정재남 개발자님의 인프런 강의인 <a href="https://www.inflearn.com/course/es6-2/dashboard" target="_blank">JavaScript ES6+ 제대로 알아보기 중급편</a>을 정리한 내용입니다.



## 1. iterable 개념

`iterable`은 내부 요소들을 공개적으로 반복할 수 있는 데이터 구조로서, `__proto__`에 `Symbol.iterator` 메소드를 갖고 있다. `Symbol.iterator`를 호출하면 `iterator`를 반환하는데, `next` 메소드를 호출하면 value와 done의 객체를 뱉는다!!! iterable한 것들에는 `array`, `map`, `set`, `string`, `generator` 등이 해당하는데, object는 해당하지 않는다.

`iterable`의 특징
- Array.from 메소드를 통해 배열로 전환이 가능하다.
```javascript
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);

console.log(Array.from(map));
// 0: ["a", 1]
// 1: ["b", 2]
// 2: ["c", 3]
```



- 펼치기 연산자(spread operator)를 통해 배열로 전환이 가능하다.
```javascript
const arrFromMap = [...map];
```



- 해체 할당이 가능하다
```javascript
const [mapA, ,mapC] = map; 
```



- for ... of를 사용 가능하다.
```javascript
for(let x of [1, 2, 3]) {
  console.log(x);
}

// 1
// 2
// 3
```



- `Promise.all`, `Promise.race` 명령을 수행할 수 있다
```javascript
const iterableVar = [
  new Promise((resolve, reject) => { setTimeout(resolve, 5000, 10)}),
  new Promise((resolve, reject) => { setTimeout(resolve, 3000, 20)}),
  1004,
  '가나다라',
  new Promise((resolve, reject) => {setTimeout(resolve, 4000, 30)}),
];

Promise.all(iterableVar)
	.then(value => console.log(v));
	.catch(error => console.log(error));

// [10, 20, 1004, '가나다라', 30]
// Promise.all은 인자로 iterable을 받으며, 모든 결과가 다 나올 때 then 구문을 실행한다.
```



- generator - yield\* 문법으로 이용이 가능하다(yield\*은 각각을 yield로 만들라는 것과 같다)
```javascript
const arr = [1, 2, 3];

const makeGenerator = iterable => function* (){
  yield* iterable;
  // yield 1
  // yield 2
  // yield 3
}

const arrGen = makeGenerator(arr)();
```



- 위의 로직들은 [Symbol.iterator]로 반환되는 iterator의 next() 메소드를 기반으로 작동된다! 😳

## 2. iterator
`iterator`는 반복을 위해 설계된 특별한 인터페이스를 가진 `객체`이다.
- `iterator`의 객체 내부에는 `next()` 메소드가 존재.
- `next()` 메소드는 `value`와 `done` 프로퍼티를 지닌 객체를 반환
- `done` 프로퍼티는 `boolean`이다.

아래는 `iterator` 형태의 예시이다.

```javascript
const iterator = {
  items: [1, 2, 3],
  count: 0,
  next() {
    const done = this.count >= this.items.length;
    return {
      done,
      value: !done? this.items[this.count++]: undefined
    }
  }
};

console.log(iter.next());
// {done: false, value: 1}
```
위와 같이 정의한 `iterator`를 object에 `[Symbol.iterator]` 정의해 반환토록 하면 `iterable`한 데이터 구조가 사용할 수 있는 해체 할당 등을 사용할 수도 있다. 아래에서 규칙에 맞게 `iterator`를 구현해 보자. 규칙을 정리하면 다음과 같다.

- object는 Symbol.iterator 메소드가 없다.
- 그래서 Symbol.iterator 메소드를 추가해주면 object는 iterable하게 될 수 있다.
- 대신 Symbol.iterator 메소드는 iterator를 반환해야 하는데,
- 이 iterator는 next 메소드를 갖는 객체여야 한다.
- next 메소드를 호출하면 value와 done 프로퍼티를 가지는 object를 반환한다!
- object는 이제 iterable이다!!!😭

```javascript
const createIterator = function() {
  let cnt = 0;
  const items = Object.entries(this);
  return {
    next() {
      return {
        done: cnt >= items.length,
        value: items[cnt++]
      }
    }
  }
};

const obj = {
  a: 1,
  b: 2,
  c: 3,
  d: 4,
  [Symbol.iterator]: createIterator
}
console.log(...obj);
// (2) ["a", 1] (2) ["b", 2] (2) ["c", 3] (2) ["d", 4]
```

## 3. 정리
- for...of, spread operator, forEach 등은 내부적으로
- Symbol.iterator를 실행한 결과로서 객체를 들고 있으면서
- 그 객체 내부의 next() 메소드를 done이 true가 될 때까지 반복적으로 호출한다.
- iterator를 정해진 규칙에 맞게 구현할 수 있다는 점에서 [`덕 타이핑(duck typing)`](https://ko.wikipedia.org/wiki/%EB%8D%95_%ED%83%80%EC%9D%B4%ED%95%91)이라 할 수 있다.