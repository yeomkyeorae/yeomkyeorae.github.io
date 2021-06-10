---
title: "JavaScript ES6+ 중급(1) Symbol"
excerpt: "javascript ES6 Symbol"
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



## 1. 특성

- primitive value로서 유일무이하고 고유한 존재이다.
- private member에 대한 needs에서 탄생했다.
- 기본적인 열거 대상에서 제외된다. (예: for ... in문)
- 암묵적인 형변환이 불가하다.

## 2. 예제
예제를 통해 위 특성들을 파악해 보자. 먼저 `Symbol`은 `new` 연산자 없이 생성하며( Symbol([string]) ), 문자열이 아닌 타입은 자동으로 `toString` 처리한다.
```javascript
const a = Symbol('hello');
const b = Symbol('hello');
console.log(a == b, a === b);	// false false
```
위에서 변수 a, b에 같은 인자를 넣어 Symbol을 생성했지만, Symbol은 고유한 존재이므로, 두 변수에 담긴 값들은 서로 다른 값이다.

이번에는 Symbol을 형변환 시켜보자. 보통 문자와 숫자를 더하면 문자로 자동으로 형변환되지만, Symbol의 경우에는 에러를 발생시킨다. Symbol과 숫자를 더해도 마찬가지이다.
```javascript
const a = Symbol('hi');
console.log(a + 1);
// Uncaught TypeError: Cannot convert a Symbol value to a number at ...
```

다음에는 object를 통해 Symbole이 열거 대상에서 어떻게 나타나는지 보자. 먼저 예제로 사용할 object를 만들어 보자.
```javascript
const NAME = Symbol('이름');
const AGE = Symbol('나이');
const giraffe = {
	[NAME]: 'girin',
  	[AGE]: 20,
  	height: 300
}
const tiger = {
	[NAME]: 'horang',
  	[AGE]: 5,
  	height: 120
}
const chicken = {
	[NAME]: 'dak',
  	[AGE]: 1,
  	height: 20
}
```
유의할 점은 Symbol 자체를 key에 넣게 되면 해당 value에 접근할 수 없기 때문에 변수를 선언한 뒤 대괄호 표기법을 사용해야 한다. 예를 들어 아래와 같이 object를 만들면 `Symbol('a')`을 알고 있는 변수가 없으므로, value 1에 접근이 안된다(`Reflect.ownKeys`를 사용하면 가능).
```javascript
const obj = {
  [Symbol('a')]: 1
};
```
다시 돌아와서 위의 객체에 대해 반복문을 돌려보면, Symbol 변수가 아닌 height만 값이 출력되는 것을 알 수 있다.
```javascript
for(const key in giraffe) {
  console.log(key, giraffe[key]);
}
// height 300
```
이는 `forEach`에서도 마찬가지이다. Symbol로 된 key에 접근할 수 있는 방법은`Object.getOwnPropertySymbols(giraffe).forEach() ...`와 같은 방식이 있지만, 이는 Symbol이 아닌 key는 제외하게 된다. Symbol 구분을 하지 않고 접근하고자 한다면 `Reflect.ownKeys`를 사용할 수 있다.

## 3. private member
즉시 실행 함수로 된 예제를 먼저 보자.
```javascript
const obj = (() => {
  const _privateMember1 = Symbol('private1');
  const _privateMember2 = Symbol('private2');
  return {
    [_privateMember1]: '외부에서 보이는데 접근이 마땅치 않은 놈',
    [_privateMember2]: '위와 마찬가지인 놈',
    publicMember: '접근이 수월한 놈'
  }
})()

console.log(obj);
console.log(obj[Symbol('private1')]);
//{
//  publicMember: "접근이 수월한 놈", 
//  Symbol(private1): "외부에서 보이는데 접근이 마땅치 않은 놈", 
//  Symbol(private2): "위와 마찬가지인 놈"
//}
// undefined
```
즉시 실행 함수 내부에 선언된 `_privateMember1`와 `_privateMember2`는 블록 스코프에 갇혀서 블록스코프 외부에서는 접근을 할 수 없다. 물론 위에서 소개한 `Object.getOwnPropertySymbols`와 `Reflect.ownKeys`를 사용하면 접근이 가능하다. 정재남님에 따르면 이는 private member를 흉내낸 정도라고 한다.

추후에 private member를 선언하는 방식이 공식적으로 나온다고 하는데, 이미 나와있는지 한번 찾아봐야겠다.🧐

## 3. 공유 심볼 - Symbol.for
Symbol에는 전역 공간에서 public member로서 공유심볼이라 놈이 있다! 이 놈은 어디서 만들든 스코프에 갇히지 않고 공유된다. 선언 방법은 단순히 `Symbol.for([string])`로 변수를 만들면 된다.
```javascript
const a = Symbol.for('hello');
const b = Symbol.for('hello');
console.log(a === b)	// true
```
기존 Symbol과 달리 인자를 같게 하는 Symbol.for 두 변수를 비교했을 때 true가 나오게 된다. 이는 Symbol.for를 관리하는 전역 공간이 존재함을 알 수 있다.
```javascript
const obj = (() => {
  const GREETING = Symbol.for('hi');
  return {
  	[GREETING]: '공유가 됩니다'
  }
})();
console.log(obj[Symbol.for('hi')]); // '공유가 됩니다'
```
또한 기존 Symbol과 달리 Symbol.for에 쓰인 문자열만 알고 있으면 객체 프로퍼티에 접근할 수도 있다.

## 4. 표준 심볼
Symbol에는 개발자가 자기 구미에 맞게 본래 기능을 다르게 구현할 수 있도록 하는 property들을 제공한다. 이를 표준 심볼이라 한다. 그 중에는 `Symbol.iterator`, `Symbol.match`, `Symbol.replace`, `Symbol.split`, `Symbol.toStringTag` 등이 있다. 이 중에서 문자열을 나누는 조건을 설정할 수 있는 `Symbol.split`을 예로 들어 보겠다.
```javascript
const str = '매일 매일 주말이었으면 좋겠어';

console.log(str.split(' '));
// ["매일", "매일", "주말이었으면", "좋겠어"]

String.prototype[Symbol.split] = function(string) {
  let result = '';
  let residue = string;
  let index = 0;
  
  do {
    index = residue.indexOf(this);
    if(index <= -1) {	// this가 없으면 멈춘다
      break;
    }
    result += residue.substr(0, index) + ' 아아 ';
   	residue = residue.substr(index + this.length);
  } while(true)
  result += residue;
  
  return result;
}

console.log(str.split(' '));
// 매일 아아 매일 아아 주말이었으면 아아 좋겠어
```
본래 문자열의 split 메소드는 delimiter를 기준으로 문자열을 구분한 요소들을 array로 return 한다. 하지만 위에서는 `Symbol.split`를 활용해 기존 split 메소드의 작동 방식을 재정의하였다. delimiter 위치에 ` 아아 ` 문자를 넣어 array가 아닌 전체 문자열로 반환하도록 바꿀 수도 있다.

Symbol은 언제 어떻게 써야할지 아직 크게 와닿지 않는다. 나중에 점점 필요한 순간이 오겠지?😳