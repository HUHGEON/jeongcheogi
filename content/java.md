# Java
Java는 2023년 이후 회차마다 1~4문제, 대개 3~4문제씩 나온다. 주제는 꽤 정해져 있는 편이다. 상속·오버라이딩·`super`, 생성자 호출 순서, 오버로딩에서 어느 메서드가 선택되는지, static, 예외 흐름, `==`와 `equals`가 반복되고 키워드 빈칸(`static`, `new`, `super`, `implements`)도 섞여 나온다. 그중에서도 중심이 되는 건, 클래스가 3~4개 얽힌 코드를 주고 "어느 클래스의 메서드가 실행되는가"를 묻는 문제다. 추적 표, 비트, 형변환, 나눗셈 같은 공통 규칙은 [코딩 공통] 페이지에 따로 있다.

## 상속·오버라이딩·동적 바인딩·super
출제: 26-2, 26-1, 25-3, 23-3, 21-2, 20-4
함정: 참조 변수의 타입이 아니라 `new`로 만든 객체의 타입이 어느 메서드가 실행될지 정한다.

### 개념
Java 문제의 중심은 클래스 3~4개가 상속으로 얽힌 코드에서 "결국 어느 클래스의 메서드가 실행되는가"를 묻는 것이다. 이걸 정하는 규칙이 **동적 바인딩**인데, 쉽게 말해 메서드는 변수의 타입이 아니라 실제로 `new`한 객체를 따라간다는 뜻이다. 반대로 필드와 static 메서드는 변수 타입을 따른다. 아래 단계마다 이 차이가 작은 코드 하나씩으로 드러난다.

먼저 상속 문제를 풀 때 쓰는 규칙을 표로 정리하면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 규칙 | 내용 |
|---|---|
| 동적 바인딩 | `Shape s = new Square(3); s.area()`는 Square의 area()를 실행한다. 변수 타입 Shape는 "어떤 메서드를 부를 수 있는가"만 정한다 |
| 오버라이딩 | 부모와 이름·매개변수가 같은 메서드를 자식이 다시 정의. `@Override` 표시는 선택 |
| 부모 메서드 안의 호출 | 부모 메서드 안에서 `area()`를 불러도 자식이 오버라이딩했으면 자식 것이 실행된다. 단, 어느 시그니처를 부를지는 부모 클래스 기준으로 이미 정해진다(오버로딩 묶음의 26-1 풀이) |
| `super.메서드()` | 부모 쪽 버전을 명시적으로 호출. 바로 위 부모부터 찾아 올라가며(부모에 없으면 조부모) 자기 클래스의 오버라이딩은 건너뛴다 |
| 자식 전용 메서드 | `Shape s`로는 Square에만 있는 메서드를 못 부른다. `((Square) s).m()`으로 다운캐스팅해야 한다 |
| 배열 순회 | `Shape[] arr = {new Shape(), new Rect()}` 각 원소의 실제 타입대로 실행 |

#### 부모 타입 변수에 자식 객체 담기
상속 문제에서 먼저 읽어야 할 줄은 `Animal a = new Dog();`처럼 부모 타입 변수에 자식 객체를 넣는 코드다. 이렇게 담을 수는 있는데, 그럼 Dog에만 있는 `fetch()`도 `a`로 부를 수 있을까?

```java
class Animal {
    String sound() { return "..."; }
}
class Dog extends Animal {
    String sound() { return "bark"; }
    String fetch() { return "fetch"; }
}
public class Main {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a.fetch());
    }
}
```
컴파일 결과:
```
Main.java:11: error: cannot find symbol
        System.out.println(a.fetch());
                            ^
  symbol:   method fetch()
  location: variable a of type Animal
1 error
```

실행해 보기도 전에 컴파일 오류가 난다. 컴파일러는 `a`의 타입인 Animal만 보고 "Animal에 fetch()가 있는가"를 따지기 때문이다. 그래서 변수 타입은 **어떤 메서드를 부를 수 있는가**를 정하는 역할을 한다. 참고로 이 페이지의 오류 메시지는 JDK 24의 javac로 찍은 것이고, 문구는 JDK 버전에 따라 조금 다를 수 있다. 꼭 부르고 싶다면 아래처럼 다운캐스팅한다.

```java
Animal a = new Dog();
System.out.println(((Dog) a).fetch());
```
출력:
```
fetch
```

#### 메서드는 누구 것이 불리나
그럼 Animal과 Dog 둘 다 가진 `sound()`를 `a`로 부르면 어느 쪽이 실행될까? 변수 타입이 Animal이니 "..."이 나올 것 같지만, 돌려 보면 다르다.

```java
class Animal {
    String sound() { return "..."; }
    String intro() { return "I say " + sound(); }
}
class Dog extends Animal {
    String sound() { return "bark"; }
}
class Puppy extends Dog { }
public class Main {
    public static void main(String[] args) {
        Animal a = new Dog();
        Animal p = new Puppy();
        System.out.println(a.sound());
        System.out.println(a.intro());
        System.out.println(p.sound());
    }
}
```
출력:
```
bark
I say bark
bark
```

첫 줄이 "bark"인 건 실제 객체가 Dog이고, Dog가 `sound()`를 **오버라이딩**했기 때문이다. 어느 메서드를 실행할지는 실행 시점에 객체를 보고 정한다. 두 번째 줄이 많이들 틀리는 곳인데, `intro()`는 Animal에 있지만 그 안에서 부르는 `sound()`도 똑같이 객체를 따라가서 Dog의 것이 불린다. 세 번째 줄의 Puppy는 `sound()`를 고치지 않았다. 이럴 때는 실제 클래스에서 시작해 위로 찾아 올라가니 바로 위 Dog의 것이 실행된다.

#### 필드는?
메서드가 객체를 따라간다면 필드도 그럴까? 아래 코드는 부모와 자식에 이름이 같은 필드 `name`을 하나씩 두었다.

```java
class Animal {
    String name = "animal";
    String getName() { return name; }
}
class Dog extends Animal {
    String name = "dog";
    String getName() { return name; }
}
public class Main {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a.name);
        System.out.println(a.getName());
    }
}
```
출력:
```
animal
dog
```

`a.name`은 "animal"이다. 필드는 오버라이딩되지 않고 **변수 타입**으로 정해지기 때문이다. 그런데 같은 값을 메서드 `getName()`으로 꺼내면 Dog의 메서드가 실행되고, Dog의 메서드는 Dog의 name을 읽으니 "dog"가 나온다. 자세한 건 필드 hiding 주제에서 다룬다.

#### static 메서드는?
static 메서드도 필드처럼 변수 타입을 따른다. 인스턴스 메서드와 나란히 놓고 보면 차이가 확실하다.

```java
class Animal {
    static String kind() { return "Animal"; }
    String sound() { return "..."; }
}
class Dog extends Animal {
    static String kind() { return "Dog"; }
    String sound() { return "bark"; }
}
public class Main {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a.kind() + " " + a.sound());
    }
}
```
출력:
```
Animal bark
```

똑같은 `a`로 불렀는데 static인 `kind()`는 Animal 것이, 인스턴스 메서드인 `sound()`는 Dog 것이 나왔다. static 메서드는 오버라이딩이 아니라 가려질(hiding) 뿐이라서 그렇다. 이 부분은 static 주제에서 더 본다.

#### super로 부모 쪽 버전 부르기
자식이 오버라이딩했어도 부모 쪽 버전이 필요할 때가 있다. 이때 `super.메서드()`를 쓴다. 그럼 부모에 그 메서드가 없으면 어떻게 될까?

```java
class Animal {
    String sound() { return "animal"; }
}
class Dog extends Animal {
    String sound() { return "dog"; }
}
class Puppy extends Dog {
    String sound() { return "puppy/" + super.sound(); }
}
class Cat extends Animal { }
class Kitten extends Cat {
    String sound() { return "kitten/" + super.sound(); }
}
public class Main {
    public static void main(String[] args) {
        System.out.println(new Puppy().sound());
        System.out.println(new Kitten().sound());
    }
}
```
출력:
```
puppy/dog
kitten/animal
```

Puppy의 `super.sound()`는 바로 위 Dog의 것을 부른다. Kitten의 부모 Cat에는 `sound()`가 없으니 한 단계 더 올라가 Animal의 것이 불린다. 정리하면 `super`는 자기 클래스의 오버라이딩을 건너뛰고 바로 위 부모부터 찾아 올라가다가 처음 만나는 메서드를 부른다.

### 예제
```java
class Shape {
    double area() { return 0; }
    String info() { return "Shape " + area(); }
}
class Rect extends Shape {
    int w, h;
    Rect(int w, int h) { this.w = w; this.h = h; }
    double area() { return w * h; }
    String info() { return "Rect " + super.info(); }
}
class Square extends Rect {
    Square(int s) { super(s, s); }
    String info() { return "Square/" + super.info(); }
}
public class Main {
    public static void main(String[] args) {
        Shape s = new Square(3);
        System.out.println(s.area());
        System.out.println(s.info());
        Shape[] arr = {new Shape(), new Rect(2, 4)};
        double t = 0;
        for (Shape x : arr) t += x.area();
        System.out.println(t);
    }
}
```
출력:
```
9.0
Square/Rect Shape 9.0
8.0
```
변수 s의 타입은 Shape지만 실제로 만든 객체는 Square다. 그런데 Square는 area()를 고치지 않았으니 바로 위 Rect의 area()가 실행되어 9.0이 나온다. info()는 조금 더 따라가야 한다. Square의 info()에서 `super`로 Rect의 info()를 부르고, 거기서 다시 `super`로 Shape의 info()를 불러 문자열을 순서대로 이어붙인다. 여기서 헷갈리는 게 Shape.info() 안의 area()인데, 이것도 동적 바인딩이라 Shape의 area()가 아니라 9.0이 들어간다. 마지막 배열은 원소마다 실제 타입의 area()를 쓰므로 0 + 8이다.

### 자주 틀리는 포인트
- 메서드를 찾는 방향은 "객체의 실제 클래스에서 시작해 위로"다. 그래서 Square에 area()가 없으면 Rect로 올라가고, 거기에도 없으면 Shape까지 올라간다.
- `super.info()`는 바로 위 부모부터 위로 찾아 올라가다가 처음 만나는 info()를 부른다. 부모에 없으면 조부모의 것이 불리는 식이다. 그 안에서 또 `super`를 부르면 거기서 한 단계 더 위로 간다.
- 반환형이 double인 메서드는 `9.0`처럼 소수점이 붙어 출력된다. 답을 `9`라고 쓰면 틀린다.
- 오버라이딩할 때 접근 범위는 넓히는 것만 된다. default를 public으로 넓히는 건 되지만, public을 default로 좁히면 컴파일 오류다. 26-2가 바로 넓히는 쪽이었다. 부모의 `int hap()`(default)을 자식이 `public int hap()`으로 넓혔기 때문에 정상 실행되고, `aaa.hap()`의 1+5+3 = 9와 `bbb.hap()`의 10×50 = 500을 더해 답은 **509**다. 자세한 건 접근 제어자 주제를 참고하면 된다.

## 생성자 호출 순서와 this()·super()
출제: 25-1, 24-1, 23-1, 22-2, 20-3, 20-2
함정: 자식 생성자의 첫 줄에 `this(...)`나 `super(...)`가 없으면 `super()`가 숨어 있다. 부모 생성자가 먼저 끝나고 자식 본문이 실행된다.

### 개념
**생성자(Constructor)**란 `new`로 객체를 만들 때 자동으로 호출되어 필드를 초기화하는 메서드를 말한다. 모양으로 알아보면 빠른데, 이름이 클래스 이름과 같고 반환형이 없다. void조차 쓰지 않는다. 생성자를 하나도 쓰지 않으면 컴파일러가 매개변수 없는 기본 생성자를 알아서 넣어 준다. 여기서 많이들 놓치는 게, 매개변수 있는 생성자를 하나라도 쓰면 이 기본 생성자가 생기지 않는다는 점이다. 그래서 그때 `new P()`를 쓰면 컴파일 오류가 난다. 20-3에는 주석이 가리키는 부분의 이름을 묻는 단답이 나왔는데, 답이 생성자였다.

생성자 문제는 결국 "무엇이 어떤 순서로 찍히는가"를 묻는다. 한 번에 외우려 하지 말고 클래스 하나에서 시작해 상속, `super(...)`, `this(...)`, 초기화 블록 순으로 하나씩 붙여 보자.

`new B(7)`를 한 번 실행하면 안에서는 어떤 순서로 일이 벌어질까? 먼저 표로 정리하면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 순서 | 내용 |
|---|---|
| 1 | 클래스 로딩 시 static 블록(부모 → 자식). 프로그램에서 한 번만 |
| 2 | 자식 생성자 진입. 첫 줄이 `this(...)`면 같은 클래스의 다른 생성자로 간다 |
| 3 | 첫 줄이 `super(...)`이거나 아무것도 없으면 부모 생성자 `super()` 호출 |
| 4 | 부모 쪽에서도 같은 규칙(부모의 this/super, 부모의 초기화 블록, 부모 본문) |
| 5 | 자식의 필드 초기화와 인스턴스 초기화 블록 `{ }` |
| 6 | 자식 생성자 본문 |

#### 기본 생성자는 언제 사라지나
위에서 매개변수 있는 생성자를 하나라도 쓰면 기본 생성자가 생기지 않는다고 했다. 그럼 `P(int x)`만 있는 클래스에 `new P()`를 쓰면 컴파일러는 어떻게 반응할까?

```java
class P {
    int x;
    P(int x) { this.x = x; }
}
public class Main {
    public static void main(String[] args) {
        P a = new P(5);
        P b = new P();
    }
}
```
컴파일 결과:
```
Main.java:8: error: constructor P in class P cannot be applied to given types;
        P b = new P();
              ^
  required: int
  found:    no arguments
  reason: actual and formal argument lists differ in length
1 error
```

`new P(5)`는 괜찮고 `new P()`만 오류다. 오류 메시지가 "int 하나가 필요한데 인자가 없다"고 말하고 있다. P에는 `P(int x)` 하나뿐이고 컴파일러가 기본 생성자를 넣어 주지 않았기 때문이다.

#### 자식을 만들면 부모 생성자가 먼저 돈다
이번에는 상속이 들어간다. B의 생성자에는 A를 부르는 코드가 전혀 없다. 그럼 `new B()`를 하면 "B()"만 찍힐까?

```java
class A {
    A() { System.out.println("A()"); }
}
class B extends A {
    B() { System.out.println("B()"); }
}
public class Main {
    public static void main(String[] args) {
        new B();
    }
}
```
출력:
```
A()
B()
```

"A()"가 먼저 찍혔다. 자식 생성자의 첫 줄에 `this(...)`나 `super(...)`가 없으면 컴파일러가 `super()`를 몰래 넣어 두기 때문이다. 그래서 B() 본문보다 부모 A()가 먼저 끝난다.

#### 부모에 기본 생성자가 없으면 super(인자)
그런데 숨어 있는 `super()`는 인자가 없는 호출이다. 부모에 매개변수 있는 생성자만 있으면 어떻게 될까?

```java
class A {
    A(int x) { System.out.println("A(" + x + ")"); }
}
class B extends A {
    B() { System.out.println("B()"); }
}
public class Main {
    public static void main(String[] args) {
        new B();
    }
}
```
컴파일 결과:
```
Main.java:5: error: constructor A in class A cannot be applied to given types;
    B() { System.out.println("B()"); }
        ^
  required: int
  found:    no arguments
  reason: actual and formal argument lists differ in length
1 error
```

B() 줄에서 오류가 난다. 숨어 있는 `super()`가 A()를 찾는데 A에는 `A(int x)`밖에 없기 때문이다. 그래서 이때는 자식이 직접 `super(인자)`를 써 줘야 한다. B()를 `B() { super(3); System.out.println("B()"); }`로 고치면 이렇게 나온다.

출력:
```
A(3)
B()
```

#### this()로 같은 클래스의 다른 생성자 거치기
`this(...)`는 같은 클래스의 다른 생성자를 부른다. 그럼 `this()`로 한 번 거쳐 가면 부모 생성자는 두 번 불릴까?

```java
class A {
    A() { System.out.println("A()"); }
}
class B extends A {
    B() { System.out.println("B()"); }
    B(int y) { this(); System.out.println("B(" + y + ")"); }
}
public class Main {
    public static void main(String[] args) {
        new B(7);
    }
}
```
출력:
```
A()
B()
B(7)
```

A()는 한 번만 찍혔다. B(int)의 첫 줄이 `this()`라서 B(int)에는 `super()`가 숨지 않고, 거쳐 간 B()에서만 `super()`가 불린다. 호출은 B(7)에서 B()로, B()에서 A()로 들어가지만, 출력은 가장 깊은 A()의 본문부터 거꾸로 나온다.

#### 초기화 블록과 static 블록은 언제 도나
마지막으로 static 블록 `static { }`, 인스턴스 초기화 블록 `{ }`, 필드 초기화까지 넣었다. 객체를 두 번 만들면 무엇이 두 번 찍힐까?

```java
class A {
    static { System.out.println("A static"); }
    { System.out.println("A block"); }
    A() { System.out.println("A()"); }
}
class B extends A {
    static { System.out.println("B static"); }
    int n = init();
    { System.out.println("B block"); }
    B() { System.out.println("B()"); }
    int init() { System.out.println("B field"); return 1; }
}
public class Main {
    public static void main(String[] args) {
        new B();
        System.out.println("--");
        new B();
    }
}
```
출력:
```
A static
B static
A block
A()
B field
B block
B()
--
A block
A()
B field
B block
B()
```

