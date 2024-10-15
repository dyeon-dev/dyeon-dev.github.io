---
title: Git Convention
author: cotes
date: 2024-10-15 11:33:00 +0800
categories: [Blogging, Git]
tags: [commit message, git]
math: true
mermaid: true
---

우테코 프리코스 7기의 1주차 학습목표이다.
![image](https://github.com/user-attachments/assets/ce07ff2d-3c31-4a22-938f-6f70640e2ec4)

이번 기회에 참여하면서 Git의 커밋 단위로  [AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)을 참고하여 커밋 메세지를 작성해보고 정리해보았다.

> 목차
>
> - 목표
> - CHANERLOG.md 생성
>   - 중요하지 않은 커밋 식별
>   - 히스토리 탐색 시 더 많은 정보 제공
> - 커밋 메시지 형식
>   - 메시지 헤더 (header)
>     - `<type>`에 들어갈 수 있는 항목들
>     - `<scope>`에 들어갈 수 있는 항목들
>     - `<subject>` 텍스트
>   - 메시지 본문 (body)
>   - 메시지 바닥글 (footer)
>     - 중요한 변경 사항 (Breaking Changes)
>     - 해결된 이슈 (Referencing Issues)
>   - 예시

## 목표

해당 글에서 말하는 Git Convention의 목표는 다음과 같다.

- **기록을 탐색할 때 더 나은 정보를 제공**할 수 있다.
- **git bisect**로 커밋을 무시하도록 허용(포맷팅과 같은 중요한 커밋이 아님) 가능
  - git bisect란 커밋의 특정 범위 내에서 이진탐색을 통해 문제가 발생한 최초의 커밋을 찾는데 도움을 주는 git의 기능

## CHANERLOG.md 생성

변경 내역엔 다음 세가지 내용을 포함한다.

1. 새로운 특징 (new features)
2. 버그 수정 (bug fixes)
3. 주요 변경 내용 (breaking changes)

배포 할 때 해당 스크립트를 사용해서 관련 커밋에 대한 링크와 함께 위 내용들을 생성할 수 있다.
물론, 실제로 배포하기 전에 변경 내역을 수정하고 배포할 수도 있다.

- 최근 배포 이후의 모든 제목(커밋 메세지의 첫 줄) 목록을 출력
    ```
    제목(subject) : 커밋 메시지의 첫번째 줄

    git log <last tag> HEAD --pretty=format:%s
    ```
- 이번 릴리즈의 새로운 기능
    ```
    git log <last release> HEAD --grep feature
     ```

### 중요하지 않은 커밋 식별

git bisect를 사용하여 이진 탐색할 때
다음과 같이 중요하지 않은 커밋을 무시할 수 있다.

- 서식 변경(공백/빈 줄 추가/제거, 들여쓰기)
- 세미콜론 누락
- 주석 등

```
git bisect skip $(git rev-list --grep irrelevant <good place> HEAD)
```

### 히스토리 탐색 시 더 많은 정보 제공

일종의 "컨텍스트" 정보를 추가하면 좋다.

다음 메시지들을 보자 :

- fix comment stripping
- fixing broken links
- Bit of refactoring
- Check whether links do exist and throw exception
- Fix sitemap include (to work on case sensitive linux)

이 메시지만 보고는 어느 부분이 변했는지 정확히 알 수 없다.
따라서 docs, docs-parser, compiler, senario-runner와 같이 어디가 변경됐는지 알려주는게 좋을 것이다.

물론, 변경된 파일들을 일일히 찾아보면 알 수 있겠지만... 그건 느릴 것다.
그리고 git history들을 보면 어디가 변했는지 명시하려고 하는건 보이지만, 단지 규약이 없을 뿐이다.

## 💬 커밋 메시지의 형식

커밋 메시지의 각 줄은 100자를 넘기지 말아야 한다.
```
<type>(<scope>): <short summary>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

### 메세지 헤더 (header)

```
<type>(<scope>): <short summary>
  │       │             │
  │       │             └─⫸ 명령문, 현재 시제로 작성합니다. 대문자를 사용하지 않으며, 마침표로 끝내지 않습니다.
  │       │
  │       └─⫸ Commit Scope: animations|bazel|benchpress|common|compiler|compiler-cli|core|
  │                          elements|forms|http|language-service|localize|platform-browser|
  │                          platform-browser-dynamic|platform-server|router|service-worker|
  │                          upgrade|zone.js|packaging|changelog|docs-infra|migrations|
  │                          devtools
  │
  └─⫸ Commit Type: build|ci|docs|feat|fix|perf|refactor|test
```

#### `<type>` 에 들어갈 수 있는 항목들

- feat : 새로운 기능 추가
- fix : 버그 수정
- docs : 문서 관련
- style : 스타일 변경 (포매팅 수정, 들여쓰기 추가 등)
- refactor : 코드 리팩토링
- test : 테스트 관련 코드
- build : 빌드 관련 파일 수정
- ci : CI 설정 파일 수정
- perf : 성능 개선
- chore : 그 외 자잘한 수정

 새로운 Angular 9 규약에서는 chore가 삭제되고, build, ci, perf가 추가되었다.

#### `<scope>`에 들어갈 수 있는 항목들

커밋 변경의 위치를 ​​지정하는 것이면 무엇이든 된다. scope는 생략이 가능하다.

ex) $location, $browser, $compile, $rootScope, ngHref, ngClick, ngView etc...

함수 이름이나 메소드 이름을 넣어주면 어디가 변경됐는지 쉽게 알 수 있을 것이다.

#### `<subject>` 텍스트

- 명령형, 현재형을 사용한다.
- 첫 글자를 대문자로 않는다.
- 끝에 점(.)이 없다.

ex) “change”이지 “changed”도 아니고 “changes”도 아님

### 메시지 본문 (body)

- 명령형, 현재형을 사용한다.
- 변경 이유를 포함하고 이전과 차이점을 설명한 문서
  - <http://365git.tumblr.com/post/3308646748/writing-git-commit-messages> <http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html>

### 메시지 바닥글 (footer)

#### 주요 변경 내역들 (Breaking Changes)

모든 주요 변경 내역들은 다음과 함께 하단에 언급되어야 한다.

- 변경점 (description of the change)
- 변경 사유 (justification)
- 마이그레이션 지시 (migration instructions)

#### 해결된 이슈 (Referencing Issues)

해결된 이슈는 커밋 메시지 하단에 Closes #<이슈번호> 와 같이 기록되어야 한다.

`Closes #234`

해결된 이슈가 여러개인 경우는 다음과 같이 쓸 수 있다.

`Closes #123, #245, #992`

> Angular 9 규약에서는 "Fixes" 키워드를 사용하기도 한다.

```
BREAKING CHANGE: <주요 변경 내역 요약>
<BLANK LINE>
<변경점 + 마이그레이션 지시>
<BLANK LINE>
<BLANK LINE>
Fixes #<이슈번호>
```

### 예시

<details>
<summary>예시</summary>
<div markdown="1">

```
feat($browser): onUrlChange event (popstate/hashchange/polling)
Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available
Breaks $browser.onHashChange, which was removed (use onUrlChange instead)
```

```
fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```

```
feat(directive): ng:disabled, ng:checked, ng:multiple, ng:readonly, ng:selected

New directives for proper binding these attributes in older browsers (IE).
Added coresponding description, live examples and e2e tests.

Closes #351
```

```
style($location): add couple of missing semi colons
```

```
docs(guide): updated fixed docs from Google Docs

Couple of typos fixed:
- indentation
- batchLogbatchLog -> batchLog
- start periodic checking
- missing brace
```

```
feat($compile): simplify isolate scope bindings

Changed the isolate scope binding options to:
  - @attr - attribute binding (including interpolation)
  - =model - by-directional model binding
  - &expr - expression execution binding

This change simplifies the terminology as well as
number of choices available to the developer. It
also supports local name aliasing from the parent.

BREAKING CHANGE: isolate scope bindings definition has changed and
the inject option for the directive controller injection was removed.

To migrate the code follow the example below:

Before:

scope: {
  myAttr: 'attribute',
  myBind: 'bind',
  myExpression: 'expression',
  myEval: 'evaluate',
  myAccessor: 'accessor'
}

After:

scope: {
  myAttr: '@',
  myBind: '@',
  myExpression: '&',
  // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
  myAccessor: '=' // in directive's template change myAccessor() to myAccessor
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```
</div>
</details>

---

그동안은 여러 블로그 글들을 참고하며 다른 컨벤션 방식들을 짬뽕해서 사용하거나 header, body, footer 내용을 상세하게 알고 사용하지 않았다.

이번 기회에 확실히 커밋 메세지 작성법을 정리하면서
배포, 커밋의 중요도에 따른 작성법과 관련된 내용도 알게 되었고, 기존보다 상세한 컨벤션 방법까지 알게되어서 좋았다! 😄
