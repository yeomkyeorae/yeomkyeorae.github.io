---
title: "Heroku로 토이프로젝트를 배포하면서 겪은 문제 해결 과정들"
excerpt: "헤로쿠 토이프로젝트 배포"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - deploy
tags:
  - deploy
  - heroku
---



이 글은 개인 토이프로젝트를 Heroku로 배포하면서 겪은 문제 해결 과정들을 기록한 글입니다.

토이프로젝트는 `푸드마커`라는 이름을 갖고, 맛집 방문 이력 기록, 앞으로 방문할 맛집들을 기록 등을 하는 서비스입니다. 

서버는 `Node.js`와 클라이언트는 `React`로 구현했고, 이를 Heroku로 배포하고자 했습니다.

참고로 아래 해결 방법이 옳은 방법이 아닐 수도 있습니다.



## 문제 1. server 실행시 빌드된 client 파일 실행

webpack을 활용해 빌드된 client 파일인 `index.html`을 어떻게 실행해야할지 검색을 하다 `concurrently` 패키지를 알게 되었다.

```bash
$ npm install concurrenly
```

`concurrently`를 설치하고

```json
{
  "scripts": {
    "start": "concurrently \"npm run dev:server\" \"npm run dev:client\"",
    "dev:server": "nodemon run start",
    "dev:client": "cd client-app && npm run dev"
  }
}
```

`package.json`에 위와 같이 scripts를 작성해 `server`를 구동하고, `client`의 빌드된 파일을 연결시키고자 했다.

그리고 Heroku로 배포 후 url를 접속했을 때 마주한건 Heroku error code=H10 !

(나중에 scripts에 배포용으로 production을 적용해야 할 듯 싶다)



## 문제 2. Heroku error code=H10

stackoverflow에서 검색한 결과 client 쪽 `package.json`의 scripts에 수정을 가해야 할 듯 싶었다.

프로젝트를 create-react-app으로 생성해서 그런지 `"dev": "react-scripts start"`로 적혀 있는데 이것이 react 버전과 호환이 안되는 듯 싶었다.

답변이 제시한 것은 이를 `server -s build`로 변경하라고 조언했다.

```bash
$ npm install serve
```

따라서 `serve` 패키지를 설치하고, 일단 로컬에서 빌드된 파일을 연결시킨 결과.

`node` 서버가 빌드된 클라이언트 파일을 인식하기 시작했다.

하지만 Heroku로 다시 배포해서 보니 왠걸 로컬과 달리 빌드된 파일을 인식하지 못하는 것이 아닌가



## 문제 3. Heroku는 왜 빌드된 파일을 인식하지 못하나

Heroku에 배포할 때 서버 `package.json` scripts에 다음과 같은 명령어를 추가했었다.

```json
{
  "scripts": {
    "...": "...",
    "heroku-postbuild": "cd client-app && npm install && npm run build"
  }
}
```

Heroku 빌드 이후 기재한 명령어를 실행하라는 것인데, 클라이언트 디렉토리 들어가 패키지들을 설치하고 빌드를 진행하라는 것이었다.

그런데 로그들을 보니 패키지들을 모두 설치하고 빌드 명령도 무사히 전달돼 빌드가 진행되기 전에 다음과 같은 문구가 도출되었다.

`heroku one cli for webpack must be installed`  

잉? devDependencies 패키지들이 설치되지 않았나?

그래서 아래와 같은 scripts를 추가했다.

```json
{
  "scripts": {
    "...": "...",
    "heroku-prebuild": "npm install --dev",
    "heroku-postbuild": "cd client-app && npm install && npm run build"
  }
}
```

그리고 다시 배포를 진행했는데, devDependencies 패키지들이 설치된 것처럼 보였지만

```bash
Heroku run bash -a "project name"
```

위 명령어로 Heroku 서버에 접속해 확인한 결과

이상하게 `package.json`에 `webpack-cli`가  존재함에도 Heroku 배포시에만 설치되지 않는 것처럼 보였다.

Heroku 서버에 접속해 `webpack-cli`를 설치하고 성공적으로 빌드해도, 빌드된 파일이 Heroku 서버를 나갔다 들어오면 사라져 있었다

