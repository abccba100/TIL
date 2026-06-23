## 1. Lazy Loading 기본 개념


### 1.1 Lazy Loading이란?

말은 어렵게 생겼지만, Lazy Loading이란 필요한 시점에 필요한 것만 불러오는 것이다.

반대 개념은 Eager Loading으로, 앱 시작할 때 모든 리소스를 미리 다 불러오는 방식이다.

### 1.2 필요 이유

앱을 처음 열면 브라우저는 JS 파일을 전부 다운로드하고 파싱한다. 근데 사용자가 지금 보는 건 메인 화면 하나인데, 설정 화면, 마이페이지, 통계 화면 JS까지 전부 받아오고 있는 것이다.

```
앱 실행
-> 메인화면 JS 다운로드
-> 설정 JS 다운로드       <- 지금 안 봄
-> 마이페이지 JS 다운로드  <- 지금 안 봄
-> 통계 JS 다운로드       <- 지금 안 봄
-> 화면 표시 (이미 몇 초 지남)
```

이렇게 되면 사용자는 실제로 필요하지도 않은 코드가 다 받아질 때까지 기다려야 하기 때문에 초기 로딩이 느려진다.

## 2. Lazy Loading의 종류


### 2.1 이미지 Lazy Loading (브라우저 기본)

브라우저에서 `loading="lazy"` 속성 하나만 추가하면, 스크롤해서 이미지가 보일 때 로드된다.

html에서 lazy loading을 사용하는 예시는 다음과 같다.

```html
<!-- 나쁜 예 → 페이지 열리자마자 이미지 전부 다운로드 -->
<img src="photo.jpg" />

<!-- 좋은 예 → 스크롤해서 보일 때 다운로드 -->
<img src="photo.jpg" loading="lazy" />
```

### 2.2 JS 코드 분할 (Code Splitting)

번들러(Vite, Webpack)가 JS를 여러 청크로 나눠서, 해당 화면에 진입할 때만 그 청크를 네트워크로 가져오는 것이다.

코드를 작성하는 방식 자체를 바꾸는 게 아니라, `import()` 문법을 통해 번들러에게 "이건 나중에 가져와도 돼"라고 알려주는 방식이다.

### 2.3 React.lazy + Suspense

`React.lazy`를 사용하면 컴포넌트가 처음 렌더링될 때 JS를 가져온다.

아래는 `React.lazy` 를 사용하는 예시 코드이다.

`Suspense`의 `fallback`은 JS를 가져오는 동안 보여줄 로딩 UI를 넣으면 된다.

```jsx
const MyPage = React.lazy(() => import('./MyPage'));

function App() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <MyPage />
    </Suspense>
  );
}
```

## 3. Lazy Loading 적용하기


### 3.1 Lazy Loading을 적용하지 않은 경우

아래는 모든 페이지를 앱 시작 시 한꺼번에 import하는 코드다.

```jsx
import HomePage from './pages/HomePage';
import MyPage from './pages/MyPage';
import SettingPage from './pages/SettingPage';
import StatPage from './pages/StatPage';

function App() {
  return (
    <Router>
      <Route path="/" component={HomePage} />
      <Route path="/my" component={MyPage} />
      <Route path="/setting" component={SettingPage} />
      <Route path="/stat" component={StatPage} />
    </Router>
  );
}
```

아래와 같은 문제가 발생한다.

- 사용자가 `/` 메인 화면만 보더라도 MyPage, SettingPage, StatPage JS까지 전부 다운로드된다.
- 앱이 커질수록 초기 로딩 시간이 점점 느려진다.
- 사용자는 아직 가지도 않은 화면의 코드를 기다리고 있는 셈이다.

### 3.2 Lazy Loading을 적용한 경우

`React.lazy`로 각 페이지를 감싸고, `Suspense`로 로딩 UI를 처리한다.

```jsx
const HomePage = React.lazy(() => import('./pages/HomePage'));
const MyPage = React.lazy(() => import('./pages/MyPage'));
const SettingPage = React.lazy(() => import('./pages/SettingPage'));
const StatPage = React.lazy(() => import('./pages/StatPage'));

function App() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <Router>
        <Route path="/" component={HomePage} />
        <Route path="/my" component={MyPage} />
        <Route path="/setting" component={SettingPage} />
        <Route path="/stat" component={StatPage} />
      </Router>
    </Suspense>
  );
}
```

이렇게 하면 사용자가 `/my` 로 이동할 때 비로소 MyPage JS를 받아온다.
클린 코드는 짧은 코드가 아니듯, Lazy Loading도 코드가 줄어드는 게 아니라 **필요한 시점에 필요한 것만 처리하는 것**이 핵심이다.

## 4. Eager Loading 기본 개념


### 4.1 Eager Loading이란?

말은 Lazy Loading의 반대지만, Eager Loading이란 필요 여부와 관계없이 미리 다 불러오는 것이다.

무조건 나쁜 방식은 아니다.

### 4.2 Eager Loading이 더 나은 경우

**1. 무조건 쓰는 리소스**

앱 시작하면 100% 보이는 헤더, 네비게이션, 폰트 같은 것들. 어차피 바로 필요하니까 미리 불러오는 게 맞다.

**2. 첫 화면이 로그인 페이지인 경우**

앱 시작하면 대부분 로그인이 첫 화면이다. 이걸 Lazy로 하면 오히려 첫 화면이 더 늦게 뜬다.

**3. Prefetching**

사용자가 곧 이동할 것 같은 페이지를 미리 받아두는 것. 쇼핑몰에서 상품 목록을 보고 있으면 상세 페이지로 이동할 가능성이 높으니까, 그 JS를 미리 받아두는 방식이다.

**4. 의도적인 Eager Loading (극단적인 경우)**

로딩 화면을 띄워놓고 그동안 모든 리소스를 다 받아버리는 방식이다. 중간에 끊기는 게 UX적으로 훨씬 나쁜 상황에서 의도적으로 선택한다.

예) 게임 인트로 로딩 화면, 오프라인 지원 앱, 주식 트레이딩처럼 화면 전환 딜레이가 치명적인 실시간 앱

### 4.3 결국 기준은 하나

나쁜 예 → 무조건 Lazy

```jsx
// 로그인 페이지도 Lazy로 하면 첫 화면이 느려진다
const LoginPage = React.lazy(() => import('./pages/LoginPage'));
```

좋은 예 → 상황에 맞게

```jsx
// 첫 화면은 Eager
import LoginPage from './pages/LoginPage';

// 이후 화면은 Lazy
const MyPage = React.lazy(() => import('./pages/MyPage'));
const StatPage = React.lazy(() => import('./pages/StatPage'));
```

**무조건 Lazy가 좋은 게 아니라, 필요한 시점에 맞게 불러오는 게 핵심이다.**

---

프론트엔드 최적화 관련해서 공부를 하면서 느끼는 점이 "이거 하나 적용한다고 얼마나 빨라지겠어" 이런 생각을 많이 했는데  
결국에는 이런 기술 하나 하나가 모여서 큰 차이를 만들어내고 사용자가 사용하기 좋은 서비스를 만드는 첫 걸음이 되는 것 같다고 생각한다.  
Lazy loading 같은 기술을 사용할 때도 무작정 사용하는 게 아니라 왜 이 기술을 사용해야 하는지, 적합한지 등을 따져보고 사용하는 개발자가 되고 싶다.  
그래서 다음에는 Lazy loading에 반대 개념인 Eager loading도 공부해볼까 한다.  
