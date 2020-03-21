---
title: "ECMAscript6(ES6, ECMAscript2015)에 대해서 알아보자"
excerpt: "ECMAscript6 문법"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - javascript
tags:
  - javascript
  - ECMA
  - ES6
---

`ECMAscript`는 줄여서 `ES`라고 불립니다. 여기서 `ECMA는` European Computer Manufactures Accosiation의 약자로서 정보 통신 시스템을 위한 국제적인 표준화 기구입니다. 현재는 `Ecma 인터내셔널`로 이름을 바꾸었습니다.



ES는 Ecma 인터내셔널이 만든 기술 규격인 `ECMA-262`에서 정의한 표준화된 스크립트 언어입니다. 스크립트 언어의 대표격인 `javascript`언어가 여기에 포함돼 있습니다.



## 1. ECMAscript6(ES6)

ECMAscript6에서 6은 6th Edition임을 의미한다. ECMAscript 2015라고도 불리우는데 단순히 2015년도에 발표했기 때문입니다. ES6기존 ES5와 다른 점은 let, const의 사용, 화살표를 이용한 함수 표현 등이 있습니다. 

 `javascript` 예제를 기준으로 한번 살펴보도록 하겠습니다.



### 1. let

ES6 이전에는 재선언과 재할당이 가능한 `var` 을 사용해 변수를 정의했었습니다. `함수스코프`를 갖는 `var`와 다른  `let`의 특징은 `let`의 경우에는 `재선언`이 불가능하며, `블록스코프`를 갖는다는 점입니다.

`블록스코프`는 아래 코드처럼 말 그대로 변수의 유효범위가 `{ }` 괄호 안에서 유지된다는 의미입니다.

```
var num = 100;
// num is 100
{
	let num = 1000;
    // num is 1000
}
// num is 100
```



예제를 보면서 `let`을 이해해 보도록 하겠습니다. 

각각의 함수 내에  `var` 형식의 `myVar` 변수와 `let` 형식의 `myLet` 변수를 선언한 뒤 `블록스코프` 내부에서 변수를 재할당 한 후 블록 바깥에서 그 값을 확인해 보도록 하겠습니다.

```
let varFunction = function() {
	var myVar = 0
	if (true) {
		var myVar = 1
		console.log(myVar) // 1
	}
	console.log(myVar)     // 1
}
```

```
let LetFunction = function() {
	let myLet = 0
	if (true) {
		let myLet = 1
		console.log(myLet)  // 1
	}
	console.log(myLet)      // 0
}
```

위의 코드는 각각 `var`와 `let` 형식을 활용해 블록 내부에서 변수를 재할당 하였습니다. 

함수 스코프를 갖는 `var`의 경우에는 블록 내부에서 변수를 `재할당`한 것이 함수 전체에 영향을 끼쳐서 둘 다 `1`을 출력했습니다. 반면, `let`의 경우에는 블록 내부에서만 `재할당`이 영향을 끼쳐서 두 번째 출력에는 기존 할당 값인 `0`을 출력하였습니다.



### 2. const

`const`는 ES6에서 `let`과 함께 등장한 변수 형식입니다. `let`과 마찬가지로 `블록스코프`를 가지며 `let`과의 차이점은 변수 `재할당`과 `재선언`이 모두 불가능하다는 점입니다. 

아래 코드처럼 변수에 새로운 값을 재할당하면 `Uncaught TypeError`가 발생하게 됩니다.

```
const num = 100
num = 1000 		// Uncaught TypeError 오류 발생
```



### 3. Arrow 함수

`=>` 기호를 사용해서 함수를 표현할 수 있게 되었습니다. 기존 함수를 정의할 때는 `function`과 `return` 키워드를 사용했지만, 간단한 함수의 경우에는 `=>`를 사용해서 간략하게 함수를 정의할 수 있습니다.

```
// ES5
var sum = function(x, y) {
	return x + y
}

// ES6
const sum = (x, y) => x + y
```

`(x, y)`와 같이 괄호를 사용해서 함수의 인자를 표현하고 `=>` 화살표 다음 그 `return` 값을 표현할 수 있습니다.

arrow 함수는 `var`보다는 `const` 를 사용하는 게 더 나은 방법이라고 합니다. 그 이유는 함수 표현이 항상 상수(const) 값을 나타내기 때문이라고 합니다. arrow 함수에서는 `this` 인자를 사용할 수 없습니다.



### 4. Class

객체를 표현할 수 있는 `class` 키워드가 등장했습니다. 예제를 먼저 보겠습니다.

```
class Friend {
	constructor(nickname) {
		this.nickname = nickname
	}
}
myFriend = new Friend("babo")
```

클래스 내부에 `constructor` 메소드를 통해 클래스의 property를 할당할 수 있습니다. 정의된 클래스를 `new` 인자를 통해서 객체를 생성할 수 있습니다.



### 5. Array의 find 메소드

`python`의 `list`와 유사한 `javascript`의 `array`의 메소드에 `find`가 생겼습니다. `find`는 함수를 인자로 받아 `array`가 가지는 값을 순서대로 판별합니다. 함수의 기준에 맞는 첫번째 array 값이 반환됩니다. 

```
function myFunction(value, index, array) {
	return value < 3
}

var numbers = [5, 7, 3, 2, 1]
var underThree = numbers.find(myFunction)
console.log(underThree)  // 2
```

`myFunction` 은 3보다 작은 값을 반환하는 함수이므로 `find`는 `number`에 있는 element 중 3보다 작은 2를 반환하게 됩니다. (1도 해당되지만 2가 먼저 등장하기 때문에 2를 반환합니다.)

그 외에 `Array.findIndex()`, `Number` 객체의 다양한 property들이 등장하였습니다.