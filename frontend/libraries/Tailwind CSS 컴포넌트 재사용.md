## 1. `@apply`


### 1.1 `@apply` 문법

```
.btn-primary {
  @apply bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600;
}
```

여러 유틸리티 클래스를 하나의 커스텀 클래스 이름으로 묶어주는 Tailwind 문법이다.

이렇게 CSS 파일에 적어두면 `className="btn-primary"` 하나로 저 유틸리티들을 다 적용한 것과 같은 효과를 낸다.

### 1.2 왜 안티패턴인가

결국 다시 `.css` 파일에 이름 붙은 클래스를 만드는 거다.

Tailwind를 쓰는 이유가 파일 분리 안 하고, 이름 안 짓고, 죽은 CSS 안 쌓이기였는데, `@apply`로 커스텀 클래스를 만들면 이 세 가지 문제가 그대로 돌아온다.

variant가 늘어날수록(`btn-primary`, `btn-outline`, `btn-ghost`...) CSS 파일도 같이 늘어나서, 결국 예전 방식이랑 다를 게 없어진다.

## 2. cva (Class Variance Authority)


### 2.1 cva가 필요한 이유

cva는 Class Variance Authority의 줄임말이다. 클래스의 변형(variance)을 관리하는 도구라는 뜻이다.

같은 컴포넌트인데 상황에 따라 스타일이 달라지는 걸 variant라고 부른다. Button을 예로 들면 primary(파란 배경), outline(테두리만), ghost(배경 없음)처럼 여러 버전이 필요한 경우다.

cva가 없으면 이런 걸 if/else로 직접 관리해야 한다.

```
function Button({ variant }) {
  let classes = "inline-flex items-center justify-center rounded-lg font-medium"

  if (variant === "primary") {
    classes += " bg-blue-500 text-white hover:bg-blue-600"
  } else if (variant === "outline") {
    classes += " border border-blue-500 text-blue-500 hover:bg-blue-50"
  } else if (variant === "ghost") {
    classes += " text-gray-600 hover:bg-gray-100"
  }

  return <button className={classes}>...</button>
}
```

variant가 늘어날수록 if/else가 계속 늘어나고, size 같은 다른 축까지 추가되면 이 로직이 순식간에 복잡해진다.

게다가 variant에 오타를 내도(`"primry"`) TypeScript가 못 잡아준다.

### 2.2 cva 사용법

```
pnpm add class-variance-authority
```

```
// button.styles.ts
import { cva, type VariantProps } from "class-variance-authority"

export const buttonStyles = cva(
  "inline-flex items-center justify-center rounded-lg font-medium",
  {
    variants: {
      variant: {
        primary: "bg-blue-500 text-white hover:bg-blue-600",
        outline: "border border-blue-500 text-blue-500 hover:bg-blue-50",
        ghost: "text-gray-600 hover:bg-gray-100",
      },
      size: {
        sm: "px-3 py-1.5 text-sm",
        md: "px-4 py-2 text-base",
        lg: "px-6 py-3 text-lg",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
)

export type ButtonVariants = VariantProps<typeof buttonStyles>
```

`buttonStyles`는 함수다.

`{ variant: "primary" }`를 넣으면 해당하는 클래스들을 다 합쳌서 문자열로 돌려준다.

```
buttonStyles({ variant: "primary", size: "lg" })
// → "inline-flex items-center ... bg-blue-500 text-white hover:bg-blue-600 px-6 py-3 text-lg"
```

첫 번째 인자로 넘긴 공통 클래스는 어떤 variant를 골라도 항상 붙어 나온다.

`variants` 안에 정의한 값만 variant로 넘길 수 있게 타입이 자동으로 제한되니까, 오타를 내면 TypeScript가 바로 잡아준다.

## 3. clsx로 조건부 클래스 조합


### 3.1 clsx가 필요한 이유

React에서 className을 조건부로 넣으려고 하면 코드가 지저분해지기 쉽다.

```
function Badge({ isActive, isDisabled }) {
  return (
    <span
      className={
        "px-2 py-1 rounded" +
        (isActive ? " bg-blue-500" : " bg-gray-300") +
        (isDisabled ? " opacity-50" : "")
      }
    >
      배지
    </span>
  )
}
```

문자열을 `+`로 이어붙이다 보면 띄어쓰기 빠뜨리는 실수도 잦고, 조건이 늘어날수록 가독성이 확 떨어진다.

### 3.2 clsx 사용법

```
pnpm add clsx
```

```
import clsx from "clsx"

clsx(
  "px-2 py-1 rounded",
  isActive ? "bg-blue-500" : "bg-gray-300",
  isDisabled && "opacity-50"
)
```

인자로 받은 값들을 하나씩 훑으면서, `false`나 `undefined`, `null`처럼 falsy한 값은 무시하고, 문자열만 골라서 띄어쓰기로 이어붙여준다.

`isDisabled && "opacity-50"`도 `isDisabled`가 `false`면 그냥 `false`가 되니까 clsx가 알아서 건너뛴다.

## 4. tailwind-merge로 클래스 충돌 처리


### 4.1 clsx만으로는 부족한 이유

```
buttonStyles({ variant: "primary" })가 이미 "bg-blue-500"을 포함하고 있는데
밖에서 <Button className="bg-red-500" />처럼 오버라이드하고 싶다고 하자
```

```
clsx(buttonStyles({ variant: "primary" }), "bg-red-500")
// 결과: "... bg-blue-500 ... bg-red-500"
```

clsx만 쓰면 `bg-blue-500`이랑 `bg-red-500`이 둘 다 최종 문자열에 남는다.

CSS는 나중에 선언된 규칙이 우선이니까 어쪻다가 `bg-red-500`이 이기긴 하는데, 이건 Tailwind가 CSS를 생성하는 순서에 달린 운빨이다. 버전이 바뀌면 순서가 바뀌을 수도 있어서 안전하지 않다.

### 4.2 tailwind-merge 사용법

```
pnpm add tailwind-merge
```

```
import { twMerge } from "tailwind-merge"

twMerge(clsx(buttonStyles({ variant: "primary" }), "bg-red-500"))
// 결과: "... bg-red-500"  (bg-blue-500은 자동으로 제거됨)
```

twMerge는 `bg-*`가 같은 속성(배경색)이라는 걸 인식해서, 나중에 온 `bg-red-500`만 남기고 앞의 `bg-blue-500`을 확실하게 제거해준다.

실무에서는 clsx랑 tailwind-merge를 같이 묶은 `cn` 유틸을 하나 만들어두고 쓰는 게 표준이다.

```
// src/shared/lib/cn.ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## 5. 결론


### 5.1 완성본

```
// Button.tsx
import type { ButtonHTMLAttributes } from "react"
import { buttonStyles, type ButtonVariants } from "./button.styles"
import { cn } from "@/shared/lib/cn"

interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    ButtonVariants {}

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonStyles({ variant, size }), className)}
      {...props}
    />
  )
}
```

사용은 이렇게 깔끔해진다.

```
<Button variant="primary" size="lg">제출</Button>
<Button variant="outline">취소</Button>
```

---
확실히 진짜 실무에서 사용된다는 게 느껴졌다 다시 한 번 깊게 파봐도 괜찮은 주제 같다. 
```

---
