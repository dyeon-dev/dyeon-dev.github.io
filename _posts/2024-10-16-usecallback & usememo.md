---
title: React 최적화 (useCallback, React.memo, useMemo)
date: 2024-10-16 11:33:00 +0800
categories: [Blogging, React]
tags: [React rendering, useCallback, React.memo, useMemo]
math: true
mermaid: true
---

우리가 리액트 개발을 하다보면 수많은 렌더링을 마주하게 된다.

그 중에서도 컴포넌트 간에 영향을 미치지 않는데도 함께 리렌더링되는 과정을 겪어보았을 것이다.

이는 성능 측면에서 좋지 않은 영향을 끼치고 profiler 툴을 사용해서 직접 확인해볼 수 있을 것이다.

그럼 리렌더링을 방지하고 성능 최적화를 위한 도구 사용법을 알아보자!

### 쓸데없는 렌더링이 되는 경우 예시 코드

```jsx
function Parent() {
  const [valueForFirstChild, setValueForFirstChild] = useState(null)

  const handleClick = () => {}

  useEffect(() => {
    setTimeout(() => {
      setValueForFirstChild('changedValue')
      console.log("updated")
    },3000)

  }, [])
  return (
    <>
      <FirstChild value={valueForFirstChild}/>
      <SecondChild onClick={handleClick}/>
    </>
  );
}

export default Parent;
```

```jsx
function FirstChild({ value }) {
    return <div>{value}</div>;
}
```

```jsx
function SecondChild({ onClick }) {
  return (
    <div>
      {Array.from({ length: 1000 }).map((_, idx) => {
        if (idx === 999) {
          // 1000번째 GrandChild 렌더링 시점
          console.log("1000번째 GrandChild 컴포넌트가 렌더링되었습니다.");
        }
        <GrandChild key={idx + 1} order={idx} />
      })}
    </div>
  );
}

export default SecondChild
```

1. Parent 컴포넌트가 렌더링 되고 난 후, useEffect 내부의 로직들이 실행이 된다.
2. state의 내용이 변경이 되면서, Parent 컴포넌트에는 리렌더링이 일어나게 된다.
3. 리액트의 렌더링이란 함수를 호출하는 것이므로 Parent 컴포넌트가 호출이 되면서 FirstChild 컴포넌트와 SecondChild 컴포넌트도 리렌더링 된다.
4. Parent, FirstChild 컴포넌트의 경우 둘 다 값의 변경이 있으므로 리렌더링이 되는 것이 당연하다.
5. 하지만 SecondChild 컴포넌트는 변경된 값이 없는데 똑같은 정보를 보여주기 위해서 리렌더링이 되는 것은 불필요하다.
6. 게다가 SecondChild 컴포넌트들이 리렌더링 되면 1000개의 GrandChild 컴포넌트들 리렌더링 될 것이다.

### Profiler

<img width="466" alt="image" src="https://github.com/user-attachments/assets/8229f8e7-ae15-49e2-bc20-3bbbb14deb58">
<img width="690" alt="image" src="https://github.com/user-attachments/assets/1670e547-293b-40da-b99b-84fec5442060">

### 컴포넌트는 언제 리렌더링 될까?

1 ) 자신이 전달받은 **props**가 변경될 때

2 ) 자신의 **state**가 바뀔 때

3 ) 부모 컴포넌트가 리렌더링 될 때

4 ) forceUpdate 함수가 실행될 때

### **SecondChild가 불필요하게 리렌더링 되는 것을 막는 방법**

매번 Parent 컴포넌트가 리렌더링이 되면서 handleClick 함수가 재생성되고, 이때 재생성되는 handleClick은 이전과는 **다른 참조값**을 가지게 된다. 따라서 다른 함수가 되기 때문에(함수는 참조 타임의 데이터) **Parent 컴포넌트가 리렌더링 될 때 SecondChild 컴포넌트도 리렌더링이 된다.**

_1 ) 자신이 전달받은 **props**가 변경될 때_ 렌더링이 일어난다는 조건에 만족하기 때문에 리렌더링이 일어나게 되는 것이다.

그렇다면 함수의 참조값이 바뀌지 않게 해주면, SecondChild 컴포넌트의 불필요한 리렌더링을 방지해줄 수 있지 않을까?

함수의 참조값이 바뀌지 않게 하는 방법은?

⇒ **함수를 메모이제이션 해주는 리액트 Hook인 useCallback을 사용하면 된다.**

## useCallback

**메모이제이션이란?**

