## 1. 클래스 상속


### 1.1 상속 개념

자바에서 상속은 연관 있는 클래스들의 공통 요소를 하나의 클래스로 뽑아내고, 그걸 다른 클래스가 물려받아 쓰는 방식이다.

```java
// 상속 안 쓴 버전 - 중복 많음
class Dog {
    int legCount;
    int tailCount;
    void bark() { }
}

class Cat {
    int legCount;
    int tailCount;
    void meow() { }
}
```

```java
// 상속 쓴 버전 - 공통 요소를 Animal로 뽑아냄
class Animal {
    int legCount;
    int tailCount;
}

class Dog extends Animal {
    void bark() { }
}

class Cat extends Animal {
    void meow() { }
}
```

기존에 있던 클래스(Animal)를 부모 클래스 또는 상위 클래스라 부르고, 상속받아 새로 만든 클래스(Dog, Cat)를 자식 클래스 또는 하위 클래스라 부른다.

상속의 장점은 다음과 같다.

- 코드 중복 제거
- 클래스 간 관계가 명확해져 가독성 향상
- 공통 코드만 고치면 되므로 유지보수 편의성 증가

자바는 클래스 하나당 부모 클래스를 딱 하나만 가질 수 있다. 두 부모 클래스가 같은 이름의 메서드를 갖고 있으면 어느 쪽 메서드를 써야 할지 애매해지기 때문이다.

### 1.2 상속 클래스의 생성자 호출 순서

자식 클래스를 인스턴스화하면, 부모 클래스의 생성자가 먼저 실행되고 그 다음 자식 클래스의 생성자가 실행된다.

```java
class Parent {
    Parent() {
        System.out.println("부모 생성자 실행");
    }
}

class Child extends Parent {
    Child() {
        System.out.println("자식 생성자 실행");
    }
}

public class Main {
    public static void main(String[] args) {
        Child ch = new Child();
        // 실행 결과:
        // 부모 생성자 실행
        // 자식 생성자 실행
    }
}
```

자식 클래스는 부모 클래스의 필드나 기능을 물려받아 쓰는 구조이므로, 부모가 먼저 온전히 초기화되어 있어야 자식이 그 위에서 안전하게 초기화될 수 있다.

### 1.3 super 키워드

super는 부모 클래스로부터 상속받은 필드나 메서드를 자식 클래스에서 참조할 때 쓰는 키워드다. this가 자기 자신을 가리키듯, super는 부모를 가리킨다.

```java
class Parent {
    int a = 10;
}

class Child extends Parent {
    int a = 20;

    void display() {
        System.out.println(a);       // 20 (자식 변수)
        System.out.println(this.a);  // 20 (자식 변수, 명시적)
        System.out.println(super.a); // 10 (부모 변수)
    }
}
```

부모와 자식이 같은 이름의 필드를 갖고 있을 때, super를 붙이면 부모 쪽 필드를 명확히 지정해서 가져올 수 있다.

### 1.4 super() 메서드

super()는 부모 클래스의 생성자를 호출하는 문법이다. this()가 같은 클래스의 다른 생성자를 호출한다면, super()는 부모 클래스의 생성자를 호출한다.

자바는 자식 클래스 생성자의 첫 줄에 자동으로 super()가 숨어서 실행되도록 정해져 있다. 그래서 직접 안 써도 부모의 매개변수 없는 생성자가 알아서 호출된다.

```java
class Parent {
    int a;
    int b;
}

class Child extends Parent {
    int c;

    Child() {
        // super(); 가 생략되어 있음
        c = 20;
    }
}
```

문제는 부모 클래스에 매개변수 있는 생성자만 직접 정의해놨을 경우다. 부모가 생성자를 하나라도 직접 만들면 부모의 기본 생성자는 더 이상 자동으로 생기지 않는다.

```java
class Employee {
    String name;

    Employee(String name) { // 매개변수 있는 생성자를 직접 만듦
        this.name = name;
    }
    // Employee()라는 기본 생성자는 이제 존재하지 않음
}

class Developer extends Employee {
    double salary;

    Developer(String name) {
        // 여기서 자동으로 super()가 실행되려 하는데
        // Employee()가 없어서 컴파일 에러
    }
}
```

이럴 땐 자식 생성자에서 super(name)처럼 부모 생성자의 시그니처에 맞게 직접 호출해줘야 한다.

```java
class Developer extends Employee {
    double salary;

    Developer(String name) {
        super(name); // 부모의 Employee(String name) 생성자를 명시적으로 호출
    }
}
```

super()도 this()와 마찬가지로 반드시 생성자의 첫 줄에서만 호출 가능하다.

### 1.5 메서드 오버라이딩

오버라이딩은 부모 클래스에 이미 정의된 메서드를 자식 클래스에서 같은 시그니처로 재정의하는 것이다. 오버로딩(같은 이름, 다른 매개변수로 여러 개 정의)과는 다른 개념이라 구분이 필요하다.

- **오버로딩**
    
    메서드 이름 동일, 매개변수 다름, 리턴 타입 상관없음
    
- **오버라이딩**
    
    메서드 이름 동일, 매개변수 동일, 리턴 타입 동일
    

```java
class Parent {
    void display() {
        System.out.println("부모의 display()");
    }
}

class Child extends Parent {
    void display() {
        System.out.println("자식의 display()");
    }
}

public class Main {
    public static void main(String[] args) {
        Child ch = new Child();
        ch.display(); // "자식의 display()" - 오버라이딩된 버전이 실행됨
    }
}
```

오버라이딩 조건은 메서드 선언부(이름, 매개변수, 리턴 타입)가 기존 메서드와 완전히 같아야 한다. 그리고 부모 메서드보다 접근 제어자를 더 좁은 범위로 줄일 수 없다(부모가 protected면 자식은 protected나 public만 가능, private로 좁히면 컴파일 에러).

부모 메서드를 자식 안에서 호출하고 싶을 때는 super를 붙이면 된다.

```java
class Child extends Parent {
    void display() {
        super.display(); // 부모의 display()를 먼저 실행
        System.out.println("자식의 추가 로직");
    }
}
```


---
문법적으로는 대충 보면 비슷해 보였는데 규칙이라든지 동작 같은 게 Java가 JS에 비해서 더 빡빡한 느낌이 드는 것 같다.  
오히려 이런 규칙, 동작 같은 게 빡빡하고 까다로운 언어가 더 마음에 드는 것 같기도 하다.  
