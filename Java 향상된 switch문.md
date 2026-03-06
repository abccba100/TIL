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
int number = 1;

String result = switch (number) {
    case 1 -> "홀수";
    case 2 -> "짝수";
    default -> "범위를 벗어난 숫자";
};

System.out.println(result); // => "홀수"
```

break를 생략할 수 있다 단 모든 case가 반환하는 자료형이 같아야 한다.  

---

빠진 내용도 하나씩 추가해보고 꾸준히 복습해야겠다
나중에 풀스택 개발자가 될 생각하니 벌써 설렌다
