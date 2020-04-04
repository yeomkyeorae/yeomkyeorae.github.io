---
title: "React 기초 개념(1)"
excerpt: "component, props, state"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - react
tags:
  - react
---

해당 게시물은 [Nomad Coders의 ReactJS로 웹 서비스 만들기 과정](https://academy.nomadcoders.co/courses/enrolled/216871)을 수행해 정리한 것입니다. 

## 1. Introduction

순서대로 설치해 줍시다.

[Node.js 설치](https://nodejs.org/en/)

`npx` 설치

```bash
$ npm install npx -g
```

설치한 `npx`를 이용해서 `react` app을 만들어 보겠습니다. 

```bash
$ npx create-react-app movie_app
```

`npx create-react-app "App이름"`을 입력하면 쉽게 `react` app을 생성할 수 있습니다. [create-react-app](https://github.com/facebook/create-react-app)을  

사용하지 않으면 `webpack`과 `babel`을 설치해야 된다고 합니다.

```bash
$ cd movie_app
$ npm start
```

생성한 react app 폴더로 들어가서 `npm start`를 하면 react가 실행되는 것을 확인할 수 있습니다.



## 2. JSX & Props

### 1. JSX 기본 개념 및 Component 생성

`JSX(JavaScript XML)` 은 React에서 나온 유일한 개념입니다. `JSX`는 자바스크립트 코드 안에 HTML가 자리잡고 있습니다. Component는 이 `JSX` 형태를 갖습니다. `Potato`라는 이름의 Component를 추가해 보겠습니다.

```javascript
// Potato.js
import React from "react";

function Potato() {
    return (
        <h3>I love potato</h3>
    )
}

export default Potato;
```

- 모든 Component는 `import React from "react";`로 명시해주어야 Component로 사용할 수 있습니다.
- `return` 은 `HTML` 코드를 반환합니다.

이를 `App.js`에 `import`하여 사용하면 아래 코드와 같습니다.

```javascript
import React from 'react';
import Potato from "./Potato";

function App() {
  return (
    <div className="App">
      <h1>Hello</h1>
      <Potato />
    </div>
  );
}

export default App;
```

- `import Potato from "./Potato"` 하여 Component 생성 준비를 합니다.
- `<Potato />` 형태로 Component를 원하는 위치에서 생성할 수 있습니다. 



### 2. Props

Props를 사용해서 상위 parent Component에서 하위 children Component로 데이터를 보낼 수 있습니다. 이때 사용하는 것이 Props입니다. 편의상 위에 작성했던 `Potato` Component를 import하는 대신 `Food` component로 대체하고 function에 Props를 적용해 보겠습니다.

```javascript
import React from 'react';

function Food(props) {
  return <h1>I like { props.fav }</h1>
}

function App() {
  return (
    <div className="App">
      <h1>Hello</h1>
      <Food fav="kimchi" />
      <Food fav="samgyepsal" />
      <Food fav="chocolate" />
    </div>
  );
}

export default App;
```

- function을 활용해 `Food`라는 이름의 component를 만들고, App component 안에 `<Food fav="음식 이름" />`로 `Food` component를 생성했습니다. 생성할 때 `fav`라는 변수에 "음식이름"을 할당해 전달하면 `Food`는 object 형식의 props 인자로 받아 function 내부에서 활용할 수 있습니다. `ES6` 문법에서는 `{ props.fav }`와 같이 접근합니다.
- 아래와 같은 형식으로 사용할 수도 있습니다.

```javascript
function Food({fav}) {
    return <h1>I like { fav }<h1>
}
```



### 3. 동적 컴포넌트 생성

 데이터가 다르지만 동일한 컴포넌트를 생성할 때 `map`을 이용하여 수 있습니다.

`array`에 `map`을 적용할 경우 원소 하나하나에 함수를 적용해서 return 받을 수 있게 됩니다. 예시는 아래와 같습니다.

```javascript
numbers = [1, 2, 3, 4]
numbers.map(num => {
    return num + 1
})
// [2, 3, 4, 5]
```

`map`을 활용해서 `Food` 컴포넌트를 생성해 보도록 하겠습니다.

```javascript
import React from 'react';

function Food({name, picture}) {
  return <div>
    <h2>I like {name}</h2>  
    <img src={picture} />
  </div>
}

const foodILike = [
  {
    id: 1,
    name: "Kimchi",
    image: "https://cdn.crowdpic.net/detail-thumb/thumb_d_CDC14868821FF3F20C77BC8BC15E6355.jpg"
  },
  {
    id: 2,
    name: "Kimbab",
    image: "https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcS0_5woTu2Kk3_SjZWLva_W_QsxKTL1zNo5TXgpWbtf8h_y2rjH"
  },
  {
    id: 3,
    name: "Samgyeobsal",
    image: "https://pds.joins.com/news/component/htmlphoto_mmdata/201702/27/117f5b49-1d09-4550-8ab7-87c0d82614de.jpg"
  }
]

function App() {
  return (
    <div className="App">
      { foodILike.map(dish => (
      <Food key={dish.id} name={dish.name} picture={dish.image}/>
      ))}
    </div>
  );
}

export default App;
```

- `Food` 컴포넌트에 `props` 적용해 `foodILike`에 있는 데이터를 전달할 수 있습니다.
- 먼저 `HTML`에 `javascript`코드를 활용하기 위해 {}을 생성합니다.
- 그리고 `foodILike`에 `map`을 적용해 내부 함수에 `<Food name={dish.name} picture={dish.image}>`와 같이 코드를 작성하면 `foodILike`에 있는 데이터 순서대로 `props`로 데이터를 전달해 `Food` 컴포넌트가 순서대로 생성되는 것을 볼 수 있습니다.
- Component를 반복적으로 생성할 때 `key`를 props로 넘겨주어 구분짓는 것이 좋습니다.



### 4. PropTypes

Prop으로 전달되는 데이터의 타입, 필수 유무, 이름 등을 확인할 수 있게 도와줍니다.

> 설치

```bash
$ npm i prop-types
```

아래와 같은 형식으로 사용할 수 있습니다.

```javascript
import PropTypes from 'prop-types';

// ...

Food.propTypes = {
  name: PropTypes.string.isRequired,
  picture: PropTypes.string.isRequired,
  rating: PropTypes.number
}
```

`isRequired`를 삭제하여 해당 props가 필수가 아님을 명시해 줄 수 있습니다. `propTypes`와 다른 형식이 접근해오면 console 상에서 오류를 발생시킵니다. 



## 3. State

 State는 보통 우리가 동적 데이터와 함께 작업할 때 만들어집니다. 즉, 현재는 존재하지 않지만 동적으로 변하는 데이터입니다.

 `Function component`는 function이고 무엇인가를 return하고 screen에 표시합니다. 하지만 `class component`는 class이고 react component로 부터 확장되고 screen에 표시됩니다. 또한 render 메소드를 필요로 합니다.

### 1. Why?

class component는 `state`를 갖고 있습니다. `state`는 `object`이고 변하는 data를 가질 수 있습니다. `state`는 `class component`에서 표현하며 다음과 같이 나타낼 수 있습니다.

```javascript
class App extends React.Component{
  state = {
    count: 0
  };
  add = () => {console.log('add')};
  minus = () => {console.log('minus')};
  render() {
    return (
    <div>
      <h1>The number is {this.state.count}</h1>
      <button onClick={this.add}>Add</button>
      <button onClick={this.minus}>Minus</button>
    </div>
    )
  }
}
```

- `state`나 `정의한 함수`를 `render` 내부에서 접근할 때는 `this`를 사용합니다.
- `class component`는 `React.Component`를 `extends`합니다.

### 2. 알아야할 점

```javascript
add = () => { this.setState(current => ({count: current.count + 1}))};
minus = () => { this.setState(current => ({count: current.count - 1}))};
```

- `state`를 `set`하고 직접적으로 `state`를 바꾸려고 하면 안됩니다.
- 대신, `setState`에 함수를 사용합니다.
- 매 순간 `setState`를 호출할 때 마다 react는 `새로운 state`와 함께 `render function`을 호출합니다.

### 3. Component Life Cycle

> Mounting

`render`되기 전에 `constructor()`등 함수를 호출합니다. `componentDidMount()`

> Updating

`render`된 이후 `state`가 업데이트될 때 마다 특정 함수들을 호출합니다. `componentDidUpdate()`