기존에 수행한 연산의 결과값을 어딘가에 저장해두고, 필요할 때 재사용하는 기법이다.

useCallback을 사용하여 함수를 감싸주게 되면, 의존성 배열이 변하지 않는 이상, 컴포넌트가 리렌더링 될 때마다, 변수에 같은 함수가 할당된다.

useCallback을 Parent 컴포넌트에 적용해 보면,
```jsx
function Parent() {
 const [valueForFirstChild, setValueForFirstChild] = useState(null);
    
    const handleClick = **useCallback(() => {},[])**;
    
    useEffect(()=>{
     setTimeout(()=>{
         setValueForFirstChild('changedValue')
         console.log("updated")
        }, 3000)
    },[]);
    
    return(
      <>
         <FirstChild value={valueForFirstChild} />
         <SecondChild onClick={handleClick} />
      </>
    );
}
```
handleClick을 useCallback으로 감싸주어서 컴포넌트가 리렌더링 되어도 변수는 같은 함수를 참조하게 되었다. 이번에 확인 해보면 SecondChild는 리렌더링이 일어나지 않을까?

예상과는 달리, Profiler의 결과를 확인해보면 SecondChild 컴포넌트는 여전히 리렌더링이 일어난다.

그 이유는 다음과 같다.

## 리액트의 렌더링 단계

리액트의 렌더링 단계를 통해서 좀 더 자세히 살펴보자. 리액트는 렌더링 과정에서 3가지 페이즈를 거친다.

1. **Trigger Phase**

    특정 트리거를 이용해 화면이 변경될 것이라는 요청을 제공한다. 리액트에서 대표적으로 useState 훅을 이용해서 상태 변경이 되었으니 화면 변경을 하라는 요청을 준다. ex) 버튼, 특정 스크롤 내리기 등

2. **Render Phase**
Trigger Phase에서 렌더링 요청을 한 다음에는 컴포넌트를 호출해 화면에 표시할 내용을 파악한다.

    초기 렌더링에서는 React에서 App Component를 호출하는 과정을 진행한다.

    컴포넌트(함수) 호출 → React Element를 반환하고 새로운 Virtual DOM을 생성해준다.

    초기 렌더링이 아니라면, **재조정**(Reconciliation)의 과정(이전 Virtual DOM과 현재 Virtual DOM을 비교하는 것)을 거친 후, **Real DOM에 비교해서 업데이트가 필요한 부분을 체크**한다.

3. **Commit Phase**

    Render Phase에서 체크했던 변경이 필요한 Real DOM목록을 **Real DOM에 반영**해주는 단계이다.

    만약 변경이 필요한 부분이 없다면, Commit phase는 skip된다.

    **즉, 렌더링 최적화를 할 수 있는 부분은 Commit Phase이다.**

4. **Browser Paint**

    공식 문서에 의하면 실제로 이 단계를 브라우저 렌더링이라고 할 수 있지만 나머지 부분과 혼동을 피하고자 Painting이라고 명시한다.

    해당 과정은 모든 렌더링이 완료된 이후 DOM을 업데이트 한 후 브라우저에서 화면을 다시 그리는 단계다.

Parent 컴포넌트를 Babel로 컴파일하게 되면, 다음과 같은 코드로 변환된다.

```jsx
function Parent() {
    const [valueForFirstChild, setValueForFirstChild] = useState(null);

    const handleClick = **useCallback(() => {},[])**;
    
    useEffect(()=>{
     setTimeout(()=>{
         setValueForFirstChild('changedValue')
         console.log("updated")
        }, 3000)
    },[]);

    return React.createElement(
      React.Fragment,
      null,
      React.createElement(FirstChild, {
        value: valueForFirstChild,
      }),
      React.createElement(SecondChild, {
        onClick: handleClick,
      })
    );
}
```

Parent 컴포넌트가 리렌더링되면, 내부의 로직들이 다 실행되며, SecondChild에 해당하는 React.createElement가 다시 실행된다.

결국 해당 과정에서  **Render Phase 과정을 거치게 되는 것이고, Commit Phase에 대해서는 불필요한 요청을 제거했다.**

**따라서 아직 최적화가 되지 않은 것이다.** 그래서 useCallback을 사용해서 함수가 같은 값을 메모이제이션하도록 해주었다 하더라도 리렌더링이 일어날 수 밖에 없는것이다.

그렇다고해서 useCallback을 사용했을 때, 렌더링 최적화에 아무 효과가 없는 것은 아니다!!!

