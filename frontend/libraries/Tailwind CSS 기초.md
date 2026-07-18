## 1. Tailwind CSS 개념


### 1.1 Tailwind CSS 기본 개념

Tailwind CSS는 React에서 쓰는 유틸리티 우선(utility-first) CSS 프레임워크이다.

원래는 `.css` 파일에 클래스를 따로 정의하고 이름으로 연결해서 쓰지만, 그러면 스타일 파일이랑 컴포넌트 파일이 분리돼서 왔다갔다 해야 하고,  
클래스 이름을 계속 새로 지어야 하고, 컴포넌트를 지워도 CSS는 같이 안 지워져서 죽은 CSS가 쌓이는 문제가 있다. Tailwind는 이걸 클래스 자체에 스타일을 다 때려박는 방식으로 해결한다.

### 1.2 유틸리티 우선(utility-first) 방식

클래스 하나가 CSS 속성 하나(또는 아주 작은 단위)에 대응한다.

```
<button className="px-4 py-2 rounded-lg bg-blue-500 text-white hover:bg-blue-600">
  제출
</button>
```

- `px-4` ← padding-left/right: 1rem
- `py-2` ← padding-top/bottom: 0.5rem
- `rounded-lg` ← border-radius: 0.5rem
- `bg-blue-500` ← background-color
- `hover:bg-blue-600` ← :hover일 때 배경색 변경

## 2. Tailwind 동작 원리


### 2.1 JIT(Just-In-Time) 컴파일러

Tailwind는 `bg-blue-500`, `px-4` 같은 클래스를 미리 다 CSS로 만들어두는 게 아니다. 색상 종류 × 채도 단계 × 속성 조합만 해도 CSS 파일이 수십 MB가 될 거라서,  
대신 빌드할 때 소스 코드(.tsx, .jsx 등)를 텍스트로 스캔해서 실제로 등장하는 클래스명만 골라 CSS를 생성한다. 이걸 JIT(Just-In-Time)라고 부른다.

### 2.2 동적 클래스명이 안 먹히는 이유

JIT 스캐너는 소스 코드에 있는 완전한 문자열만 클래스로 인식한다. 그래서 문자열을 조합해서 클래스명을 만들면 인식을 못 한다.

```
// 작동 안 함
function Badge({ color }) {
  return <span className={`bg-${color}-500`}>배지</span>
}
```

소스 코드에 `bg-${color}-500`이라는 완성 안 된 문자열만 있고, `bg-red-500`이나 `bg-green-500` 같은 완전한 문자열이 어디에도 없어서 스캐너가 인식을 못 한다.  
그래서 빌드된 CSS에 해당 스타일이 아예 없고, 런타임에 `color`가 뭐가 되든 스타일이 안 먹힌다.

```
// 완전한 클래스명을 매핑 객체로 다 나열해야 함
const colorMap = { red: "bg-red-500", green: "bg-green-500" }
function Badge({ color }) {
  return <span className={colorMap[color]}>배지</span>
}
```

## 3. Tailwind 설치 및 설정


### 3.1 v3 vs v4 — 설정 방식이 완전히 바뀜

v3까지는 설정할 게 세 군데로 나뉘어 있었다.

1. `tailwind.config.js` ← JS 파일로 스캔 범위(`content`)랑 커스텀 테마를 지정

```
module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"], // 스캔할 파일 범위를 직접 지정해야 함
  theme: {
    extend: {
      colors: { brand: "#3b82f6" },
    },
  },
}
```

`content`에 스캔할 경로를 안 적어두면 JIT가 그 파일은 아예 안 훑어서 클래스가 안 먹힌다.

1. `postcss.config.js` ← Tailwind가 PostCSS라는 CSS 변환 도구 위에서 동작하는 플러그인이라 따로 등록해야 함

```
module.exports = {
  plugins: { tailwindcss: {}, autoprefixer: {} },
}
```

1. CSS 파일에 지시어 3줄

```
@tailwind base;      /* 브라우저 기본 스타일 리셋 */
@tailwind components; /* 컴포넌트 레이어 */
@tailwind utilities;  /* 유틸리티 클래스 전부 */
```

**v4부터는 이 세 군데가 CSS 파일 한 줄로 합쳐졌다.**

```
/* src/app/styles/index.css */
@import "tailwindcss";
```

이 한 줄이 예전의 `@tailwind` 세 줄을 대신하고, 스캔 범위도 `@tailwindcss/vite` 플러그인이 프로젝트 구조를 보고 자동으로 찾아줘서 `content` 경로를 따로 안 적어도 된다. `postcss.config.js`도 필요 없어졌다.

```
// vite.config.ts
import tailwindcss from "@tailwindcss/vite"
export default defineConfig({ plugins: [react(), tailwindcss()] })
```

커스텀 색상도 예전엔 `tailwind.config.js`의 `theme.extend`에 JS 객체로 적었는데, v4는 CSS 안에서 `@theme` 블록으로 적는다.

```
@import "tailwindcss";

@theme {
  --color-brand-500: #3b82f6;
}
```

### 3.2 간격(spacing) 스케일

임의의 px 값 대신 0.25rem(4px)을 기본 단위로 하는 정해진 숫자 스케일을 쓴다.

- `p-1` ← 4px
- `p-2` ← 8px
- `p-4` ← 16px
- `p-6` ← 24px

`p`(전체) / `px`(좌우) / `py`(상하), margin은 `m` 접두사.

### 3.3 반응형 (mobile-first)

프리픽스 없는 클래스가 모바일 기본 스타일이고, `sm:`(640px~) `md:`(768px~) `lg:`(1024px~) 순으로 해당 너비 이상일 때 값을 덮어쓴다.

```
<div className="text-sm md:text-base lg:text-lg">반응형 텍스트</div>
```

여기서 `md:text-base`는 "768px 구간에서만"이 아니라 "768px 이상 전부"를 뜻하고, 더 큰 프리픽스(`lg:`)가 나오면 그게 또 덮어쓴다.


---
새로 시작하는 프로젝트에서 Tailwind CSS를 사용할 거라서 문법만 약간 공부해보려고 했는데,  
어쩌다 보니 Tailwind CSS 버전 차이, 흐름이랑 동작 과정을 공부해버렸다.  
그리고 유틸리티 우선 방식을 생각해낸 사람이 누구인지는 모르지만 진짜 똑똑한 것 같다는 생각이 들었다.  
기존 CSS의 문제점을 유틸리티 우선 방식이라는 되게 간단한 방식으로 해결해낸 게 좀 많이 놀랍다.  
