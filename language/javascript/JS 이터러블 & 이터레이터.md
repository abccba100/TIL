## 1. Symbol


### 1.1 Symbol 개념

JavaScript에서 다른 개념에 비해 중요하다고 생각하는 개념은 아니지만, 이터러블 & 이터레이터 개념과 연관이 있기 때문에 간단하게만 정리하면 아래와 같다.  

Symbol은 이런 식으로 절대 겹치지 않는 유일한 값을 만들어내는 타입이다.

```jsx
const a = Symbol('id');
const b = Symbol('id');

a === b // false! 이름이 같아도 다른 값
```

이런 Symbol은 `Symbol.iterator` 은 이터러블 메서드에 접근할 수 있게 해준다.

## 2. 이터러블 (Iterable) & 이터레이터 (Iterator)


### 2.1 이터러블이란?

이터러블은 순서대로 값을 꺼낼 수 있다고 선언된 객체이다.

쉽게 판단하는 방법은 `Symbol.iterator` 메서드를 가지고 있으면 이터러블이다.

```jsx
// 배열은 이터러블
const arr = [1, 2, 3];
arr[Symbol.iterator]; // 함수가 있음 -> 이터러블 O

// 일반 객체는 이터러블이 아님
const obj = { a: 1 };
obj[Symbol.iterator]; // undefined -> 이터러블 X
```

### 2.2 이터레이터란?

이터레이터는 실제로 값을 하나씩 꺼내주는 객체이다.

- 왜 객체일까?
    
    `Symbol.iterator` 로 접근해서 메서드를 실행시키면 아마도 값을 꺼내주는 메서드들을 반환해주는 객체를 반환해주는 것 같다.
    

`next()` 메서드를 호출하면 { value, done } 형태로 값을 반환한다.  
여기서 done은 해당 이터러블의 끝인지를 의미한다.

```jsx
const arr = [10, 20, 30];
const iter = arrSymbol.iterator; // 이터레이터 생성

iter.next(); // { value: 10, done: false }
iter.next(); // { value: 20, done: false }
iter.next(); // { value: 30, done: false }
iter.next(); // { value: undefined, done: true } -> 끝
```

for of 문은 이터러블을 이터레이터를 이용해서 순회하기 때문에 앞서 말했던 내용들에 따라  
굳이 배열이 아니더라도 이터러블이라면 for of 문을 이용해서 순회가 가능하다.

### 2.3 이터러블과 이터레이터 관계

이터러블과 이터레이터의 관계를 나타내면 아래와 같다.

```
이터러블 (Iterable)
	└─ Symbol.iterator 호출
				└─ 이터레이터 (Iterator) 반환
							└─ next() 호출 → { value, done }
```

---

이터러블과 이터레이터도 심화 개념까지 가면 많이 복잡해서 공부를 망설이긴 했지만,  
그래도 기본 수준 정도는 공부하고 나니 for of문 동작 원리도 알게 되고 왜 배열이 아닌 다른 자료형들도 순회가 가능했는지 이해가 되어서 좋은 학습이었다고 생각된다.
