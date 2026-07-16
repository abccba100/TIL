## 1. JS 문제


### 1.1 동적 타입

동적 타입은 어떻게 보면 JS의 장점이자 단점이다.  
JS에서 TS로 넘어가는 이유가 여러개 있겠지만, 대부분은 동적 타입이라는 문제점에서 뻗어나가는 것 같다.  

아래처럼 `sumNumber` 라는 a, b 숫자를 받아서 더한 후에 반환해주는  함수가 있을 때 딱히 별 문제가 없어 보인다.

```jsx
const sumNumber = (a, b) => {
	return a + b
}
```

아래 처럼 함수에 값을 넘겨도 코드 동작에는 전혀 문제가 없다.  
’우리가 이 함수의 목적과 다르게 인자를 하나만 넘기거나 문자를 넘겨도 오류가 나지 않는다.’ 점이 문제인 것이다.  

```jsx
sumNumber(100); // NaN
sumNumber("a", "b"); // ab
```

## 2. 해결 방안


### 2.1 JSDoc

JSDoc은 JS 코드에 주석으로 타입/설명 등을 달아서 타입 및 에러 확인 등도 가능한 문서화 표준인이다.  

JSDoc을 이용할 땐 아래처럼 코드를 작성할 수 있다.  
하지만 JSDoc도 TS를 대체하기에는 주석의 성격이 강하고 강제성을 부여해 사용하기에는 어렵다는 문제가 있다.  

```jsx
// @ts-check

/**
 * @param {string} name
 * @returns {number}
 */
function getLength(name) {
  return name.length;
}

getLength(123); // 에러: Argument of type 'number' is not assignable to parameter of type 'string'.
```

### 2.2 propType

propType은 React 컴포넌트에서 props의 타입을 검사하기 위해 사용할 수 있는 속성이다.  

propType을 이용할 땐 아래처럼 코드를 작성할 수 있다.  
하지만 propType도 마찬가지로 TS를 대체하기에는 타입 불일치 시에 에러가 아닌 경고를 띄우는 점, 컴포넌트가 아닌 전체 애플리케이션의 타입 검사를 하느데는 사용할 수 없다는 점, React에서만 사용할 수 있다는 점 등의 문제가 있다.  

```jsx
import PropTypes from 'prop-types';

function Greeting({ name, age }) {
  return <div>{name} is {age} years old</div>;
}

Greeting.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};

<Greeting name={123} /> // 콘솔에 warning만 뜸, 앱은 그대로 돌아감
```

### 2.3 Dart

Dart는 구글에서 JS를 대체하기 위해 만든 언어로 JS의 동적 타입 문제를 근본적으로 해결했지만,  
과거 (인터넷 익스플로러 vs 넷스케이프 내비게이터) 같은 브라우저 전쟁처럼 JS와 Dart로 개발의 파편화를 불러올 수 있어 Dart를 바라보는 시선이 좋지 않아 웹 개발에서 잘 쓰이지 않았다.  
결론적으로 웹 개발에서 JS의 동적 타입 문제를 해결하지 못했다.  

## 3. TS 사용 이유


### 3.1 안정성 보장

TS는 컴파일 단계에서 타입 검사를 해주기 때문에 JS에서 일어나던 타입 에러를 줄일 수 있고, 런타임 에러를 사전에 방지할 수 있다는 점에서 안정성을 보장할 수 있다.

### 3.2 개발 생산성 향상

IDE에서 제공하는 타입 자동 완성 기능으로 변수와 함수의 타입을 추론한다.  
React에서 어떤 prop을 넘겨야 하는지 매번 직접 확안하지 않아도 사용부에서 바로 확인이 가능하기 때문에 개발 생산성이 향상된다.

### 3.3 협업에 유리

코드에서 정의된 인터페이스를 읽으면서 코드를 보게 되기 때문에 복잡한 애플리케이션 개발 및 협업에서도 빠른 코드 이해가 가능해 협업에서 유리하다.



---
우아한 타입 스크립트 책을 읽으면서 TS 문법 심화나 개념을 탄탄하게 다지기 위해서 책을 빌리고 오늘 조금 읽었다.  
그런데 정작 지금까지 TS를 쓰는 이유라고 해봤자 '안정성' 정도로만 생각했지 진짜 정확히 JS에서 어떤 부분이 문제여서 TS를 쓰고 지금까지 어떤 해결 방안들이 있었고 왜 근본적으로 문제를 해결하지 못 했는지는 깊게 생각해본 적이 없어서 이렇게 TS 사용 이유를 정리해보게 되었다.  