static 블록은 클래스가 처음 쓰일 때 부모, 자식 순으로 한 번만 돌고, 두 번째 `new B()`에서는 나오지 않는다. 그다음부터는 부모 쪽(A의 초기화 블록, A() 본문)이 먼저 끝나고, 자식의 필드 초기화와 초기화 블록, 마지막으로 B() 본문이 돈다.

#### 생성자 안에서 오버라이딩된 메서드를 부르면
25-1처럼 생성자 안에서 오버라이딩된 메서드를 부르는 코드는 따로 조심해야 한다. 아래에서는 부모 생성자 A()가 `show()`를 부르고, B의 x는 5로 초기화했다. 첫 줄에 5가 나올까?

```java
class A {
    A() { show(); }
    void show() { System.out.println("A show"); }
}
class B extends A {
    int x = 5;
    String s = "hi";
    B() { show(); }
    void show() { System.out.println("B show " + x + " " + s); }
}
public class Main {
    public static void main(String[] args) {
        new B();
    }
}
```
출력:
```
B show 0 null
B show 5 hi
```

A() 안의 `show()`도 동적 바인딩으로 B의 것이 불린다. 문제는 시점이다. 부모 생성자가 도는 동안에는 B의 필드 초기화가 아직 일어나지 않아서 x는 0, s는 null로 보인다. B() 본문에 들어왔을 때는 초기화가 끝난 뒤여서 5와 hi가 나온다.

### 정리

`this(...)`와 `super(...)`는 생성자 첫 줄에만 쓸 수 있고, 둘 중 하나만 쓴다. 그리고 앞에서 매개변수 있는 생성자를 쓰면 기본 생성자가 생기지 않는다고 했었다. 그래서 매개변수 있는 생성자만 있는 부모를 상속하면 자식은 반드시 `super(인자)`를 명시해야 하고, 이 자리가 빈칸 단골이다.

### 예제
```java
class A {
    A() { this(1); System.out.println("A()"); }
    A(int x) { System.out.println("A(" + x + ")"); }
}
class B extends A {
    static { System.out.println("B static"); }
    { System.out.println("B block"); }
    B() { super(); System.out.println("B()"); }
    B(int y) { this(); System.out.println("B(" + y + ")"); }
}
public class Main {
    public static void main(String[] args) {
        System.out.println("start");
        new B(7);
        System.out.println("--");
        new B();
    }
}
```
출력:
```
start
B static
A(1)
A()
B block
B()
B(7)
--
A(1)
A()
B block
B()
```
`new B(7)`는 먼저 `this()`로 B()에 가고, B()는 `super()`로 A()에, A()는 다시 `this(1)`로 A(int)에 간다. 이렇게 가장 깊이 들어간 A(int)가 먼저 출력된다. 그다음 A() 본문, B의 초기화 블록, B() 본문, B(7) 본문 순으로 찍힌다. static 블록은 첫 `new` 직전에 한 번만 나오고, 두 번째 `new B()`에서는 다시 나오지 않는다.

### 자주 틀리는 포인트
- 출력 순서는 "가장 깊은 생성자의 본문부터"다. 둘을 섞으면 틀리니 호출 순서와 실행(출력) 순서를 구분해 적는다.
- `this()`로 다른 생성자를 거쳐 가면 `super()`는 그 거쳐 간 생성자에서 한 번만 호출된다. 두 번 불리는 일은 없다.
- 24-1처럼 "실행 순서를 번호로 나열"하라는 문제가 나오면, 코드 각 줄에 번호를 붙여 두고 위 표의 1~6 순서대로 따라가면 된다.
- 25-1처럼 생성자 본문에서 오버라이딩된 메서드를 부르면 조심해야 한다. 그 시점에는 자식 필드가 아직 초기화 전이라 0이나 null이 보일 수 있다.

## 추상 클래스·인터페이스·implements
출제: 25-3, 25-2, 24-2, 23-1, 20-3, 20-2
함정: 인터페이스 변수로 받아도 실행되는 것은 구현 클래스의 메서드다. 기출 빈칸은 `implements`(25-3)와 업캐스팅의 `new`(20-2)였고, `extends`·`abstract`도 같은 자리에 나올 수 있으니 구분해 둔다.

### 개념
추상 클래스와 인터페이스는 "자식이 이 메서드를 꼭 만들어라"라고 틀만 정해 두는 장치다. 그래서 둘 다 혼자서는 객체가 될 수 없고, 실제로 실행되는 건 언제나 그 틀을 채운 자식(구현 클래스)의 메서드다. 기출 빈칸으로는 `implements`가 나왔고 `abstract`, `extends`도 같은 자리에 올 수 있으니, 컴파일러가 무엇을 막는지부터 차례로 짚는다.

추상 클래스와 인터페이스는 둘 다 `new`로 직접 만들 수 없다는 점은 같지만, 나머지는 꽤 다르다. 표로 비교하면 이렇다.

| 항목 | 추상 클래스 | 인터페이스 |
|---|---|---|
| 선언 | `abstract class Base` | `interface Greet` |
| 상속 키워드 | `extends` (하나만) | `implements` (여러 개 가능) |
| 메서드 | 추상·일반 메서드 섞을 수 있음 | 추상 메서드가 기본(public abstract 생략). `default`·`static`(그리고 Java 9부터 `private`) 메서드는 본문을 가질 수 있다 |
| 필드 | 일반 필드 가능 | 상수(`public static final`)만 |
| 객체 생성 | `new Base()` 불가. 자식으로만 | `new Greet()` 불가. 구현 클래스 또는 익명 클래스 `new Greet() { ... }` |
| 구현 메서드 접근 | | 반드시 `public` |

#### 추상 클래스는 new로 만들 수 없다
본문 없이 선언만 한 메서드를 추상 메서드라 하고, 앞에 `abstract`를 붙인다. 이런 메서드가 있는 클래스를 `new`로 만들면 어떻게 될까?

```java
abstract class Base {
    abstract int size();
    int twice() { return size() * 2; }
}
class Impl extends Base {
    int size() { return 3; }
}
public class Main {
    public static void main(String[] args) {
        Base b = new Impl();
        System.out.println(b.size() + " " + b.twice());
        Base x = new Base();
    }
}
```
컴파일 결과:
```
Main.java:12: error: Base is abstract; cannot be instantiated
        Base x = new Base();
                 ^
1 error
```

`new Base()` 줄만 오류다. 본문이 없는 `size()`를 부를 방법이 없으니 객체 자체를 못 만들게 막는 것이다. 이 줄을 지우고 돌리면 `Base` 타입 변수에 자식 Impl을 담아 쓰는 건 문제없다.

출력:
```
3 6
```

`twice()`는 Base의 일반 메서드지만 그 안의 `size()`는 Impl의 것이 불려 3 × 2 = 6이 된다. 이렇게 추상 클래스는 추상 메서드와 일반 메서드를 섞어 가질 수 있다.

#### 추상 메서드가 있으면 클래스도 abstract
반대로 추상 메서드를 넣고 클래스에는 `abstract`를 빼먹으면 어떻게 될까?

```java
class Base {
    abstract int size();
}
```
컴파일 결과:
```
Main.java:1: error: Base is not abstract and does not override abstract method size() in Base
class Base {
^
1 error
```

추상 메서드가 하나라도 있으면 클래스에 `abstract`를 붙여야 한다. 자식이 추상 메서드를 다 구현하지 않았을 때도 같은 모양의 오류가 자식 클래스 줄에 난다.

#### 인터페이스 구현은 implements, 메서드는 public
인터페이스는 `interface`로 선언하고, 구현하는 클래스는 `extends`가 아니라 `implements`를 쓴다. 그런데 구현 메서드에 `public`을 빼면 어떻게 될까?

```java
interface Greet {
    String hello();
}
class Kor implements Greet {
    String hello() { return "annyeong"; }
}
```
컴파일 결과:
```
Main.java:5: error: hello() in Kor cannot implement hello() in Greet
    String hello() { return "annyeong"; }
           ^
  attempting to assign weaker access privileges; was public
1 error
```

메시지 끝의 "was public"이 이유다. 인터페이스의 `hello()`는 `public abstract`가 생략된 형태로 원래 public인데, Kor가 구현하면서 default로 좁히려 한 것이다.

#### 인터페이스 변수로 받아도 실행은 구현 클래스
`implements` 뒤에는 인터페이스를 여러 개 쓸 수 있다. 그리고 `default` 메서드는 본문이 있어서 구현 클래스가 고치지 않으면 그대로 쓰인다. 아래는 두 구현 클래스를 Greet 배열에 담아 차례로 부르는 코드다.

```java
interface Greet {
    String hello();
    default String bye() { return "bye"; }
}
interface Named {
    String name();
}
class Kor implements Greet, Named {
    public String hello() { return "annyeong"; }
    public String name() { return "kor"; }
}
class Eng implements Greet {
    public String hello() { return "hello"; }
    public String bye() { return "see you"; }
}
public class Main {
    public static void main(String[] args) {
        Greet[] arr = {new Kor(), new Eng()};
        for (Greet g : arr) System.out.println(g.hello() + " " + g.bye());
        Named n = new Kor();
        System.out.println(n.name());
    }
}
```
출력:
```
annyeong bye
hello see you
kor
```

변수 타입은 모두 Greet인데 `hello()`는 각 객체의 것이 나왔다. 상속에서 본 동적 바인딩이 인터페이스에도 그대로 적용된다. Kor는 `bye()`를 고치지 않아 default의 "bye"를, Eng는 오버라이딩해서 "see you"를 찍는다.

#### 인터페이스의 필드는 상수
인터페이스에 필드를 쓰면 `public static final`이 생략된 상수가 된다. 값을 바꿔 보면 바로 드러난다.

```java
interface Config {
    int MAX = 10;
}
public class Main {
    public static void main(String[] args) {
        System.out.println(Config.MAX);
        Config.MAX = 20;
    }
}
```
컴파일 결과:
```
Main.java:7: error: cannot assign a value to static final variable MAX
        Config.MAX = 20;
              ^
1 error
```

`MAX` 앞에는 아무 제어자도 쓰지 않았지만 컴파일러는 이것을 final 상수로 보고 대입을 막는다. 객체 없이 `Config.MAX`처럼 인터페이스 이름으로 읽을 수 있는 것도 static이 생략돼 있어서다.

#### 이름 없이 바로 구현하기: 익명 클래스
인터페이스도 `new Greet()`처럼 직접 만들 수는 없다. 대신 뒤에 `{ }`를 붙여 그 자리에서 구현하면 된다.

```java
Greet g = new Greet() {
    public String hello() { return "anon"; }
};
System.out.println(g.hello() + " " + g.bye());
```
출력:
```
anon bye
```

`{ }` 안이 이름 없는 구현 클래스의 본문이고, 여기서 손대지 않은 `bye()`는 default의 "bye"로 남는다.

### 정리

20-2에 나온 업캐스팅 빈칸은 `Parent p = new Child();`에서 `new` 자리였다. 그리고 표에 나온 익명 클래스 `new Greet() { public String hello() { ... } }`는 쉽게 말해 인터페이스를 이름 없이 그 자리에서 바로 구현하는 방법이다.

### 예제
```java
interface Greet {
    String hello();
    default String bye() { return "bye"; }
}
abstract class Base implements Greet {
    abstract int size();
    public String hello() { return "hi" + size(); }
}
class Impl extends Base {
    int size() { return 3; }
    public String bye() { return "see you"; }
}
public class Main {
    public static void main(String[] args) {
        Greet g = new Impl();
        Base b = (Base) g;
        System.out.println(g.hello() + " " + g.bye());
        System.out.println(b.size() + " " + (g instanceof Base));
        Greet anon = new Greet() { public String hello() { return "anon"; } };
        System.out.println(anon.hello() + " " + anon.bye());
    }
}
```
출력:
```
hi3 see you
3 true
anon bye
```
hello()는 Base에 있지만, 그 안에서 부르는 size()는 Impl의 것이라 "hi3"이 된다. bye()는 Impl이 default 메서드를 오버라이딩했으므로 "see you"다. 반면 익명 클래스는 bye()를 고치지 않았으니 default 그대로 "bye"가 나온다.

### 자주 틀리는 포인트
- 추상 메서드가 하나라도 있으면 클래스에 `abstract`를 붙여야 한다. 그리고 자식은 그 추상 메서드를 전부 구현해야 객체를 만들 수 있다.
- 인터페이스 메서드를 구현할 때 `public`을 빼면 컴파일 오류가 난다. 인터페이스의 추상 메서드는 원래 public이라서, 이것을 빼면 "접근 범위를 좁힐 수 없다"는 규칙에 걸리기 때문이다.
- 25-2처럼 인터페이스와 예외가 같이 나오면, 구현 클래스의 메서드 안에서 던진 예외가 호출한 쪽의 catch로 간다고 보면 된다. 자세한 흐름은 예외 흐름 주제에 있다.
- `g instanceof Base`는 실제 객체(Impl)를 기준으로 본다. 그 객체가 Base의 자손이면 true다.

## 오버로딩 (인자 타입으로 메서드 선택)
출제: 26-1, 25-1, 24-3, 23-1, 21-2
함정: 오버로딩은 컴파일 시점에 인자의 "선언 타입"으로 고른다. 정확히 같은 타입 → 넓히기(int→long→double) → 박싱(int→Integer) → Object 순이다.

### 개념
**오버로딩(Overloading)**이란 이름은 같고 매개변수가 다른 메서드를 여러 개 두는 것을 말한다. 호출 한 줄에 후보가 여럿 걸리니 컴파일러가 그중 하나를 골라야 하는데, 이 고르는 순서가 정해져 있다. 정확히 같은 타입이 먼저이고, 없으면 넓히기, 그다음 박싱, 마지막이 가변 인자다.

오버로딩 문제는 결국 "후보가 여러 개일 때 어느 것이 뽑히는가"를 묻는다. 먼저 인자별로 후보를 고르는 순서를 표로 정리하면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 호출 인자 | 후보가 있을 때 선택 순서 |
|---|---|
| `f(1)` | int → long → float → double → Integer → Number → Object |
| `f('a')` | char → int → long → ... (char는 int로 넓혀진다. Character 박싱보다 int 넓히기가 먼저) |
| `f(1L)` | long → float → double → Long → Object |
| `f(1.0f)` | float → double → Float → Object |
| `f("s")` | String → Object |
| `f(Integer.valueOf(1))` | Integer → Number → Object → (언박싱 int는 그 뒤) |
| 제네릭 클래스 안의 `T value`로 `f(value)` | 상한 없는 T는 컴파일 때 Object로 취급 → Object 버전 (`<T extends Number>`면 Number 버전) |

#### 정확히 맞는 타입이 있으면 그것
출발점은 후보가 다 갖춰진 경우다. int, long, double 버전이 다 있으면 각 호출은 어디로 갈까?

```java
public class Main {
    static void f(int x) { System.out.println("int"); }
    static void f(long x) { System.out.println("long"); }
    static void f(double x) { System.out.println("double"); }
    public static void main(String[] args) {
        f(1);
        f(1L);
        f(1.0);
    }
}
```
출력:
```
int
long
double
```

`1`은 int, `1L`은 long, `1.0`은 double 리터럴이라 각자 타입이 똑같은 버전으로 간다. 리터럴 뒤에 붙은 `L`, `f`나 소수점이 인자의 타입을 바꾼다는 것만 놓치지 않으면 된다.

#### 없으면 넓히기
이번에는 int와 double 버전만 남긴다. char, long, float를 넘기면 어떻게 될까?

```java
public class Main {
    static void f(int x) { System.out.println("int"); }
    static void f(double x) { System.out.println("double"); }
    public static void main(String[] args) {
        f('a');
        f(1L);
        f(1.0f);
    }
}
```
출력:
```
int
double
double
```

딱 맞는 버전이 없으면 더 넓은 기본형으로 넓혀서 받는다. 기본형은 int, long, float, double 순서로 넓은 쪽으로만 바뀌고, char는 int부터 이 순서에 올라탄다. 그래서 `'a'`는 int로, `1L`과 `1.0f`는 double로 넓어졌다. long은 int로 줄어들 수 없기 때문에 `f(1L)`이 int 버전으로 가는 일은 없다.

#### 넓힐 곳도 없으면 박싱
기본형끼리 넓힐 곳이 없으면 그때 int를 Integer로 감싸는 박싱을 한다. 아래는 후보 구성이 서로 다른 세 클래스에 똑같이 `f(1)`을 넘긴 코드다.

```java
class X {
    static void f(Integer x) { System.out.println("X Integer"); }
    static void f(Object x) { System.out.println("X Object"); }
}
class Y {
    static void f(Object x) { System.out.println("Y Object"); }
}
class Z {
    static void f(long x) { System.out.println("Z long"); }
    static void f(Integer x) { System.out.println("Z Integer"); }
}
public class Main {
    public static void main(String[] args) {
        X.f(1);
        Y.f(1);
        Z.f(1);
    }
}
```
출력:
```
X Integer
Y Object
Z long
```

X에서는 박싱한 Integer가 딱 맞는다. Y처럼 Object 버전 하나뿐이어도 오류가 아니다. Integer로 박싱한 다음 부모 타입인 Object로 받는다. 눈여겨볼 곳은 Z다. Integer 버전이 눈에 띄지만 넓히기가 박싱보다 먼저라서 long 버전이 뽑힌다.

#### 마지막은 가변 인자
`int... xs`처럼 개수가 정해지지 않은 매개변수를 가변 인자라고 한다. 이건 앞의 세 단계로도 후보를 못 찾았을 때만 고려된다.

