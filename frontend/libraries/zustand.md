## 1. zustand 개념


### 1.1 zustand 기본 개념

zustand는 React에서 사용하는 전역상태 관리 라이브러리이다.  

원래는 Context API라는 걸 사용하지만 사용도 까다롭고 성능도 다른 라이브러리에 비해 떨어진다.  

### 1.2 zustand 기본 구조

zustand의 기본 구조는 다음과 같다.  

- `store`  ← 데이터 저장소
    - `state`  ← 상태 데이터
    - `action`  ← 상태 데이터를 변경하는 함수

## 2. zustand 사용


### 2.1 zustand 설치

```bash
npm install zustand
```

### 2.2 zustand 기본 사용 방법

**zustand 호출**

```tsx
import { create } from "zustand"
```

**기본적으로 TypeScript 기준에서는 store를 생성할 때 필요한 type을 정의해야 한다.**

```tsx
interface AuthState {
  accessToken: string | null
  setToken: (token: string) => void
  logout: () => void
}
```

**store 생성 구조**

```tsx
const store 이름 = create<위에서 생성한 type>()()

/* 기본적으로는 이런 구조를 가지지만 TypeScript에서는 
	 아래와 같은 축약형으로 쓰는 걸 허용한다 */
const store 이름 = create<위에서 생성한 type>()
```

**store 생성**

```tsx
export const useAuthStore = create<AuthState>((set) => ({
  accessToken: null, // <- 기본 상태 데이터 지정

  setToken: (token) =>
    set({
      accessToken: token,
    }),

  logout: () =>
    set({
      accessToken: null,
    }),
}))
```

### 2.3 middleware 사용

middelware는 store에서 추가적인 기능을 사용할 수 있게 해주는 도구이다.  

middleware에는 다양한 키워드가 존재하고 키워드 마다 고유한 기능을 가지고 있다.  

**persist**

웹/앱이 종료되어도 상태를 가지고 있을 수 있도록 해주는 미들에워이다.  

```tsx
import { create } from "zustand"
import { persist } from "zustand/middleware"

interface AuthState {
  token: string | null
  setToken: (token: string) => void
}

export const useAuthStore = create<AuthState>(
  persist(
	    (set) => ({
      token: null,

      setToken: (token) =>
        set({
          token,
        }),
    }),
    {
      name: "auth-storage", // <- 상태 데이터에 접근할 때 쓰는 key
    }
  )
)
```

**immer**

상태 업데이틀를 편하게 해주는 미들웨어이다.  

```tsx
// 원래는 이렇게 상태를 변경해야 하지만,
set((state) => ({
  user: {
    ...state.user,
    name: "sunwoo"
  }
}))

// immer을 쓰면 아래처럼 직접 상태를 변경할 수 있다.
set((state) => {
  state.user.name = "sunwoo"
})

// 위에서 쓰이는 state는 store 내에 있는 데이터이다.

// 사용 예시 코드
import { create } from "zustand"
import { immer } from "zustand/middleware/immer"

interface UserState {
  user: {
    name: string
    age: number
  }
  setName: (name: string) => void
}

export const useUserStore = create<UserState>(
  immer((set) => ({
    user: {
      name: "",
      age: 0,
    },

    setName: (name) =>
      set((state) => {
        state.user.name = name
      }),
  }))
)
```

---
원래는 Context API 쓰다가 zustand 쓰니까 너무 편하다.  
앞으로도 이런 라이브러리 공부 많이 해야겠다.  
