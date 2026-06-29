## 1. Unit 테스트란?


### 1.1 개념

가장 작은 단위로 함수 하나, 모듈 하나를 독립적으로 검증하는 테스트다.

외부 의존(API, DB, 다른 모듈)을 끊고 그 함수 자체만 테스트한다.

## 2. 순수 함수 vs 비순수 함수

---

### 2.1 차이

```jsx
// 순수 함수 - 같은 입력, 항상 같은 출력
function add(a, b) {
  return a + b
}

// 비순수 함수 - 외부 상태에 의존
let count = 0
function increment() {
  count++
  return count
}
```

순수 함수는 외부 상태를 건드리지 않고 입력만으로 출력이 결정되기 때문에 Unit 테스트하기 쉽다.

비순수 함수는 외부 상태에 따라 결과가 달라져서 테스트 전에 상태를 매번 초기화해야 한다. API 호출, `Date.now()`, `Math.random()` 같은 것도 비순수라 Unit 테스트하기 까다롭다. 이건 mock으로 대체하는데 그건 나중에 다룰 내용이다.

## 3. Jest 문법


### 3.1 테스트 구조

Jest(그리고 Vitest)에서 테스트를 작성하는 기본 구조다.

```jsx
describe('add 함수', () => {
  it('1 + 2는 3이다', () => {
    expect(add(1, 2)).toBe(3)
  })
})
```

- `describe` — 관련 테스트를 묶는 그룹
- `it` / `test` — 테스트 케이스 하나
- `expect` — 실제 값
- `toBe` — 기대값과 비교하는 matcher

- matcher이란?
    
    matcher은 어떻게 비교할지 정하는 함수이다.
    값과 값을 비교할 수도 있고, 객체나 배열의 내용을 비교할 수도 있고, 함수가 에러를 던지는 지 여부를 확인할 수도 있다.
    

### 3.2 Assertion

테스트의 핵심은 "이 값이 내가 기대한 값이랑 같냐"를 검증하는 것이다. 이걸 assertion이라고 한다.

`expect(실제값).matcher(기대값)` 형태로 쓴다. 값이 다르면 테스트 실패다.

## 4. AAA 패턴


### 4.1 개념

테스트 코드를 Arrange / Act / Assert 3단계로 구조화하는 패턴이다.

```jsx
it('1 + 2는 3이다', () => {
  // Arrange — 준비
  const a = 1
  const b = 2

  // Act — 실행
  const result = add(a, b)

  // Assert — 검증
  expect(result).toBe(3)
})
```

- **Arrange** — 테스트에 필요한 값, 환경 준비
- **Act** — 테스트 대상 함수 실행
- **Assert** — 결과 검증

간단한 테스트는 한 줄로 써도 되지만, 복잡해질수록 이 구조로 나누면 읽기 쉬워진다.

## 5. Vitest

---

### 5.1 개념

Vite 기반 프로젝트에서 쓰는 테스트 프레임워크다.  
Jest API를 그대로 가져다 써서 `describe`, `it`, `expect`, `toBe` 전부 동일하다. 
Jest는 Babel이나 TypeScript 환경에서 별도 transform 설정이 필요하고 `jest.config.js`를 따로 잡아줘야 하지만, Vitest는 `vite.config.js`를 그대로 공유해서 Vite 프로젝트라면 설치만 하면 바로 돌아간다.


---
Integration나 E2E 테스트는 도입하지 못해도 앞으로 진행하는 프로젝트 혹은 이전에 진행했던 프로젝트에 직접 적용해보면서 더 공부하고 유지보수성도 높여보고 싶다.  
특히 Unit 테스트를 도입하면 함수의 이상이 있는지 바로 확인할 수 있으니 디버깅이나 리팩토링 과정에 많이 유용할 것 같아서 개인 프로젝트에서라도 먼저 빨리 적용해보고 싶은 마음이 크다.  
