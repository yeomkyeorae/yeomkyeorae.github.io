---



```
title: "TypeScript 기본 문법 정리"
excerpt: "typescript 기본 문법"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - typesciprt
tags:
  - typescript
```

---

이 글은 캡틴판교 장기효님의 인프런 강의 [타입스크립트 입문 - 기초부터 실전까지](https://www.inflearn.com/course/%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%9E%85%EB%AC%B8/dashboard)의 내용을 예제 코드 위주로 간략하게 정리한 글입니다.



## 1. 기본 TS 타입 선언

### 문자열

```typescript
let str: string = 'hello';
```

### 숫자

```typescript
let num: number = 100;
```

### 배열

```typescript
let arr: Array<number> = [10 , 20, 30];
let arr2: number[] = [10, 20, 30];
let arr3: Array<string> = ["lion", "tiger"];
let arr4: [string, number] = ["sejong", 182];
```

### 객체

```typescript
let obj: object = { name: "yeom", age: 29 };
let person: { name: string, age: number };
```

### Boolean

```typescript
let isAvaliable: boolean = true;
```



## 2. 함수 선언

parameter와 return 값에 대해 타입 선언을 할 수 있다.

```typescript
function sum(a: number, b: number): number {
  return a + b;
}
```

optional parameter일 경우 `?` 를 사용한다.

```typescript
function log(time: string, result?: string, option?: string) {
	console.log(time, result, option);
}

log("2021-10-04", "success");
```



## 3. 인터페이스(interface)

`interface`는 자주 사용하는 타입들을 object 형태의 묶음으로 정의해 `새로운 타입`을 만들어 사용하는 기능이다.

### interface 선언

```typescript
interface User {
  age: number;
  name: string;
}
```

### 변수 활용

```typescript
const yeom: User = { age: 30, name: 'kyeorae'}
```

### 함수 인자로의 활용

```typescript
function getUser(user: User) {
  console.log(user);
}

getUser({ age: 10, name: 'hayoon'});
```

### 함수 구조 활용 

```typescript
interface Sum {
  (a: number, b: number): number;
}

let sumFunc: Sum: 
sumFunc = function(a: number, b: number): number {
  return a + b;
}
```

### 배열 활용

```typescript
interface StringArray {
  [index: number]: string;
}

let arr: StringArray = ['a', 'b', c];
```

### 객체 활용

```typescript
interface StringRegexObject {
  [key: string]: RegExp;
}

const obj: StringRegexObject {
  cssFile: /\.css$/,
  jsFile: /\.js$/
}
```

### Interface 확장

`extends` 키워드를 사용한다

```typescript
interface Person {
  name: string;
  age: number;
}

interface Developer extends Person {
  skill: string;
}

const juniorDeveloper = {
  name: 'yeom',
  age: 29,
  skill: 'sleep'
}
```



## 4. 타입 별칭(type aliases)

`type` 키워드는 `interface`와는 다르게 새로운 타입을 생성하는 것이 아닌 `별칭`을 부여하는 것이다. `extends` 키워드는 사용할 수 없다. 

### 타입 별칭 선언 및 활용

```typescript
type MyString = string;
const str: MyString = 'I love you';

type Todo = { 
  id: string;
  title: string;
  done: boolean;
};

function getTodo(todo: Todo) {
  console.log(todo);
}
```



## 5. 연산자(Operator)

### Union Type

한 가지 이상의 type을 선언하고자 할 때 사용할 수 있다. `|` 기호를 사용한다

```typescript
function logMessage(value: string | number) {	// 함수 호출시 value 인자의 타입에는 string 또는 number
  if(typeof value === 'string') {
    value.toString();
  } else if(typeof value === 'number') {
    value.toLocaleString();
  } else {
    throw new TypeError('value must be string or number');
  }
}

logMessage('hello');
logMessage(100);
```

### Intersection Type

합집합과 같은 개념. 함수 호출의 경우. 함수 인자에 명시한 type을 모두 제공해야 한다. `&` 키워드를 사용한다.

```typescript
interface Zoo {
    name: string;
    location: string;
    price: number;
}

interface Animal {
    name: string;
    count: number;
}

// Union: Zoo나 Animal에 해당하는 인자를 제공해야 한다
askZookeeper({ name: 'tiger', count: 3});

// Intersection: Zoo나 Animal에 해당하는 인자를 줘야한다(합집합 같은 개념)
askZookeeper2({ name: '어린이대공원', location: '서울시 광진구', price: 10000, count: 10000 });
```



## 6. Enum

`enum` 키워드를 사용하면 일종의 default 값을 선언할 수 있다.

### 숫자형 enum

자동으로 0에서 1씩 증가하는 값을 부여

```typescript
enum Shoes {
  Nike,				// 0
  Adidas,			// 1
  NewBalance	// 2
}

const myShoes = Shoes.Nike;	// 0
```

### 문자형 enum

```typescript
enum Player {
  Koke = '코케',
  Saul = '사울'
}

const player = Player.Koke;	// 코케
```



## 7. Class

### Class에서의 타입 선언

```typescript
class Tiger {
  // constructor 위에 선언
  private name: string;
  public age: number;
  readonly log: string;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```



## 8. 제네릭(generics)

`제네릭`을 활용하면 인자를 넘겨 호출하는 시점에 타입을 결정할 수 있다. 제네릭을 활용하면 동일한 기능을 하는 함수를 일일이 만들 필요가 없으며, 타입 추론에 있어서 강점을 가진다.

### 제네릭 선언

`<T>`와 같이 타입을 선언한다. 알파벳은 통상 `T`로 정해져 있다.

```typescript
function logText<T>(text: T): T {
  consol.log(text);
  return text;
}

logText<string>('Hello My World!');
```

### interface에 제네릭 선언

```typescript
interface Dropdown<T> {
	value: T;
  selected: boolean;
}

const obj: Dropdown<string> = { value: 'hamburger', selected: true };
```

### 제네릭 타입 제한

#### 1. 배열 힌트

```typescript
function logTextLength<T>(text: T[]): T[] {
    console.log(text.length);
    text.forEach(text => {
        console.log(text);
    });
    return text;
}

logTextLength<string>(['hi', 'hello']);
```

#### 2. 정의된 타입 이용(extends)

```typescript
interface LengthType {
    length: number;
}

function logTextLen<T extends LengthType>(text: T): T {
    console.log(text.length);
    return text;
}

logTextLen('abc');
// logTextLen(100); // 숫자에 length property가 없으므로 에러
logTextLen({ length: 100 });
```

#### 3. keyof

interface에 정의된 key 값만을 허용

```typescript
interface ShoppingItem {
    name: string;
    price: number;
    stock: number;
}

function getShoppingItemOption<T extends keyof ShoppingItem>(itemOption: T): T {
    return itemOption;
}

// 'name', 'price', 'stock'만 인자로 가능
getShoppingItemOption('price');
```



## 9. 타입 추론(type inference)

### 기본 변수 타입 추론

```typescript
// string으로 추론
let a = 'abc';	

// a: number로 추론
// b: string으로 추론
// return value는 string으로 추론
function getValue(a = 10) {
  let b = 'hello';
  return a + b;
}
```

### interface 추론

```typescript
// interface 1개
interface Dropdown<T> {
    value: T;
    title: string;
}
const shoppingItem: Dropdown<number> = {
    value: 10000,
    title: 'shoe'
}

// interface 2개
interface Dropdown2<T> {
    value: T;
    title: string;
}
interface DetailedDropdown<K> extends Dropdown2<K> {
    tag: K;
    desc: string;    
}
const detailed: DetailedDropdown<string> = {
    value: '10000',
    title: 'shoe',
    tag: '10000',
    desc: 'description'
}
```



## 10. 타입 단언(type assertion)

`as` 키워드를 사용해 타입을 정함으로써 typescript에게 타입을 알려줄 수 있다. 주로 DOM API를 조작할 때 사용한다고 한다.

### 타입 단언 예제

```typescript
// div가 있는지 장담할 수 없음, HTMLDivElement | null
// 따라써 typescript에게 타입 단언해 타입을 알려 줄 수 있다.
const div = document.querySelector('div') as HTMLDivElement;
div.innerText = 'test';
```



## 11. 타입 가드(type guard)

`Union Type`을 사용하는 경우 공통된 속성만 접근이 되므로, 로직상 공통되지 않은 속성에 접근하고자 할 때 불편함이 따를 수 있다. 타입 단언으로 공통되지 않은 속성에 접근하고자 하는 방법이 있지만, 코드가 지저분해지므로 타입 가드 방법을 사용한다.

### 타입 가드 예제

```typescript
function isDeveloper(target: Developer | Humanoid): target is Developer {
    return (target as Developer).skill !== undefined;
}

if(isDeveloper(tom)) {
    console.log(tom.name);
    console.log(tom.skill);
} else {
    console.log(tom.name);
    console.log(tom.age)
}
```



## 12. 타입 호환(type compatibility)

typescript에서는 더 큰 타입 구조를 갖는 변수에 작은 타입 구조를 갖는 변수를 할당할 수 있다. 예제를 보자.

### 타입 호환 예제

```typescript
let add = function(a: number) {
    // ...
}
let sum = function(a: number, b: number) {
    // ...
}
// 아래는 에러가 난다
// add = sum;

// 에러가 나지 않는다. sum의 구조가 더 크다고 볼 수 있으므로.
sum = add;
```

