## 1. Context API 불필요한 렌더링 문제

### 1.1 함수 참조로 인한 불필요한 렌더링

개념부터 설명하면 어떤 컴포넌트가 context의 함수를 참조하고 있는 경우 context의 값이 변경 되어 Provider가 리렌더링 되면서 context의 함수가 재생성되어 컴포넌트 입장에서는 참조하던 값이 변경된 것이니 리렌더링되는 문제가 발생한다.

아래와 같은 context가 존재하고 컴포넌트에서는 `toggleTheme` 를 참조하여 사용할 때 불필요한 렌더링이 발생한다.

아래 코드를 예시로 다시 설명하자면,
`ThemeContext` 의 내부 값인 `theme` 값이 변경되게 되면 Provider가 리렌더링 된다.  
Provider가 리렌더링되면 안에 있는 `toggleTheme` 함수도 재생성되게 된다.  
이러면 `ToggleButton` 은 `toggleTheme` 을 참조하고 있기 때문에 React는 `toggleTheme` 이 재생성될 때  `ToggleButton` 이 참조하던 값이 변경되었다고 `ToggleButton` 을 리렌더링 시키게 된다.  
결론적으로는 함수의 동작이 변경된 것은 아니나 내부 값 변경으로 인해 함수까지 재생성되어 함수를 참조하고 있는 컴포넌트에게 리렌더링이 일어나는 문제인 것이다.  

```jsx
// ThemeContext.tsx
import { createContext, useState } from "react";

export const ThemeContext = createContext("light");
export const ThemeToggleContext = createContext(() => {});

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("light");

  // 매 렌더링마다 새로 생성되는 함수
  const toggleTheme = () => {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  };

  return (
    <ThemeContext.Provider value={theme}>
      <ThemeToggleContext.Provider value={toggleTheme}>
        {children}
      </ThemeToggleContext.Provider>
    </ThemeContext.Provider>
  );
}
```

```jsx
function ToggleButton() {
  const toggleTheme = useContext(ThemeToggleContext);

  console.log("ToggleButton rendered");

  return <button onClick={toggleTheme}>테마 변경</button>;
}
```

함수가 재생성될 때 발생하는 문제이다 보니 매우 간단하게 해결이 가능하다.  
아래와 같이 `toggleTheme` 을 useCallback으로 감싸면 Provider가 리렌더링 되어도 `toggleTheme` 는 재생성되지 않으니 컴포넌트에게도 불필요한 리렌더링이 발생하지 않는다.

```jsx
const toggleTheme = useCallback(() => {
  setTheme((prev) => (prev === "light" ? "dark" : "light"));
}, []);
```

- 여기서 든 생각이 “Provider는 화면에 직접적으로 보이는 것이 아닌데 왜 useState를 사용하고 리렌더링이 발생되어야 할까?” 여서 찾아보니
    
    생각보다 단순했다 consumer에게 새로운 값을 전파하기 위해서 리렌더링을 트리거로 해서 새로운 값을 전파해줘야 해서 값이 변하면 리렌더링되는 useState를 사용해야하는 것이다.
    

### 1.2 객체 중 일부 값 참조로 인한 불필요한 렌더링

개념부터 설명하면 context의 여러 값들을 하나의 객체로 이루어져 있다.  
context의 값을 참조하여 사용하는 consumer가 객체들의 값 중 일부만 사용할 때, consumer가 사용하지 않는 다른 값이 변경되어도 객체 자체가 다시 생성되기 떄문에 context를 참조하는 모든 consumer가 리렌더링되는 문제가 발생한다.  

아래와 같은 코드에서  `VolumeDisplay` ,`BrightnessDisplay` 같은 consumer가 context의 일부 값만 참조할 땐 불필요한 리렌더링이 발생한다.

아래 코드를 예시로 다시 설명하자면,  
`VolumeDisplay` 가 현재 `volume` 이라는 값만 사용하고 있지만, 사실 `SettingsContext`  자체를 참조하고 있기 때문에 `SettingsContext` 에서 `volume` 이 아니라 `brightness` 가 변경되어 객체가 재생성 되었을 때,  
`volume` 만을 사용하고 있는 `VolumeDisplay` 도 결국 `SettingsContext` 를 참조하고 있기 때문에 불필요한 리렌더링이 발생하게 된다.

```jsx
// SettingsContext.tsx
import { createContext, useState } from "react";

export const SettingsContext = createContext({ volume: 50, brightness: 50 });
export const SettingsActionContext = createContext(
  (_: number) => {}
);

export function SettingsProvider({ children }: { children: React.ReactNode }) {
  const [volume, setVolume] = useState(50);
  const [brightness, setBrightness] = useState(50);

  return (
    <SettingsContext.Provider value={{ volume, brightness }}>
      <SettingsActionContext.Provider value={setVolume}>
        {children}
      </SettingsActionContext.Provider>
    </SettingsContext.Provider>
  );
}
```

```jsx
function VolumeDisplay() {
  const { volume } = useContext(SettingsContext);
  console.log("VolumeDisplay rendered");
  return <div>볼륨: {volume}</div>;
}

function BrightnessDisplay() {
  const { brightness } = useContext(SettingsContext);
  console.log("BrightnessDisplay rendered");
  return <div>밝기: {brightness}</div>;
}
```

- 그렇다면 값 별로 Provider를 만들면 되는 거 아닐까?
    
    우선 값 별로 Provider를 만들어서 사용하면 위와 같은 문제는 해결이 된다.  
    단, 사용자가 버튼을 눌러서 새로운 아이템(값)을 추가해야 된다면 어떨까?  
    React 입장에서는 없던 Provider를 새로운 node로 만들어서 tree에 삽입해야 하는데  
    이 부분에서 비용적인 문제가 크다고 한다.  
    결국은 작은 비용을 아끼기 위해서 더 큰 비용을 지불하는 꼴이 될 수 있기 때문에 문제의 근본적인 해결방법은 아니다.
    

- 그럼 어떤 해결책이 있을까?
    
    Zustand, Recoil, Redux Toolkit 같은 라이브러리에서는 여러 값들 중 일부 값을 사용할 때  
    context의 객체 자체를 참조하는 것이 아닌, store에서 일부 값만 참조하는 Selector 기능을 제공하기 때문에 위에서 겪던 문제들을 해결할 수 있다.



---
Context API의 렌더링 관련된 문제에 대해서 자세히는 몰랐는데 이번에 공부해보니 생각보다 재밌었다  
그리고 React에서도 Selector 기능이 추가된다는 이야기를 들었던 것 같은데 빨리 추가 되었으면 좋겠다  
Zustand와 같은 상태관리 외부 라이브러리를 써야 하는 이유가 한 개 더 늘어난 것 같다  