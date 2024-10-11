---
title: Async-await 동작원리 (ft. Generator)
date: 2024-09-25 11:33:00 +0800
categories: [Blogging, 자바스크립트]
tags: [generator, async await, 동작원리]
math: true
mermaid: true
---

이전글에서 제너레이터와 Promise, Async-await에 대해서 알아봤었다.

Async-await의 동작원리에는 제너레이터가 중요한 요소인데, 이 사실에 좀 더 초점을 맞춰서 제너레이터가 어떤 내부적인 동작을 하는지 알아보겠다.

Async-await은 ES8에서 도입되었다! 그 전까지는 Promise나 제너레이터 방식을 통해서 비동기 처리를 해줬었다.

그렇다면 Async-await을 사용하면 구형 브라우저에서 동작하기 위해서 Babel을 통해 트랜스 파일링을 하게 된다.

내부 동작 방식을 파악하기 위해서 [babel의 try it out](https://babeljs.io/repl#?browsers=&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABAQwBQEpEG8IIM5wA2ApgHSFwDmqA5MjegL4BQzyeAnpIqJLAogBGGbM0TjEuMARLkqtQQEYGYicgDuyGFBQZV4qTLIVqNQQCYVLYemaGix-TQgrmQA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env&prettier=false&targets=&version=7.16.6&externalPlugins=&assumptions=%7B%7D)에서  트랜스파일링을 해보면 다음과 같은 결과를 얻을 수 있다.
![alt text](/assets/img/4/image.png)

확인해보면 _asyncToGenerator, Promise 등 함수들이 사용된 것을 볼 수 있다.
async-await은 내부적으로 Generator와 Promise의 산물인 것이다!

***즉, async-await은 동작하기 위해 제너레이터 함수로 변경이 되어서 실행된다.***

그렇다면 제너레이터에 대해서 좀 더 자세하게 살펴보겠다.

---

## Generator란?

- 제너레이터는 코드 블록의 **실행을 일시 중지했**다가 **필요한 시점에 재개**할 수 있는 특수한 함수
- 일반 함수는 실행 결과로 반환값이 출력되지만, 제너레이터 함수는 실행 결과로  Generator 객체가 출력된다.

### Generator 객체란?

![alt text](/assets/img/4/image-1.png)
Generator 객체는 Iterator와 iterable 인터페이스를 준수한다.

- Iterator란? next 메서드를 가지고, next 메서드는 iteratorResult 객체를 반환한다.
- iteratorResult 객체란? **value(iterator가 반환하는 값)** 와 **done(iterator의 끝남 여부를 알려주는 값)** 프로퍼티를 갖는 객체이다.

**코드 관점**
> next() → yield → next() → yield → … → next() → return

`await`은 `yield`로 변경된다. 그리고 Generator의 `next`를 실행한다. 그럼 다시 `yield`에 의해 멈추게 되고 실행된   함수가 resolve되면 또 다시 Generator `next`를 호출한다. 그럼 다음 `yield`까지 실행되고 이런식으로 계속 반복되다가 마지막 `next`가 실행되었을 때 `return`문을 만나게 되면 함수가 끝나게 된다.

- async-await 함수 ⇒ 제너레이터 함수로 변경된다.
- 제너레이터 함수가 실행될 때 yield를 기준으로 promise에서 then 메소드로 넘겨가면서 실행한다.
- 이때 then 메소드로 넘어간 뒷부분이 **마이크로태스크큐**로 들어가게 되면서 콜스택이 비워진 다음에 이어서 실행된다.
  
## Generator가 중단된 yield (중단점)를 어떻게 기억할까?

![alt text](/assets/img/4/image-2.png) 출처: <https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-generator.prototype.next>

- Generator 객체는 내부적으로 **GeneratorState**와 **GeneratorContext** slot을 가지고 있다.
- **GeneratorState** → Generator의 진행상태와 관련된 값이 저장된다.
- **GeneratorContext** → Generator 함수의 **실행 컨텍스트**가 저장된다.

**GeneratorState Type**에는 4가지가 있다.

1. **Suspended Start** - Generator 함수를 실행했을 때의 상태
2. **Executing** - Generator 객체의 next 메서드를 실행한 시점의 상태
3. **Suspended Yield** - Generator 함수의 실행 도중 yield 키워드를 만난 시점의 상태
4. **Completed** - Generator 함수의 실행 도중 return 키워드를 만나거나 함수의 끝에 도달했을 때의 상태

![alt text](/assets/img/4/image-3.png)

**GeneratorContext 실행 컨텍스트**는 다음과 같이 실행된다.

실행 컨텍스트 스택에 해당 과정을 통해 Generator 함수의 실행 컨텍스트의 push와 pop이 일어난다.
Generator 함수 내부에 있는 모든 코드가 실행이 되면 Generator 객체의 상태는 Completed가 되고 실행 컨텍스트가 pop이 된다.

    1. 실행 컨텍스트 스택에 전역 실행 컨텍스트가 먼저 생성된다.
    2. Generator 함수의 실행 컨텍스트가 스택에 push 된다.
    3. Generator 객체가 생성된다. 이때 객체는 Suspended Start 상태로 생성되고, 함수 내부 코드가 실행되기 전까지 일시 중지된다.
    4. 다음 next 메서드를 호출하게 되면 다시 아까 Generator 함수의 실행 컨텍스트가 실행 컨텍스트 스택에 push 된다. 
    5. 그 다음에 Generator 함수의 코드 일부분이 실행된다. 이때의 Generator 객체의 상태는 Executing 상태가 된다. 
    6. 그리고 Generator 함수의 내부에서 실행이 되다가 yield 키워드를 만나게 되면 그때 다시 실행 컨텍스트가 pop된다. 이때 Generator Object의 상태는 Suspended Yield 상태가 된다.
    7. 다시 next 메서드를 만나서 실행하게 되면 실행컨텍스트 스택에 다시  Generator 함수의 실행 컨텍스트가 push되고 그때 Generator Object 상태는 Executing 상태가 된다.
    8. Generator 함수 내부에 있는 모든 코드가 실행이 되면 Generator 객체의 상태는 Completed가 되고 실행 컨텍스트가 pop이 된다.

### Generator 비동기 처리 패턴

기존에는 Generator + Promise → 비동기 처리 코드를 동기식 코드처럼 작성하는 패턴으로 코드를 작성했다.

하지만 해당 패턴을 사용하기 위해서는 Generator를 연속적으로 실행하는 유틸 함수가 필요했다.

이때 더 편하게 동기식 코드처럼 작성하는 방법을 고안해낸 것이  ⇒ Async-await의 등장이었다.

마지막으로 정리해보자면,

- Async-await은 Generator를 사용한 비동기 처리 패턴보다 더 편리하게 비동기 처리를 하기 위해 등장했다.
- Async-await의 동작 원리를 파악하기 위해서는 Generator 비동기 처리 패턴을 알고 있으면 수월하게 이해할 수 있을 것이다!