useCallback을 사용하면 Render Phase는 실행되고 Commit phase는 실행되지 않지만, useCallback을 활용해서 props 값을 이전과 같이 유지해주었기 때문에 렌더링에 최적화 효과가 있긴 하다.

하지만 여전히 SecondChild 컴포넌트는 불필요한 렌더링을 반복하고 있다. Render Phase까지 막아줄 방법은 없을까?

**⇒ React.memo를 사용하면 된다.**

### React.memo

컴포넌트가 `React.memo()`로 래핑 될 때, React는 컴포넌트를 렌더링하고 결과를 메모이징(Memoizing)한다.

그리고 다음 렌더링이 일어날 때 `props`가 이전 `props`와 같다면, **컴포넌트의 리렌더링을 막아주고 마지막에 리렌더링된 메모이징(**Memoizing) **결과를 재사용하는 고차 컴포넌트(Higher Order Component, HOC)**이다.

따라서 불필요한 리렌더링을 건너뛸 수 있다.

React.memo가 props 혹은 props의 객체를 비교할 때는 기본적으로 **얕은(shallow) 비교를** 한다.

**얕은 비교**란?

_원시타입의 데이터_는 값이 같은지만 비교하고, _참조타입의 데이터_는 참조 값만 비교하는 것이다.

두번째 인자로 비교함수를 넣어주게 되면 해당 함수를 활용해서 비교를 해주도록 할 수도 있다.

```jsx
function Component({ content }) {
  return <p>{content}</p>;
}

export default React.memo(Component);
// React.memo(Component, [areEqual(prevProps, nextProps)]); -> prevProps와 nextProps가 같다면 true를 반환할 것이다.
```

React.memo를 SecondChild 컴포넌트에 적용했을때의 모습을 살펴보면,

```jsx
function SecondChild({ onClick }) {

  return (
   {Array.from({ length: 1000 }).map((_, idx) => {
        if (idx === 999) {  // 1000번째 GrandChild 렌더링 시점
            console.log("1000번째 GrandChild 컴포넌트가 렌더링되었습니다.");
        }
        return (
       <div onClick={onClick}>
         {Array.from({ length: 1000}).map((_, idx)=>(
           <GrandChild key={idx + 1} order={idx} />
         ))
       </div>
        );
    })}
  );
}

export default **React.memo(SecondChild);**
```

SecondChild 컴포넌트가 렌더링에 진입하기전에, props값인 onClick에 대해서 이전값과 현재값이 다른지를 비교해준다.

서로 같은 값이라면 SecondChild는 렌더링이 진행되지 않는다.

### Profiler

<img width="467" alt="image" src="https://github.com/user-attachments/assets/f6aeeefd-9c5d-4be2-82a7-a3ef594a47ad">
<img width="698" alt="image" src="https://github.com/user-attachments/assets/adb11df7-b80e-4b2f-b80c-5875f6959611">

SecondChild 컴포넌트가 렌더링 되지 않았음을 Profiler를 통해서 확인할 수 있다.

### 그럼 SecondChild 컴포넌트가 “객체”를 props로 받게 되면 어떻게 될까?

```jsx
function Parent() {
  const [valueForFirstChild, setValueForFirstChild] = useState(null)

  const value = { a: 1 };
  
  useEffect(() => {
    setTimeout(() => {
      setValueForFirstChild('changedValue')
      console.log("updated")
    },3000)
  }, [])

  return (
    <>
      <FirstChild value={valueForFirstChild}/>
      <SecondChild value={value}/>
    </>
  );
}

export default Parent;
```

```jsx
function SecondChild({ value }) {
  return (
    <div>
      {value.a}
   {Array.from({ length: 1000 }).map((_, idx) => {
        if (idx === 999) {  // 1000번째 GrandChild 렌더링 시점
            console.log("1000번째 GrandChild 컴포넌트가 렌더링되었습니다.");
        }
        return (
       <div onClick={onClick}>
         {Array.from({ length: 1000}).map((_, idx)=>(
           <GrandChild key={idx + 1} order={idx} />
         ))
       </div>
        );
    })}
    </div>
  );
}

export default React.memo(SecondChild);
```
같은 상황이지만 props로 넘겨받은 데이터가 함수에서 변하지 않는 value라는 값으로 바뀌었다. 이런 경우 Parent 컴포넌트가 리렌더링 되면 SecondChild는 아쉽게도 리렌더링이 발생한다.
그 이유는, Parent 컴포넌트가 리렌더링 되면서 **value라는 객체가 매번 생성되기 때문에** SecondChild 컴포넌트에게 **매번 다른 참조값을 가진 value를 props로 전달받아** React.memo가 정상적으로 작동하지 않는다.

