## 1. Playwright란?


### 1.1 개념

Microsoft가 만든 E2E 테스트 프레임워크다. 실제 브라우저(Chromium, Firefox, WebKit)를 자동으로 조작해서 유저처럼 동작하는 테스트를 작성할 수 있다.

여기서 자동으로 조작한다는 게 신기했는데, 코드로 브라우저에 명령을 내리면 실제 브라우저가 그대로 실행하는 방식이다. 사람이 직접 클릭하는 것과 차이가 없다.

## 2. 주요 개념


### 2.1 핵심 구성요소

- **Page** — 브라우저 탭 하나. 테스트의 기본 단위
- **Locator** — 요소를 찾는 방법. RTL의 쿼리랑 비슷한 개념
- **Action** — 클릭, 입력, 이동 같은 유저 동작
- **Assertion** — `expect(locator).toBeVisible()` 같은 검증

### 2.2 Locator

Playwright에서 요소를 찾는 핵심 개념다.

```jsx
page.getByRole('button', { name: '로그인' })
page.getByText('환영합니다')
page.getByPlaceholder('이메일을 입력하세요')
page.getByTestId('submit-btn')
```

RTL이랑 API가 거의 비슷하다. 차이점이 궁금해서 찾아봤는데, RTL은 이미 렌더링된 가상 DOM에서 요소를 찾고 Playwright는 실제 브라우저에서 찾는다는 게 다르다.

## 3. 테스트 구조


### 3.1 기본 구조

```jsx
import { test, expect } from '@playwright/test'

test('로그인 성공', async ({ page }) => {
  await page.goto('https://example.com/login')
  await page.getByPlaceholder('이메일').fill('test@test.com')
  await page.getByPlaceholder('비밀번호').fill('1234')
  await page.getByRole('button', { name: '로그인' }).click()
  await expect(page).toHaveURL('/dashboard')
})
```

- `test` — Jest의 `it`량 같은 역할
- `page.goto` — URL 이동
- `fill` — input에 텍스트 입력
- `click` — 클릭
- `expect(page).toHaveURL` — 현재 URL 검증

## 4. 왜 async/await인가?


### 4.1 이유

처음에 왜 모든 동작에 `await`를 붙이는지 의문이었다.

브라우저 동작은 전부 비동기라서 결과를 기다려야 하기 때문이다.

`fill`, `click`, `goto` 전부 Promise를 반환하는데, `await` 없이 다음 줄로 넘어가버리면 이전 동작이 끝나기도 전에 다음 동작을 실행하게 된다. 예를 들어 페이지 이동이 끝나기 전에 버튼을 클릭하려고 하면 요소를 못 찾아서 테스트가 실패한다.

## 5. Cypress와의 차이


### 5.1 비교

Cypress도 E2E 테스트 도구인데 왜 요즘 Playwright를 더 많이 쓰는지 궁금했다.

Playwright는 Chromium, Firefox, WebKit을 모두 지원하지만 Cypress는 기본적으로 Chrome 계열만 지원한다.

또 멀티탭, 멀티 브라우저도 Playwright가 더 자유롭다.

### 5.2 멀티탭 / 멀티 브라우저

멀티탭은 브라우저에서 탭 여러 개를 동시에 열어서 테스트하는 것이다. 예를 들어 관리자 탭과 유저 탭을 동시에 열어서 관리자가 공지를 올리면 유저 탭에 바로 뜨는지 확인하는 식으로 활용한다.

멀티 브라우저는 Chrome, Firefox, Safari를 동시에 띄워서 같은 테스트를 돌리는 것이다. 브라우저마다 동작이 다를 수 있어서 크로스 브라우징 검증할 때 쓴다.

Cypress는 이게 제한적이거나 유료 플랜에서만 돼서 Playwright한테 밀리는 이유 중 하나다.


---
아직 Cypress는 공부해보지 못하고 Playwright를 먼저 공부해봐서 그런가 Playwright가 압도적으로 좋아 보인다.  
다음에는 Cypress나 다른 E2E 테스트 도구를 공부해보고 내 프로젝트에는 어떤 도구를 도입할지 한 번 깊게 생각해보고, 팀 프로젝트에서는 친구들이랑도 깊게 이야기 나눠보고 싶다.  
늘 느끼는 거지만, 프로젝트를 진행하는 것도 재밌지만 프로젝트를 진행하기 전에 어떤 도구를 어떤 이유로 도입할지 생각해보고 이야기 나누는 과정이 뭔가 성장하는 느낌도 들고  
내가 이 프로젝트에서 왜 이 도구를 도입했는지 설명할 수 있게 되니까 의미 있다는 생각도 든다.  
테스트를 공부하면 할 수록 내 생각보다 훨씬 많이 유용하고 "이것도 할 수 있네?"가 많아서 재밌다.  
