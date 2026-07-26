## 1. 접근 제어자(access modifier)


### 1.1 개념

클래스, 필드, 메서드, 생성자에 붙여서 어디까지 접근을 허용할 것인가를 정하는 키워드다.

자바는 네 단계로 나뉜다.

- `public` — 어디서든 접근 가능
- `protected` — 같은 패키지 + 상속 관계까지 접근 가능
- default(생략) — 같은 패키지 안에서만 접근 가능
- `private` — 같은 클래스 내부에서만 접근 가능

### 1.2 public

어디서든 접근 가능하다. 패키지가 다르든 상속 관계가 없든 상관없이 전부 열려있다.

```java
public class Car {
    public String modelName;

    public void drive() { }
}
```

### 1.3 private

선언된 클래스 내부에서만 접근 가능하다. 다른 클래스는 물론이고, 자기 자신을 상속받은 자식 클래스에서도 접근 못 한다.

```java
class Car {
    private int speed;

    void accelerate() {
        speed = 10; // 같은 클래스 내부라 접근 가능
    }
}

class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.speed = 10; // 컴파일 에러! 클래스 외부에서 private 접근 불가
    }
}
```

private는 캡슐화를 구현할 때 쓰는 제어자다. 캡슐화란 필드를 외부에서 직접 못 건드리게 숨기고, 정해진 메서드(getter/setter)를 통해서만 접근하게 만드는 설계 원칙이다.

```java
class Car {
    private int speed;

    public int getSpeed() {
        return speed;
    }

    public void setSpeed(int speed) {
        this.speed = (speed < 0) ? 0 : speed; // 검증 로직을 한 곳에 모음
    }
}
```

필드를 public으로 열어두면 이런 검증 없이 아무 값이나 들어갈 수 있어서, private + getter/setter 조합이 표준 관습으로 자리잡았다.

### 1.4 default(아무것도 안 붙인 상태)

제어자를 아예 안 쓰면 default가 적용된다. 같은 패키지 안에서만 접근 가능하다.

여기서 패키지란 클래스들을 묶어 관리하는 폴더 단위다. `package` 문으로 선언하고, 실제 디렉토리 구조도 그 경로를 따라간다.

```java
package com.example.myapp;

class Car { }
```

같은 패키지란 같은 package 문으로 선언된 클래스들끼리를 말한다.

```
com/example/a/Parent.java   → package com.example.a;
com/example/a/Sibling.java  → package com.example.a;  ← Parent와 같은 패키지
com/example/b/Other.java    → package com.example.b;  ← Parent와 다른 패키지
```

```java
class Car { // 제어자 생략 → default
    int speed; // 제어자 생략 → default
}
```

Sibling은 Parent와 같은 패키지라 default 필드에 접근 가능하지만, Other는 다른 패키지라 접근이 막힌다.

### 1.5 protected

같은 패키지 안에서는 default처럼 접근 가능하고, 추가로 패키지가 달라도 상속 관계에 있는 자식 클래스에서는 접근 가능하다.

```java
package com.example.a;

public class Parent {
    protected int a;
}
```

```java
package com.example.b;

import com.example.a.Parent;

class Child extends Parent {
    void display() {
        System.out.println(a); // 패키지는 다르지만 상속 관계라 접근 가능
    }
}
```

같은 패키지 또는 상속 관계, 두 조건 중 하나만 만족해도 되는 구조라 default보다 범위가 넓다.

## 2. 기타 제어자


### 2.1 static

인스턴스가 아니라 클래스 자체에 속한다. 모든 인스턴스가 하나의 값을 공유하고, 인스턴스 없이 클래스명으로 바로 호출 가능하다.

```java
class Car {
    static int totalCount = 0; // 모든 인스턴스가 공유

    Car() {
        totalCount++; // 객체 생성될 때마다 공유 값 증가
    }
}
```

### 2.2 final

더 이상 변경할 수 없다는 뜻으로, 붙는 대상에 따라 의미가 다르다.

- 필드에 붙으면 재할당 불가(상수화)
- 메서드에 붙으면 자식 클래스에서 오버라이딩 금지
- 클래스에 붙으면 상속 자체가 금지

```java
class Car {
    final int maxSpeed = 200; // 필드: 재할당 불가
}

class Parent {
    final void display() { } // 메서드: 오버라이딩 불가
}

final class Engine { } // 클래스: 상속 불가
```

상수를 선언할 때는 static과 final을 같이 써서, 클래스에 하나만 존재하고 절대 안 바뀌는 값으로 만든다.

```java
class MathConstant {
    static final double PI = 3.14159;
}
```

## 3. 제어자 조합 규칙

---

### 3.1 규칙

- 접근 제어자는 하나만 골라야 한다
    
     public과 private를 동시에 쓸 수는 없음
    
- static과 final은 접근 제어자와 별개라서 같이 쓸 수 있다
    
    접근 제어자를 먼저, 그다음 static이나 final을 쓰는 게 관례
    
- abstract와 final은 동시에 쓸 수 없다
    
    abstract는 반드시 구현하라는 뜻이고 final은 확장 불가라는 뜻이라 의미가 모순됨
    

```java
public static final double PI = 3.14159; // 접근 제어자 + static + final
```


---
이제 뭔가 진짜 Java 공부하는 느낌이 든다 빨리 spring boot까지 배워서 토이 프로젝트라도 하나 하고 싶은 마응이 크다.
