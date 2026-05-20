## 1. Sass


### 1.1 Sass 개념

Sass는 CSS 전처리기 언어로 CSS의 확장 언어에 가깝다.

- CSS 전처리기 언어란?
    
    브라우저는 CSS만 해석할 수 있다. 하지만, Sass는 순수 CSS가 아닌 CSS를 프로그래밍 언어처럼 쓰려고 만든 확장 언어이기 때문에 바로 해석할 수 없기 때문에 Sass를 CSS로 바꿔주는 걸 CSS 전처리기라고 한다.
    

Sass는 Syntactically Awesome Style Sheets의 약자로 직역하면 구문론적으로 놀라운 스타일 시트인데,  
나중에 문법을 보면 기존 CSS랑 비교했을 때 매우 놀랍기는 하다.

### 1.2 Sass 등장 배경

지금도 CSS를 쓰다 보면 느끼는 게 계속 쓰다보면 코드를 쓴 나조차도 헷갈린다.  
코드가 길어지면서 읽기도 힘들고 복잡해지고, 중요한 점이 중복되는 코드가 많아진다.  
위와 같은 문제점들을 모두 해결하기 위해 나온 게 Sass다.

### 1.3 Sass의 장점

1. **변수 사용**
    
    Sass는 아래와 같이 변수를 선언하고 사용할 수 있다.
    
    ```sass
    $primary: #2ecc71;
    
    .button {
      background: $primary;
    }
    ```
    
2. **중첩 셀렉터**
    
    기존 CSS와 Sass의 문법을 비교해서 보면 Sass가 왜 aswome한지 알 수 있다.
    Sass가 훨씬 더 직관적이고 높은 가독성을 제공한다.
    
    ```css
    /* CSS */
    .card .title {
    }
    
    .card .description {
    }
    ```
    
    ```sass
    // Sass
    .card {
      .title {
      }
    
      .description {
      }
    }
    ```
    
3. **중첩 프로퍼티**
    
    중첩되는 프로퍼티를 하나의 프로퍼티 안에서 작성할 수 있다.
    
    ```css
    /* CSS */
    .container {
    	flex-direction: column;
    	flex-wrap: nowrap;
    }
    ```
    
    ```sass
    // Sass
    .container {
    	flex: {
    		direction: column;
    		wrap: nowrap;
    	}
    }
    ```
    
4. **믹스인 (mixin)**
    
    공통 스타일 선언하고 사용할 수 있게 하여 코드 재사용을 높이고 중복 코드를 줄였다.
    
    ```sass
    @mixin flex-center {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    
    .box {
      @include flex-center;
    }
    ```
    

그 외에도 미디어쿼리 중첩, 연산자 활용, 사칙연산, 상속 등 많은 기능들을 제공한다.

## 2. SCSS


### 2.1 SCSS 개념

SCSS도 Sass와 마찬가지로 CSS 전처리기 언어다.

사실 SCSS도 Sass 기술이라는 큰 개념에 포함되는 것이라 설명할 게 없다.

Sass (전처리기 전체를 뜻하는 브랜드/기술 이름)  
├── 1. Sass 문법 (.sass 파일 / 들여쓰기 방식)  
└── 2. SCSS 문법 (.scss 파일 / CSS 친화 방식)  

이런 식으로 Sass라는 기술 아래에 Sass 문법과 SCSS 문법이 있는 것인데 위에서 소개한 Sass 장점의 문법들은 모두 SCSS의 문법이다.

### 2.2 Sass/SCSS 문법 비교

Sass 문법은 아래처럼 `;` 과 `{}` 을 사용하지 않는다.

```sass
$primary: #2ecc71

.button
  background: $primary
```

SCSS 문법은 위와 마찬가지 이지만, `;` 과 `{}` 를 사용하고 여러 추가 문법들을 제공한다.

```scss
$primary: #2ecc71

.button
  background: $primary
```


---
듣기는 많이 듣기도 하고 실제로 .sass 파일이나 .scss 파일도 몇번 보긴 했는데, 실제로 공부하고 보니까 생각보다 더 유용한 것 같다 나중에 토의 프로젝트에 한 번 적용시켜 보면 재밌을 듯 싶다.
