## 1. 교차 타입


### 1.1 교차 타입이란?

교차 타입은 여러 타입의 프로퍼티를 모두 합집합으로 하는 새로운 타입을 만든다.  
타입을 만들 때는 프로퍼티를 모두 합집합으로 하지만, ‘교차 타입’ 이라는 이름을 가져서 약간 이상하다.

### 1.2 코드

아래는 교차 타입을 사용하는 예시 코드이다.  
아래처럼 `Timestamped`라는 타입과 `Named`라는 타입을 합쳐서 새로운 타입을 만들어낼 수 있다.  
단,`string & number`  이런 식으로 원시 타입끼리 교차하면 `never`  타입이 된다.

```tsx
type Timestamped = { createdAt: Date };
type Named = { name: string };
type NamedAndTimestamped = Timestamped & Named;
// { createdAt: Date; name: string } — 둘 다 가져야 함
```

### 1.3 실사용

- **믹스인 패턴**
여러 개의 작은 타입 조각을 조합해서 최종 타입을 만드는 경우로  
예를 들어 컴포넌트에서 `type ButtonProps = BaseProps & AccessibilityProps & VariantProps`처럼 관심사별로 타입을 쪼개놓고 합친다.

## 2. 유니온 타입


### 2.1 유니온 타입이란?

유니온 타입은 여러 타입 중 하나가 될 수 있음을 나타내는 타입이다.

유니온 타입의 진짜 힘은 타입 좁히기와 결합됐을 때 나온다.

- 리터럴 필드란?
    
    `"idle"`, `"loading"`처럼 특정 값 하나로 고정된 타입을 리터럴 타입이라고 하고, 객체 안에서 이런 리터럴 타입을 값으로 갖는 필드를 리터럴 필드라고 한다. 아래 코드에서 `status` 필드가 리터럴 필드다.
    
- 판별 유니온이란?
    
    판별 유니온은 객체마다 공통으로 가진 리터럴 필드(구분자 역할)를 기준으로 서로 다른 형태를 가지는 유니온 타입이다. 이 구분자 필드 값을 보고 TS가 나머지 필드의 타입까지 자동으로 좁혀준다.
    

### 2.2 코드

아래는 판별 유니온을 사용하는 예시 코드이다.

`status` 값에 따라 타입이 자동으로 좁혀져서, `success`가 아닌 상태에서 `data`에 접근하려 하면 컴파일 에러가 난다.

```tsx
type FetchState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

### 2.3 실사용

- **비동기 상태 관리**
    
    직접 fetch 상태를 관리할 때 `idle | loading | success | error` 판별 유니온은 표준 패턴처럼 쓰인다.
    
- **컴포넌트 variant props**
    
    `variant: "primary" | "secondary" | "ghost"` 같은 문자열 유니온은 디자인 시스템 컴포넌트에서 항상 쓰인다.
    

## 3. 인덱스 시그니처


### 3.1 인덱스 시그니처란?

key의 이름을 미리 다 알 수 없을 때, "이런 패턴의 key가 올 수 있다"고 선언하는 문법이다.  
단, 오타를 낸 key도 타입 체크를 통과한다는 점과 모든 값이 같은 타입이어야 한다는 제약이 있다.

### 3.2 코드

```tsx
type StringMap = {
  [key: string]: string;
};

const colors: StringMap = {
  red: "#FF0000",
  blue: "#0000FF",
};
```

### 3.3 실사용

- 실무에서 직접 쓰는 경우는 적은 편이다. key 종류를 알면 `Record<"ko" | "en" | "ja", string>` 같은 유니온 기반 Record로 대체하는 경우가 많다.
- 그래도 다국어 리소스 파일이나 feature flag 목록처럼 key를 진짜 예측할 수 없는 경우엔 쓰인다.

## 4. 인덱스드 액세스 타입


### 4.1 인덱스드 액세스 타입이란?

`obj["key"]`로 값을 꺼내듯, 타입 레벨에서 다른 타입의 특정 프로퍼티 타입만 뽑아 쓰는 문법이다.

원본 타입이 바뀌면 파생 타입도 같이 바뀌어서, 타입을 중복 정의하지 않아도 된다.

### 4.2 코드

```tsx
interface User {
  id: number;
  name: string;
  address: { city: string; zipCode: string };
}

