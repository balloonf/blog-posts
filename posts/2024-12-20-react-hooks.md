---
title: "React Hook 완전 정복 가이드"
date: "2024-12-20"
tags: ["React", "Hook", "useState", "useEffect", "JavaScript"]
published: true
---

# React Hook 완전 정복 가이드

React Hook은 함수형 컴포넌트에서 상태와 생명주기를 관리할 수 있게 해주는 강력한 기능입니다.

## 기본 Hook들

### 1. useState
상태 관리를 위한 가장 기본적인 Hook입니다.

```javascript
const [count, setCount] = useState(0);

const increment = () => {
  setCount(count + 1);
};
```

### 2. useEffect
컴포넌트의 생명주기를 관리합니다.

```javascript
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

### 3. useContext
전역 상태를 쉽게 공유할 수 있습니다.

```javascript
const theme = useContext(ThemeContext);
```

## 고급 Hook들

### 1. useReducer
복잡한 상태 로직을 관리할 때 유용합니다.

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

### 2. useMemo
연산 결과를 메모이제이션하여 성능을 최적화합니다.

```javascript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

### 3. useCallback
함수를 메모이제이션하여 불필요한 리렌더링을 방지합니다.

```javascript
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

## 커스텀 Hook 만들기

재사용 가능한 로직을 커스텀 Hook으로 만들 수 있습니다.

```javascript
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}
```

## Hook 사용 규칙

1. **최상위에서만 호출**: 반복문, 조건문, 중첩 함수 안에서 Hook을 호출하지 마세요.
2. **React 함수에서만 호출**: 일반 JavaScript 함수에서는 Hook을 호출하지 마세요.

## 마무리

React Hook을 제대로 이해하고 사용하면 더 깔끔하고 재사용 가능한 컴포넌트를 만들 수 있습니다.

다음 포스트에서는 실제 프로젝트에서 Hook을 활용하는 방법을 알아보겠습니다.
