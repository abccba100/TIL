## 1. SEO 기본 개념


### 1.1 SEO란?

말은 어렵게 생겼지만, SEO(Search Engine Optimization)란 검색 엔진이 내 페이지를 잘 찾고 읽을 수 있도록 최적화하는 것이다.

구글에 "맛집 추천"을 검색했을 때 첫 번째로 뜨는 사이트와 10페이지에 뜨는 사이트는 SEO 수준이 다른 것이다. 아무리 잘 만든 서비스도 검색에 안 뜨면 사용자가 찾아올 수 없다.

### 1.2 크롤러가 페이지를 읽는 방식

검색 엔진은 크롤러(Crawler)라는 봇을 이용해서 웹 페이지를 돌아다니며 내용을 수집한다. 크롤러가 페이지를 읽는 방식은 아래와 같다.

```
크롤러 방문
-> HTML 읽기
-> 페이지 제목, 설명, 내용 수집
-> 링크 타고 다음 페이지로 이동
-> 수집한 정보를 검색 엔진 데이터베이스에 저장
-> 검색 결과에 반영
```

여기서 중요한 건 크롤러는 기본적으로 HTML을 읽는다는 것이다. JS를 실행해서 그려지는 화면이 아니라, 서버에서 받아온 HTML 자체를 본다.

## 2. SEO 최적화


### 2.1 시맨틱 태그

시맨틱 태그란 의미를 가진 HTML 태그다. `<div>` 대신 `<header>`, `<main>`, `<article>` 같은 태그를 쓰면 크롤러가 페이지 구조를 더 잘 이해한다.

```html
<!-- 나쁜 예 -> 크롤러가 구조를 파악하기 어렵다 -->
<div class="header">...</div>
<div class="content">...</div>
<div class="footer">...</div>

<!-- 좋은 예 -> 크롤러가 구조를 명확히 파악한다 -->
<header>...</header>
<main>...</main>
<footer>...</footer>
```

### 2.2 meta 태그

meta 태그는 페이지에 대한 정보를 검색 엔진과 브라우저에 알려주는 태그다. 검색 결과에 뜨는 제목과 설명이 여기서 온다.

```html
<head>
  <!-- 검색 결과에 뜨는 제목 -->
  <title>selectD | SQLD 합격을 위한 최선의 선택</title>

  <!-- 검색 결과에 뜨는 설명 -->
  <meta name="description" content="AI 튜터와 함께하는 SQLD 시험 준비" />

  <!-- SNS 공유 시 뜨는 미리보기 -->
  <meta property="og:title" content="selectD" />
  <meta property="og:description" content="AI 튜터와 함께하는 SQLD 시험 준비" />
  <meta property="og:image" content="https://selectd.com/thumbnail.png" />
</head>
```

`og` 태그는 카카오톡이나 슬랙에 링크를 공유했을 때 뜨는 미리보기 카드를 만들어준다.

### 2.3 이미지 alt 속성

크롤러는 이미지를 직접 볼 수 없다. `alt` 속성으로 이미지가 어떤 내용인지 텍스트로 알려줘야 한다.

```html
<!-- 나쁜 예 → 크롤러가 이미지 내용을 모른다 -->
<img src="banner.png" />

<!-- 좋은 예 → 크롤러가 이미지 내용을 파악한다 -->
<img src="banner.png" alt="SQLD 시험 합격을 위한 selectD 앱 소개" />
```

### 2.4 페이지 로딩 속도

구글은 페이지 로딩 속도를 SEO 점수에 반영한다. 느린 페이지는 검색 순위가 낮아진다. 이미지 최적화, Lazy Loading 적용이 여기서 직접 연결된다.

## 3. React와 SEO


### 3.1 CSR이 SEO에 불리한 이유

React는 기본적으로 CSR(Client Side Rendering) 방식이다. CSR은 서버에서 빈 HTML을 받아온 뒤, JS가 실행되면서 화면을 그린다.

```
[CSR 방식]
서버 -> 빈 HTML 전달 -> JS 다운로드 -> JS 실행 -> 화면 렌더링

[크롤러가 보는 것]
<div id="root"></div>  <- 내용이 없다
```

크롤러가 방문했을 때 JS가 아직 실행되기 전이라면, 크롤러는 빈 페이지를 보게 된다. 수집할 내용이 없으니 검색 결과에 제대로 반영되지 않는다.

### 3.2 react-helmet으로 meta 태그 관리

CSR 환경에서도 페이지마다 다른 meta 태그를 넣고 싶을 때 `react-helmet` 라이브러리를 사용한다.

```jsx
import { Helmet } from 'react-helmet';

function ProductPage({ product }) {
  return (
    <>
      <Helmet>
        <title>{product.name} | selectD</title>
        <meta name="description" content={product.description} />
      </Helmet>
      <div>{product.name}</div>
    </>
  );
}
```

페이지마다 `<Helmet>`을 넣으면 해당 페이지에 맞는 title과 meta 태그가 자동으로 바뀐다. 단, CSR의 근본적인 문제(빈 HTML)는 해결되지 않는다.

### 3.3 SSR/SSG로 해결하기

근본적인 해결책은 서버에서 완성된 HTML을 내려주는 SSR(Server Side Rendering) 또는 SSG(Static Site Generation)다.

```
[SSR 방식]
서버 -> 완성된 HTML 전달 -> 크롤러가 내용을 바로 읽을 수 있음
```

Next.js가 React에서 SSR/SSG를 지원하는 대표적인 프레임워크다. SEO가 중요한 서비스(쇼핑몰, 블로그, 랜딩 페이지)는 Next.js를 쓰는 이유가 여기에 있다. SSR/SSG 자체는 별도로 깊게 다룰 주제다.


---

React로 프로젝트할 때 SEO 최적화하면 Next.js 만큼은 아니더라도 조금은 따라잡을 수 있을 거라고 생각했는데  
사실상 크게 의미가 없어서 앞으로 서비스 개발 시에 기술 스택 선택에 대해서 다시 생각해봐야 할 것 같다.