```java
public class Main {
    static void f(Integer x) { System.out.println("Integer"); }
    static void f(int... xs) { System.out.println("varargs " + xs.length); }
    public static void main(String[] args) {
        f(1);
        f(1, 2);
        f();
    }
}
```
출력:
```
Integer
varargs 2
varargs 0
```

`f(1)`은 가변 인자 버전이 int를 그대로 받을 수 있는데도 박싱이 필요한 Integer 버전으로 갔다. 가변 인자가 가장 뒷순위이기 때문이다. 인자가 두 개이거나 하나도 없으면 받을 수 있는 게 가변 인자 버전뿐이라 그쪽으로 간다.

#### 값이 아니라 선언 타입으로 고른다
지금까지는 리터럴을 넘겼다. 변수를 넘기면 그 변수에 실제로 무엇이 들어 있느냐가 아니라 변수를 **어떤 타입으로 선언했는가**로 고른다. 제네릭 클래스의 `T`를 넘기는 경우도 마지막 줄에 넣었다.

```java
class Printer {
    static void print(Integer v) { System.out.println("Integer"); }
    static void print(Object v) { System.out.println("Object"); }
}
class Box<T> {
    T value;
    Box(T value) { this.value = value; }
    void show() { Printer.print(value); }
}
public class Main {
    public static void main(String[] args) {
        Object o = 1;
        Printer.print(o);
        Printer.print(1);
        new Box<Integer>(1).show();
    }
}
```
출력:
```
Object
Integer
Object
```

`o` 안에는 Integer 1이 들어 있지만 선언 타입이 Object라서 Object 버전이다. 오버로딩은 컴파일할 때 정해지고, 컴파일러는 변수 안의 값을 모른다. `Box<Integer>`로 만든 마지막 줄도 Object가 나오는데, `Box` 안에서 상한 없는 `T`는 컴파일 때 Object로 취급되기 때문이다. 24-3이 바로 이 구조였다.

#### 오버라이딩과 겹치면 두 단계로
오버로딩은 컴파일 때 시그니처를 고르고, 오버라이딩은 실행 때 객체를 보고 고른다. 둘이 한 코드에 겹치면 어떻게 될까? 아래 코드는 26-1과 같은 구조다.

```java
class A {
    void f(Object o) { System.out.println("A.f(Object)"); }
    void g() { f("a"); }
}
class B extends A {
    void f(Object o) { System.out.println("B.f(Object)"); }
    void f(String s) { System.out.println("B.f(String)"); }
}
public class Main {
    public static void main(String[] args) {
        A a = new B();
        a.g();
        a.f("a");
        new B().f("a");
    }
}
```
출력:
```
B.f(Object)
B.f(Object)
B.f(String)
```

`a.g()` 안의 `f("a")`는 A 클래스에 적힌 코드다. 컴파일러는 이 자리에서 A에 있는 후보만 보는데, A에는 `f(Object)` 하나뿐이므로 시그니처가 `f(Object)`로 굳는다. 그다음 실행할 때 실제 객체 B를 보고, B가 오버라이딩한 `f(Object)`가 실행된다. 선언 타입이 A인 `a`로 부른 `a.f("a")`도 같은 과정을 거친다. B 타입으로 직접 부른 마지막 줄에서만 B의 `f(String)`이 후보에 들어와 선택된다.

### 정리

이름이 비슷해서 오버로딩과 오버라이딩을 혼동하기 쉬운데, 둘은 고르는 시점부터 다르다. 오버로딩은 같은 클래스(또는 상속 관계)에서 이름은 같고 매개변수가 다른 메서드들을 말하고, 어느 것을 부를지는 컴파일 시 인자 타입으로 정해진다. 반면 오버라이딩은 부모 메서드를 자식이 같은 시그니처로 다시 정의하는 것이고, 어느 것이 실행될지는 실행 시 객체 타입으로 정해진다.

### 예제
```java
public class Main {
    static void f(int x) { System.out.print("int "); }
    static void f(long x) { System.out.print("long "); }
    static void f(double x) { System.out.print("double "); }
    static void f(Integer x) { System.out.print("Integer "); }
    static void f(Object x) { System.out.print("Object "); }
    static int add(int a, int b) { return a + b; }
    static String add(String a, String b) { return a + b; }
    static int add(char a, char b) { return a + b; }
    public static void main(String[] args) {
        f(1); f(1L); f(1.0f); f('a');
        f(Integer.valueOf(1)); f("s");
        System.out.println();
        System.out.println(add(1, 2) + " " + add("1", "2") + " " + add('1', '2'));
        char c = 'a';
        System.out.println(c + 1);
        System.out.println((char) (c + 1));
    }
}
```
출력:
```
int long double int Integer Object 
3 12 99
98
b
```
`f(1.0f)`는 float 버전이 없으니 double로 넓혀서 받는다. `f('a')`도 char 버전이 없어 int로 넓히는데, 이게 Character로 박싱해 Object로 가는 것보다 우선이다. 여기서 Integer 버전으로 가지 않을까 싶을 수 있지만, char는 Integer로는 박싱되지 않는다. 마지막으로 `add('1', '2')`는 char 버전이 있으니 그것이 선택되고, 49 + 50 = 99를 int로 반환한다.

### 자주 틀리는 포인트
- `add(char, char)`라도 반환형이 int면 문자 합이 숫자로 나온다. 그래서 인자 타입만 보지 말고 반환형과 출력 서식까지 확인한다.
- `Object` 버전만 있을 때 `f(1)`을 부르면 오류일 것 같지만 오류가 아니다. int가 Integer로 박싱되어 Object로 간다.
- 24-3은 특히 헷갈리기 쉽다. Integer·Object·Number를 받는 print가 함께 있는데, 넘기는 인자가 제네릭 클래스 `Collection<T>`의 필드 `T value`다. 위에서 오버로딩은 컴파일 시점의 선언 타입으로 고른다고 했었다. 그런데 제네릭 클래스 안에서 상한 없는 T는 Object로 취급된다(타입 소거). 그래서 `new Collection<Integer>(0)`으로 만들었더라도 `print(value)`는 print(Object)로 정해지고, 답은 **B0**이다. 비교해 보면, 같은 Printer에 `Integer v = 0; print(v)`를 직접 부르면 Integer 버전(A0)이다. `print(0)`도 int를 넓혀 받을 버전이 없으니 박싱한 뒤 Integer 버전(A0)으로 간다. 만약 `<T extends Number>`였다면 Number 버전(C0)이 된다. 덧붙여 `f(long)`과 `f(Integer)`만 있으면 `f(1)`은 넓히기가 박싱보다 먼저라 long 버전이다.
- 25-1처럼 문자열을 받는 오버로딩이 재귀로 이어지는 문제는, 매 호출마다 인자 타입이 무엇인지 적으면서 내려간다.
- 오버로딩과 오버라이딩이 한 문제에 겹치면 두 단계로 나눠 푼다. 1단계(컴파일 시점)에서는 호출이 적힌 자리의 선언 타입으로 시그니처를 정한다. 예를 들어 부모 A의 메서드 g() 안에서 `f("a")`를 부른다고 해보자. 이 자리에서 후보는 A에 있는 f(Object)뿐이라 `f(Object)`로 확정된다. 2단계(실행 시점)에서는 실제 객체를 본다. 실제 객체 B가 `f(Object)`를 오버라이딩했으므로 B의 f(Object)가 실행된다. 여기서 많이들 헷갈리는 게 B에만 있는 `f(String)`이다. 이것은 오버라이딩이 아니라 새 오버로딩이라서 A 쪽 코드에서는 후보가 아니다. 그래서 26-1 `A a = new B(); a.g()`의 답은 3이 아니라 **2**다. `a.f("a")`도 2이고, `new B().f("a")`처럼 B 타입으로 직접 부를 때만 3이다.

## static 멤버와 static 메서드 hiding
출제: 25-2, 25-1, 23-3, 23-1, 21-2
함정: static 메서드는 오버라이딩되지 않는다. 참조 변수의 타입으로 결정된다(hiding). 인스턴스 메서드와 반대다.

### 개념
`static`이 붙은 멤버는 객체가 아니라 클래스에 딸린다. 그래서 객체를 몇 개 만들든 static 필드는 하나뿐이고, static 메서드는 객체 없이 `클래스.메서드()`로 부른다. 이 "객체가 없다"는 성질 하나에서 공유, `this` 사용 불가, hiding이 모두 따라 나온다.

먼저 static과 관련된 규칙을 표로 보면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 항목 | 내용 |
|---|---|
| static 필드 | 클래스에 하나. 모든 객체가 공유. 생성자에서 `count++`하면 객체 수가 된다 |
| static 메서드 | 객체 없이 `클래스.메서드()`. 안에서 `this`·인스턴스 변수 사용 불가(23-3 오류 라인 문제) |
| static 메서드 hiding | 부모와 자식이 같은 static 메서드를 가지면 `Parent p = new Child(); p.who()`는 Parent의 것 |
| 인스턴스 메서드 오버라이딩 | 같은 상황에서 `p.me()`는 Child의 것 |
| 자식 클래스로 static 접근 | `Child.count`는 Parent의 static 필드와 같은 것 |
| static 블록 | 클래스가 처음 쓰일 때 한 번 실행 |

#### static 필드는 모든 객체가 하나를 공유한다
생성자에서 static 필드를 하나씩 올리면 무엇이 남을까?

```java
class Counter {
    static int count = 0;
    int id;
    Counter() { count++; id = count; }
}
public class Main {
    public static void main(String[] args) {
        Counter a = new Counter();
        Counter b = new Counter();
        Counter c = new Counter();
        System.out.println(Counter.count + " " + a.count);
        System.out.println(a.id + " " + b.id + " " + c.id);
    }
}
```
출력:
```
3 3
1 2 3
```

`count`는 세 객체가 같이 쓰는 변수 하나라서 만든 객체 수인 3이 된다. `a.count`처럼 객체를 통해 읽어도 같은 변수다. 반면 `id`는 객체마다 따로 있어서, 만들어지던 순간의 count 값 1, 2, 3이 각각 남았다.

#### static 메서드 안에서는 this를 못 쓴다
static 메서드는 객체 없이 불릴 수 있다. 그럼 그 안에서 인스턴스 변수를 쓰면 어떻게 될까?

```java
public class Main {
    int v = 10;
    static int sv = 20;
    static void show() {
        System.out.println(sv);
        System.out.println(v);
        System.out.println(this.v);
    }
    public static void main(String[] args) {
        show();
    }
}
```
컴파일 결과:
```
Main.java:6: error: non-static variable v cannot be referenced from a static context
        System.out.println(v);
                           ^
Main.java:7: error: non-static variable this cannot be referenced from a static context
        System.out.println(this.v);
                           ^
2 errors
```

static 변수 `sv`를 쓴 5번 줄은 괜찮고, 인스턴스 변수 `v`와 `this`를 쓴 6, 7번 줄이 오류다. static 메서드가 불릴 때는 "이 객체"라고 가리킬 대상이 없으니 당연한 결과다. 23-3처럼 오류 나는 줄을 고르는 문제에서 이 줄이 답이 된다.

#### 부모와 자식에 같은 static 메서드가 있으면
상속 묶음에서 static 메서드는 변수 타입을 따른다고 했다. 이번에는 인스턴스 메서드를 나란히 두고, 자식 클래스 이름으로 static 필드도 읽는다.

```java
class Parent {
    static int count = 0;
    Parent() { count++; }
    static String who() { return "Parent"; }
    String me() { return "parent"; }
}
class Child extends Parent {
    static String who() { return "Child"; }
    String me() { return "child"; }
}
public class Main {
    public static void main(String[] args) {
        Parent p = new Child();
        Child c = new Child();
        System.out.println(p.who() + " " + p.me());
        System.out.println(c.who() + " " + c.me());
        System.out.println(Parent.count + " " + Child.count);
    }
}
```
출력:
```
Parent child
Child child
2 2
```

`p`와 `c`는 둘 다 Child 객체다. 그런데 static인 `who()`는 변수 타입대로 `p`에서는 Parent 것이, `c`에서는 Child 것이 나왔다. 컴파일러는 static 메서드 호출을 `Parent.who()`, `Child.who()`로 바꿔서 처리하기 때문이다. 이렇게 자식의 static 메서드가 부모 것을 덮지 못하고 가리기만 하는 것을 hiding이라고 한다. 마지막 줄의 `Child.count`도 Child에 따로 생긴 변수가 아니라 Parent의 count와 같은 변수다.

#### static 블록은 클래스가 처음 쓰일 때 한 번
`static { }` 블록은 클래스를 처음 쓰는 순간 딱 한 번 돈다.

```java
class Config {
    static int max;
    static {
        System.out.println("static block");
        max = 3;
    }
}
public class Main {
    public static void main(String[] args) {
        System.out.println("start");
        System.out.println(Config.max);
        System.out.println(Config.max);
    }
}
```
출력:
```
start
static block
3
3
```

"start"가 먼저 찍힌 것을 보면 프로그램 시작 때가 아니라 `Config.max`를 처음 읽는 순간 블록이 돌았다. 두 번째로 읽을 때는 다시 돌지 않는다.

### 예제
```java
class Parent {
    static int count = 0;
    int id;
    Parent() { count++; id = count; }
    static String who() { return "Parent"; }
    String me() { return "parent" + id; }
}
class Child extends Parent {
    static String who() { return "Child"; }
    String me() { return "child" + id; }
}
public class Main {
    public static void main(String[] args) {
        Parent p = new Child();
        Parent q = new Parent();
        Child c = new Child();
        System.out.println(Parent.count + " " + p.id + " " + c.id);
        System.out.println(p.who() + " " + p.me());
        System.out.println(c.who() + " " + c.me());
        System.out.println(Child.count);
    }
}
```
출력:
```
3 1 3
Parent child1
Child child3
3
```
Child를 만들 때도 Parent() 생성자가 돌기 때문에 count가 늘어난다. 객체를 3개 만들었으니 3이다. 두 번째 줄은 잘 봐야 한다. `p.who()`는 static이라 변수 타입 Parent의 것이 불려 "Parent"가 나오고, `p.me()`는 인스턴스 메서드라 객체 타입 Child의 것이 불려 "child1"이 나온다.

### 자주 틀리는 포인트
- 똑같은 `Parent p = new Child()`인데도 static 메서드는 Parent 것이, 인스턴스 메서드는 Child 것이 불린다. 그러니 메서드 선언에 `static`이 있는지부터 본다.
- static 메서드는 객체 없이 부를 수 있는 메서드라서, 그 안에서 `this`나 인스턴스 변수를 객체 없이 직접 쓰면 컴파일 오류다. `obj.v`처럼 객체를 통해서 쓰는 건 괜찮다. "오류가 나는 줄 번호"를 묻는 문제라면 바로 이 줄이 답이다.
- 21-2에는 static 빈칸이 나왔다. 객체 없이 `클래스.메서드()`로 부르고 있거나, 모든 객체가 공유하는 카운터라면 그 자리는 `static`이다.
- 싱글톤 패턴도 static과 엮여 있는데, static 필드와 private 생성자, static getInstance()로 만든다. 자세한 건 별도 주제에서 다룬다.

## 재귀
출제: 26-2, 25-1, 24-2, 23-3, 20-4
함정: 문자열 재귀는 `substring(1)`로 한 글자씩 줄여 가고, 결과를 "재귀 결과 + 첫 글자"로 붙이면 뒤집힌다.

### 개념
재귀란 메서드가 자기 자신을 다시 부르는 것을 말한다. 매번 문제를 조금씩 줄여서 부르다가, 더 줄일 수 없는 **기저 조건**에 닿으면 값을 돌려주고, 그 값이 거꾸로 올라오면서 답이 완성된다. 시험에 나오는 재귀는 모양이 몇 가지로 정해져 있으니, 아래 코드들은 호출마다 출력문을 끼워 넣어 내려가고 올라오는 과정을 그대로 찍는다.

먼저 재귀 모양을 유형별로 나란히 놓으면 아래 표가 된다. 하나씩은 아래 단계에서 코드로 확인한다.

| 유형 | 모양 | 결과 |
|---|---|---|
| 합·팩토리얼 | `n == 0 ? 0 : n + sum(n-1)` | sum(10) = 55 |
| 문자열 뒤집기 | `rev(s.substring(1)) + s.charAt(0)` | "abcd" → "dcba" |
| 연속 중복 제거 | 첫 두 글자가 같으면 첫 글자 버리고 재귀 | "aabbbca" → "abca" |
| 분할(이진 탐색) | 중간 비교 후 왼쪽/오른쪽 절반으로 재귀 | 인덱스 또는 -1 |
| 상속+재귀(20-4) | 부모의 재귀 메서드를 자식이 오버라이딩하면 재귀 호출도 자식 것으로 간다 | |

#### 내려갔다가 올라온다: 합
`sum(3)`은 `3 + sum(2)`이고, `sum(2)`는 또 `2 + sum(1)`이다. 호출될 때와 값을 돌려줄 때 한 줄씩 찍으면, 값은 어떤 순서로 정해질까?

```java
public class Main {
    static int sum(int n) {
        System.out.println("sum(" + n + ") call");
        int r = (n == 0) ? 0 : n + sum(n - 1);
        System.out.println("sum(" + n + ") = " + r);
        return r;
    }
    public static void main(String[] args) {
        sum(3);
    }
}
```
출력:
```
sum(3) call
sum(2) call
sum(1) call
sum(0) call
sum(0) = 0
sum(1) = 1
sum(2) = 3
sum(3) = 6
```

