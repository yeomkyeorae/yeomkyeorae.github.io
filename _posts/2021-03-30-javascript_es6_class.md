---
title: "JavaScript ES6+ 중급(5) Class"
excerpt: "javascript ES6 Class"
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



## 1. Class 이전, 생성자 함수

 ES5에서는 생성자 함수를 사용해서 class를 대신해 사용하였다. 예를 들면 다음과 같다.
 ```javascript
function Person(name) {
  this.name = name;
}

Person();				// 함수여서 바로 호출할 수도 있고
const p = new Person();	// new 연산자를 활용해 인스턴스를 생성할 수도 있다

// prototype에 정의한 method는 prototype method이다
// prototype method는 인스턴스를 통해 호출한다.
Person.prototype.getName = function() {
  return this.name;
}
// 생성자 함수 자체에 정의한 함수는 static method이다
// static method는 인스턴스 없이 생성자 함수로부터 호출한다.
Person.isPerson = function() {	// static method
  return obj instanceof this;
}

const p1 = new Person('겨레');
console.log(p1.getName());
console.log(Person.isPerson(p1));
 ```
위의 예제에서의 생성자 함수 `Person`과 그 인스턴스 `p1`간의 구조를 나타내면 다음과 같다. 
 ```
Person . prototype
			  /
(new)		 /
  |			/
  v		   /
  		  /
  p1  (__proto__, 생략 가능)
  
 p1.__proto__ === Person.prototype
 p1.__proto__.getName() === Person.prototype.getName()
 p1.getName() === Person.prototype.getName()
 ```
 위에서 `p1`은 `__proto__`를 통해 생성자 함수의 prototype에 정의된 `prototype method`에 접근할 수 있지만, `static method`는 접근할 수 없다. 또한 인스턴스 `p1`이 어떤 메소드를 호출할 때 `__proto__`에 정의돼 있지 않으면 대각선으로 올라가서 `prototype`에 있는지 확인한 후 호출한다(`__proto__`를 계속 따라간다).

이제 위에서 만든 생성자 함수를 간단히 `class`로 표현하면 다음과 같다.
```javascript
class Person {
  constructor(name) { 
    this.name = name; 
  }
  
  getName() {	// prototype method
    return this.name;
  }
  
  static isPerson(obj) {	// static method
    return obj instanceof this;
  }
}
```

## 2. Class 개념
1. `class`의 선언은 아래와 같은 방식을 사용할 수 있다.
```javascript
// 클래스 리터럴
class Person1 {

}

// 기명 클래스 표현식
let Person2 = class Person2 {

}

// 익명 클래스 표현식
let Person2 = class {

}
```



2. `class`는 `let, const`와 마찬가지로 TDZ가 존재하며, 블록스코프를 갖는다. 또한 `Class` 내부는 `strict mode`가 강제된다.
```javascript
if(true) {
  class A { }
  const a = new A();
  if(true) {
    const b = new A();	
    class A { }		// let, const와 마찬가지로 호이스팅이 되지만 TDZ에 걸려 reference error
  }
}
const c = new A();	// reference error
```



3. `class` 내부에서 정의한 메소드들은 `prototype`으로 접근했을 때 열거 대상에서 제외된다는 특징을 갖는다.
```javascript
class A {
  a() {}
  b() {}
  static c() {}
}

for(let p in A.prototype) {
  console.log(p);	// 아무것도 출력되지 않는다. prototype에 직접 함수를 정의하면 가능하다.
}
```



4. `class`에서 `new` 명령어로 호출할 수 있는 메소드는 `constructor`가 유일하다. 또한 `class`는 생성자 함수로서 작동해서 `new` 명령어 없이는 호출할 수 없다.

5. `class`를 선언할 때 `constructor`에서 `class`의 변수를 바꾸고자 할 때는 `class`의 변수가 `const`처럼 작동하지만, 인스턴스 생성 후 `class`의 변수를 바꾸고자 할 때는 `let`처럼 작동한다.
```javascript
class C {
  constructor() {
    C = 'C';
  }
}
const c = new C();
// Uncaught TypeError: Assignment to constant variable.

C = '10';
// C가 '10'으로 변경된다
```



6. `class` 자체를 함수의 인자로 넘길 수 있다.
```javascript
const instanceGenerator = (className, ...params) => new className(...params);

class Person {
  constructor(name) { 
    this.name = name; 
  }
  
  printName() { 
    console.log(this.name); 
  }
}

const kr = instanceGenerator(Person, '겨레');
```



7. 대괄호 표기법을 활용해서 `class`의 메소드를 정의할 수 있다. (computed property names)
```javascript
const method = 'printName';
class Person {
  constructor(name) {
    this.name = name;
  }
  
  [method]() {
    console.log(this.name);
  }
}
```



8. `generator`를 `class`의 메소드로 정의할 수 있다.
```javascript
class A {
  *gene() {}
}
```



## 3. 상속
기존에 class 이전에 생성자 함수로 상속을 구현하기 위해서 하위 개념 함수의 prototype에 상위 개념 함수를 new 명령어로 인스턴스를 만들어 대입하는 과정을 활용했다. 하지만 class에서는 단순히 `extends`를 활용하면 가능하다.
```javascript
class Square {
  constructor(width) {
    this.width = width;
  }
  
  getArea() {
    return this.width * (this.height || this.width);
  }
}

class Rectangle extends Square {
  constructor(width, height) {
    super(width);
    this.height = height;
  }
}

const rect = new Rectange(10, 30);
console.log(rect.getArea());	// 300
```
`super()`는 상위 클래스의 constructor를 호출하는 함수이고, constructor 안에서만 호출이 가능하다. 유의할 점은 `super()`는 constructor 내부에서 `this`를 사용하기 이전에 사용해야 한다는 점이다. 위에서 `super(width)`는 `this.width = width`와 같다. `Rectangle`에 사각형 넓이를 구하는 메소드가 없음에도 호출이 가능한 이유는 상위 클래스인 `Square`를 상속받았기 때문이다. `Rectangle`에 메소드가 정의되지 않았기 때문에 `__proto__`를 타고 올라가 `Square`의 메소드에 접근할 수 있게 된다.

class가 상속받을 수 있는 것에는 class뿐만 아니라 생성장 함수도 상속을 받을 수 있다.
```javascript
function Person(name) {
  this.name = name;
}

class Employee extends Person {
  constructor(name, position) {
    super(name);
    this.position = position;
  }
}

const kr = new Employee('겨레', 'Master');
```

class는 기존에 정의된 내장 함수를 상속해서 새롭게 메소드를 재정의(오버라이딩)할 수도 있다.
```javascript
class NewArr extends Array {
  toString() {
    return `[${super.toString()}]`'
  }
}

const arr = new NewArr(100, 200, 300);
console.log(arr.toString());	// [100, 200, 300]
```

메소드 내부에서 사용하는 `super` 키워드는 상위 클래스의 메소드에 접근할 수 있도록 한다. 위 예제에서 `NewArr` 클래스에서 `toString` 사용한 `super.toString`은 `Array`에 정의된 `toString` 메소드이다. 

javascript class에서 추상 클래스가 존재하지 않지만 비슷하게 활용할 수 있다.
```javascript
class Shape {
  constructor() {
    if(new.target === Shape) {
      throw new Error('이 클래스는 추상 클래스입니다.');
    }
  }
  getSize() {}
}
```

`private member`는 추가될 예정이라고 하는데 한번 찾아 봐야겠다. 🧐


 [Class 이해를 위해 읽어보면 좋은 글](https://roy-jung.github.io/161007_is-class-only-a-syntactic-sugar/)