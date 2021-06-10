---
title: "JavaScript ES6+ 중급(4) Generator"
excerpt: "javascript ES6 Generator"
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



## 1. 개념

 `generator`는 함수 표시(function) 앞에 `*`가 붙는 녀석이다. 이 녀석은 `next()` 메소드를 호출하면 `yield`(넘겨주다, 양도하다) 키워드를 만나기 전까지를 실행한다. 선언과 간단한 활용은 아래와 같이 한다. `next()` 메소드를 호출하면 객체에 `value`와 `done` 프로퍼티가 있는 것을 알 수 있다.

 ```javascript
function* gen() {
  console.log(1);
  yield 1;
  console.log(2);
  yield 2;
  console.log(3);
  yield 3;
}
const gen = gene();
gen.next();	
// 1
// {value: 1, done: false}

gen.next();	
// 2
// {value: 2, done: false}

gen.next();	
// 3
// {value: 3, done: false}

gen.next();	
// {value: undefined, done: true}
 ```
객체 내부에 축약형 함수를 선언할 경우 `*`은 변수명 앞에 둔다. class에서도 마찬가지다.
```javascript
const obj = {
  gen1: function* () {.
  *gen2 () { yield }
}

class A {
  *gen () { yield }
}
                      
```

## 2. iterator로서의 generator
```javascript
const obj = {
  a: 1,
  b: 2,
  c: 3,
  *[Symbol.iterator] () {
    for(let prop in this) {
      yield [prop, this[prop]];
    }
  }
}

console.log(...obj);
// ["a", 1], ["b", 2], ["c", 3]

for(let p of obj) {
  console.log(p);
}
// ["a", 1]
// ["b", 2]
// ["c", 3]
```
객체에 `iterator`를 적용하기 위해서 `Symbol.iterator`가 객체를 반환해야 되고... 그 객체는 next() 메소드를 반환해야 되고... done 프로퍼티를 반환해야 한다 등의 규칙을 적용해야 했지만, `generator`를 활용해 `yield`를 적절히 사용하면 복잡하게 구현할 필요성이 줄어들게 된다. 


`yield` 키워드 뒤에 `*`가 붙어 있고, 그 뒤에 `iterator` 요소가 오면, `iterator`의 내부 요소들을 하나씩 반환한다.
```javascript
function* gen() {
  yield* [1, 2, 3];
  yield* 'abc';
}

const gene = gen();
gene.next();
// {value: 1, done: false}

gene.next();
// {value: 2, done: false}

gene.next();
// {value: 3, done: false}

gene.next();
// {value: 'a', done: false}
// ...
```
`yield` 뒤에 `generator`를 사용할 수도 있다. 그러면 참 복잡해진다. 큼...🤣

## 3. yield와 값의 할당
예제부터 보자.
```javascript
function gen() {
  let first = yield 1;
  console.log(first);
  
  let second = yield first + 2;
  console.log(second);
}

const gene = gen();
gene.next();
// {value: 1, done: false)

gene.next();
// {value: NaN, done: false}
```
위 예제에서 `첫번째 yield`를 만났을 때 `value 1`을 반환하는데 마치 이 값은 `first 변수`에 할당될 것만 같다. 하지만 그렇지 않다. yield는 value와 done의 객체를 반환하지만 `변수에 값을 할당하지 않는다`. 그래서 `두 번째 yield`를 했을 때 반환되는 value 값은 `NaN`이 나오게 된다. first에 해당하는 값을 할당하고자 한다면 두 번째 next 메소드를 호출할 때(`gene.next(10)`) 인자에 값을 넣어주면 된다.

## 4. 비동기 처리 예시
아래 예제는 `1000번째` 이후 유저들을 가져와서 `4번째(1004)`, `6번째(1006)` 유저들의 정보를 가져오는 것이다. 천천히 코드를 음미하면서 이해해 보자 😳. 특히 `generator`를 인자로 넘기고, `fetchWrapper`에서 `then` 이후에 다시 `next`를 호출하는 것을 이해해 보자!

```javascript
const fetchWrapper = (gen, url) => fetch(url)
	.then(res => res.json())
	.then(res => gen.next(res));

function getNthUserInfo() {
  const [gen, from, nth] = yield;
  const req1 = yield fetchWrapper(gen, `https://api.github.com/users?since=${from || 0}`);
  const userId = req1[nth - 1 || 0].id;
  console.log(userId);
  
  const req2 = yield fetchWrapper(gen, `https://api.github.com/user/${userId}`);
  console.log(req2);
}

const runGenerator = (generator, ...rest) => {
  const gen = generator();
  gen.next();
  gen.next([gen, ...rest]);
}

runGenerator(genNthUserInfo, 1000, 4);
runGenerator(genNthUserInfo, 1000, 6);
```