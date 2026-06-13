# TIL

> 작더라도 하루 한 개씩은 개념 공부하자  
> 복습도 좋으니 그날 정리한 걸 써보자  
> 꼭 개념 공부, 정리가 아니라도 그날 배운 점, 느낀 점, 후회한 점을 써보자

## 폴더 구조

```
TIL/
├── language/              # 프로그래밍 언어 (문법, 런타임, 언어 기능)
├── frontend/
│   ├── react/             # React · Next.js 등 프레임워크
│   ├── web/               # CSS · 브라우저 등 웹 플랫폼 기본
│   ├── libraries/         # zustand, react-query 등 서드파티 라이브러리
│   ├── architecture/      # FSD 등 프로젝트 구조
│   └── toss/              # 토스 콘텐츠 기반 개념 정리
├── backend/               # 서버 · 프레임워크 · 인프라
├── algorithm/             # 코딩 테스트 · 자료구조 · 문제 풀이
├── tools/                 # Git hook, 모니터링, 패키지 매니저 등
└── journal/               # 개발 일지 · 느낀 점
```

### 분류 기준

| 어디에 둘까? | 기준 |
|---|---|
| **language** | JavaScript `this`, Java `switch`처럼 **언어 문법·동작** |
| **frontend/react** | React, Next.js 등 **프레임워크** 사용법 |
| **frontend/web** | Sass, Reflow처럼 **CSS·브라우저·렌더링** 기본 |
| **frontend/libraries** | zustand, TanStack Query 등 **서드파티 라이브러리** |
| **frontend/architecture** | FSD 등 **프로젝트·코드 구조** |
| **frontend/toss** | 토스 글·영상 보고 정리한 **개념** |
| **backend** | Spring, API, DB, 서버 설정 |
| **algorithm** | LeetCode, BOJ, 그리디·DP 등 **문제 풀이** |
| **tools** | husky, Sentry, npm/yarn/pnpm |
| **journal** | 개념 정리가 아닌 **경험·생각** |

### 새 글 추가 규칙

1. **폴더는 최대 2단계** — `frontend/react/` 까지만. 파일이 5개 넘으면 그때 하위 폴더 추가
2. **같은 출처 시리즈는 폴더로** — 토스처럼 연속으로 정리하는 콘텐츠는 `frontend/toss/`에
3. **라이브러리는 React와 분리** — 프레임워크가 아닌 npm 패키지는 `frontend/libraries/`
4. **루트에 파일 두지 않기** — README만 루트에 유지
5. **이름은 주제 중심** — `React useForm`, `zustand`처럼 무엇을 배웠는지 드러나게

---

## 목록

### language

#### Java
- [Java 시작](language/java/Java%20시작.md)
- [Java 변수 선언](language/java/Java%20변수%20선언.md)
- [Java 상식](language/java/Java%20상식.md)
- [Java 향상된 switch문](language/java/Java%20향상된%20switch문.md)

#### JavaScript
- [JS 실행 컨텍스트](language/javascript/JS%20실행%20컨텍스트.md)
- [JS 이터러블 & 이터레이터](language/javascript/JS%20이터러블%20&%20이터레이터.md)
- [JS 디바운싱 및 쓰로틀링](language/javascript/JS%20디바운싱%20및%20쓰로틀링.md)

### frontend

#### react
- [React useForm](frontend/react/React%20useForm.md)
- [React 렌더링 최적화 기본](frontend/react/React%20렌더링%20최적화%20기본.md)

#### web
- [Sass, SCSS 개념 공부](frontend/web/Sass,%20SCSS%20개념%20공부.md)
- [Reflow & Repaint](frontend/web/Reflow%20&%20Repaint.md)

#### libraries
- [zustand](frontend/libraries/zustand.md)

#### architecture
- [FSD 아키텍처](frontend/architecture/FSD%20아키텍처.md)

#### toss
- [지속 가능한 컴포넌트](frontend/toss/지속%20가능한%20컴포넌트.md)
- [클린코드](frontend/toss/클린코드.md)

### backend

#### spring
- [Spring Boot 시작](backend/spring/Spring%20Boot%20시작.md)

### algorithm
- [자료구조](algorithm/알고리즘%20[자료구조].md)
- [그리디](algorithm/알고리즘%20[그리디].md)
- [구현](algorithm/알고리즘%20[구현].md)

### tools
- [패키지 매니저](tools/패키지%20매니저.md)
- [husky 개념 공부](tools/husky%20개념%20공부.md)
- [Sentry 개념 공부](tools/Sentry%20개념%20공부.md)

### journal
- [개발하면서 느낀 점 1](journal/개발하면서%20느낀%20점1.md)
- [개발하면서 느낀 점 2](journal/개발하면서%20느낀%20점2.md)
- [개발하면서 느낀 점 3](journal/개발하면서%20느낀%20점3.md)
- [개발하면서 느낀 점 4](journal/개발하면서%20느낀%20점4.md)