type Address = User["address"]; // { city: string; zipCode: string }

const ROUTES = ["/home", "/about", "/contact"] as const;
type Route = (typeof ROUTES)[number]; // "/home" | "/about" | "/contact"
```

### 4.3 실사용

- `as const` 배열에서 유니온 타입을 뽑아내는 `(typeof arr)[number]` 패턴이 라우트, 권한 목록 등에서 자주 쓰인다.
- 거대한 API 응답 타입에서 특정 필드 타입만 뽑아 재사용할 때도 쓴다.

## 5. 맵드 타입


### 5.1 맵드 타입이란?

`keyof`로 얻은 key들을 순회하면서 프로퍼티를 변형해 새 타입을 만드는 문법이다.

TS 내장 유틸리티 타입인 `Partial`, `Readonly`, `Pick`, `Record`가 전부 이 문법으로 구현돼 있다.

### 5.2 코드

```tsx
type MyPartial<T> = { [K in keyof T]?: T[K] };
```

### 5.3 실사용

- 직접 처음부터 작성하는 경우는 적지만, `Partial<FormValues>`, `Pick<User, "id" | "name">`, `Omit<User, "password">`처럼 내장 유틸리티를 통해 매일 간접적으로 쓴다.

## 6. 템플릿 리터럴 타입


### 6.1 템플릿 리터럴 타입이란?

문자열 리터럴 타입을 조합해서 새로운 문자열 패턴 타입을 만드는 문법이다.  
조합이 늘어나면 곱집합으로 늘어나서 타입 추론이 느려질 수 있다는 점을 기억해야 한다.

### 6.2 코드

```tsx
type Direction = "top" | "right" | "bottom" | "left";
type Margin = `margin-${Direction}`;
// "margin-top" | "margin-right" | "margin-bottom" | "margin-left"
```

### 6.3 실사용

- react-hook-form의 필드 경로(`"address.city"`) 타입 추론처럼 라이브러리 내부에 숨어 있는 경우가 많고, 직접 설계해서 쓰는 빈도는 낮은 편이다.

## 7. 제네릭


### 7.1 제네릭이란?

타입을 함수의 매개변수처럼 다루는 문법이다.

`any`를 쓰면 재사용성은 얻지만 타입 안전성을 잃고, 타입별로 함수를 중복 작성하면 안전성은 얻지만 재사용성을 잃는다. 제네릭은 이 둘을 동시에 잡기 위해 존재한다.

### 7.2 코드

```tsx
async function fetchData<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return res.json();
}

const user = await fetchData<User>("/api/user");
```

### 7.3 실사용

- TanStack Query의 `useQuery<TData, TError>`처럼 API 클라이언트 레이어에서 거의 반드시 쓰인다.
- `List<T>`, `Select<T>` 같은 재사용 가능한 컴포넌트나 `useLocalStorage<T>` 같은 커스텀 훅에서도 자주 쓴다.


---
제네릭이나 교차 타입 같은 건 평소에도 많이 쓰고 자주 보는 개념이라 복습했다는 느낌이 강했지만, 맵드 타입 같은 건 직접적으로 잘 사용되지도 않고,  
유틸 구현 시에 내부에서 사용되다 보니 개념적으로도 접하지 않았을 것 같은데 우아한 타입 스크립트 책에서 나오는 개념이라 공부해봤다.  
유용하다기 보다는 신기한 것도 있고 TS에 대해 조금 더 이해한 것 같아서 뭔가 뿌듯함? 성장한? 그런 느낌이 든다.  