호출은 3, 2, 1, 0 순서로 내려가는데 값은 0, 1, 3, 6 순서로 확정된다. `sum(3)`은 `sum(2)`의 값이 나올 때까지 덧셈을 못 하고 기다리기 때문이다. 기저 조건 `n == 0`에서 처음으로 진짜 값 0이 나오고, 거기서부터 거꾸로 더해 올라온다.

#### 한 글자씩 줄이기: 문자열 뒤집기
문자열 재귀는 `substring(1)`로 첫 글자를 떼고 나머지로 재귀한다. 결과를 "재귀 결과 + 첫 글자"로 붙이면 왜 뒤집힐까? 각 호출이 돌려주는 값을 찍게 해 두었다.

```java
public class Main {
    static String rev(String s) {
        if (s.length() <= 1) return s;
        String r = rev(s.substring(1)) + s.charAt(0);
        System.out.println("rev(" + s + ") = " + r);
        return r;
    }
    public static void main(String[] args) {
        rev("abcd");
    }
}
```
출력:
```
rev(cd) = dc
rev(bcd) = dcb
rev(abcd) = dcba
```

가장 안쪽 `rev("d")`는 기저 조건이라 "d"를 그대로 돌려준다. 거기에 c를 뒤에 붙여 "dc", 다시 b를 붙여 "dcb", 마지막에 a를 붙여 "dcba"가 된다. 떼어 낸 첫 글자가 매번 맨 뒤로 가니 결과가 뒤집힌다.

#### 조건에 따라 버리거나 남기기: 연속 중복 제거
이번에는 첫 두 글자가 같으면 첫 글자를 버리고, 다르면 남긴 채 재귀한다. "aabbbca"는 무엇이 될까? 매 호출의 `s`도 함께 찍었다.

```java
public class Main {
    static String dedup(String s) {
        System.out.println(s);
        if (s.length() < 2) return s;
        if (s.charAt(0) == s.charAt(1)) return dedup(s.substring(1));
        return s.charAt(0) + dedup(s.substring(1));
    }
    public static void main(String[] args) {
        System.out.println("= " + dedup("aabbbca"));
    }
}
```
출력:
```
aabbbca
abbbca
bbbca
bbca
bca
ca
a
= abca
```

매번 한 글자씩 줄어드는 건 같지만, 앞 두 글자가 같은 "aa", "bb" 단계에서는 첫 글자가 결과에 붙지 않고 버려진다. 결과에 남는 건 다음 글자와 달랐던 a, b, c와 마지막 a뿐이라 "abca"다.

#### 절반씩 줄이기: 이진 탐색
이진 탐색은 정렬된 배열의 가운데 값을 보고 왼쪽이나 오른쪽 절반으로만 재귀한다. 9를 찾는 동안 lo, hi, mid는 어떻게 바뀔까?

```java
public class Main {
    static int bin(int[] a, int lo, int hi, int key) {
        if (lo > hi) return -1;
        int mid = (lo + hi) / 2;
        System.out.println("lo=" + lo + " hi=" + hi + " mid=" + mid + " a[mid]=" + a[mid]);
        if (a[mid] == key) return mid;
        return a[mid] < key ? bin(a, mid + 1, hi, key) : bin(a, lo, mid - 1, key);
    }
    public static void main(String[] args) {
        int[] a = {1, 3, 5, 7, 9, 11};
        System.out.println("= " + bin(a, 0, 5, 9));
    }
}
```
출력:
```
lo=0 hi=5 mid=2 a[mid]=5
lo=3 hi=5 mid=4 a[mid]=9
= 4
```

처음 mid는 (0 + 5) / 2인데 정수 나눗셈이라 2다. a[2] = 5가 9보다 작으므로 오른쪽 절반(lo = 3)으로 가고, 두 번째 mid 4에서 9를 찾아 인덱스 4를 돌려준다. 끝까지 못 찾으면 lo가 hi를 넘어서는 순간 -1이 나온다.

#### 상속과 재귀가 겹치면
부모의 재귀 메서드를 자식이 오버라이딩했다면, 재귀로 다시 부르는 `compute()`는 누구 것일까? 아래는 20-4와 같은 구조이고, 비교하려고 부모 쪽에는 `compute(num - 1) + compute(num - 2)`를 넣었다.

```java
class Parent {
    int compute(int num) {
        if (num <= 1) return num;
        return compute(num - 1) + compute(num - 2);
    }
}
class Child extends Parent {
    int compute(int num) {
        if (num <= 1) return num;
        return compute(num - 1) + compute(num - 3);
    }
}
public class Main {
    public static void main(String[] args) {
        Parent obj = new Child();
        System.out.println(obj.compute(4));
        System.out.println(new Parent().compute(4));
    }
}
```
출력:
```
1
3
```

재귀 안의 `compute()`도 동적 바인딩이다. 그래서 Child 객체에서 시작하면 재귀 끝까지 Child의 식이 쓰이고, 부모 식으로 계산한 3과는 다른 값이 나온다.

20-4를 직접 풀어 보자. `Parent obj = new Child(); obj.compute(4)`에서는 자식의 `compute(n) = compute(n-1) + compute(n-3)`(기저 `n <= 1`이면 n 반환)가 재귀 끝까지 쓰인다. 작은 것부터 쌓아 올리면 c(2) = c(1) + c(-1) = 1 + (-1) = 0, c(3) = c(2) + c(0) = 0 + 0 = 0, c(4) = c(3) + c(1) = 0 + 1 = **1**이다. 여기서 실수하기 쉬운 곳이 c(-1)이다. 기저 조건이 `return num`이라 c(-1)은 0이 아니라 -1인데, 이것을 0으로 놓으면 2라는 오답이 나온다.

### 정리

추적할 때는 [코딩 공통]의 재귀 추적법대로 호출 트리를 그리면 된다. 문자열 재귀라면 매 호출에서 s가 무엇인지 옆에 적어 둔다.

### 예제
```java
public class Main {
    static int sum(int n) { return n == 0 ? 0 : n + sum(n - 1); }
    static String rev(String s) {
        if (s.length() <= 1) return s;
        return rev(s.substring(1)) + s.charAt(0);
    }
    static String dedup(String s) {
        if (s.length() < 2) return s;
        if (s.charAt(0) == s.charAt(1)) return dedup(s.substring(1));
        return s.charAt(0) + dedup(s.substring(1));
    }
    static int bin(int[] a, int lo, int hi, int key) {
        if (lo > hi) return -1;
        int mid = (lo + hi) / 2;
        if (a[mid] == key) return mid;
        return a[mid] < key ? bin(a, mid + 1, hi, key) : bin(a, lo, mid - 1, key);
    }
    public static void main(String[] args) {
        System.out.println(sum(10) + " " + rev("abcd") + " " + dedup("aabbbca"));
        int[] a = {1, 3, 5, 7, 9, 11};
        System.out.println(bin(a, 0, 5, 9) + " " + bin(a, 0, 5, 4));
    }
}
```
출력:
```
55 dcba abca
4 -1
```
rev("abcd") = rev("bcd") + 'a' = (rev("cd") + 'b') + 'a' = ... = "dcba"처럼, 첫 글자가 계속 뒤에 붙으면서 뒤집힌다. dedup은 "aa"에서 a를 하나 버리고, "bbb"에서 두 개를 버린다. bin은 매 호출의 mid를 따라가면 된다. bin(9)는 mid=2(5)에서 오른쪽으로 가 lo=3이 되고, mid=4(9)에서 찾아 4를 돌려준다. bin(4)는 mid=2(5)에서 왼쪽으로 가 hi=1, mid=0(1)에서 오른쪽으로 가 lo=1, mid=1(3)에서 다시 오른쪽으로 가 lo=2가 된다. 이제 lo=2 > hi=1이니 -1이다.

### 자주 틀리는 포인트
- `s.charAt(0) + dedup(...)`에서 charAt은 char지만 뒤가 String이라 문자열 연결이 된다. 그런데 둘 다 char라면 숫자 덧셈이 되므로 양쪽 타입을 꼭 본다.
- 이진 탐색의 `mid = (lo + hi) / 2`는 정수 나눗셈이다. 매 호출의 lo, hi, mid를 표로 적어 가며 따라간다.
- 20-4처럼 재귀 안에서 오버라이딩된 메서드를 부르면, 자식 객체일 때 재귀도 자식 메서드로 이어진다.

## 배열 (length, 2차원 가변, 반환, 순위 계산)
출제: 22-3, 22-2, 21-1, 20-4, 20-1
함정: `length`는 배열의 필드(괄호 없음), `length()`는 String의 메서드, `size()`는 컬렉션이다.

### 개념
Java 배열은 `new`로 만드는 객체다. 그래서 만들자마자 기본값으로 채워져 있고, 변수에는 배열 자체가 아니라 배열을 가리키는 참조가 들어간다. 이 두 가지를 알면 2차원 가변 배열, 배열 반환, 객체 배열의 null, `println`의 이상한 출력까지 한 번에 설명된다.

먼저 배열을 만들고 재는 식과 순위 계산 식을 표 하나에 모았다. 하나씩은 아래 단계에서 코드로 확인한다.

| 식 | 뜻 |
|---|---|
| `int[] a = new int[4]` | 0으로 채워진 길이 4. `a.length` = 4 |
| `int[][] m = new int[3][]` | 행만 3개. 각 행은 null. `m[i] = new int[i+1]`로 길이가 다른 행 |
| `m.length`, `m[2].length` | 행 수, 2행의 열 수 |
| `for (int v : a)` | 향상된 for. 원소 값만 읽는다 |
| 배열 반환 | 메서드가 `int[]`를 돌려주면 호출 쪽 변수가 같은 배열을 가리킨다 |
| 순위 계산 | `rank[i] = 1; for j: if (sc[j] > sc[i]) rank[i]++;` 자기보다 큰 수의 개수 + 1 |
| `Arrays.toString(a)` | `[70, 90, 80]` 모양 |

#### 만들면 0으로 채워지고, 길이는 length
`new int[4]`를 하고 아무것도 넣지 않으면 무엇이 들어 있을까? 길이를 재는 세 가지 방법도 같이 찍었다.

```java
import java.util.ArrayList;
public class Main {
    public static void main(String[] args) {
        int[] a = new int[4];
        String s = "java";
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        System.out.println(a[0] + " " + a[3]);
        System.out.println(a.length + " " + s.length() + " " + list.size());
    }
}
```
출력:
```
0 0
4 4 1
```

int 배열은 0으로 채워진다. 길이는 배열이 `a.length`, 문자열이 `s.length()`, 리스트가 `list.size()`다. 배열의 `length`는 메서드가 아니라 필드라서 괄호가 없다. 빈칸에서 괄호 하나로 답이 갈리는 부분이다.

#### 2차원 배열과 길이가 다른 행
`new int[3][]`처럼 뒤쪽 크기를 비워 두면 행 3개만 생긴다. 이때 각 행에는 무엇이 들어 있을까?

```java
public class Main {
    public static void main(String[] args) {
        int[][] m = new int[3][];
        System.out.println(m[0]);
        for (int i = 0; i < 3; i++) m[i] = new int[i + 1];
        System.out.println(m.length + " " + m[0].length + " " + m[2].length);
    }
}
```
출력:
```
null
3 1 3
```

2차원 배열은 "배열을 담은 배열"이다. 처음에는 행 자리에 아직 아무 배열도 없어서 null이고, `m[i] = new int[i+1]`로 행마다 길이 1, 2, 3인 배열을 따로 넣었다. 그래서 행 수는 `m.length`, 각 행의 열 수는 `m[i].length`로 따로 잰다.

#### 배열을 반환하거나 넘기면 같은 배열
메서드가 배열을 돌려주거나 배열을 매개변수로 받으면 복사본이 생길까? 바꿔 보면 안다.

```java
public class Main {
    static int[] make(int n) {
        int[] r = new int[n];
        for (int i = 0; i < n; i++) r[i] = i * i;
        return r;
    }
    static void addOne(int[] arr) {
        for (int i = 0; i < arr.length; i++) arr[i]++;
    }
    public static void main(String[] args) {
        int[] a = make(4);
        int[] b = a;
        addOne(a);
        b[0] = 100;
        System.out.println(a[0] + " " + a[1] + " " + a[3]);
    }
}
```
출력:
```
100 2 10
```

`make(4)`가 만든 {0, 1, 4, 9}를 `a`가 받고, `b = a`와 `addOne(a)`도 모두 같은 배열을 가리킨다. 그래서 `addOne`이 더한 1도, `b[0] = 100`도 `a`에서 그대로 보인다. 복사본은 어디에도 생기지 않았다.

#### 객체 배열은 null로 시작한다
위에서 만든 int 배열은 0으로 채워졌다. 그럼 `new Box[3]`은 Box 객체 3개로 채워질까?

```java
class Box { int v = 5; }
public class Main {
    public static void main(String[] args) {
        Box[] arr = new Box[3];
        System.out.println(arr[0]);
        arr[0] = new Box();
        System.out.println(arr[0].v);
        try {
            System.out.println(arr[1].v);
        } catch (NullPointerException e) {
            System.out.println("NullPointerException");
        }
    }
}
```
출력:
```
null
5
NullPointerException
```

`new Box[3]`은 참조를 담을 칸 3개만 만들고 칸은 null로 채운다. `arr[0] = new Box()`로 객체를 넣은 칸만 쓸 수 있고, 손대지 않은 `arr[1]`의 필드를 읽으면 NullPointerException이 난다.

#### 배열을 println으로 바로 찍으면
마지막으로 배열을 `println`에 그대로 넣으면 어떻게 찍힐까?

```java
import java.util.Arrays;
public class Main {
    public static void main(String[] args) {
        int[] a = {70, 90, 80};
        char[] c = {'h', 'i'};
        System.out.println(a);
        System.out.println(c);
        System.out.println(Arrays.toString(a));
    }
}
```
출력:
```
[I@2b2fa4f7
hi
[70, 90, 80]
```

int 배열은 내용 대신 `[I@` 뒤에 해시코드가 붙은 값이 나온다. `@` 뒤의 숫자는 JVM과 실행 환경에 따라 달라지니 외울 값이 아니다. char 배열만 `println(char[])`가 따로 있어서 "hi"처럼 내용이 찍히고, 내용을 보고 싶으면 `Arrays.toString`을 쓴다.

### 정리

여기서 혼동하기 쉬운 게 객체 배열이다. `Box[] arr = new Box[3]`을 해도 Box 객체가 생기는 게 아니라 원소가 null이다. 그래서 `arr[i] = new Box()`를 해 줘야 비로소 쓸 수 있다.

### 예제
```java
public class Main {
    static int[] make(int n) {
        int[] r = new int[n];
        for (int i = 0; i < n; i++) r[i] = i * i;
        return r;
    }
    public static void main(String[] args) {
        int[] a = make(4);
        System.out.println(a.length + " " + a[a.length - 1]);
        int[][] m = new int[3][];
        for (int i = 0; i < 3; i++) m[i] = new int[i + 1];
        System.out.println(m.length + " " + m[2].length);
        int[][] g = {{1, 2, 3}, {4, 5, 6}};
        int s = 0;
        for (int[] row : g) for (int v : row) s += v;
        System.out.println(s + " " + g[1][2]);
        int[] sc = {70, 90, 80};
        int[] rank = new int[3];
        for (int i = 0; i < 3; i++) {
            rank[i] = 1;
            for (int j = 0; j < 3; j++) if (sc[j] > sc[i]) rank[i]++;
        }
        System.out.println(rank[0] + " " + rank[1] + " " + rank[2]);
        System.out.println(java.util.Arrays.toString(sc));
    }
}
```
출력:
```
4 9
3 3
21 6
3 1 2
[70, 90, 80]
```
make(4)는 {0,1,4,9}를 만들어 돌려준다. 가변 배열은 행 길이가 1,2,3이라 m[2].length = 3이다. 순위는 자기보다 큰 수를 세면 되는데, 70보다 큰 수가 2개라 3등, 90은 1등, 80은 2등이다.

### 자주 틀리는 포인트
- `System.out.println(a)`로 int 배열 같은 것을 직접 찍으면 내용이 아니라 `[I@1b6d3586` 같은 `타입@해시코드`가 나온다. char 배열만 예외인데, `println(char[])`가 따로 있어서 내용이 그대로 찍힌다. 그래서 출력 문제에서는 반복문이나 `Arrays.toString`을 쓴다.
- 2차원 배열 크기는 20-4에 빈칸으로, 21-1에 출력으로 나왔다. 행 수는 `m.length`, 열 수는 `m[i].length`인데, 열 수는 행마다 다를 수 있기 때문에 i를 붙인다.
- 순위 계산에서 비교 부호가 `>=`이면 자기 자신까지 세게 되어 모두 1씩 커진다.
- 배열을 `==`로 비교하는 건 참조 비교다. 자세한 건 ==와 equals 주제에서 본다.

## ==와 equals, String 메서드
출제: 24-3, 24-2, 23-2
함정: 리터럴 `"java"`끼리는 `==`가 true(같은 상수 풀), `new String("java")`는 false. 내용 비교는 항상 `equals`.

### 개념
객체에 `==`를 쓰면 "두 변수가 같은 객체를 가리키는가"를 비교하고, `equals`는 "내용이 같은가"를 비교한다. String도 객체라서 이 차이가 그대로 적용되는데, 리터럴을 상수 풀에서 공유하는 규칙 때문에 `==`가 true로 나오는 경우가 섞여 헷갈린다.

먼저 `==` 결과가 갈리는 식을 표로 모으면 이렇다. 하나씩은 아래 단계에서 코드로 확인한다.

