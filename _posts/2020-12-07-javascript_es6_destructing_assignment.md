---
title: "JavaScript ES6+ (8) destructing assignment"
excerpt: "javascript ES6 해체 할당"
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



# Destructing assignment

destructing assignment: 해체 할당
## 1. 배열의 해체 할당

해체 할당은 말 그대로 해체해서 적재적소에 할당하는 것을 말합니다🤣🤣
말해서 뭐합니까. 예제를 보겠습니다.

```javascript
const sports = ['soccer', 'baseball', 'basketball'];
const [sport1, sport2, sport3] = sports;
console.log(sport1, sport2, sport3);
// soccer baseball basketball
```
두 번째 줄에서 `const`로 선언한 변수들에 `sports` 배열에 있는 요소들이 순서대로 할당된 것을 볼 수 있습니다. 쉽죠잉?
comma를 사용해서 원하는 위치의 값을 가져와서 할당할 수도 있습니다.
```javascript
const [ , second] = ['soccer', 'baseball', 'basketball'];
console.log(second);	// baseball
```
### rest parameter
이전에 배운 `rest parameter`와도 연동해서 사용할 수 있습니다.
```javascript
const nums = [10, 20, 30, 40, 50];
const [ , second, ...rest1] = nums;
const [ , , ...rest2] = nums;
```
### default parameter
그리고 `default parameter`도 사용할 수 있습니다.
```javascript
const [a = 10, b = 20] = [50, undefined];
const [num1, num2 = num1 * 2] = [10];
console.log(a, b);		// 50 20
console.log(num1, num2);	// 10 20
```
### 다차원 배열
2차원 이상의 배열에서도 당연히🥕 적용이 됩니다.
```javascript
const arr = [1, [2, [3, 4], 5], 6];
const [a, [b, [, d], ], f] = arr;
console.log(a, b, d, f) 	// 1 2 4 6
```
### 변수 교환
마지막으로 두 변수의 값을 교환할 수도 있습니다!
```javascript
let a = 100;
let b = 200;
[a, b] = [b, a];
console.log(a, b);	// 200 100
```

## 2. 객체의 해체 할당
### 1. 기본 사용법
객체도 배열과 마찬가지로 해체 할당을 할 수 있습니다. 기본적인 사용 방법은 아래 예제와 같습니다.
```javascript
const gaeko = {
  name: '김윤성',
  age: 40,
  gender: 'male'
}

const {
  name: n,
  age: a,
  gender: g
} = gaeko;

const {
  name,
  age,
  gender
} = gaeko;

console.log(n, a, g);	// '김윤성' 40 'male'
```
 기존 객체를 `gaeko`라는 변수명으로 선언하고 이를 대입하고자 할 때, 대입하는 변수명을 `key(name, age, gender)`로 받아 들이고 새로운 변수명을 `value(n, a, g)`로 선언한 것을 볼 수 있습니다. `value`를 사용하지 않고, `key`만 입력하면 `key`와 동일한 이름으로 변수가 만들어집니다.

### 2. 중첩 객체일 경우
객체 내 객체가 존재하는 중첩 객체일 경우를 보겠습니다. 이 경우에는 
```javascript
const dynamicDuo = {
  member: {
    gaeko: '김윤성',
    choiza: '최재호'
  },
  album: {
    first: 'Taxi Driver',
    second: 'Double Dynamite',
    third: 'Enlightened'
  }
}

const {
  member: {
    gaeko,
    choiza
  },
  album: albumInfo,
  album: {
    first: firstAlbum,
    second: secondAlbum
  },
  album: {
    third
  }
} = dynamicDuo;
```
여기서 특이한 점은 `album` 접근자를 여러 번 사용해서 변수를 추출할 수 있다는 점이다.

### 3. default parameter
배열의 해체 할당과 마찬가지로 객체에서도 `default parameter`를 설정할 수 있습니다.
```javascript
const jiSungPark = {
  position: 'midfielder',
  goal: 12
}

const {
  position: place = 'goalkeeper',
  goal: goals = 0,
  age: age = 20
} = jiSungPark
```

### 4. 활용 예제
마지막으로 함수로 활용하는 간단한 예제를 보고 마무리하도록 하겠습니다. 함수 인자로 객체의 해체 할당을 이용하고, default parameter를 부여하였습니다.
```javascript
const getArea = function({ width = 5, height = 4 }) {
  return width * height;
};

console.log(getAreat({ width: 10 }));
```
함수를 이런 식으로 작성하면 혹여나 함수 인자로 width나 height가 오지 않더라도 오류를 피해갈 수 있게 됩니다.

ES6+ 제대로 알아보기 정리를 마무리하도록 하겠습니다😭😭