---
title: "프론트엔드 Husky 사용해서 commit 단위로 Lint 검사"
description: "프론트엔드 Husky 사용해서 commit 단위로 Lint 검사"
date: 2023-07-24
update: 2023-07-24
tags:
    - frontend
    - test
---

> 해당 글은 우아한테크코스 5기 팀바팀 크루 [루루](https://github.com/hafnium1923)가 작성했습니다!

# Husky 사용해서 commit 단위로 Lint 검사(Git Desktop 오류코드 127)

## Husky란?

husky는 commit 또는 push 할 때 lint에 맞는지 확인하고, 테스트를 실행하는 것에 사용할 수 있다.

팀마다 컨벤션이 다양할 텐데 인간이 코드를 작성하는지라 어쩔 수 없이 컨벤션에 맞지않게 코드를 작성할 수도 있다.

그럴때 husky를 사용하면 자동으로 검사해서 commit이든 push든 막아준다.

팀바팀은 husky로 lint검사만 진행하는 것으로 결정했다.

1. lint 검사는 commit 단위로 진행하는 것이 더 많은 수정을 막는다

2. test를 commit단위로 진행하는 것은 필요없는 commit이 더 많기도 하고 전체 파일에서의 통과 여부가 더 중요하기 때문에 PR 단위로 진행한다.

라는 이유였다!

### Husky 설치하기

```
npm i -d husky
```

명령어 넣기

```
//package.json
"scripts": {
  ...
    "prepare": "cd .. && husky install frontend/.husky && cd frontend"
  },
```

우리는 하나의 레포에서 각각 백엔드 프론트엔드 파일을 만들어서 사용하고 있기 때문에 frontend 폴더로 이동하는 것도 추가했다

### lint-staged 설치하기

lint-staged란 문법 오류나 스타일 오류를 분석하고 표시, 수정하는 도구이다.

```
npm i -d lint-staged
```

설치했으면 설정도 해주자

```
//package.json
"lint-staged": {
    "*.{ts,tsx}": [
      "eslint --cache"
    ],
    "*.{ts,tsx,md,json}": [
      "prettier --write"
    ]
  }
```

각각 확장자로 끝나는 파일들을 어떤 거로 검사할건지 정해주는 거다.

+) --cache는 캐싱해주는 옵션으로 한 파일만 고쳤는데 나머지 코드들이 의미없이 포맷팅되는 상황을 막아준다. 즉, 전과 후를 비교해 비교를 최소화해준다.

### pre-commit 파일

```
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

cd frontend
npx lint-staged
```

마찬가지로 프론트엔드 폴더로 이동 후 커맨드를 실행시키게 하자

추가적으로 깃 데스크탑을 사용하면 조금 바뀐다.

## Git Desktop에서 husky 오류

내가 깃 데탑을 써서 그런가, 아님 윈도우라 그런가, 아님 그 둘다라서 husky 억까를 당했다. storybook과 npm배포에 이어 또 억까.. 그냥 한달에 한번 환경억까당한다고 생각해야 마음 편할듯.

커밋하려고 하니까 이런 에러가 떴다. (아래 가린건 신경안써도 된다) 근데 VScode에서는 commit이 아주아주 잘된다.

![https://blog.kakaocdn.net/dn/beyYO1/btsn6HDjKtq/u1sDGEqQV5vFj1lgkfzlkk/img.png](https://blog.kakaocdn.net/dn/beyYO1/btsn6HDjKtq/u1sDGEqQV5vFj1lgkfzlkk/img.png)

이거 두개 이슈 보면서 환경변수 추가했는데 안됐다.(추가하라는거 다 한듯?)

[Code 127 error · Issue #1038 · typicode/husky
Troubleshoot [ x ] Before creating an issue, please check: https://typicode.github.io/husky/#/?id=troubleshoot Context We had husky 7.0.1 working fine in our repo. But a few days ago it stopped wor...
github.com](https://github.com/typicode/husky/issues/1038)

[Win 10 Github Desktop pre-commit hook · Issue #1072 · typicode/husky
Hello, I have a problem with my pre-commit hook or husky in general when using Github Desktop. On the first look, it seems to be #1038, but the problem is. It works perfectly fine in VS code. My en...
github.com](https://github.com/typicode/husky/issues/1072)

### 해결방법

진짜 간단하다.

.husky\pre-commit 에서 이렇게 하면 된다.

```
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

cd frontend
npx.cmd lint-staged
```

npx  -> npx.cmd

단, 이렇게 하면 맥에서는 아예 작동을 하지 않는다ㅋㅋ.. 그래서 나는 일단 브랜치에서 작업할 때마다 .cmd를 붙여주고있다.

다른 방법 알면 알려주세요...

끋!