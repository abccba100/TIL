## 1. 렌더링 최적화 개념


### 1.1 렌더링 최적화 기본 개념

React에서 ‘렌더링 최적화’라고 하면 기본적으로 부모가 리렌더링 될 때 자식까지 함께 리렌더링되는 문제를 해결하는 것을 떠올린다.

왜냐하면 React에서는 부모가 리렌더링 시에 자식의 값 변경 유무에 상관없이 리렌더링 되기 때문에 프로젝트가 커지면서 자식도 함께 커진다면 무시할 수 없는 수준의 불필요한 리렌더링 비용이 발생할 수 있ㄷ

## 2. 렌더링 최적화 방법


### 2.1 React.memo

`React.memo`를 사용하면 자식의 리렌더링을 스킵할 수 있다.

아래 코드처럼 `React.memo` 를 사용하게 되면 Child의 props가 변하지 않았다면 Child의 리렌더링은 일어나지 않는다.

```jsx
const Child = React.memo(function Child() {
  return <h1>Hello</h1>;
});
```

하지만, `React.memo`에도 문제가 하나 있다.
아래 코드처럼 자식 컴포넌트에 props로 함수를 넘기는 경우에는 함수가 매번 재생성되기 때문에 `React.memo` 를 사용해도 리렌더링이 일어난다.
이 문제를 해결하기 위해 나온 것이 `useCallback` 이다.

```jsx
<Child onClick={() => console.log("hello")} />
```

### 2.2 useCallback

`useCallback` 을 사용하면 함수를 기억해두기 때문에 함수를 새로 생성하지 않는다.

아래 코드처럼 `useCallback` 을 사용하면 리렌더링 될 때도 이전에 만들어둔 함수를 참조하기 때문에 props가 변했다고 인지하지 않아서 리렌더링되지 않는다.

```jsx
const handleClick = useCallback(() => {
  console.log("hello");
}, []);
```

### 2.3 useMemo

`useMemo` 는 한 번 계산된 값을 캐싱하여 불필요한 계산 비용을 줄일 수 있다.

아래 코드처럼 `useMemo` 를 사용하면 리렌더링 되었을 때 result의 값을 다시 계산하지 않고 전에 계산되어 캐싱된 값을 사용한다.

```jsx
const result = useMemo(() => {
  return expensiveCalculation();
}, []);
```

---

앞으로 깊게 공부할 렌더링 최적화를 공부하기 위해 다시 복습을 해보니 생각보다 중요한 것들인데 실제로 프로젝트할 때는  
크게 신경쓰지 않고 많이 넘겼던 것 같아 후회도 많이 되고 앞으로 프로젝트에서는 제대로 된 최적화를 해야겠다는 다짐을 하게 됐다.
