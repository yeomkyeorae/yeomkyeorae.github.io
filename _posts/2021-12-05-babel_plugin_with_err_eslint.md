---
title: "babel plugin을 적용하면서 발생한 eslint 헤매기"
excerpt: "babel plugin 적용 후 생긴 lint 빨간 줄 없애기"
toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
categories:
  - babel
  - eslint
tags:
  - babel
  - eslint
---



 토이프로젝트를 만들면서 `nullish-coalescing-operator`,` optional-chaining` 문법을 사용하고자 babel plugin을 설치하고, babel 설정 파일에 plugin을 등록하고, 코드 상에서 사용해봤습니다.

문제 없이 동작하는데 VS code 상에서 lint 에러가 빨간색으로 발생하는데 눈을 거슬리게 합니다. 

그래서 팔을 걷어 붙히고 이 빨간 lint 에러를 고쳐보고자 했습니다.



## 1. eslint 설정 파일 생성 및 babel-parser 적용

우선 검색한 결과 `.eslintrc` eslint 설정 파일에 `parser`를 `babel-parser`로 변경해야 한다는 답변이 많아서 그대로 진행해 보았습니다. 

기존에 `package.json`에 설정된 `eslint` 옵션(eslintConfig)을 지우고 새로 `.eslintrc` 파일을 생성했습니다. 

그 전에 babel-parser을 사용하기 위해 `eslint`와 `babel-lint`를 설치하였습니다.

```bash
$ npm install eslint --save-dev
$ npm install babel-eslint --save-dev
```

그리고 eslint 설정 파일을 생성해 내부에 다음과 같이 입력해 주었습니다.

```json
{
    "parser": "babel-eslint",
    "parserOptions": {
        "ecmaVersion": 2020
    }
}
```

`parserOptions`에 `ECMA` 버전도 함께 명시해 주었습니다. 

그리고 실행한 결과는?

## 2. eslint 의존성 문제 발생

`The react-scripts package provided by Create React App requires a dependency: "eslint": "^6.6.0"` 이라는 메시지가 나옵니다. 

혹시 몰라  `package-lock.json` 파일과 기존 설치된 `node_modules` 파일을 삭제해 재설치했지만 어림도 없습니다. 

검색해서 찾아낸 결론은 기 설치된 `react-scripts`가 `3.4.3`버전이고, eslint를 `6.6.0`버전을 지원하는데 위에서 설치한 `eslint`의 버전은 `7.32.0`이라 의존성 문제가 발생한 것이었습니다. 

이에 따라 `react-scripts`를 `4.0`대의 버전으로 업데이트하고, node_module을 재설치 해 다시 실행해 봅니다.



## 3. Definition for rule was not found

이번에는 `Definition for rule 'react-hooks/exhaustive-deps' was not found`라는 메시지가 나옵니다.

stackoverflow의 도움을 구한 결과. `.eslintrc` 파일에 react plugin을 설정해 주지 않은 결과였습니다. 

이를 추가해 주었습니다.

```json
{
    "parser": "babel-eslint",
    "parserOptions": {
        "ecmaVersion": 2020
    },
    "plugins": ["react", "react-hooks"]
}
```

그럼 이번에는 과연?



## 4. Parser load 실패

`.eslintrc` 파일을 저장할 때 마다 VS code 우측 하단부에서 에러 메시지가 올라옵니다.

 `eslint failed to load parser 'babel-eslint' eslint extension`. 

여전히 빨간색의 lint 에러는 사라지지 않고, 유심히 보지 않았던 메시지를 읽어보니 parser가 동작하지 않는 것으로 보입니다. 

아래에는 다음과 같은 문구도 있습니다. 

`createRequire is not a function ...`

저게 뭔 말인지 싶어 검색해 보니 이구동성으로 VS code를 업데이트 하라고 합니다. 

그 동안 얼마나 업데이트를 안했던 거지 싶어하며 VS code 업데이트를 진행하고자 했습니다.



## 5. VS code 업데이트 on Mac

VS code 업데이트를 하고자 위 상단 메뉴에서 `Check for Updates`를 업데이트를 할 수 없다고 합니다. 

에러 메시지를 읽어보면 `Cannot update while running on a read-volume...`  라며 끝부분에서 `you'll need to move the application out of the Downloads directory`라고 친절하게 해결방안을 설명해줍니다. 

아마도 Mac에서는 VS code가 응용프로그램으로 등록돼 있어야 하나 봅니다.

그래서 다운로드 폴더에 방치돼 있던 VS Code를 응용프로그램 폴더로 옮깁니다. 옮긴 이후에 신기하게도 군말 없이 VS code가 업데이트를 받아들입니다. 

그리고 기적적으로 lint 에러의 빨간줄이 사라지는 것을 목격합니다. lint를 적용하는 것은 아직도 어색하고 헷갈리기만 합니다. 오히려 코드를 작성하는 것이 쉬워 보이기도 합니다. 그렇지만 생산성 향상을 위해서 lint를 하나씩 적용해 가는 것도 의미있는 일이라고 생각합니다. 

아직 알아야 할 것은 넘쳐나는 것 같습니다!





## 참고자료

1. [Eslint fails to parse and red-highlights optional chaining (?.) and nullish coalescing (??) operators](https://stackoverflow.com/questions/57378411/eslint-fails-to-parse-and-red-highlights-optional-chaining-and-nullish-coal)
1. [eslint - Optional chaining error with vscode](https://stackoverflow.com/questions/61628947/eslint-optional-chaining-error-with-vscode)
1. [ESLint The react-srcipts package provided by Create React App requires a dependency :: 의존성 문제](https://hini7.tistory.com/144)
1. [Definition for rule 'react-hooks/exhaustive-deps' was not found](https://stackoverflow.com/questions/59611822/definition-for-rule-react-hooks-exhaustive-deps-was-not-found)
1. [ESLint createRequire is not a function Referenced from: ...](https://hini7.tistory.com/142)
1. [mac에 ms viscose 설치하기](https://oneboard.tistory.com/3)
