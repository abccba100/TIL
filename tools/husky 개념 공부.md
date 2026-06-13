## 1. husky 기본 개념


### 1.1 husky란?

husky는 Git Hook을 쉽게 쓰게 해주는 개발 도구이다.

### 1.2 husky 사용 이유

오류가 있는 코드도 그냥 commit이 가능하다.
또한 협업 동료들 마다 코드의 형태 같은 것이 다른 상태에서 commit 될 수 있다.

그래서 이런 오류를 검사하는 도구들이 있지만, husky가 없으면 개발자가 직접 commit 전에 검사 도구들을 실행시켜야 했다.  
그 결과, 개발자가 검사를 깜빡하고 commit 하여 위와 같은 일이 발생할 수 있기 때문에 husky를 사용한다.

**husky는 commit 전 검사를 실행시켜 이런 불상사를 사전에 방지할 수 있다.** 

**husky가 직접 검사하는 것이 아닌 검사하는 도구들을 실행시킨다.**

| 실제 검사 | 담당 도구 |
| --- | --- |
| 문법 검사 | ESLint |
| 코드 정렬 | Prettier |
| 변경 파일만 검사 | lint-staged |
| 테스트 | Jest, Vitest |

## 2. husky 사용 방법


### 2.1 husky 설치

```bash
npm install husky --save-dev
npx husky init
```

husky가 설치되면

```bash
.husky/
 ├─ pre-commit
 ├─ pre-push <-추가 명령어 입력 시 생성
 └─ _/
```

pre-commit, pre-push는 각각 commit, push 명령어를 수행하기 전에 실행된다.

_/ 폴더는 husky 엔진 파일이 들어있는 내부용 폴더이므로 몰라도 된다.

### 2.2 husky 폴더 구성

pre-commit 안에는 보통 이렇게 구성된다.

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

여기서 npx lint-staged 이게 핵심이다.
npx lint-staged는 변경된 파일에만 검사 도구들을 적용시킨다.

**package.json** 에 이렇게 검사 도구들이 있다.

```bash
{
  "lint-staged": {
    "*.{js,ts,jsx,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

## 3. husky 전체 흐름


### 3.1 husky 단계 별 보기

```bash
[ Git ]
  ↓
[ Husky ]
  ↓
[ pre-commit ]
  ↓
[ lint-staged ]
  ↓
[ ESLint / Prettier / 기타 검사 도구 ]
```

위 설명에서 본 것처럼 생각보다 간단한 흐름으로 이루어져 있다.
