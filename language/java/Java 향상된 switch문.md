## 1. 향상된 switch문


### 1.1 원래 switch문

```java
int number = 1;

switch (number) {
    case 1:
        System.out.println("홀수");
        break;
    case 2:
        System.out.println("짝수");
        break;
    default:
        System.out.println("범위를 벗어난 숫자");
        break;
}
```

매 case 마다 break를 써줘야하고 쓰기 불편하다  

### 1.2 향상된 switch문

```java
String result = switch (number) {
    case 1, 3, 5 -> "홀수";
    case 2, 4, 6 -> 2; // => 컴파일 에러
    default -> "범위를 벗어난 숫자";
}
```

break를 생략할 수 있다 단 모든 case가 반환하는 자료형이 같아야 한다.  
예시 코드처럼 반환 값의 자료형이 서로 다르면 컴파일 에러가 난다.  

### 1.3 yield 사용

```java
int number = 2;

String result = switch (number) {
    case 1, 3, 5 -> {
        System.out.println("홀수입니다!");
        yield "홀수";
    }
    case 2, 4, 6 -> {
        System.out.println("짝수입니다!");
        yield "짝수";
    }
    default -> {
        System.out.println("범위를 벗어난 숫자");
        yield "?";
    }
};
```

향상된 switch문에서는 yield를 사용해서 값을 반환할 수 있다.  

switch문은 함수가 아니기 때문에 return 키워드를 쓰면 안된다.  

---

빠진 내용도 하나씩 추가해보고 꾸준히 복습해야겠다  
나중에 풀스택 개발자가 될 생각하니 벌써 설렌다  

추가로 yield에 대해 공부했는데 꽤 중요한 키워드 같다.  
앞으로도 이렇게 작더라도 꾸준하게 공부해야겠다.  
