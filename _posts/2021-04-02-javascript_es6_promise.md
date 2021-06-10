---
title: "JavaScript ES6+ 중급(6) Promise"
excerpt: "javascript ES6 Promise"
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



## 1. Promise의 상태 값

`Promise`는 콜백 함수가 콜백 함수를 호출하는 중첩 구조, 콜백 지옥을 탈출하고자 하는 의도에서 등장했다. `Promise`를 출력해 보면 함수인 것을 알 수 있다. `static` 메소드에는 `all, any, race, reject, resolve` 등이 있고, `Promise`의 인스턴스가 사용하는 메소드에는 `then, catch` 등이 있는 것을 알 수 있다.
```javascript
console.dir(Promise);

// ƒ Promise()
// 	all: ƒ all()
// 	allSettled: ƒ allSettled()
// 	any: ƒ any()
// 	arguments: (...)
//	caller: (...)
// 	length: 1
//	name: "Promise"
//	prototype: Promise
//		catch: ƒ catch()
//		constructor: ƒ Promise()
//		finally: ƒ finally()
//		then: ƒ then()
//		Symbol(Symbol.toStringTag): "Promise"
//		__proto__: Object
//	race: ƒ race()
//	reject: ƒ reject()
//	resolve: ƒ resolve()
//	Symbol(Symbol.species): (...)
//	get Symbol(Symbol.species): ƒ [Symbol.species]()
//	__proto__: ƒ ()
//	[[Scopes]]: Scopes[0]
```
`Promise`의 상태에는 2가지가 있다. 어떤 값이 비동기 처리가 되기 전 상태를 `unsettled(pending, not thenable)`라고 하며. 비동기 처리가 끝난 상태를 `settled(resolved, thenable)`라고 한다. 

`settled` 상태에는 `성공(fulfilled)`과 `실패(rejected)`로 구분한다. 예제를 보자.
```javascript
const promiseTest = param => new Promise((resolve, reject) => {
  setTimeout(() => {
    if(param) {
      resolve('해결!!!');
    } else {
      reject(Error('실패!!!'));
    }
  }, 1000);
});

const testRun = param => promiseTest(param)
	.then(text => console.log(text))
	.catch(err => console.error(err))

const a = testRun(true);	// 해결!!!
const b = testRun(false);	// 실패!!!
```
`new` 연산자로 `Promise` 인스턴스를 생성할 때 인자에는 함수가 들어간다. 이 함수는 첫 번째 인자로 `성공 시 호출 함수`, 두 번째 인자로 `실패 시 호출 함수`를 받는다.

이제 `testRun` 함수에 true 또는 false 인자를 넘겼을 때 `unsetteld`에서 `setTimeout` 이후 `settled` 상태가 되고, `Promise`에서 정의한 함수 로직에 의해 `resolve`인지 `reject`인지 판별된다. 이에 따라서 호출되는 함수가 달라진다. `resolve`될 경우 `then` 구문이 실행되고, `reject`될 경우 `catch` 구문이 실행된다. 이때 `text`, `err`와 같이 값을 받을 수 있다.

## 2. 문법
`then()`과 `catch()` 구문은 언제나 `Promise`를 반환한다. 위 예제에서는 `then`, `catch`에서 출력만 했지만, 무언가 값을 `return` 하면 `Promise`가 반환이 된다. 다음 `then`, `catch` 구문을 탈 수 있다(?).
```javascript
const testRun = param => promiseTest(param)
	.then(text => return 10)
	.then(number => console.log(number))
	.catch(err => console.error(err))
````

첫 번째 예제처럼 `Promise` 인스턴스를 생성하는 것 따로 호출하는 것 따로 할 수도 있지만 하나로 구현할 수도 있다.
```javascript
const simplePromiseBuilder = value => {
  return new Promise((resolve, reject) => {
    if(value) {
      resolve(value);
    } else {
      reject(value);
    }
  })
  .then(res => console.log(res))
  .catch(err => console.error(err))
}