| 식 | 결과 | 이유 |
|---|---|---|
| `"java" == "java"` | true | 리터럴은 상수 풀에서 공유 |
| `"java" == new String("java")` | false | new는 새 객체 |
| `"ja" + "va" == "java"` | true | 상수끼리의 연결은 컴파일 때 합쳐진다 |
| `a.substring(0,2) + "va" == a` | false | 실행 중에 만든 새 문자열 |
| `a.equals(b)` | 내용 비교 | 항상 이것으로 비교 |
| `int[] x == int[] y` | false | 배열도 참조 비교. 내용은 `Arrays.equals(x, y)` |

#### ==는 같은 객체인가, equals는 내용이 같은가
같은 "java"를 리터럴로 두 번, `new String`으로 한 번 만들면 `==` 결과는 어떻게 갈릴까?

```java
public class Main {
    public static void main(String[] args) {
        String a = "java";
        String b = "java";
        String c = new String("java");
        System.out.println(a == b);
        System.out.println(a == c);
        System.out.println(a.equals(c));
    }
}
```
출력:
```
true
false
true
```

`a == b`가 true인 건 같은 리터럴을 상수 풀에 하나만 두고 `a`와 `b`가 그것을 같이 가리키기 때문이다. `new String("java")`는 내용이 같아도 새 객체를 만드니 `a == c`는 false다. 내용만 보는 `equals`는 셋 다 true다.

#### 언제 합쳐지나: 컴파일 때 vs 실행 중
그럼 문자열을 `+`로 이어 붙여 "java"를 만들면 리터럴과 같은 객체일까?

```java
public class Main {
    public static void main(String[] args) {
        String a = "java";
        String d = "ja" + "va";
        String e = a.substring(0, 2) + "va";
        System.out.println(e);
        System.out.println((a == d) + " " + (a == e) + " " + a.equals(e));
    }
}
```
출력:
```
java
true false true
```

`e`도 분명 "java"인데 `a == e`는 false다. `"ja" + "va"`는 리터럴끼리라 컴파일러가 미리 "java"로 합쳐 두고, 결국 상수 풀의 그 객체를 가리킨다. 반면 `a.substring(0, 2) + "va"`는 프로그램이 실행되는 중에 계산되어 새 문자열 객체가 된다.

#### 배열도 참조 비교
배열은 내용이 같으면 `==`나 `equals`가 true일까?

```java
import java.util.Arrays;
public class Main {
    public static void main(String[] args) {
        int[] x = {1, 2};
        int[] y = {1, 2};
        int[] z = x;
        System.out.println((x == y) + " " + x.equals(y) + " " + Arrays.equals(x, y));
        System.out.println(x == z);
    }
}
```
출력:
```
false false true
true
```

배열에서는 `x.equals(y)`도 false가 나온다. 배열의 `equals`는 내용을 보지 않고 `==`처럼 같은 객체인지만 따지기 때문이다. 내용 비교는 `Arrays.equals(x, y)`로 하고, 같은 배열을 가리키는 `z`만 `==`가 true다.

#### 자주 나오는 String 메서드
String 메서드는 결과 모양을 정확히 알아야 답을 쓸 수 있다. "Hello Java" 하나에 여러 메서드를 한꺼번에 적용했다.

```java
public class Main {
    public static void main(String[] args) {
        String s = "Hello Java";
        System.out.println(s.length() + " " + s.charAt(1) + " " + s.substring(0, 2));
        System.out.println(s.indexOf("J") + " " + s.indexOf("z") + " " + s.contains("Ja"));
        System.out.println(s.toUpperCase() + " " + "  hi  ".trim() + " " + "".isEmpty());
        System.out.println("Java".equals("java") + " " + "Java".equalsIgnoreCase("java"));
    }
}
```
출력:
```
10 e He
6 -1 true
HELLO JAVA hi true
false true
```

`length()`는 공백까지 세서 10이다. 인덱스는 0부터라 `charAt(1)`은 e이고, `substring(0, 2)`는 끝 2를 빼고 0, 1번만 잘라 "He"다. `indexOf`는 처음 나온 위치를, 없으면 -1을 돌려준다. `equals`는 대소문자를 구분하고, 구분하지 않으려면 `equalsIgnoreCase`를 쓴다.

