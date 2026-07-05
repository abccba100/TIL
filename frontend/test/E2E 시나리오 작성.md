## 1. E2E 시나리오


### 1.1 개념

E2E 테스트는 코드 단위가 아니라 사용자가 실제로 하는 행동을 기준으로 시나리오를 설계한다.

처음에는 함수나 컴포넌트처럼 코드 단위로 생각하게 되는데, E2E는 "유저가 이 서비스를 어떻게 쓰는가"를 먼저 생각하고 그걸 코드로 옮기는 방식이다. 그래서 테스트 코드 짜기 전에 어떤 흐름을 테스트할지 먼저 설계해야 한다.

## 2. Happy Path vs Edge Case


### 2.1 Happy Path

아무 문제 없이 정상적으로 흘러가는 시나리오다. 제일 먼저 작성해야 하는 시나리오이고, 핵심 기능이 동작하는지 검증한다.

**ex) 로그인 happy path**

1. 이메일, 비밀번호 입력
2. 로그인 버튼 클릭
3. 대시보드로 이동

### 2.2 Edge Case

정상 흐름에서 벗어난 예외 상황이다. happy path가 통과된 다음에 작성한다.

**ex) 로그인 edge case**

- 비밀번호 틀렸을 때 에러 메시지 표시
- 이메일 형식이 아닐 때 검증 메시지
- 네트워크 오류 시 처리

## 3. 시나리오 설계 기준


### 3.1 E2E로 커버할 것

모든 흐름을 E2E로 짜는 게 아니라 기준을 잡고 선택해야 한다.

- **비즈니스 핵심 흐름**
    
    터지면 서비스 자체가 마비되는 것
    
- **여러 페이지를 걸치는 흐름**
    
    단일 컴포넌트 테스트로는 못 잡는 것
    
- **회귀 가능성이 높은 흐름**
    
    과거에 버그가 났던 곳
    

### 3.2 E2E로 커버하지 않을 것

- **버튼 색상, 폰트 크기 같은 UI 디테일**
- **단순 렌더링 확인**
    
    컴포넌트 테스트 영역이라서 E2E에서는 진행하지 않음
    
- **모든 edge case**
    
    다 짜면 유지보수가 너무 힘들어져 중요한 것만
    

## 4. POM (Page Object Model)


### 4.1 왜 필요한가

시나리오가 많아지면 같은 요소를 여러 테스트에서 반복해서 찾게 된다는 걸 처음에 몰랐다.

로그인 버튼 locator를 10개 테스트에서 각각 쓰다가 버튼 텍스트가 바뀌면 10군데를 다 고쳐야 하는 상황이 온다.

### 4.2 개념

POM은 페이지 단위로 locator와 액션을 한 곳에 모아서 재사용하는 패턴이다.

### 4.3 class vs 함수

Playwright 공식 문서가 class 예시를 많이 보여주는데, React에서는 class를 거의 안 쓰니까 꼭 class를 써야 하는지 궁금했다.  
POM은 패턴이지 class를 강제하는 게 아니라서 함수로도 독같이 구현할 수 있다.  
React 코드베이스와 통일하고 싶으면 함수형으로 써도 전혀 문제 없어서 팀 콘벤션 따라가면 될 것 같다.

```jsx
// class 방식
class LoginPage {
  constructor(page) {
    this.page = page
    this.emailInput = page.getByPlaceholder('이메일')
    this.passwordInput = page.getByPlaceholder('비밀번호')
    this.loginButton = page.getByRole('button', { name: '로그인' })
  }

  async login(email, password) {
    await this.emailInput.fill(email)
    await this.passwordInput.fill(password)
    await this.loginButton.click()
  }
}

// 함수 방식
function createLoginPage(page) {
  return {
    emailInput: page.getByPlaceholder('이메일'),
    passwordInput: page.getByPlaceholder('비밀번호'),
    loginButton: page.getByRole('button', { name: '로그인' }),

    async login(email, password) {
      await this.emailInput.fill(email)
      await this.passwordInput.fill(password)
      await this.loginButton.click()
    }
  }
}
```


---
E2E 테스트라고 하면 어떤 작업이나 하나의 흐름 같은 걸 테스트한다고 해서 "그냥 그렇구나" 생각했는데,  
막상 나한테 "E2E 테스트 시나리오 작성해서 코드로 작성해주세요" 라고 요청하면 어디서부터 해야할지 막막할 것 같기도 하고  
아직 어떤 식으로 시나리오를 짜야할지 잘 감이 안잡힌다. 특히 흐름이 계속 이어질 때 어디서 끊어야 할지 잘 모르겠다.  
이건 경험의 영역 같기도 해서 다른 사람들이 작성한 E2E 코드를 보면서 시나리오를 따라가보면 조금은 이해가 되지 않을까 싶다.  