⇒ 이럴 때는 **useMemo**를 사용하면 된다.

### useMemo

useCallback은 _**함수**_에 대한 메모이제이션을 제공하는 훅이고, useMemo는 _**값**_에 대한 메모이제이션을 제공하는 훅이다.

의존성 배열에 들어있는 값이 변경되지 않는 이상, 리렌더링 될 때마다, 같은 값을 반환해준다.

```jsx
function Parent() {
  const [valueForFirstChild, setValueForFirstChild] = useState(null)

  const value = { a: 1 };
  
  const memoizationValue = **useMemo(() => value,[])**

  useEffect(() => {
    setTimeout(() => {
      setValueForFirstChild('changedValue')
      console.log("updated")
    },3000)
  }, [])

  return (
    <>
      <FirstChild value={valueForFirstChild}/>
      <SecondChild value={memoizationValue}/>
    </>
  );
}

export default Parent;
```

코드에 적용시켜보면, memoized된 value 객체를 SecondChild 컴포넌트에 props로 전달해줬을 때 **매번 새로운 value을 전달해주지 않기 때문에 정상적으로 React.memo가 동작하면서 불필요한 렌더링을 방지**해 준다.

---

## **useCallback, useMemo, React.memo를 모든 곳에서 사용해주면 좋은걸까?**

이들도 하나의 코드이고 내부적으로 특정한 동작을 실행시켜 주어야하기 때문에, 하나 하나가 모두 비용이다.
무분별하게 다 훅으로 걸어서 메모리상에 잡아 놓을 필요는 없고 올바른 곳에 충분히 고민하고 사용해야 한다.

### 근본적인 코드를 먼저 개선하자

최적화 도구들을 사용하기 전에 근본적인 코드를 먼저 개선하는 것이 좋다. 불필요한 렌더링이 발생할 수 밖에 없는 코드에서 근본적인 문제를 해결하지 않은 상태에서 최적화 도구를 사용하는 것은 문제를 더 키우는 것이다.

예를 들어 다음 코드를 보자.

개선 전
```jsx
function Component() {
  const forceUpdate = useForceUpdate();
  
  return (
    <>
      <button onClick={forceUpdate}>force</button>
      <Consoler value="fixedValue" />
    </>
  );
}
```

- Consoler 컴포넌트를 부모 컴포넌트안에서 사용
- 부모 컴포넌트 Component가 리렌더링이 될 때,  Consoler 컴포넌트도 불필요하게 리렌더링이 되고 있다.

개선 후
```jsx
function Component({ childeren }) {
  const forceUpdate = useForceUpdate();
  return (
    <>
      <button onclick={forceUpdate}>force</button>
      {children}
    </>
  );
}

function App() {
  <div>
    <Component>
      <Consoler value="fixedValue" />
    </Component>
  </div>;
}
```

- children을 사용해서 Consoler 컴포넌트를 주입
- **React는 무언가 변경되지 않는 한, children prop을 심층적으로 검사하지 않기 때문에 prop이나 상태에 대한 변경 사항이 없으면 Consoler에 대한 렌더링을 건너뛸 수 있다.**

개선 후 코드를 babel로 변환 시킨 코드는 다음과 같다.

```jsx

function Component({ childeren }) {
  const forceUpdate = useForceUpdate();
  return React.createElement(
    React.Fragment,
    null,
    React.createElement(
      'button',
      {
        onClick: forceUpdate,
      },
      'force'
    ),
    children
  );
}
```

- return문에는 Consoler 컴포넌트에 대한 React.createElement가 존재하지 않기 때문에, 부모 컴포넌트가 호출 되더라도 Consoler 컴포넌트는 호출되지 않는다.

### 최적화가 필요할 때만 최적화를 하자

1. React.memo()를 사용한 렌더링 최적화
2. useMemo 훅을 사용한 렌더링 최적화
3. children을 사용한 렌더링 최적화 (최적화 도구를 쓰지 않고도 가능!)

**React에서는 성능 개선을 위한 하나의 도구로 메모이제이션을 사용한다.**

**대부분의 상황에서 최적화 도구로 React는 메모이징 된 컴포넌트의 리렌더링을 피할 수 있지만, 렌더링을 막기 위해 메모이제이션에 의존하면 안된다.**