(임의로 접속해 파일을 생성하면 안되나 보다)

최후의 선택으로 `heroku-postbuild` 명령어에 추가적으로 `webpack-cli`를 설치한 결과 배포가 되기는 되었다

```json
  "scripts": {
		"...": "...",
    "heroku-prebuild": "npm install --dev",
    "heroku-postbuild": "cd client-app && npm install && npm install webpack-cli --dev && npm run build"
  },
```

그런데 이번에는 favicon과 배경 이미지가 보이지 않기 시작했다.



## 문제 4. URIError

### favicon 인식

먼저 favicon를 인식할 때 발생한 URIError를 검색한 결과 stackoverflow에서 제시한 해결책은 다음과 같았다

클라이언트 `index.html` 파일에 입력한 `/$PUBLIC_URL%/` 인식하지 못한다는 것이었다.

따라서 webpack 설정 파일에서 `file-loader`에 favicon 파일 형식인 `ico`을 추가하고,

```javascript
{
  test: /\.(jpeg|png|ico)$/,
  loader: "file-loader",
  options: {
    publicPath: "./",
    name: "[name].[ext]"
  }
}
```

플러그인에서 `HtmlWebPackPlugin`에 favicon 경로를 추가해 주었다.

```javascript
plugins: [
  new HtmlWebPackPlugin({
    template: "./public/index.html",
    filename: "index.html",
    favicon: "./public/favicon.ico"
  })
]
```

### 배경 이미지 인식

배경 이미지의 경우에는 `file-loader`에서 설정한 부분과 실제로 생성되는 파일 경로와 파일 이름이 일치하지 않아서 문제였다.

따라서 위 favicon 인식 부분에서 `file-loader` 와 같이 `options`의  `publicPath`를 수정하고, `name` 부분에 지정돼 있었던 `[hash]`를 지어주었다.

(이 부분은 webpack을 더 깊게 이해해서 해결해야 할 거 같다. 임시방편)

그리고 로그인 이후 마주한 에러는 다음과 같았다.

`regeneratorRuntime is not defined` !



## 문제 5. regeneratorRuntime is not defined

검색한 결과 이 에러는 react에서 async/await 비동기 함수를 사용하면 발생하는 것이라고 한다. async/await를 인식할 수 있도록 babel 플러그인을 설치해 주었다.

```bash
$ npm install --save-dev @babel/plugin-transform-runtime
$ npm install --save @babel/runtime
```

설치한 플러그인을 babel 설정 파일에 입력해 주었다.

```javascript
{
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

이후 다행하게도 무사히 위에서 발생한 문제들은 더 이상 나타나지는 않았다. 하지만 위에 나온 개념들에 대한 깊은 이해 없이 배포하고자 해서

임시방편 같은 해결 방법이 아닌가 싶다. 

Heroku, 배포 디렉토리 구성, package.json, webpack, babel 등에 대한 이해도를 증진시켜야겠다.

우선 개인 토이프로젝트를 이용할 수 있도록 배포를 끝까지 진행해볼 예정이다.



다음 과정은 AWS의 S3를 활용해 이미지를 저장할 수 있도록 해보아야 겠다.



## 참고자료

1. [Express + React 연동 및 Heroku에 배포하기](https://velog.io/@recordboy/Express-React-%EC%97%B0%EB%8F%99-%EB%B0%8F-Heroku%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)
2. [React app runs locally, crashes when on Heroku error code=H10](https://stackoverflow.com/questions/44319832/react-app-runs-locally-crashes-when-on-heroku-error-code-h10)
3. [react build한 파일들과 express에 연결하기](https://velog.io/@taelee/react-build%ED%95%9C-%EC%A0%95%EC%A0%81%ED%8C%8C%EC%9D%BC%EB%93%A4-express%EC%97%90-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0)
4. [URIError: Failed to decode param '/%PUBLIC_URL%/favicon.ico'](https://stackoverflow.com/questions/50824024/urierror-failed-to-decode-param-public-url-favicon-ico)
5. [[React] regeneratorRuntime is not defined 에러 해결](https://velog.io/@haebin/React-regeneratorRuntime-is-not-defined-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0)