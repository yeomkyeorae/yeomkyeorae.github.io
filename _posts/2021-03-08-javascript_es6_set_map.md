---
title: "JavaScript ES6+ 중급(2) Set, WeakSet, Map, WeakMap"
excerpt: "javascript ES6 Set, WeakSet, Map, WeakMap"
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



## 1. Set

 ### (1) 기본 개념
 `Set`은 말 그대로 중복을 허용하지 않는 집합이다. 파이썬에서 알고리즘 테스트를 준비하면서 set을 사용했던 기억이 있다. 물론 dict를 더 많이 사용하긴 했었지만. `Set`은 `new` 연산자를 활용해서 생성할 수 있다. `Set()` 괄호안에는 Iterable한 객체가 올 수 있다.
 ```javascript
const arr = [1, 2, 3, 3, 4, 2, 4, 5, 5, 5];
const s = new Set(arr);
console.log(s);
// Set(5) {1, 2, 3, 4, 5}
 ```

### (2) Set 기능
`Set`에는 주로 다음과 같은 기능들이 있다.
- 추가(add)
- 삭제(delete)
- 초기화(clear)
- 요소 개수 확인(size)
- 포함 여부 확인(has)

하나의 예제로 위의 기능들을 한번에 확인해 보자.
```javascript
const s = new Set();
s.add(1);
s.add(2);
s.add(3);
s.add(2);
console.log(s);
// Set(5) {1, 2, 3}

s.delete(3);
console.log(s);
// Set(5) {1, 2}

console.log(s.size);	// 2

console.log(s.has(2));	// true
console.log(s.has(3));	// false

s.clear();
console.log(s.size);	// 0
```

### (3) SetIterator
`Set`은 iterable한 객체이다. 따라서 spread operator를 적용할 수 있다. 또한 `Set`에서 `entries(), keys(), values()`를 사용할 수 있는데, 도출되는 값은 `SetIterator`다. 특이한 점은 `Set`에는 인덱스가 없으므로 위 세가지 메소드로 도출되는 값은 모두 `Set`에 포함된 value 값이라는 점에서 동일하다.
```javascript
const s = new Set([1, 2, 3, 3, 4, 2, 2, 5, 1]);
console.log(s.entries());
console.log(s.keys());
console.log(s.values());

// SetIterator {1 => 1, 2 => 2, 3 => 3, 4 => 4, 5 => 5}
// SetIterator {1, 2, 3, 4, 5}
// SetIterator {1, 2, 3, 4, 5}
```
`Set`의 값의 중복 제거, Iterable한 객체를 전체 순회시, 값의 유무를 빠르게 판단할 때 활용적이다.

## 2. WeakSet
`WeakSet`은 약한 집합이다. 어느 부분이 약할까? 바로 `참조 카운트를 증가시키지 않는다`는 점이다. 다른 변수가 객체 변수를 참조할 때 객체 변수의 참조 카운트를 1 증가시키는데, 참조 카운트가 0이 되면 GC(garbage collector)에 의해 수거 대상이 되어 객체 변수에 대한 정보를 제거한다. 따라서 `WeakSet`이 참조하는 객체 변수의 참조 카운트가 0이 되면 `WeakSet` 내부에 있는 정보가 사라질 수 있다.

`WeakSet`의 특이한 점은 참조형 데이터만을 요소로 삼을 수 있어 값을 하나 하나 순회하는 작업을 할 수 없다는 점(iterable 하지 않다)이다. 예를 들어 위 `Set`에서 다른 메소드들인 `keys(), values(), entries(), size, forEach() 등`을 사용할 수 없다.

그렇다면 도대체 `WeakSet`은 언제 사용할까? 모르겠다. 약한 놈은 이 세상에서 살아갈 수 없나 보다.

## 3. Map
`Map`에 대해 알아보기 전에 객체의 단점을 알아 보자.

- iterable 하지 않다. `for ... in`을 쓰면 되지 않는가? 그것은 페이크다!!
```javascript
const obj = {
  name: 'giraffe',
  age: 29
};
// for ... in 객체 순회
for(let key in obj) console.log(key, obj[key]);
// name giraffe
// age 29

// 객체 프로토타입 method에 함수를 넣는다면?
Object.prototype.method = function() {};
for(let key in obj) console.log(key, obj[key]);
// name giraffe
// age 29
// method f () {}
```
위 예제를 보면 객체 프로토타입에 method에 함수를 입력 후 `for ... in` 순회를 한 결과 입력한 함수까지 도출되는 것을 볼 수 있다. 이 결과를 보면 `for ... in` 순회는 `prototype`도 검사하는 것을 알 수 있다. 이를 해결하기 위해 `for ... in` 내부에 `hasOwnProperty` 메소드로 key가 있는 게 맞는지 확인하는 로직을 추가할 수 있다. 