simplePromiseBuilder(10).then(res => {});	// 이어서 then을 또 활용할 수 있다.
```
만약 `Promise` 인스턴스 생성시 인자로 넘긴 함수 내부에서 `resolve`와 `reject` 함수를 조건에 상관없이 둘 다 실행시키면 누가 실행될까? 아래 코드를 실행해 보자.
```javascript
const pro = new Promise((resolve, reject) => {
  resolve();
  reject();
  console.log('Promise!!!');
});

pro.then(() => {
  console.log('then!!!');
})

pro.catch(() => {
  console.log('catch!!!');
})

console.log('hi!!!');
```
코드 실행시 콘솔에 찍히는 결과의 순서는 `Promise!!!`, `hi!!!`, `then!!!`이다. 이로 말미암아 유추할 수 있는 결론은 먼저, then, catch 구문은 실행큐로부터 후순위로 실행된다는 점이다. 그리고 `resolve`와 `reject` 함수를 먼저 호출한 것만 반영된다는 점이다. 실제로 두 함수 모두 호출되지만, 먼저 호출한 `resolve`에 의해 `settled` 상태로 변경되고, `reject` 호출은 `settled` 상태로 변경됐기 때문에 반영이 되지 않는다.

`Promise` 인스턴스를 생성하지 않고도 `resolve`, `reject` 메소드를 호출할 수도 있다.
```javascript
Promise.resolve(100)
	.then(res => console.log(res))
	.catch(err => console.error(err))

Promose.reject(200)
	.then(res => console.log(res))
	.catch(err => console.error(err))
```

이전에 설명했던 `덕타이핑` 개념은 Promise의 then 구문에도 적용이 된다. then 구문을 커스터마이징 할 수 있는 것이다. catch는 적용이 되지 않는다. `thenable`한 객체를 생성하기 위한 조건은 객체에 then 메소드가 존재하고, `resolve`, `reject`를 실행할 수 있도록 하면 된다.
```javascript
const thenable = {
  then(resolve, reject) {
    resolve(1004);
  }
}

const pro = Promise.resolve(thenable);
console.log(pro);

// Promise {<pending>}
//	__proto__: Promise
//	[[PromiseState]]: "fulfilled"
//	[[PromiseResult]]: 1004
```
정상적으로 Promise가 결과를 반환한 것을 볼 수 있다.

## 3. Promise chaining
먼저 여러 개의 `Promise`, `then`, `catch`로 얽힌 예제를 보면서 이해해 보자.
```javascript
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('1번째 프로미스')		// 1초 뒤에 resolve => then
  }, 1000)
})
.then(res => {
  console.log(res);		// '1번째 프로미스'
  return '2번째 프로미스'; // return 한 결과는 Promise의 resolve 값으로 들어간다
})
.then(res => {
  console.log(res);		// '2번째 프로미스'
  return new Promise((resolve, reject) => {	// 새로운 Promise 인스턴스를 return 한다
    setTimeout(() => {
      resolve('3번째 프로미스');	// 1초 뒤에 resolve => then
    }, 1000)
  })
})
.then(res => {
  console.log(res);		// '3번째 프로미스'
  return new Promise((resolve, reject) => {
  	setTimeout(() => {
      reject('4번째 프로미스');	// 1초 뒤에 reject => catch
    }, 1000)
  })
})
.then(res => {		// 위에서 reject를 호출했으므로 넘어감
  console.log(res);
})
.catch(err => {
  console.log(err);		// 빨간 에러 메시지('4번째 프로미스')
  return new Error('이 에러는 then에 잡힌다!!!');	
  // new Error는 프로미스가 아니고 일반 값이므로, resolve된 상태에 값이 담긴 채로 then으로 넘어간다.
})
.then(res => {
  console.log(res);		// 'Error: 이 에러는 then에 잡힌다!!!'
  throw new Error('이 에러는 catch에 잡힌다!!!');	// throw 한 결과는 catch에 잡힌다.
})
.then(res => {
  console.log('출력 되지 않음');
})
.catch(err => {
  console.error(err);	// 빨간 에러 메시지('Error: 이 에러는 catch에 잡힌다!!!')
})
                     