#### split이 만드는 조각 수
`split`에서는 빈 조각을 어떻게 세는지를 묻는다. 구분자가 연속되거나 끝, 앞에 있으면 조각 수가 어떻게 달라질까?

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("a,b,,c".split(",").length);
        System.out.println("a,b,".split(",").length);
        System.out.println(",a".split(",").length);
        System.out.println("a.b".split(".").length);
        System.out.println("a.b".split("\\.").length);
    }
}
```
출력:
```
4
2
2
0
2
```

연속된 쉼표 사이의 빈 문자열은 살아남아 "a,b,,c"는 4조각이다. 그런데 "a,b,"의 맨 끝 빈 조각은 버려져 2조각이고, ",a"의 맨 앞 빈 조각은 남아서 2조각이다. `split(".")`가 0인 이유는 인자가 정규식이기 때문이다. 정규식에서 `.`은 "아무 글자 하나"를 뜻해 모든 글자가 구분자가 되고, 그렇게 생긴 빈 조각은 모두 끝 쪽 빈 조각으로 취급되어 함께 버려진다. 점 자체로 나누려면 `split("\\.")`를 쓴다.

### 정리

자주 나오는 String 메서드는 `length()`, `charAt(i)`, `substring(a, b)`(b 미포함), `indexOf("c")`(없으면 -1), `split(",")`, `toUpperCase()`, `equals`, `isEmpty()`, `trim()`, `contains` 정도다. 이 중에서 `split`은 따로 기억할 규칙이 많다. 연속된 구분자 사이에는 빈 문자열이 생기고, 끝의 빈 문자열은 버린다. 그런데 맨 앞의 빈 문자열은 남는다. 예를 들어 `",a".split(",")`는 2개다. 또 인자가 정규식이라서 `split(".")`는 길이 0이 된다. 점으로 나누고 싶다면 `split("\\.")`를 써야 한다.

### 예제
```java
public class Main {
    public static void main(String[] args) {
        String a = "java";
        String b = "java";
        String c = new String("java");
        String d = "ja" + "va";
        String e = a.substring(0, 2) + "va";
        System.out.println((a == b) + " " + (a == c) + " " + a.equals(c));
        System.out.println((a == d) + " " + (a == e) + " " + a.equals(e));
        int[] x = {1, 2}, y = {1, 2};
        System.out.println((x == y) + " " + java.util.Arrays.equals(x, y));
        String s = "a,b,,c";
        String[] t = s.split(",");
        System.out.println(t.length + " " + t[2].isEmpty() + " " + s.charAt(2));
        System.out.println(s.length() + " " + s.indexOf("c") + " " + s.toUpperCase());
    }
}
```
출력:
```
true false true
true false true
false true
4 true b
6 5 A,B,,C
```
"a,b,,c"를 ","로 나누면 "a", "b", "", "c" 네 조각이 나온다. 연속된 쉼표 사이에 빈 문자열이 생긴 것이라 t[2]는 빈 문자열이다. charAt(2)는 'b'이고, indexOf("c")는 5다.

### 자주 틀리는 포인트
- 24-3은 문자열 배열의 이웃 원소를 `equals`로 비교한 문제였다. `sM[2] = new String("A")`라도 내용은 똑같이 "A"이므로 `"A".equals(new String("A"))`는 true다. 그래서 두 번 모두 O가 찍혀 답은 **OOAAA**다. 그럼 같은 코드가 `==`였다면 어떨까? 리터럴끼리인 sM[0], sM[1]은 true지만, 리터럴과 `new String`인 sM[1], sM[2]는 false라서 ONAAA가 된다. 이렇게 답이 갈리니 비교식이 `==`인지 `equals`인지부터 확인한다.
- `equals`는 대소문자를 구분한다. 그래서 `equalsIgnoreCase`를 쓴 게 아니라면 "Java"와 "java"는 다르다.
- `substring(0, 2)`는 인덱스 0, 1 두 글자다. 끝 인덱스는 포함하지 않기 때문이다.
- 24-2에 나온 `split` 결과 길이는, 구분자 사이의 빈 조각은 세고 맨 끝 빈 조각은 빼고 센다. 그래서 "a,b,"는 길이 2다.

## 예외 흐름 (try-catch-finally)
출제: 25-2, 25-1, 24-3
함정: `finally`는 `return`이 있어도, 예외가 catch되지 않아도 실행된다. 단 `System.exit()`로 프로그램을 끝내면 finally도 실행되지 않는다. 출력 순서에서 finally가 return 값보다 먼저 찍힌다.

### 개념
**예외(Exception)**란 0으로 나누기나 배열 범위 밖 접근처럼 실행 중에 생긴 문제를 말한다. 예외가 나면 프로그램은 그 자리에서 멈추고 맞는 `catch`를 찾아 나서는데, 그 사이에 `finally`가 끼어든다. 시험은 이 흐름을 출력 순서로 묻기 때문에, 상황을 하나씩 바꿔 가며 무엇이 찍히는지 보자.

try-catch-finally는 상황마다 흐름이 정해져 있다. 먼저 표로 정리하면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 상황 | 흐름 |
|---|---|
| try에서 예외 없음 | try 끝까지 → finally → 다음 문장 |
| try에서 예외, 맞는 catch 있음 | 예외 지점에서 try를 빠져나옴 → catch → finally → 다음 문장 |
| try에서 예외, 맞는 catch 없음 | finally 실행 후 호출한 쪽으로 예외가 올라간다 |
| try나 catch에 return | return 값을 정한 뒤 finally를 실행하고 반환한다 |
| 여러 catch | 위에서부터 처음 맞는 하나만. 부모 예외 타입(Exception)을 위에 두면 아래는 컴파일 오류 |

#### 예외가 나면 try의 나머지는 건너뛴다
try 안에서 예외가 나면 그 아래 문장도 실행될까?

```java
public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("1");
            int x = 10 / 0;
            System.out.println("2");
        } catch (ArithmeticException e) {
            System.out.println("catch");
        } finally {
            System.out.println("finally");
        }
        System.out.println("next");
    }
}
```
출력:
```
1
catch
finally
next
```

"2"는 찍히지 않았다. `10 / 0`에서 ArithmeticException이 나는 순간 try를 빠져나와 catch로 가기 때문이다. 예외는 catch에서 처리가 끝났으므로, 흐름은 finally를 거친 뒤 try 문 다음 문장 "next"까지 정상으로 이어진다.

#### return이 있어도 finally는 돈다
try 안에서 `return`을 만나면 메서드가 바로 끝날까? 그리고 finally에서 변수를 바꾸면 반환값도 바뀔까?

```java
public class Main {
    static int f() {
        int x = 1;
        try {
            return x;
        } finally {
            x = 99;
            System.out.println("finally");
        }
    }
    public static void main(String[] args) {
        System.out.println(f());
    }
}
```
출력:
```
finally
1
```

finally의 "finally"가 반환값 1보다 먼저 찍혔다. `return x`를 만나면 반환할 값 1을 먼저 정해 두고, finally를 실행한 다음에 돌아간다. 그래서 finally에서 x를 99로 바꿔도 이미 정해 둔 1이 그대로 나온다.

#### 맞는 catch가 없으면 위로 올라간다
이번에는 catch가 잡지 못하는 예외다. `Integer.parseInt("a")`는 NumberFormatException을 던지는데, g()에는 ArithmeticException용 catch만 있다.

```java
public class Main {
    static void g() {
        try {
            Integer.parseInt("a");
            System.out.println("g end");
        } catch (ArithmeticException e) {
            System.out.println("g catch");
        } finally {
            System.out.println("g finally");
        }
    }
    public static void main(String[] args) {
        try {
            g();
            System.out.println("after g");
        } catch (NumberFormatException e) {
            System.out.println("main catch");
        }
    }
}
```
출력:
```
g finally
main catch
```

g() 안에서 맞는 catch가 없어도 finally는 실행된다. 그다음 예외는 g()를 부른 main으로 올라가고, main의 catch가 잡는다. 이때 main의 "after g"는 예외 때문에 건너뛰어진다.

#### catch가 여러 개면 위에서부터 하나만
catch를 여러 개 두면 위에서부터 차례로 보고 처음 맞는 하나만 실행한다.

```java
try {
    int[] a = new int[2];
    a[2] = 1;
} catch (ArithmeticException e) {
    System.out.println("arith");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("index");
} catch (RuntimeException e) {
    System.out.println("runtime");
}
```
출력:
```
index
```

ArrayIndexOutOfBoundsException은 RuntimeException의 하위 클래스여서 세 번째 catch에도 맞는다. 하지만 두 번째 catch에서 이미 잡혔기 때문에 "index" 하나만 찍힌다. 그럼 부모 타입을 맨 위로 올리면 어떻게 될까?

```java
public class Main {
    public static void main(String[] args) {
        try {
            int[] a = new int[2];
            a[2] = 1;
        } catch (Exception e) {
            System.out.println("exception");
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("index");
        }
    }
}
```
컴파일 결과:
```
Main.java:8: error: exception ArrayIndexOutOfBoundsException has already been caught
        } catch (ArrayIndexOutOfBoundsException e) {
          ^
1 error
```

Exception이 모든 예외를 먼저 잡아 버리니 아래 catch는 절대 실행될 수 없다. 컴파일러가 이걸 "이미 잡혔다"는 오류로 막는다.

#### finally도 건너뛰는 경우: System.exit
finally는 return이 있어도, 예외가 catch되지 않아도 실행된다고 했다. 그런데 `System.exit()`를 만나면 다르다.

```java
try {
    System.out.println("try");
    System.exit(0);
} finally {
    System.out.println("finally");
}
```
출력:
```
try
```

`System.exit(0)`은 메서드를 빠져나가는 게 아니라 프로그램 자체를 끝낸다. 그래서 finally로 갈 기회가 없다.

#### finally에 return이 있으면
드물지만 finally에도 `return`을 쓰면 어느 값이 나올까?

```java
static int f() {
    try {
        throw new RuntimeException("x");
    } catch (RuntimeException e) {
        return 1;
    } finally {
        return 2;
    }
}
```
출력:
```
2
```

catch가 1을 반환값으로 정해 두었지만 finally의 `return 2`가 그 값을 덮어쓴다.

### 정리

자주 나오는 예외는 `ArithmeticException`(0으로 나눔), `ArrayIndexOutOfBoundsException`, `NullPointerException`, `NumberFormatException`(`Integer.parseInt("a")`)이다. 이들은 모두 `RuntimeException`의 자식이라서 `catch (RuntimeException e)`로도, `catch (Exception e)`로도 잡힌다. 예외는 `throw new X("msg")`로 직접 던질 수 있고, 넣은 메시지는 `e.getMessage()`로 꺼낸다.

### 예제
```java
public class Main {
    static int f(int n) {
        try {
            if (n == 0) throw new ArithmeticException("zero");
            if (n < 0) throw new IllegalArgumentException("neg");
            return 10 / n;
        } catch (ArithmeticException e) {
            System.out.print("A:" + e.getMessage() + " ");
            return -1;
        } finally {
            System.out.print("F ");
        }
    }
    public static void main(String[] args) {
        System.out.println(f(5));
        System.out.println(f(0));
        try {
            System.out.println(f(-1));
        } catch (RuntimeException e) {
            System.out.println("main:" + e.getMessage());
        }
        int[] arr = new int[2];
        try {
            arr[2] = 1;
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("idx");
        } finally {
            System.out.println("done");
        }
    }
}
```
출력:
```
F 2
A:zero F -1
F main:neg
idx
done
```
f(5)는 return 2가 먼저 정해지고, finally가 "F "를 찍은 다음에야 println이 2를 찍는다. f(0)은 catch가 "A:zero "를 찍고 return -1을 정한 뒤, finally가 "F "를 찍고 마지막에 -1이 나온다. f(-1)이 가장 헷갈리는데, IllegalArgumentException은 f 안의 catch에 맞지 않는다. 그래서 finally의 "F "만 찍고 main의 catch로 올라가 "main:neg"가 나온다. 이때 f(-1)의 println은 실행되지 않는다.

### 자주 틀리는 포인트
- `System.out.println(f(0))`에서는 f 안의 출력이 먼저 나오고, 반환값 출력은 그 뒤에 나온다. 그래서 한 줄에 "A:zero F -1"처럼 섞여 찍힌다.
- 예외가 catch되지 않은 호출의 println은 실행되지 않고, 그 줄 대신 바깥 catch의 출력이 나온다.
- finally에 return이 있으면 try·catch의 return을 덮어쓴다. 기출에서는 드물지만, 혹시 보이면 finally 값이 답이다.
- 25-2처럼 인터페이스 구현 메서드 안에서 던진 예외도 다르지 않다. 같은 규칙으로 호출자 쪽 catch를 찾아 올라간다.

## 반복문과 출력 형식
출제: 22-3, 22-2, 21-1, 20-3
함정: `print`와 `println`의 차이, 그리고 `+`로 이어붙일 때 숫자 덧셈이 되는 구간이 어디까지인지.

### 개념
반복문 문제의 함정은 로직보다 "출력이 정확히 어떤 모양인가"에 있다. 줄바꿈이 어디서 들어가는지, `+`가 덧셈인지 문자열 연결인지, `printf` 칸이 몇 칸인지를 차례로 짚고, 반복이 끝난 뒤 변수 값과 별 찍기 모양으로 넘어간다.

먼저 출력과 반복 관련 항목을 표로 보면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 항목 | 내용 |
|---|---|
| `print` / `println` | 줄바꿈 없음 / 있음. `println()`만 쓰면 빈 줄 |
| `printf("%d %5.2f %s%n", ...)` | C와 같은 서식. `%n`이 줄바꿈 |
| `while (i <= 100) { sum += i; i += 2; }` | 홀수 합. 끝난 뒤 i 값도 자주 묻는다 |
| 별 찍기 | 바깥 r행, 안쪽 r개. 안쪽이 `c <= r`인지 `c <= n - r`인지로 모양이 정해진다 |
| 최대 배수 | 조건을 만족할 때마다 덮어쓰면 마지막(최대) 값이 남는다 |
| `1 + 2 + "=" + 1 + 2` | "3=12". 문자열을 만나기 전까지만 덧셈 |

#### print와 println
`print`와 `println`을 섞어 쓰면 줄이 어떻게 나뉠까?

```java
System.out.print("A");
System.out.print("B");
System.out.println("C");
System.out.println();
System.out.println("D");
```
출력:
```
ABC

D
```

`print`는 줄을 바꾸지 않아서 A, B, C가 한 줄에 붙는다. `println`은 찍은 뒤 줄을 바꾸고, 괄호 안이 빈 `println()`은 줄바꿈만 해서 빈 줄 하나가 생긴다.

#### +는 문자열을 만나기 전까지만 덧셈
`+`는 숫자끼리면 덧셈, 한쪽이 문자열이면 연결이다. 그리고 왼쪽부터 차례로 계산한다. 그럼 아래 세 식은 각각 무엇을 찍을까?

```java
System.out.println(1 + 2 + "=" + 1 + 2);
System.out.println("=" + 1 + 2);
System.out.println("=" + (1 + 2));
```
출력:
```
3=12
=12
=3
```

첫 줄은 `1 + 2`가 먼저 3이 되고, "="를 만난 뒤부터는 1과 2가 글자로 붙어 "3=12"다. 둘째 줄은 처음부터 문자열이라 1과 2가 그냥 이어진다. 괄호로 묶은 셋째 줄만 괄호 안을 먼저 더해 3이 된다.

#### printf의 칸 수
`printf`는 C와 같은 서식을 쓴다. 칸 수가 눈에 보이도록 대괄호로 감싸 찍었다.

```java
System.out.printf("[%d][%5d][%.2f][%5.2f][%s]%n", 7, 7, 3.14159, 3.14159, "ok");
System.out.printf("end%n");
```
출력:
```
[7][    7][3.14][ 3.14][ok]
end
```

`%5d`는 5칸을 잡고 오른쪽에 맞추므로 7 앞에 공백 4칸이 붙는다. `%.2f`는 소수 둘째 자리까지 "3.14"이고, `%5.2f`는 소수점을 포함한 전체 폭이 5칸이어서 앞에 공백 1칸이 생긴다. "end"가 다음 줄에 찍힌 건 `%n`이 줄바꿈을 넣었기 때문이다.

#### 반복이 끝난 뒤의 변수 값
`while`이 끝났을 때 i는 마지막으로 더한 값일까?

```java
int i = 1, sum = 0;
while (i <= 10) { sum += i; i += 2; }
System.out.println(sum + " " + i);
```
출력:
```
25 11
```

1, 3, 5, 7, 9를 더해 25다. i는 9를 더한 뒤 11이 되고, 이때 `i <= 10`이 처음 거짓이 되어 반복이 끝난다. 그래서 끝난 뒤의 i는 마지막으로 더한 9가 아니라 조건을 처음 깬 11이다.

#### 별 찍기: 안쪽 조건이 모양을 정한다
바깥 반복은 행, 안쪽 반복은 그 행에 찍을 별 개수다. 안쪽 조건만 바꾸면 모양이 어떻게 달라질까?

```java
int n = 4;
for (int r = 1; r < n; r++) {
    for (int c = 1; c <= r; c++) System.out.print("*");
    System.out.println();
}
System.out.println("--");
for (int r = 1; r < n; r++) {
    for (int c = 1; c <= n - r; c++) System.out.print("*");
    System.out.println();
}
```
출력:
```
*
**
***
--
***
**
*
```

`c <= r`이면 r행에 별이 r개라 아래로 갈수록 늘어난다. `c <= n - r`이면 r이 커질수록 n - r이 줄어 거꾸로 된 삼각형이 된다.

### 예제
```java
public class Main {
    public static void main(String[] args) {
        int i = 1, sum = 0;
        while (i <= 100) { sum += i; i += 2; }
        System.out.println(sum + " " + i);
        for (int r = 1; r <= 3; r++) {
            for (int c = 1; c <= r; c++) System.out.print("*");
            System.out.println();
        }
        int max = 0;
        for (int k = 1; k < 50; k++) if (k % 7 == 0) max = k;
        System.out.println(max);
        System.out.printf("%d-%5.2f-%s%n", 7, 3.14159, "ok");
        System.out.println(1 + 2 + "=" + 1 + 2);
    }
}
```
출력:
```
2500 101
*
**
***
49
7- 3.14-ok
3=12
```
1~99 홀수 합은 50개 × 평균 50 = 2500이다. i는 99 다음에 101이 되고, 이때 조건이 깨져 반복이 끝난다. 50 미만 7의 배수 중에서는 마지막으로 덮어쓴 49가 남는다. `%5.2f`로 찍으면 "3.14" 앞에 공백이 1칸 붙는다.

### 자주 틀리는 포인트
- `sum + " " + i`는 문자열 앞에 있는 숫자가 sum 하나뿐이라 그대로 "2500 101"이다. 만약 `sum + i + " "`였다면 문자열을 만나기 전에 2601이 먼저 계산된다.
- 반복이 끝난 뒤의 i는 조건을 처음 깬 값이다. 홀수로 늘어나니 100이 아니라 101이다.
- 객체 배열은 `new A[2]`로 참조 칸만 만들어지니, 칸마다 `new A()`를 따로 해 줘야 한다. 22-2가 이 경우였고, `a[0].n + a[1].g` = 0 + 2 = 2다.

## 참조 전달 (객체 필드·배열·String)
출제: 25-2, 22-1
함정: 객체를 넘기면 필드 변경은 반영되고, 매개변수에 `new`로 다른 객체를 넣는 것은 반영되지 않는다. String은 불변이라 바뀌지 않는다.

### 개념
메서드에 값을 넘기면 매개변수는 그 값의 복사본을 받는다. 기본형이면 숫자가 복사되고, 객체나 배열이면 그것을 가리키는 참조가 복사된다. 그래서 "메서드 안에서 바꾼 게 밖에서도 보이는가"는 무엇을 바꿨느냐에 따라 갈리고, 경우는 넷으로 나뉜다.

먼저 전체를 표로 보면 이렇다. 매개변수에 한 일과 호출자에게 보이는지를 짝지으면 아래 표가 된다.

| 매개변수에 한 일 | 호출자에게 보이는가 |
|---|---|
| `b.v = 100` (객체 필드 변경) | 보인다 |
| `arr[0] = 100` (배열 원소 변경) | 보인다 |
| `n = 100` (기본형 재대입) | 안 보인다 |
| `b = new Box(7)` (참조 재대입) | 안 보인다. 그 뒤 `b.v = 999`도 새 객체에만 |
| `s = s + "!"` (String) | 안 보인다. 새 문자열이 만들어져 지역 변수만 바뀜 |

#### 기본형은 복사본만 바뀐다
int를 넘기고 메서드 안에서 100을 넣으면 밖의 변수도 100이 될까?

```java
public class Main {
    static void change(int n) { n = 100; System.out.println("in: " + n); }
    public static void main(String[] args) {
        int n = 1;
        change(n);
        System.out.println("out: " + n);
    }
}
```
출력:
```
in: 100
out: 1
```

이름이 똑같이 n이어도 메서드의 n은 1을 복사해 받은 별개의 변수다. 안에서 100으로 바꿔도 main의 n은 1 그대로다.

#### 객체의 필드와 배열의 원소는 바뀐다
이번에는 객체와 배열을 넘기고 그 안의 값을 바꾼다. 결과도 기본형과 같을까?

```java
class Box { int v; Box(int v) { this.v = v; } }
public class Main {
    static void change(Box b, int[] arr) {
        b.v = 100;
        arr[0] = 100;
    }
    public static void main(String[] args) {
        Box box = new Box(1);
        int[] arr = {1, 2};
        change(box, arr);
        System.out.println(box.v + " " + arr[0]);
    }
}
```
출력:
```
100 100
```

복사된 건 참조라서 `b`와 `box`는 같은 Box 객체를 가리킨다. 그 객체의 필드를 바꿨으니 main에서도 100이 보이고, 배열 원소도 마찬가지다.

#### 매개변수에 new를 넣으면 연결이 끊긴다
그럼 메서드 안에서 매개변수에 새 객체를 넣고 값을 바꾸면 어떨까? 비교하려고 새 객체를 넣기 전에 필드를 한 번 바꿔 두었다.

```java
class Box { int v; Box(int v) { this.v = v; } }
public class Main {
    static void replace(Box b) {
        b.v = 50;
        b = new Box(7);
        b.v = 999;
    }
    public static void main(String[] args) {
        Box box = new Box(1);
        replace(box);
        System.out.println(box.v);
    }
}
```
출력:
```
50
```

`b = new Box(7)` 전에 바꾼 50은 반영됐고, 그 뒤의 999는 반영되지 않았다. `b = new Box(7)`은 지역 변수 `b`가 가리키는 대상을 새 객체로 바꿀 뿐이고, main의 `box`는 여전히 원래 객체를 가리킨다. 그 뒤의 변경은 새 객체에만 일어난다.

#### String은 불변, String 배열의 원소는 바뀐다
String도 객체인데 필드처럼 바뀔까? String 하나와 String 배열을 같이 넘겨서 비교한다.

```java
public class Main {
    static void change(String s, String[] arr) {
        s = s + "!";
        arr[0] = "x";
    }
    public static void main(String[] args) {
        String s = "hi";
        String[] arr = {"a", "b"};
        change(s, arr);
        System.out.println(s + " " + arr[0]);
    }
}
```
출력:
```
hi x
```

String은 한 번 만들면 내용을 바꿀 수 없다. `s + "!"`는 `"hi!"`라는 새 문자열을 만들어 지역 변수 `s`에 넣는 것이라, 바로 위의 `b = new Box(7)`과 같은 경우다. 반면 `arr[0] = "x"`는 main과 같이 쓰는 배열의 칸을 바꾼 것이라 반영된다.

### 정리

제목은 참조 전달이라고 붙였지만, 정확히 말하면 Java는 항상 "값 전달"이고 그 값이 참조(주소)일 뿐이다. 그래서 표에서 본 것처럼 그 참조로 필드를 바꾸면 호출자에게 보이고, 매개변수에 다른 객체를 넣으면 보이지 않는다. 마찬가지로 두 변수가 같은 객체를 가리키면(`Box other = box`) 한쪽으로 바꾼 필드가 다른 쪽에도 보인다.

### 예제
```java
class Box { int v; Box(int v) { this.v = v; } }
public class Main {
    static void modify(Box b, int n, int[] arr, String s) {
        b.v = 100;
        n = 100;
        arr[0] = 100;
        s = s + "!";
    }
    static void replace(Box b) { b = new Box(7); b.v = 999; }
    public static void main(String[] args) {
        Box box = new Box(1);
        int n = 1;
        int[] arr = {1, 2};
        String s = "hi";
        modify(box, n, arr, s);
        System.out.println(box.v + " " + n + " " + arr[0] + " " + s);
        replace(box);
        System.out.println(box.v);
        Box other = box;
        other.v = 5;
        System.out.println(box.v + " " + (box == other));
    }
}
```
출력:
```
100 1 100 hi
100
5 true
```
첫 줄을 보면 객체 필드와 배열 원소만 바뀌었고, int와 String은 그대로다. replace는 지역 변수 b만 새 객체로 바꿨을 뿐이라 box.v는 100 그대로 남는다. 마지막으로 other와 box는 같은 객체라서 box.v도 5다.

### 자주 틀리는 포인트
- 25-2처럼 String 배열을 넘겨 `arr[0] = "x"`로 바꾸는 건 헷갈리기 쉽다. 이것은 배열 원소 재대입이므로 반영된다. String 자체를 바꾸는 것과 구분해야 한다.
- 메서드 안에서 `b = null`이나 `b = new ...`를 했다면, 그 뒤의 변경은 호출자와 상관이 없다.
- `box == other`는 둘이 같은 객체를 가리키므로 true다. 내용이 같더라도 다른 객체라면 false가 나온다.

## 싱글톤 패턴 코드
출제: 24-1, 21-3
함정: `getInstance()`를 몇 번 불러도 객체는 하나다. 카운터 필드가 인스턴스 필드여도 공유되는 것처럼 보인다.

### 개념
싱글톤이란 쉽게 말해 객체를 하나만 만들도록 막아 둔 구조다. 막는 방법은 간단한데, 밖에서 `new`를 못 쓰게 하고 클래스가 직접 만든 객체 하나만 나눠 주면 된다. 코드 한 줄씩이 왜 필요한지 빼 보면서 확인하자.

먼저 전체를 표로 보면 이렇다. 싱글톤 코드는 아래 세 가지로 이루어진다.

| 구성 | 코드 | 역할 |
|---|---|---|
| static 인스턴스 필드 | `private static Printer instance;` | 유일한 객체를 보관 |
| private 생성자 | `private Printer() { }` | 밖에서 `new` 금지 |
| static 접근 메서드 | `static Printer getInstance() { if (instance == null) instance = new Printer(); return instance; }` | 처음 한 번만 생성, 이후 같은 객체 반환 |

#### 밖에서 new를 막는다: private 생성자
생성자에 `private`을 붙이면 클래스 밖에서 쓴 `new`는 어떻게 될까?

```java
class Printer {
    private Printer() { }
}
public class Main {
    public static void main(String[] args) {
        Printer p = new Printer();
    }
}
```
컴파일 결과:
```
Main.java:6: error: Printer() has private access in Printer
        Printer p = new Printer();
                    ^
1 error
```

private 생성자는 Printer 클래스 안에서만 부를 수 있다. 이제 객체를 만들 수 있는 곳은 Printer 자신뿐이다.

#### 하나만 만들어 나눠 준다: getInstance
그럼 Printer 안에서 객체를 만들어 static 필드에 보관하고, 달라고 할 때마다 그것을 돌려주면 된다. 생성자에 출력문을 넣고 세 번 받으면 "created"는 몇 번 찍힐까?

```java
class Printer {
    private static Printer instance;
    private Printer() { System.out.println("created"); }
    static Printer getInstance() {
        if (instance == null) instance = new Printer();
        return instance;
    }
}
public class Main {
    public static void main(String[] args) {
        Printer a = Printer.getInstance();
        Printer b = Printer.getInstance();
        Printer c = Printer.getInstance();
        System.out.println((a == b) + " " + (b == c));
    }
}
```
출력:
```
created
true true
```

"created"는 한 번만 찍혔다. 첫 호출에서는 `instance == null`이 참이어서 객체를 만들고, 두 번째부터는 보관해 둔 그 객체를 그대로 돌려준다. 그래서 a, b, c는 모두 같은 객체다.

#### getInstance에서 static을 빼면
`getInstance()`의 `static`을 지우면 어떻게 될까?

```java
class Printer {
    private static Printer instance;
    private Printer() { }
    Printer getInstance() {
        if (instance == null) instance = new Printer();
        return instance;
    }
}
public class Main {
    public static void main(String[] args) {
        Printer a = Printer.getInstance();
    }
}
```
컴파일 결과:
```
Main.java:11: error: non-static method getInstance() cannot be referenced from a static context
        Printer a = Printer.getInstance();
                           ^
1 error
```

인스턴스 메서드는 객체가 있어야 부를 수 있는데, 객체를 얻으려고 부르는 메서드라 앞뒤가 안 맞는다. 그래서 `getInstance()`는 반드시 static이고, 이 자리가 빈칸으로 자주 나온다.

### 정리

빈칸은 `static`, `instance == null`, `new Printer()`, `return instance` 자리에 나온다. 디자인 패턴 이름을 같이 묻기도 하는데, 이때 답은 Singleton이고 분류는 생성 패턴이다.

### 예제
```java
class Printer {
    private static Printer instance;
    private int count = 0;
    private Printer() { }
    static Printer getInstance() {
        if (instance == null) instance = new Printer();
        return instance;
    }
    void print() { count++; }
    int getCount() { return count; }
}
public class Main {
    public static void main(String[] args) {
        Printer a = Printer.getInstance();
        Printer b = Printer.getInstance();
        a.print(); a.print(); b.print();
        System.out.println(a == b);
        System.out.println(a.getCount() + " " + b.getCount());
    }
}
```
출력:
```
true
3 3
```
getInstance()를 두 번 불렀지만 a와 b는 같은 객체라 `==`가 true다. print()도 a와 b로 합쳐 3번 불렀으니 둘 다 3이 나온다.

### 자주 틀리는 포인트
- count는 static이 아닌데도 결과가 공유된다. 객체가 하나뿐이기 때문이다. "static이 아니니 따로 센다"고 보면 틀린다.
- `getInstance()`가 static이어야 객체 없이 부를 수 있다. 그래서 이 static이 빈칸 1순위다.
- 생성자가 private이므로 클래스 밖에서 `new Printer()`를 쓰면 컴파일 오류가 난다.

## switch fall-through
출제: 22-2, 20-1
함정: `break`가 없으면 아래 case로 떨어진다. 반복문 안에서 `continue`를 만나면 switch 뒤의 문장을 건너뛰고 다음 반복으로 간다.

### 개념
switch는 값에 맞는 case로 "들어가는 입구"만 정한다. 들어간 다음에는 `break`를 만날 때까지 아래 case를 계속 실행하는데, 이것을 fall-through라고 한다. 그래서 입구가 어디인지, 어디서 멈추는지만 따라가면 된다.

규칙은 C와 같으니 [C] switch 주제를 같이 보면 된다. Java에만 있는 것은 `switch`에 String과 enum을 쓸 수 있다는 점이다. 화살표 문법(`case 1 -> ...`)은 아래로 떨어지지 않지만, 기출은 콜론 문법으로 나온다.

| 상황 | 결과 |
|---|---|
| `case 2: s += 2; case 3: s += 3; break;` 에 n=2 | 2와 3 둘 다 더해짐 |
| default가 마지막이고 그 위 case에 break 없음 | default도 실행 |
| 반복문 안 `case 0: ...; continue;` | switch와 그 아래를 건너뛰고 다음 반복 |

#### break가 없으면 아래로 떨어진다
case 1, 2에는 break가 없고 case 3에만 있다. n이 1, 2, 3, 9일 때 s는 각각 얼마일까?

```java
static int calc(int n) {
    int s = 0;
    switch (n) {
        case 1: s += 1;
        case 2: s += 2;
        case 3: s += 3; break;
        default: s += 100;
    }
    return s;
}
public static void main(String[] args) {
    System.out.println(calc(1) + " " + calc(2) + " " + calc(3) + " " + calc(9));
}
```
출력:
```
6 5 3 100
```

n=1은 case 1로 들어가 break가 있는 case 3까지 내려가니 1 + 2 + 3 = 6이다. n=2는 2 + 3 = 5, n=3은 들어가자마자 break라 3이다. 맞는 case가 없는 9만 default로 가서 100이다.

#### String switch와 default
Java는 switch에 String도 쓸 수 있다. 그럼 맨 아래 default 위에 break가 없으면 default까지 떨어질까? 소문자 "b"는 "B"에 맞을까?

```java
static void grade(String g) {
    switch (g) {
        case "A": System.out.print("excellent ");
        case "B": System.out.print("good ");
        default: System.out.println("end");
    }
}
public static void main(String[] args) {
    grade("A");
    grade("b");
}
```
출력:
```
excellent good end
end
```

"A"는 break가 하나도 없어서 default의 "end"까지 다 찍는다. "b"는 `equals`로 비교하면 "B"와 다르니 맞는 case가 없고, 바로 default로 간다.

#### 반복문 안의 break와 continue
switch가 for 안에 있으면 `break`와 `continue`가 각각 무엇을 끝낼까?

```java
for (int i = 0; i < 4; i++) {
    switch (i) {
        case 1: System.out.print("B"); break;
        case 2: System.out.print("C"); continue;
        default: System.out.print("D");
    }
    System.out.print("|");
}
System.out.println();
```
출력:
```
D|B|CD|
```

i=1의 `break`는 switch만 빠져나와서 아래 "|"가 찍힌다. i=2의 `continue`는 switch 아래의 "|"까지 건너뛰고 다음 반복으로 넘어가서 "C" 뒤에 바로 i=3의 "D"가 붙었다.

#### 화살표 문법은 떨어지지 않는다
`case 1 ->`처럼 화살표로 쓰면 어떻게 될까?

```java
int n = 2, s = 0;
switch (n) {
    case 1 -> s += 1;
    case 2 -> s += 2;
    case 3 -> s += 3;
}
System.out.println(s);
```
출력:
```
2
```

break가 없는데도 case 2만 실행되고 끝났다. 화살표 문법은 아래로 떨어지지 않는다. 다만 기출은 콜론 문법으로 나온다.

### 예제
```java
public class Main {
    public static void main(String[] args) {
        int n = 2, s = 0;
        switch (n) {
            case 1: s += 1;
            case 2: s += 2;
            case 3: s += 3; break;
            case 4: s += 4;
        }
        System.out.println(s);
        String grade = "B";
        switch (grade) {
            case "A": System.out.print("excellent ");
            case "B": System.out.print("good ");
            case "C": System.out.print("ok ");
            default: System.out.println("end");
        }
        for (int i = 0; i < 3; i++) {
            switch (i % 2) {
                case 0: System.out.print("e"); continue;
                case 1: System.out.print("o");
            }
            System.out.print("|");
        }
        System.out.println();
    }
}
```
출력:
```
5
good ok end
eo|e
```
n=2는 case 2에서 들어가 break를 만날 때까지 내려가므로 2 + 3 = 5다. "B"는 break가 없어서 good, ok, end까지 줄줄이 떨어진다. 반복문 쪽은 i=0과 2일 때 "e"를 찍고 continue를 만나 "|"를 건너뛰고, i=1일 때만 "o|"가 찍힌다.

### 자주 틀리는 포인트
- 여기서 `break`는 switch를 빠져나갈 뿐 for를 끝내는 게 아니다. 반면 `continue`는 for의 다음 반복으로 넘어간다. 둘을 헷갈리기 쉬우니 구분해 둔다.
- String switch는 `equals`로 비교하기 때문에, 대소문자가 다르면 맞는 case가 없어 default로 간다.

## 필드 hiding
출제: 24-3
함정: 필드는 오버라이딩되지 않는다. `P p = new C(); p.v`는 P의 v, `p.get()`은 C의 get()이 돌려주는 C의 v다.

### 개념
자식이 부모와 같은 이름의 필드를 선언하면 부모 필드를 덮어쓰는 게 아니라, 한 객체 안에 v가 두 개 생기고 자식 것이 부모 것을 가린다. 이것을 필드 hiding이라고 한다. 둘 중 어느 v를 읽을지는 변수 타입이 정한다.

메서드는 객체 타입을 따른다고 했는데, 필드는 그렇지 않고 변수 타입을 따른다. 아래 표에서 `p.v`와 `p.get()`을 비교해 보면 차이가 보인다.

| 식 (`P p = new C()`, 둘 다 `int v`) | 결과 | 이유 |
|---|---|---|
| `p.v` | P의 v | 필드는 변수 타입으로 결정 |
| `p.get()` | C의 v | 메서드는 객체 타입으로 결정되고, C.get()은 C.v를 읽는다 |
| `((C) p).v` | C의 v | 캐스팅하면 C 타입으로 접근 |
| `super.v` (C 안에서) | P의 v | 가려진 부모 필드 접근 |
| `p.v = 99` | P의 v만 바뀜 | C의 v는 별개 변수 |

#### 필드는 변수 타입, 메서드는 객체 타입
같은 C 객체를 P 타입 변수와 C 타입 변수로 각각 잡고 `v`와 `get()`을 읽으면 같은 값이 나올까?

```java
class P {
    int v = 10;
    int get() { return v; }
}
class C extends P {
    int v = 20;
    int get() { return v; }
}
public class Main {
    public static void main(String[] args) {
        P p = new C();
        C c = new C();
        System.out.println(p.v + " " + p.get());
        System.out.println(c.v + " " + c.get());
    }
}
```
출력:
```
10 20
20 20
```

첫 줄의 `p.v`만 10이다. 필드는 변수 타입 P를 보고 P 쪽 v를 읽는다. `p.get()`은 메서드라 객체 타입 C의 `get()`이 실행되고, C의 `get()` 안에서 v는 C의 v라 20이다.

#### 캐스팅, super.v, 그리고 대입
가려진 P의 v는 사라진 게 아니다. 캐스팅과 `super.v`로 두 v를 각각 꺼낼 수 있다. 그럼 `p.v = 99`는 어느 쪽을 바꿀까?

```java
class P {
    int v = 10;
}
class C extends P {
    int v = 20;
    int superV() { return super.v; }
}
public class Main {
    public static void main(String[] args) {
        P p = new C();
        System.out.println(((C) p).v + " " + ((C) p).superV());
        p.v = 99;
        System.out.println(p.v + " " + ((C) p).v + " " + ((C) p).superV());
    }
}
```
출력:
```
20 10
99 20 99
```

`((C) p).v`는 C 타입으로 읽으니 20, C 안의 `super.v`는 가려진 부모 v라 10이다. `p.v = 99`도 읽을 때와 같은 규칙으로 P 쪽 v에만 들어간다. 그래서 C의 v는 20 그대로이고, `super.v`로 읽은 부모 v만 99가 됐다.

#### get()이 부모에만 있으면
C에서 `get()`을 지우면 `p.get()`은 무엇을 돌려줄까?

```java
class P {
    int v = 10;
    int get() { return v; }
}
class C extends P {
    int v = 20;
}
public class Main {
    public static void main(String[] args) {
        P p = new C();
        C c = new C();
        System.out.println(p.get() + " " + c.get() + " " + c.v);
    }
}
```
출력:
```
10 10 20
```

이번에는 C 타입 `c`로 불러도 10이다. 실행되는 `get()`이 P에 정의된 메서드이고, P의 코드 안에서 `v`는 P의 v를 뜻하기 때문이다. 같은 객체에서 `c.v`로 직접 읽으면 20이 나오는 것과 비교해 보면 된다.

### 정리

그림으로 생각하면 쉽다. 한 객체 안에 P의 v와 C의 v, 이렇게 v가 두 개 따로 들어 있다고 그리면 된다.

### 예제
```java
class P {
    int v = 10;
    int get() { return v; }
}
class C extends P {
    int v = 20;
    int get() { return v; }
    int superV() { return super.v; }
}
public class Main {
    public static void main(String[] args) {
        P p = new C();
        C c = (C) p;
        System.out.println(p.v + " " + p.get());
        System.out.println(c.v + " " + c.get() + " " + c.superV());
        p.v = 99;
        System.out.println(c.v + " " + ((P) c).v);
    }
}
```
출력:
```
10 20
20 20 10
20 99
```
먼저 p와 c는 같은 객체라는 점을 기억하자. `p.v`는 필드라서 P 쪽 변수인 10이 나오고, `p.get()`은 메서드라서 C.get()이 실행되어 20이 나온다. `p.v = 99`도 P 쪽 변수만 바꾼다. 그래서 `((P) c).v`는 99가 되고 `c.v`는 20 그대로다.

### 자주 틀리는 포인트
- 오버라이딩(메서드)과 hiding(필드·static 메서드)을 구분해야 한다. 이 중에서 객체 타입을 따르는 건 메서드뿐이다.
- 만약 C의 get()이 없고 P의 get()만 있다면, `p.get()`은 P.get()이 P의 v(10)를 돌려준다. 이렇게 메서드가 어느 클래스에 정의됐는지에 따라 읽는 v가 달라진다.

## 접근 제어자
출제: 26-2
함정: `private` 멤버는 자식 클래스에서도 직접 못 쓴다. 오버라이딩할 때 자식은 부모보다 접근 범위를 좁힐 수 없다.

### 개념
접근 제어자는 멤버를 어디까지 보여 줄지 정하는 키워드다. 시험에서는 "어느 줄이 컴파일 오류인가"나 "오버라이딩이 성립하는가"로 나오니, 컴파일러가 실제로 무엇을 막는지 오류 메시지로 확인해 보자.

먼저 접근 범위를 표로 정리하면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 제어자 | 같은 클래스 | 같은 패키지 | 자식 클래스(다른 패키지) | 전체 |
|---|---|---|---|---|
| `private` | O | | | |
| (없음, default) | O | O | | |
| `protected` | O | O | O | |
| `public` | O | O | O | O |

범위 순서는 public > protected > default > private이다. 오버라이딩할 때는 이 순서에서 부모보다 좁아지면 안 된다. 그래서 부모가 `public`이면 자식도 `public`이어야 하고, 부모가 `protected`면 자식은 `protected` 또는 `public`이어야 한다. 좁히면 컴파일 오류다. 인터페이스 메서드를 구현할 때도 반드시 `public`이다.

접근 제어자와 함께 나오는 키워드도 표로 정리하면 이렇다.

| 키워드 | 뜻 |
|---|---|
| `final` | 변수: 값 변경 불가(상수). 메서드: 오버라이딩 금지. 클래스: 상속 금지 |
| `this` / `super` | 현재 객체 / 부모 쪽 멤버. `this.v = v`는 필드와 매개변수 이름이 같을 때 필드를 가리킨다 |
| `throw` / `throws` | 예외를 직접 던진다 / 메서드 선언 뒤에 붙여 예외를 호출한 쪽에 떠넘긴다 |
| `static` | 객체 없이 클래스 단위로 공유 |
| `abstract` | 메서드: 본문 없이 선언만(자식이 구현). 클래스: 객체 생성 불가. 추상 메서드가 하나라도 있으면 클래스도 abstract여야 한다(추상 메서드 없이 abstract만 붙여도 된다) |

#### private은 자식도, 바깥도 못 쓴다
`balance`를 private으로 두면 자식 클래스와 main 중 어디서 쓸 수 있을까?

```java
class Account {
    private int balance = 100;
    int getBalance() { return balance; }
}
class Saving extends Account {
    int twice() { return balance * 2; }
}
public class Main {
    public static void main(String[] args) {
        Account a = new Account();
        System.out.println(a.balance);
    }
}
```
컴파일 결과:
```
Main.java:6: error: balance has private access in Account
    int twice() { return balance * 2; }
                         ^
Main.java:11: error: balance has private access in Account
        System.out.println(a.balance);
                            ^
2 errors
```

자식 Saving의 6번 줄도, main의 11번 줄도 같은 오류다. 상속을 받았어도 private 필드는 Account 안에서만 쓸 수 있다. 그래서 자식은 `getBalance()`처럼 부모가 열어 둔 메서드를 거쳐서 값을 얻는다.

#### 오버라이딩하면서 좁히면 오류, 넓히면 정상
부모의 public 메서드를 자식이 default(아무것도 안 붙임)로 오버라이딩하면 어떻게 될까?

```java
class Parent {
    public int hap() { return 1; }
}
class Child extends Parent {
    int hap() { return 2; }
}
```
컴파일 결과:
```
Main.java:5: error: hap() in Child cannot override hap() in Parent
    int hap() { return 2; }
        ^
  attempting to assign weaker access privileges; was public
1 error
```

"더 약한 접근 권한을 주려 한다"는 오류다. 반대로 26-2처럼 부모가 default이고 자식이 public이면 범위를 넓힌 것이라 문제없다.

```java
class Parent {
    int hap() { return 1; }
}
class Child extends Parent {
    public int hap() { return 2; }
}
public class Main {
    public static void main(String[] args) {
        Parent p = new Child();
        System.out.println(p.hap());
    }
}
```
출력:
```
2
```

정상으로 오버라이딩되었으니 `p.hap()`은 동적 바인딩으로 Child의 것이 실행된다.

#### private 메서드는 오버라이딩되지 않는다
부모의 private 메서드와 같은 이름을 자식이 만들면, 부모 코드에서 부르는 건 누구 것일까?

```java
class Parent {
    private String log() { return "parent log"; }
    String run() { return log(); }
}
class Child extends Parent {
    String log() { return "child log"; }
}
public class Main {
    public static void main(String[] args) {
        Parent p = new Child();
        System.out.println(p.run());
        System.out.println(new Child().log());
    }
}
```
출력:
```
parent log
child log
```

객체는 Child인데 `run()` 안의 `log()`는 Parent 것이 불렸다. private 메서드는 자식에게 보이지 않으니 애초에 오버라이딩 대상이 아니고, Child의 `log()`는 이름만 같은 별개의 메서드다. Child 타입으로 직접 부를 때만 Child의 것이 나온다.

#### final은 바꾸지도, 오버라이딩하지도 못한다
접근 제어자와 같이 나오는 `final`은 무엇을 막을까?

```java
class Parent {
    final int MAX = 10;
    final void stop() { }
}
class Child extends Parent {
    void stop() { }
    void reset() { MAX = 20; }
}
```
컴파일 결과:
```
Main.java:6: error: stop() in Child cannot override stop() in Parent
    void stop() { }
         ^
  overridden method is final
Main.java:7: error: cannot assign a value to final variable MAX
    void reset() { MAX = 20; }
                   ^
2 errors
```

final 메서드는 오버라이딩할 수 없고, final 변수는 값을 바꿀 수 없다. 클래스에 final을 붙이면 상속 자체가 막힌다.

### 예제
```java
class Account {
    private int balance = 100;
    protected String owner = "kim";
    int code = 7;
    public int getBalance() { return balance; }
    private void log() { System.out.print("log "); }
    void audit() { log(); System.out.println(balance); }
}
class Saving extends Account {
    public int getBalance() { return super.getBalance() * 2; }
    void show() { System.out.println(owner + " " + code); }
}
public class Main {
    public static void main(String[] args) {
        Account a = new Saving();
        System.out.println(a.getBalance());
        a.audit();
        ((Saving) a).show();
    }
}
```
출력:
```
200
log 100
kim 7
```
Saving은 private balance에 직접 접근할 수 없어서 `super.getBalance()`를 거쳐 값을 얻는다. 반면 audit()은 Account에 정의돼 있으므로 private log()와 balance를 쓸 수 있다. private은 같은 클래스 안에서만 쓸 수 있기 때문이다. 같은 파일(같은 패키지)이라 protected와 default 필드는 Saving에서 보인다. 하지만 main에 `a.balance`나 `a.log()`를 쓰면 컴파일 오류다.

### 자주 틀리는 포인트
- "오류가 나는 줄"을 묻는 문제라면, private 멤버를 클래스 밖에서 쓰는 줄이나 오버라이딩하면서 접근 범위를 좁힌 줄이 답이다.
- private 메서드는 오버라이딩 대상이 아니다. 그래서 자식에 같은 이름의 메서드가 있더라도 부모 쪽 코드에서는 부모 것이 실행된다. 동적 바인딩처럼 자식 것이 불릴 거라고 착각하기 쉬운 부분이다.
- 기출 코드는 한 파일 안에 들어 있으니 default(패키지) 접근은 항상 허용된다고 보면 된다.

## enum
출제: 25-3
함정: `values()`는 선언 순서 배열, `ordinal()`은 0부터 시작하는 순번, `name()`과 `toString()`은 상수 이름 문자열이다.

### 개념
enum은 정해진 값 몇 개만 가질 수 있는 타입이다. `enum Color { RED, GREEN, BLUE }`라고 쓰면 RED, GREEN, BLUE 세 객체가 선언 순서대로 만들어지고, 각자 이름과 순번을 가진다. 시험은 이 이름과 순번을 꺼내는 메서드의 결과를 묻는다.

먼저 enum 식을 표로 보면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 식 | 결과 |
|---|---|
| `Color.values()` | `{RED, GREEN, BLUE}` 배열 |
| `c.name()`, `c.toString()`, `"" + c` | "BLUE" |
| `c.ordinal()` | 2 (0부터) |
| `Color.valueOf("BLUE")` | Color.BLUE (없는 이름이면 예외) |
| `c == Color.BLUE` | true. enum은 `==` 비교 가능 |
| `c.compareTo(Color.RED)` | ordinal 차이 2 |
| 생성자·필드 | `LOW(1), HIGH(3); final int w; Level(int w) { this.w = w; }` 상수마다 값 보관 |
| switch | `case RED:` (Color. 접두사 없이) |

#### 이름과 순번: values, name, ordinal
`values()`, `name()`, `toString()`, `ordinal()`은 각각 무엇을 돌려줄까?

```java
enum Color { RED, GREEN, BLUE }
public class Main {
    public static void main(String[] args) {
        Color[] all = Color.values();
        System.out.println(all.length + " " + all[0]);
        Color c = Color.BLUE;
        System.out.println(c.name() + " " + c + " " + ("" + c).length() + " " + c.ordinal());
    }
}
```
출력:
```
3 RED
BLUE BLUE 4 2
```

`values()`는 선언 순서대로 담긴 배열이라 길이 3, 첫 원소 RED다. `name()`과 그냥 찍은 `c`, `"" + c` 모두 "BLUE"라는 문자열이고, 길이가 4인 것으로 따옴표 없는 이름 그대로임을 알 수 있다. `ordinal()`은 0부터 세서 세 번째인 BLUE가 2다.

#### 문자열로 찾기와 비교: valueOf, ==, compareTo
`valueOf`는 이름 문자열로 상수를 찾는다. 찾은 상수끼리 비교하면 어떤 값이 나오고, 없는 이름을 넣으면 어떻게 될까?

```java
enum Color { RED, GREEN, BLUE }
public class Main {
    public static void main(String[] args) {
        Color c = Color.valueOf("GREEN");
        System.out.println((c == Color.GREEN) + " " + c.compareTo(Color.RED) + " " + Color.RED.compareTo(Color.BLUE));
        try {
            Color.valueOf("PINK");
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}
```
출력:
```
true 1 -2
No enum constant Color.PINK
```

enum 상수는 상수마다 객체가 하나씩만 만들어지므로 `==`로 비교해도 된다. `compareTo`는 ordinal의 차이를 돌려준다. GREEN(1)과 RED(0)은 1, RED(0)와 BLUE(2)는 -2다. 없는 이름 "PINK"를 넣으면 IllegalArgumentException이 난다.

#### 상수마다 값을 넣기: 생성자와 필드
상수 뒤에 괄호로 값을 붙이면 그 값은 무엇이 될까?

```java
enum Level {
    LOW(1), MID(2), HIGH(3);
    final int w;
    Level(int w) { this.w = w; }
}
public class Main {
    public static void main(String[] args) {
        for (Level l : Level.values()) System.out.println(l + " " + l.ordinal() + " " + l.w);
    }
}
```
출력:
```
LOW 0 1
MID 1 2
HIGH 2 3
```

괄호 안의 값은 생성자 `Level(int w)`의 인자로 들어가 필드 w에 저장된다. 그래서 `l.w`로 꺼내야 하고, 0부터 매기는 `ordinal()`과는 별개의 값이다. LOW의 w는 1이지만 ordinal은 0이다.

### 예제
```java
enum Color { RED, GREEN, BLUE }
enum Level {
    LOW(1), HIGH(3);
    final int w;
    Level(int w) { this.w = w; }
}
public class Main {
    public static void main(String[] args) {
        for (Color c : Color.values()) System.out.print(c.name() + c.ordinal() + " ");
        System.out.println();
        Color c = Color.valueOf("BLUE");
        System.out.println(c + " " + (c == Color.BLUE) + " " + c.compareTo(Color.RED));
        System.out.println(Level.HIGH.w + Level.LOW.w);
        switch (c) {
            case RED: System.out.println("r"); break;
            case BLUE: System.out.println("b"); break;
            default: System.out.println("?");
        }
    }
}
```
출력:
```
RED0 GREEN1 BLUE2 
BLUE true 2
4
b
```
첫 줄은 values()를 돌면서 상수마다 이름과 순번을 찍은 것이다. `Level.HIGH.w + Level.LOW.w`는 둘 다 int라서 숫자 덧셈이 되어 3 + 1 = 4다.

### 자주 틀리는 포인트
- `ordinal()`은 0부터 센다. 선언 순서대로 매기는 번호라서, 선언 순서를 바꾸면 값도 바뀐다.
- enum 상수에 붙은 괄호 값은 생성자 인자다. 그러니 필드에 저장된 값을 꺼내려면 `.w`처럼 필드 이름으로 접근해야 한다.

## 스레드 (Runnable)
출제: 22-1
함정: `Thread`에 넘기는 것은 `Runnable` 구현 객체다. 빈칸은 `new Thread(new Car())`의 모양이고, 실행은 `start()`이지 `run()`이 아니다.

### 개념
스레드는 main과 따로 동시에 돌아가는 실행 흐름이다. 스레드가 할 일은 `run()` 메서드에 적고, 새 스레드를 띄우는 건 `start()`가 한다. 할 일을 적는 방법은 Runnable 구현과 Thread 상속 두 가지인데, 기출 빈칸은 Runnable 쪽 모양으로 나왔다.

먼저 전체를 표로 보면 이렇다. 스레드를 만드는 두 방법과 실행 메서드의 차이는 아래 표와 같다.

| 방법 | 코드 | 실행 |
|---|---|---|
| Runnable 구현 | `class Car implements Runnable { public void run() {...} }` | `new Thread(new Car()).start()` |
| Thread 상속 | `class Bike extends Thread { public void run() {...} }` | `new Bike().start()` |
| `start()` | 새 스레드에서 run()을 실행 | 순서는 보장되지 않음 |
| `run()` 직접 호출 | 그냥 메서드 호출. 새 스레드 없음 | 호출한 자리에서 순서대로 |
| `join()` | 그 스레드가 끝날 때까지 기다림 | 출력 순서를 고정할 때 |

#### Runnable을 구현해 Thread에 넘긴다
`Runnable`을 구현한 Car 객체를 `new Thread(...)`에 넣고 `start()`를 부르면 어떻게 될까?

```java
class Car implements Runnable {
    public void run() { System.out.println("car running"); }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Car());
        t1.start();
        t1.join();
        System.out.println("main end");
    }
}
```
출력:
```
car running
main end
```

`start()`가 새 스레드를 띄우고 그 스레드에서 Car의 `run()`이 실행된다. `join()`은 그 스레드가 끝날 때까지 main을 기다리게 해서 "car running"이 항상 "main end"보다 먼저 찍힌다. join이 없으면 두 줄의 순서가 보장되지 않는다.

#### start()와 run()은 무엇이 다른가
`run()`을 직접 불러도 같은 내용이 찍힌다. 그럼 차이는 무엇일까? 실행 중인 스레드 이름을 같이 찍으면 드러난다.

```java
class Car implements Runnable {
    public void run() { System.out.println("run in " + Thread.currentThread().getName()); }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Car());
        t.start();
        t.join();
        new Car().run();
    }
}
```
출력:
```
run in Thread-0
run in main
```

`start()`로 실행한 쪽은 새로 만든 스레드 Thread-0에서 돌았다. `run()`을 직접 부른 쪽은 새 스레드 없이 main에서 그냥 메서드처럼 실행됐다. 그래서 `run()` 직접 호출은 호출한 자리에서 순서대로 실행된다.

#### Runnable 객체에는 start()가 없다
Car에 `start()`를 바로 부르면 어떻게 될까?

```java
class Car implements Runnable {
    public void run() { System.out.println("car running"); }
}
public class Main {
    public static void main(String[] args) {
        new Car().start();
    }
}
```
컴파일 결과:
```
Main.java:6: error: cannot find symbol
        new Car().start();
                 ^
  symbol:   method start()
  location: class Car