또한 
- 객체는 key를 문자열로 인식한다.
- property 개수를 직접적으로 파악할 수 없다. (keys를 활용하면 가능)

이런 단점들을 해결하기 위해 Map이 등장했다.

### (1) 기본 개념
`Map`은 [key, value] 쌍으로 이루어진 요소들의 집합이다. 순서를 보장하며, 객체와 달리 iterable하다. key에는 문자열뿐만 아니라 어떤 요소도 올 수 있다. 아래는 Map을 선언하는 방법, 기초적인 메소드 예시이다.
```javascript
const map = new Map();	// Map 초기화

map.set(10, 20);		// Map key, value 쌍 추가
map.set('name', 'yeom')

map.get('name')			// Map 값 가져오기 

map.delete('name')		// Map 쌍 삭제

map.has('name')			// Map key 여부 확인

map.size	// Map 쌍 개수 확인
```
`Map`을 초기화할 때는 `Set`처럼 iterable 한 요소가 올 수 있지만, 각 요소들이 key와 value 쌍을 이루고 있어야 한다.

```javascript
const arr = [[10, 20], [20, 30], [30, 40]];
const map = new Map(arr);
console.log(map);

// Map(3) {10 => 20, 20 => 30, 30 => 40}
```

그 다음에는 `MapIterator` 요소를 반환하는 메소드를 살펴보자. 여기에는 `Set`처럼 `keys(), values(), entries()`가 있다. `MapIterator`는 `next()` 메소드를 호출할 수 있는데, `value`, `done`으로 구성된 객체를 반환한다. `value`에는 순회하는 차례의 값을 가지고 있고, `done`에는 순회가 끝났는지를 표시하는 bool값을 가지고 있다. `values()` 메소드를 예로 들어보자.
```javascript
const arr = [[10, 20], [20, 30], [30, 40]];
const map = new Map(arr);

const values = map.values();
console.log(values.next());		// {value: 20, done: false}
console.log(values.next());		// {value: 30, done: false}
console.log(values.next());		// {value: 40, done: false}
console.log(values.next());		// {value: undefined, done: true}

```

## 4. WeakMap
약한 놈이 또 등장했다. `WeakMap`은 `WeakSet`과 마찬가지로 참조 카운트를 증가시키지 않는다. key에 할당하는 변수가 반드시 참조형 변수만이 올 수 있다. 아래는 그 예시이다.
```javascript
let obj = { 1: 2, 3: 4};	// 참조 카운트: 1
const map = new Map();
map.set(obj, 5); 			// 참조 카운트: 2
obj = null;					// 참조 카운트: 1

let obj2 = { 1: 2, 3: 4};	// 참조 카운트: 1
const map2 = new WeakMap();
map2.set(obj2, 5); 			// 참조 카운트: 1
obj2 = null;				// 참조 카운트: 0
```
`WeakMap`은 `WeakSet`처럼 iterable 하지 않고, `size`와 같은 메소드를 사용할 수 없다. 그렇다면 `WeakMap`은 어떻게 사용할까? 아래 예제는 즉시 실행 함수로 class를 반환하고 그 내부에 `WeakMap`을 활용해 인스턴스를 저장한다. 인스턴스가 사라질 경우(참조 카운트가 0이 되면) `WeakMap`에 저장한 인스턴스 정보도 사라진다. 아래 내용은 `class`에 대한 학습을 마치고 더 자세히 알아봐야 겠다.

```javascript
const weakMapValueAdder = (wmap, key, addValue) => {
  wmap.set(key, Object.assign({}, wmap.get(key), addValue));
}

const Person = (() => {
  const privateMembers = new WeakMap();		// 클로저로서 변수 범위는 블록스코프에 갇힌다
  return class {
    constructor(n, a) {
      privateMembers.set(this, { name: n, age: a });
    }
    
    // getter, setter 함수들
    set name(n) {
      // WeakMap에 인스턴스 정보 저장
      weaMapValueAdder(privateMembers, this, { name: n });	
    }
    
    get name() {
      return privateMembers.get(this).name;
    }
    
    set age(a) {
      weakMapValueAdder(privateMembers, this, { age: a});
    }
    
    get age(a) {
      return privateMembers.get(this).age;
    }
  })();
```