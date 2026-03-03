## 1. Java 시작

### 1.1 Java 느낌

Java가 어떤 느낌의 언어인지만 알아봄

```
  java
public class Main {
    public static void main(String[] args) {
        System.out.println(12);
    }
}
```

[test.java] 파일 안에 하나의 class가 있고 그 안에 함수들이 있는 느낌이다

### 1.2 Java 기초 코드

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(12);
    }
}
```

class 앞에 붙는 public 키워드는 프로젝트 내 모든 파일에서 해당 class에 접근할 수 있게 해준다.  
class 앞에 아무것도 안 붙으면 같은 패킷(폴더) 안에서만 class를 쓸 수 있다.  

static 키워드가 붙은 함수는 인스턴스 선언 없이 바로 사용할 수 있다.  
위 코드에서 main 함수는 class 내부에 선언되어 있기 때문에 인스턴스 선언 후 인스턴스에 접근를  
통해 main 함수를 사용한다. 하지만, static 키워드가 붙으면 이런 과정 없이 바로 main 함수를 사용할 수 있다.  

main 함수는 코드를 실행할 때 먼저 실행되는 함수이다. (C언어 main 함수 느낌)  
main 함수는 프로젝트 내에서 하나만 선언될 수 있다.  
