---
title: "JavaScript ES6+ (3) Template literal"
excerpt: "javascript ES6 템플릿 리터럴"
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



## 1. template literal 소개

기존 작은 따옴표' '로 표현한 string의 한계를 보완하고자 `backtick`(``)을 활용한 template literal이 등장했습니다. 기존에는 기다란 문자열을 표현하려면...
```javascript
const num1 = 4;
const num2 = 6;
console.log(num1 + ' + ' + num2 + ' = ' num1 + num2);
```
와 같이 변수와 수식, 문자열을 넘나드는 곡예를 펼쳐야 했었다.
하지만, template literal의 등장으로 한결 더 쉽게 문자열을 표현할 수 있게 되었다. 위 예시를 template literal로 표현하면 아래와 같다.
```javascript
console.log(`${num1} + ${num2} = ${num1 + num2}`);
```
$표시와 중괄호 { 의 등장으로 문자열이 더 복잡하게 표현된 듯 보이지만 더 명확하게 문자열을 표현할 수 있게 되었다. 사이의 값은 띄어쓰기와 줄바꿈이 모두 적용돼 표현된다. ${}의 조합으로 값(변수)을 표현할 수 있는데, 중괄호 사이에 원하는 값을 넣으면 된다. 이를 `String interpolation`이라고 한다!

## 2. template literal 상세
### 1. multi-line인 경우 들여쓰기를 주의한다.
들여쓰기, 띄어쓰기가 모양 그대로 출력된다. 오메나😳
```javascript
const str = `abc
	def
	ghi`;
console.log(str);

// abc
// 		def
//		ghi
```

### 2. ${} 안에 내용은 자동으로 string 처리된다.
```javascript
console.log(`${[1, 2, 3, 4]}`);		// 1,2,3,4
console.log(`${{num1: 1, num2: 2}}`);	// [object, object]
console.log(`${function(){return 10;}}`);// function(){return 10;}
```

### 3. 중첩된 backtick을 사용할 수 있다.
```javascript
console.log(`Foo ${`Bar`}`);		// Foo Bar
console.log(`Foo ${`Bar ${`Baz`}`}`);	// Foo Bar Baz
```

### 4. trim 처리가 가능하다.
```javascript
function html() {
	return `
<div>
	<h1>Title</h1>
</div>
	`.trim();
}
console.log(html());

// <div>
//	<h1>Title</h1>
// <div>
```
이를 활용하여 배열 정보를 가지고 HTML 코드를 만드는 데 활용하는 등 다재다능하게 쓸 수 있다.

## 3. 번외) forEach, map, reduce
### 1. forEach
[]안에 있는 인자는 option으로서 있어도 되고 없어도 된다. 바깥에 있으면 필수인자👌. for문과 동일한 메소드.

`Array.protoType.forEach(callback[, thisArg])`
- callback: function(currentValue[, index[, originalArray]])
	
    -	currentValue: 현재 값
    -	index: 현재 인덱스
    -	originalArray: 원본 배열
    
-	thisArg: this에 할당할 대상. 생략하면 global 객체가 this에!
```javascript
const nums = [10, 20, 30];
nums.forEach(function(value, index, arr) {
  console.log(value, index, arr, this);
}, [100, 200, 300]);

//	10, 0, [10, 20, 30], [100, 200, 300]
// 	20, 1, [10, 20, 30], [100, 200, 300]
//	30, 2, [10, 20, 30], [100, 200, 300]
```

### 2. map

`Array.protoType.map(callback[, thisArg])`
- forEach와 구조가 동일.
```javascript
const nums = [10, 20, 30];
const addedNums = nums.map(function(value, index, arr) {
  return value + this[index];
}, [5, 10, 15]);

console.log(addedNums);

//	[15, 30, 45]
```

### 3. reduce
`Array.protoType.reduce(callback[, initialValue])`
- initialValue: 초기값. 생략시 첫번째 인자가 자동으로 입력되고 두번째 인자부터 작동한다.
- callback: function(accumulator, currentValue[, currentIndex[, originalArray]])

	
    - accumulator: 누적 계산 값
    - currentValue: 현재 값
    - currentIndex: 현재 인덱스
    - originalArray: 원본 배열
```javascript
const nums = [10, 20, 30];
const res = nums.reduce(function(acc, value, index) {
  console.log(acc, value, index);
  return acc + value;
}, 100);

console.log(res);

// 100, 10, 0
// 110, 20, 1
// 130, 30, 2
// 160
````
위의 예제에서 initialValue에 해당하는 100이 제거되면 nums의 첫번째 원소인 10이 initialValue가 되고, 두번째 원소부터 reduce가 적용돼 반복은 총 2회가 된다!

## 3. tag function
tag function은 함수 인자로 template literal을 넘겨주어 첫 번째 인자에는 순수 문자열을 array로 넘겨주고 두 번째 인자부터는 template literal의 string interpolation을 넘겨준다. 예제를 보자.
```javascript
const tag = function(strs, arg1, arg2) {
  return {strs: strs, args: [arg1, arg2]};
};
const res = tag `내일은 ${1}오늘은 ${2}`;
console.log(res);

// args: Array(2)
//	0: 1
//	1: 2
// strs: Array(3)
//	0: "내일은 "
// 	1: "오늘은 "
//	2: ""
//	raw: ["내일은 ", "오늘은 ", ""]	아래 String.raw에서 살펴 볼 예정
//	...
```
특이한 점은 항상 string interpolation의 개수(위에서 args) 보다 항상 문자열이 1개(위에서 strs) 더 많다. strs의 마지막 값으로 ""이 도출된 것을 볼 수 있다. 또한 tag function은 괄호 없이 호출해야 함을 유의한다. 

### 활용 예시
```javascript
const addApple = (strs, ...exps) => {
  return strs.reduce(function(acc, curr, i) {
	let res = acc + curr + '_Apple';
    if(exps[i]) res += exps[i];
  	return res;
  }, '');
}
console.log(addApple `이제 어딘가에${'사과'}가 ${'사과의 의미'}로 추가됩니다.`);

// 이제 어딘가에_Apple사과가 _Apple사과의 의미로 추가됩니다._Apple
```
특정 문자열을 추가하고자 싶을 때 위와 같은 예제로 `_Apple` 문자열을 추가할 수 있다. string interpolation을 제외하고 문자열에 `_Apple`이 추가되었다😂
비슷하게 문자열에 있는 숫자에 comma를 넣는 작업도 수행할 수 있다.

## 4. String.raw
위의 tag function 예시에서 도출된 값 중에 `raw`라는 값이 있었다. `raw`는 말 그대로 날 것의 값을 갖고 있다. 예를 들어, `console.log('hello\n')`을 실행하면 `hello`를 출력하고 줄바꿈을 처리하겠지만, `raw`에는 `hello\n` 형태로 값을 갖고 있다.
`String.raw`는 String의 내장 함수로서 tag function 형태로 호출할 수 있다.
```javascript
const res = String.raw `안녕 너 내 ${'포켓몬'}이 되어라\n\n`;
console.log(res);
//	안녕 너 내 포켓몬이 되어라\n\n
```

마치겠다😉