1 error
```

`start()`는 Thread의 메서드이고 Runnable에는 `run()` 하나뿐이다. Car는 Thread가 아니므로 반드시 `new Thread(new Car())`로 감싼 뒤 `start()`를 부른다. 반면 `extends Thread`로 만든 클래스는 그 자체가 Thread라서 `new Bike().start()`처럼 바로 부를 수 있다.

### 예제
```java
class Car implements Runnable {
    String name;
    Car(String n) { name = n; }
    public void run() { System.out.println(name + " running"); }
}
class Bike extends Thread {
    public void run() { System.out.println("bike running"); }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Car("car"));
        t.start();
        t.join();
        Bike b = new Bike();
        b.start();
        b.join();
        new Car("direct").run();
        System.out.println("main end");
    }
}
```
출력:
```
car running
bike running
direct running
main end
```
start()를 할 때마다 바로 뒤에서 join()으로 기다리므로 출력 순서가 고정된다. 마지막 `run()`은 스레드를 만들지 않는 그냥 메서드 호출이라, main에서 바로 실행된다.

### 자주 틀리는 포인트
- 22-1 원문은 `Thread t1 = new Thread(new _______());`였고, 답은 Runnable을 구현한 클래스 이름인 **Car**다. 여기서 실수하기 쉬운 게, 빈칸 앞뒤의 `new`와 `()`는 이미 적혀 있으니 다시 쓰지 않는다는 점이다. 빈칸이 다른 형태로 나온다면 자리에 따라 `Runnable`(implements 뒤)이나 `implements`, `start`(실행 메서드)가 답이 된다. 그리고 클래스가 `implements Runnable`이라면 `new Car().start()`는 컴파일 오류다. Car는 Thread가 아니기 때문이다.
- join() 없이 start()만 여러 번 하면 출력 순서는 정해지지 않는다. 그래서 기출은 순서가 결정되는 코드만 낸다.

## 람다와 함수형 인터페이스
출제: 25-2
함정: 람다는 추상 메서드가 하나뿐인 인터페이스에만 대입할 수 있다. `(a, b) -> a + b`는 그 메서드의 본문이다.

### 개념
람다는 인터페이스를 구현하는 코드를 짧게 줄여 쓴 것이다. 익명 클래스에서 클래스 이름, 메서드 이름, 반환형을 다 지우고 매개변수와 본문만 남긴 모양이라고 보면 된다. 메서드 이름을 지웠으니 어느 메서드인지 헷갈리지 않도록, 추상 메서드가 딱 하나인 인터페이스에만 쓸 수 있다.

먼저 람다 관련 항목을 표로 보면 이렇다. 아래에서 하나씩 코드로 확인한다.

| 항목 | 내용 |
|---|---|
| 함수형 인터페이스 | 추상 메서드가 하나인 인터페이스. `interface Calc { int run(int a, int b); }` |
| 람다 대입 | `Calc add = (a, b) -> a + b;` 호출은 `add.run(2, 3)` |
| 표준 인터페이스 | `Function<T,R>` apply, `Predicate<T>` test, `Runnable` run, `Supplier<T>` get, `Consumer<T>` accept |
| 스트림 | `Arrays.stream(arr).map(x -> x * 10).sum()` |

#### 익명 클래스를 줄이면 람다
같은 덧셈을 익명 클래스와 람다로 각각 쓰면 아래와 같다.

```java
interface Calc { int run(int a, int b); }
public class Main {
    public static void main(String[] args) {
        Calc add1 = new Calc() {
            public int run(int a, int b) { return a + b; }
        };
        Calc add2 = (a, b) -> a + b;
        Calc sub = (int a, int b) -> { return a - b; };
        System.out.println(add1.run(2, 3) + " " + add2.run(2, 3) + " " + sub.run(2, 3));
    }
}
```
출력:
```
5 5 -1
```

`add1`과 `add2`는 같은 일을 한다. 람다 `(a, b) -> a + b`의 `(a, b)`는 `run`의 매개변수, `a + b`는 본문이자 반환값이다. `sub`처럼 매개변수 타입을 적거나 중괄호와 `return`을 써도 같은 람다다. 호출할 때는 람다에 이름이 없으니 인터페이스의 메서드 이름 `run`으로 부른다.

#### 추상 메서드가 둘이면 람다를 못 쓴다
Calc에 추상 메서드를 하나 더 넣으면 어떻게 될까?

```java
interface Calc {
    int run(int a, int b);
    int undo(int a);
}
public class Main {
    public static void main(String[] args) {
        Calc add = (a, b) -> a + b;
    }
}
```
컴파일 결과:
```
Main.java:7: error: incompatible types: Calc is not a functional interface
        Calc add = (a, b) -> a + b;
                   ^
    multiple non-overriding abstract methods found in interface Calc