```
위의 예제에서 주목할 부분 중에 하나는 `then`, `catch` 구문 안에서 return 유무 그리고 어떤 것을 return 해주는지이다. 무언가 return을 하면 계속 Promise로서 작동해 다음 `then`이나 `catch`에 태울 수 있다. 정리하면 아래와 같다.

> `then`이나 `catch` 안에서
1. `return Promise 인스턴스`를 하면 말 그대로 Promise의 인스턴스가 리턴된 것이다.
2. `return 일반 값`을 하면 Promise 객체에 resolve 상태로 반환이 되고, 그 안에 값이 담긴다. 예를 들어) Promise {<< resolved >>: 값}
3. `return을 안하면` return undefined와 같다. (일반적으로 return을 안할 경우와 같음)
4. `Promise.resolve()` 또는 `Promise.reject()`만할 경우는 return 해주지 않는 경우와 같다. return을 해줘야 의미가 있다.


위에서 `new Erorr`를 `return` 했을 때와 `throw` 했을 때의 차이점도 존재한다. 전자는 `then` 후자는 `catch`에 잡힌다. 특이한 점은 `Promise` 내에서 Erorr를 `throw` 했을 때, `Promise` 내부에서 Error가 발생했음을 `catch`로 인지할 수 있지만, 전체 소스/서비스에 영향을 끼치는 것을 막을 수 있다.

## 4. 심화
### 1. Error Handling

각각의 `then`과 `catch`에서의 성공과 실패에 따라 어느 로직을 타는지 예제를 보면서 이해해 보자.

```javascript
// 성공 시 바로 다음 then, 실패 시 바로 다음 catch
asyncThing1()
	// 성공 시 바로 다음 then, 실패 시 바로 다음 catch
	.then(asyncThing2)
	// 성공 시 바로 다음 then, 실패 시 바로 다음 catch
	.then(asyncThing3)
	// 성공 시 바로 다음 then(asyncThing4), 실패 시 바로 다음 catch
	.catch(asyncRecovery1)	
	// 성공 시(asyncThing4) 바로 다음 then(asyncRecovery2 실행 x), 실패 시 바로 다음 catch
	.then(asyncThing4, asyncRecovery2)
	// 성공 시 바로 다음 then, 실패 시 종료
	.catch(err => console.log('Dont worry about it'))
	.then(() => console.log('All done'))
```



### 2. Promise.all()

Promise.all()은 `iterable`의 모든 요소들을 Promise 인스턴스로 간주하여 모두 `fullfilled`되는 경우에 전체 결과들을 배열 형태로 then에 전달하는 역할을 한다. 요소 중 하나라도 `reject` 되는 것이 있으면 그 하나를 `catch`로 전달하고 then은 실행되지 않는다.



```javascript
const arr = [
  1,	// 일반 값은 바로 resolved된 값으로 간주됨
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('will be resolved');
    }, 1000)
  }),
  'abcde',
  () => 'not called func',
  (() => '즉시 실행 함수')()
]

Promise.all(arr)
	.then(res => console.log(res))
	.catch(err => console.error(err))

// [1, "will be resolved", "abcde", ƒ, "즉시 실행 함수"]
```



### 3. Promise.race()

Promise.all과 마찬가지로 `iterable`를 인자로 받지만 차이점은 경주(race)를 통해 가장 먼저 `fullfilled`이나 `reject`된 값을 then이나 catch에 전달한다. 



```javascript
const arr = [
  new Promise(resolve => {
    setTimeout(() => { resolve('1번 말')}, 1000)
  }),
  new Promise(resolve => {
    setTimeout(() => { resolve('2번 말')}, 2000)
  }),
  new Promise(resolve => {
    setTimeout(() => { resolve('3번 말')}, 500)
  })
]

Promise.race(arr)
	.then(res => console.log(res))
	.catch(err => console.error(err))
// 3번 말
```



마지막으로 ES2017에 등장한 [async/await function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)의 링크를 걸어 놓으면 JavaScript 강의 정리를 마치겠습니다 😎