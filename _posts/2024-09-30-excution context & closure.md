---
title: Execution context & Closure
date: 2024-09-25 11:33:00 +0800
categories: [Blogging, 자바스크립트]
tags: [excution context, closure, call stack]
math: true
mermaid: true
---

# Execution context (실행 컨텍스트)
>
> scope, hosting, this, function, closure 등과 같은 JS 동적 언어로서의 동작원리를 담고 있는 핵심 원리

- 브라우저(혹은 Node)는 자바스크립트를 파싱할 때 자바스크립트 엔진은 일련의 과정을 거친다.
- 또한 자바스크립트 엔진은 코드를 파싱하기 위해 실행에 필요한 여러 정보를 알고 있어야 한다.
- **실행 컨텍스트**는 **실행할 코드**에 제공할 **환경 정보**를 담아놓은 객체이다.
  
## **전역 실행 컨텍스트, 전역 스코프 생성**

자바스크립트 엔진은 코드를 파싱할 때 **전역 실행 컨텍스트**와 **전역 스코프**를 만든다.

```jsx
var x = 'xxx';

function foo () {
  var y = 'yyy';

  function bar () {
    var z = 'zzz';
    console.log(x + y + z);
  }

  bar();
}

foo();
```

![image](https://github.com/user-attachments/assets/ea59d469-e597-4354-b529-5c1025cddb88)
위의 코드를 실행하면 실행 컨텍스트에는 **콜스택(Call Stack)** 이 생성되고 소멸한다.

현재 실행중인 컨텍스트에서 함수가 실행되면 새로운 컨텍스트가 생성되고, 함수가 호출됨에 따라 스택이 쌓이게 되고 제어권이 이동한다.

1. 자바 스크립트 코드를 파싱할 때 **전역 실행 컨텍스트**가 생성되고, 실행 컨텍스트 콜스택에 쌓인다.
전역 실행 컨텍스트는 애플리케이션이 종료될 때까지 유지된다.
2. 함수를 호출하면 해당 함수의 실행 컨텍스트가 생성되며 직전에 실행된 코드 블록의 실행 컨텍스트 위에 쌓인다.
사용되는 변수들은 변수 객체 안에서 값을 찾고, 없으면 스코프 체인을 따라 올라가며 찾는다.
3. 함수 실행이 끝나면 해당 함수의 실행 컨텍스트가 소멸하고 (클로저 제외) 직전의 실행 컨텍스트에 제어권을 반환한다.

### Call Stack

함수와 함수 바깥쪽에 있는 코드들이 접근할 수 있는 스코프를 보여준다.

실행되는 순서대로 Call Stack에 하나씩 쌓인다.

(비유하자면, Call Stack의 항목들은 폴더라면, 그 폴더에서 접근 가능할 수 있는 파일들이 Scope이다.)
<img width="820" alt="image" src="https://github.com/user-attachments/assets/340dbd6b-9dcc-4561-85d5-f8f9eb6d73aa">

- **전역 실행 컨텍스트** (Global Excute Context) - 최초로 생기는 (anonymous)
- **함수 실행 컨텍스트** (Function Excute Context) - 함수가 실행되면 call stack에 하나씩 쌓이고, 해당 변수에 대한 **Local 스코프**가 생성된다.

## 실행 컨텍스트 구조

![image](https://github.com/user-attachments/assets/69d23632-8ae9-441a-9e6d-88169dc9d98e)

### 변수 객체 (Variable object)

- 컨텍스트가 생성되면 자바스크립트 엔진은 실행에 필요한 여러 정보들(**변수, 매개변수(parameter)와 인수정보(arguments), 함수** **선언**)을 담을 객체를 내부적으로 생성하는 객체이다.
- 변수 객체는 코드가 실행될 때 엔진에 의해 참조되며 코드에서는 접근할 수 없다.

**전역 컨텍스트**의 경우

- 변수 객체는 유일하며 최상위에 위치하고 있는 모든 전역 변수, 전역 함수 등을 포함하는 **전역 객체**를 가리킨다.

**함수 컨텍스트**의 경우

- 변수 객체는 활성 객체를 가리키며 매개변수와 인수들의 정보를 배열 형태로 담고 있는 객체인 arguments object가 추가된다.

### Scope

- **Local**

    **함수 실행 컨텍스트**에서 실행될 때 함수가 접근할 수 있는 변수가 들어가게 된다.

  - this: Window → 현재 범위의 전역 Window 개체를 가리킴
  - 함수 내의 `var` 변수는 Global이 아닌 **Local 스코프**로 들어오게 된다.
  - 함수 내의 `let`, `const` 변수 또한 **Local 스코프**로 들어오게 된다.
- **Script**

    **전역 실행 컨텍스트**에서 실행될 때 `let`, `const`에 대한 변수가 들어간다.

- **Global**

    **전역 실행 컨텍스트**에서 실행될 때 `아무런 키워드 없이 ****변수`를 작성하거나 `var`는 **Global 스코프**로 들어간다.

    **함수 실행 컨텍스트**에서 실행될 때 함수 내의 `키워드가 없는 변수`는 **Global 스코프**로 들어간다.

    *하지만 키워드 없이 작성한다면 Global에 있는 수많은 라이브러리, 빌트인 사이에 들어가기 때문에 “*암시적 전역 변수의 문제 (변수 누출, 전역 범위 오염) / 빌트인과 충돌*”로 인해 바람직하지 않다.

==> **Local → Script → Global** 순서로 ***스코프 체이닝***이 일어나면서 변수를 찾는다.

여기서 잠시 this에 대해 알아보고 가자.

### this

- this 프로퍼티에는 this 값이 할당되는데, this에 할당되는 값은 **함수 호출 패턴에 의해 결정**된다. (무엇이 함수를 호출했는가에 의해)
- 전역 컨텍스트의 this는 window이다.
- 변수가 선언된 위치가 아니라, **실행 컨텍스트 로컬 범위와 관련**되어 있다.
- Script 스코프에서는 직접 참조할 수 없다.

    키워드 없는 변수 → Global 스코프 - window 객체에 저장되어 window.변수로 접근 가능

    `var` 변수 → Global 스코프 - window 객체에 저장되어 window.변수로 접근 가능

    `let` 변수 → Script 스코프 - Script 스코프에 저장되어 window.변수로 접근 불가능

    `const` 변수 → Script 스코프 - Script 스코프에 저장되어 window.변수로 접근 불가능

### Block Scope

- 함수 실행 컨텍스트에서의 let, const는 함수에서만 동작하는 것이 아닌 블럭 안에서 Local로 들어가는 것이다.

블럭에서 실행할 때

```jsx
function fn1() {
 v0 = 'fn1-v0'
 l0 = 'fn1-l0'
 console.log(v0, l0) // Global: fn1-v0, Global: fn1-l0
}
fn1()
console.log(v0, l0) // Global: fn1-v0, Global: fn1-l0

function fn2() {
 var v0 = 'fn2-v0'
 let l0 = 'fn2-l0'
 console.log(v0, l0) // Local: fn2-v0, Local: fn2-l0
}
fn2()
console.log(v0, l0) // Global: fn1-v0, Global: fn1-l0

{
 var v0 = 'block-v0'
 let l0 = 'block-l0'
 console.log(v0, l0) // Global: block-v0, Block: block-l0
}
console.log(v0, l0) // Global: block-v0, Global: fn1-l0
```

`var` - 블록 안에서 선언한 것과 블록 밖에서 선언한 것과 똑같은 값으로 **Global 스코프**에 들어간다.

`let` - (Script 스코프 이전의) **Block 스코프**에 들어간다.  Block 안에서 Local 한 스코프를 가지는 것이다.

**블록 내에서(ex. for문) let, const로 할당한다면 스코프에 따라서 변수가 실행되기 때문에 변형, 오염을 시키지 않는다.**

|                      | var     | let     | const   |
| -------------------- | ------- | ------- | ------- |
| global scope         | **yes** | **no**  | **no**  |
| script scope         | **no**  | **yes** | **yes** |
| function local scope | **yes** | **yes** | **yes** |
| block scope          | **no**  | **yes** | **yes** |
| 재선언               | **yes** | **no**  | **no**  |
| 재할당               | **yes** | **yes** | **no**  |

### Scope Chain(SC)

> 스코프 체인은 리스트로서 전역 객체와 중첩된 함수의 스코프의 레퍼런스를 차례로 저장하고 있다.
>

자바스크립트 엔진은 스코프 체인을 통해 **렉시컬 스코프를 파악**한다.

함수가 중첩 상태일 때 자식 함수 내에서 **부모 함수의 스코프**와 **전역 스코프까지** 참조할 수 있는데 이는 스코프 체인을 통해 가능하다.

1. 함수가 중첩되어 있으면 중첩이 될 때마다 부모 함수의 스코프가 자식 함수의 스코프 체인에 포함된다.
2. 함수 실행중에 변수를 만나면 그 변수를 우선 현재 스코프(활성 객체)에서 검색해보고, 검색에 실패하면 스코프 체인에 담겨진 순서대로 검색을 이어가게 된다.

---

## Closure

> 스코프 체인을 활용하여 자식 함수를 포함하는 부모 함수의 실행 컨텍스트가 소멸하여도 부모 스코프의 변수에 대한 해당 Lexical Environment에 계속 참조할 수 있도록 메모리에 유지되는 기능
>

자바스크립트는 dynamic scope(동적 스코프)가 아닌 static scope(정적 스코프)를 따른다.

- dynamic scope: 어디에서 호출했느냐에 따라 접근할 수 있는 유효범위가 달라지는 것
- static scope(=lexical scope): 함수를 어디에서 호출했느냐가 아니라, 어디에서 **정의**되었느냐에 따라 함수의 유효 범위가 달라지는 것

```jsx
let l0 = 'l0'

function fn2() {
 let l2 = 'l2'
 console.log(l0, l1, l2) // l1 is not defined
 // l1은 fn1()에 존재하기 때문에 static scope에 따라 호출될 수 없다.
 // 함수마다 독립적으로 실행되는 excute context에 따라 독립적인 Local 스코프만 들어가기 때문에 호출될 수 없다.
}
function fn1() {
 let l1 = 'l1'
 console.log(l0, l1) // l0 l1
 fn2()
}
fn1()
```

```jsx
let l0 = 'l0'

function fn1() {
 function fn2() {
  let l2 = 'l2'
  console.log(l0, l1, l2) // l0 l1 l2
  // 새로 생성된 Closure(fn1)에 따라 해당 함수의 변수가 저장되어 있다.
 }
 let l1 = 'l1'
 console.log(l0, l1) // l0 l1
 fn2()
}
fn1()
```

⇒ 부모 함수 안에서 자식 함수를 정의하면 자식함수를 어디에서 호출하더라도 자식함수 안에서 부모함수의 변수에 접근할 수 있다.

⇒ 또한, 부모함수는 실행을 마치면 실행 컨텍스트가 소멸되지만, 자식함수에 의해 형성된 클로저로 인해 렉시컬 환경을 활성 상태로 유지하여 부모 스코프의 변수를 보존한다.

자식 함수(클로저)에 대한 참조가 있는 한 스코프 체인은 가비지 수집되지 않는다.