1 error
```

"Calc는 함수형 인터페이스가 아니다"라는 오류다. 람다 하나로는 `run`과 `undo` 중 무엇을 구현하는지 정할 수 없기 때문이다.

#### 미리 만들어진 함수형 인터페이스
매번 Calc 같은 인터페이스를 만들지 않도록 `java.util.function`에 자주 쓰는 모양이 준비되어 있다. 눈여겨볼 것은 인터페이스마다 호출하는 메서드 이름이 다르다는 점이다.

```java
import java.util.function.*;
public class Main {
    public static void main(String[] args) {
        Function<Integer, String> f = x -> "n" + x;
        Predicate<String> p = s -> s.isEmpty();
        Supplier<Integer> sup = () -> 42;
        Consumer<String> con = s -> System.out.println("got " + s);
        System.out.println(f.apply(7) + " " + p.test("") + " " + sup.get());
        con.accept("hi");
    }
}
```
출력:
```
n7 true 42
got hi
```

Function은 받아서 돌려주는 `apply`, Predicate는 참거짓을 돌려주는 `test`, Supplier는 받지 않고 돌려만 주는 `get`, Consumer는 받기만 하는 `accept`로 부른다.

#### 람다 안에서 던진 예외
인터페이스 메서드에 `throws Exception`이 있으면 람다 본문에서 예외를 던질 수 있다. 그 예외는 어디서 잡힐까? 코드는 25-2 풀이와 같은 구조로 짰다.

```java
interface F { int apply(int x) throws Exception; }
public class Main {
    static int run(F f) {
        try {
            return f.apply(3);
        } catch (Exception e) {
            return 7;
        }
    }
    public static void main(String[] args) {
        int a = run(x -> { throw new Exception(); });
        int b = run((int n) -> n + 9);
        System.out.println(a + " " + b + " " + (a + b));
    }
}
```
출력:
```
7 12 19
```

람다 본문은 `f.apply(3)`이 불릴 때 실행된다. 그래서 첫 번째 람다가 던진 예외는 `f.apply(3)`을 감싼 `run`의 try로 전달되고, catch가 7을 돌려준다. 두 번째는 예외 없이 3 + 9 = 12다.

### 예제
```java
import java.util.function.*;
interface Calc { int run(int a, int b); }
public class Main {
    public static void main(String[] args) {
        Calc add = (a, b) -> a + b;
        Calc mul = (a, b) -> a * b;
        System.out.println(add.run(2, 3) + " " + mul.run(2, 3));
        Function<Integer, Integer> sq = x -> x * x;
        Predicate<Integer> even = x -> x % 2 == 0;
        System.out.println(sq.apply(4) + " " + even.test(3));
        Runnable r = () -> System.out.println("run");
        r.run();
        int[] arr = {3, 1, 2};
        System.out.println(java.util.Arrays.stream(arr).map(x -> x * 10).sum());
    }
}
```
출력:
```
5 6
16 false
run
60
```
람다 본문이 인터페이스 메서드가 되므로, 같은 run(2, 3)이라도 add는 덧셈을, mul은 곱셈을 한다. 스트림은 각 원소에 10을 곱해 더하니 (30+10+20)이다.

### 자주 틀리는 포인트
- 람다 자체에는 이름이 없다. 그래서 람다 변수의 타입(Calc)과 호출 메서드 이름(run)을 짝지어 읽어야 한다.
- 익명 클래스 `new Calc() { public int run(int a, int b) { return a + b; } }`와 같은 뜻이다.
- 25-2는 `interface F { int apply(int x) throws Exception; }`에 람다를 넣은 문제였다. 람다 본문의 `throw new Exception()`은 `f.apply(3)`을 부른 `run`의 try로 전달되고, 거기서 catch가 7을 돌려준다. 두 번째 `run((int n) -> n + 9)`는 예외 없이 3 + 9 = 12다. 그래서 합은 7 + 12 = **19**다. 참고로 매개변수에 `(int n)`처럼 타입을 적어도 똑같은 람다다.
