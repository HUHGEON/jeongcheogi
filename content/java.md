# Java
Java는 2023년 이후 회차마다 1~4문제(대개 3~4문제)가 나온다. 상속·오버라이딩·`super`, 생성자 호출 순서, 오버로딩에서 어느 메서드가 선택되는지, static, 예외 흐름, `==`와 `equals`가 반복되고, 키워드 빈칸(`static`, `new`, `super`, `implements`, `extends`)도 섞여 나온다. 클래스가 3~4개 얽힌 코드에서 "어느 클래스의 메서드가 실행되는가"를 묻는 것이 핵심이다. 공통 규칙(추적 표, 비트, 형변환, 나눗셈)은 [코딩 공통] 페이지에 있다.

## 상속·오버라이딩·동적 바인딩·super
출제: 26-2, 26-1, 25-3, 23-3, 21-2, 20-4
함정: 참조 변수의 타입이 아니라 `new`로 만든 객체의 타입이 어느 메서드가 실행될지 정한다.

### 개념
| 규칙 | 내용 |
|---|---|
| 동적 바인딩 | `Shape s = new Square(3); s.area()`는 Square의 area()를 실행한다. 변수 타입 Shape는 "어떤 메서드를 부를 수 있는가"만 정한다 |
| 오버라이딩 | 부모와 이름·매개변수가 같은 메서드를 자식이 다시 정의. `@Override` 표시는 선택 |
| 부모 메서드 안의 호출 | 부모 메서드 안에서 `area()`를 불러도 자식이 오버라이딩했으면 자식 것이 실행된다 |
| `super.메서드()` | 부모 버전을 명시적으로 호출. 체인이 길면 한 단계 위만 |
| 자식 전용 메서드 | `Shape s`로는 Square에만 있는 메서드를 못 부른다. `((Square) s).m()`으로 다운캐스팅해야 한다 |
| 배열 순회 | `Shape[] arr = {new Shape(), new Rect()}` 각 원소의 실제 타입대로 실행 |

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
s는 실제로 Square이므로 area()는 Rect의 것(Square는 안 고침) → 9.0. info()는 Square → `super`로 Rect → `super`로 Shape 순으로 이어붙이고, Shape.info() 안의 area()도 동적 바인딩으로 9.0. 배열은 0 + 8.

### 자주 틀리는 포인트
- 메서드를 찾을 때 "객체의 실제 클래스에서 시작해 위로" 올라간다. Square에 area()가 없으면 Rect, 거기도 없으면 Shape.
- `super.info()`는 바로 위 부모의 info()다. 그 안에서 또 `super`를 부르면 한 단계 더 위로 간다.
- 반환형이 double인 메서드는 `9.0`처럼 소수점이 붙어 출력된다. `9`라고 쓰면 틀린다.
- 접근 제어자가 붙은 오버라이딩(26-2)은 자식이 부모보다 좁은 범위로 줄일 수 없다. 접근 제어자 주제 참고.

## 생성자 호출 순서와 this()·super()
출제: 25-1, 24-1, 23-1, 22-2, 20-3, 20-2
함정: 자식 생성자의 첫 줄에는 항상 `super()`가 숨어 있다. 부모 생성자가 먼저 끝나고 자식 본문이 실행된다.

### 개념
`new B(7)` 한 번에 실행되는 순서:

| 순서 | 내용 |
|---|---|
| 1 | 클래스 로딩 시 static 블록(부모 → 자식). 프로그램에서 한 번만 |
| 2 | 자식 생성자 진입. 첫 줄이 `this(...)`면 같은 클래스의 다른 생성자로 간다 |
| 3 | 첫 줄이 `super(...)`이거나 아무것도 없으면 부모 생성자 `super()` 호출 |
| 4 | 부모 쪽에서도 같은 규칙(부모의 this/super, 부모의 초기화 블록, 부모 본문) |
| 5 | 자식의 필드 초기화와 인스턴스 초기화 블록 `{ }` |
| 6 | 자식 생성자 본문 |

`this(...)`와 `super(...)`는 생성자 첫 줄에만 쓸 수 있고 둘 중 하나만 쓴다. 매개변수 있는 생성자만 있는 부모를 상속하면 자식은 반드시 `super(인자)`를 명시해야 한다(빈칸 단골).

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
`new B(7)` → `this()`로 B() → `super()`로 A() → `this(1)`로 A(int)가 먼저 출력. 그다음 A() 본문, B의 초기화 블록, B() 본문, B(7) 본문 순. static 블록은 첫 `new` 직전에 한 번만 나오고 두 번째 `new B()`에서는 안 나온다.

### 자주 틀리는 포인트
- 출력 순서는 "가장 깊은 생성자의 본문부터"다. 호출 순서와 실행(출력) 순서를 구분해 적는다.
- `this()`로 다른 생성자를 거치면 `super()`는 그 생성자에서 한 번만 호출된다. 두 번 호출되지 않는다.
- 24-1처럼 "실행 순서를 번호로 나열"하는 문제는 각 줄에 번호를 붙이고 위 표의 1~6 순서대로 따라간다.
- 생성자 본문에서 오버라이딩된 메서드를 부르면(25-1) 자식 필드가 아직 초기화 전이라 0이나 null이 보일 수 있다.

## 추상 클래스·인터페이스·implements
출제: 25-3, 25-2, 24-2, 23-1, 20-3, 20-2
함정: 인터페이스 변수로 받아도 실행되는 것은 구현 클래스의 메서드다. 빈칸은 `implements`·`extends`·`abstract`·`new` 중 하나다.

### 개념
| 항목 | 추상 클래스 | 인터페이스 |
|---|---|---|
| 선언 | `abstract class Base` | `interface Greet` |
| 상속 키워드 | `extends` (하나만) | `implements` (여러 개 가능) |
| 메서드 | 추상·일반 메서드 섞을 수 있음 | 추상 메서드가 기본(public abstract 생략). `default` 메서드는 본문 가능 |
| 필드 | 일반 필드 가능 | 상수(`public static final`)만 |
| 객체 생성 | `new Base()` 불가. 자식으로만 | `new Greet()` 불가. 구현 클래스 또는 익명 클래스 `new Greet() { ... }` |
| 구현 메서드 접근 | | 반드시 `public` |

업캐스팅 빈칸(20-2): `Parent p = new Child();`의 `new`. 익명 클래스 `new Greet() { public String hello() { ... } }`는 인터페이스를 그 자리에서 구현한다.

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
Base.hello() 안의 size()는 Impl의 것이라 "hi3". bye()는 Impl이 default 메서드를 오버라이딩했으므로 "see you". 익명 클래스는 bye()를 안 고쳐 default "bye".

### 자주 틀리는 포인트
- 추상 클래스는 추상 메서드가 하나라도 있으면 `abstract`를 붙여야 하고, 자식은 전부 구현해야 객체를 만들 수 있다.
- 인터페이스 메서드를 구현할 때 `public`을 빼면 컴파일 오류다("접근 범위를 좁힐 수 없다").
- 인터페이스+예외 조합(25-2)은 구현 클래스의 메서드 안에서 던진 예외가 호출한 쪽 catch로 간다. 예외 흐름 주제 참고.
- `g instanceof Base`는 실제 객체(Impl)가 Base의 자손이면 true다.

## 오버로딩 (인자 타입으로 메서드 선택)
출제: 26-1, 25-1, 24-3, 23-1, 21-2
함정: 오버로딩은 컴파일 시점에 인자의 "선언 타입"으로 고른다. 정확히 같은 타입 → 넓히기(int→long→double) → 박싱(int→Integer) → Object 순이다.

### 개념
| 호출 인자 | 후보가 있을 때 선택 순서 |
|---|---|
| `f(1)` | int → long → float → double → Integer → Number → Object |
| `f('a')` | char → int → long → ... (char는 int로 넓혀진다. Character 박싱보다 int 넓히기가 먼저) |
| `f(1L)` | long → float → double → Long → Object |
| `f(1.0f)` | float → double → Float → Object |
| `f("s")` | String → Object |
| `f(Integer.valueOf(1))` | Integer → Number → Object → (언박싱 int는 그 뒤) |

오버로딩 vs 오버라이딩: 오버로딩은 같은 클래스(또는 상속 관계)에서 이름 같고 매개변수가 다른 메서드들, 선택은 컴파일 시 인자 타입으로. 오버라이딩은 부모 메서드를 자식이 같은 시그니처로 다시 정의, 선택은 실행 시 객체 타입으로.

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
`f(1.0f)`는 float 버전이 없으니 double로 넓힌다. `f('a')`는 char 버전이 없으니 int로 넓힌다(Integer 박싱보다 우선). `add('1', '2')`는 char 버전이 있어 49 + 50 = 99를 int로 반환한다.

### 자주 틀리는 포인트
- `add(char, char)`의 반환이 int면 문자 합이 숫자로 나온다. 반환형과 출력 서식을 확인한다.
- `Object` 버전만 있을 때 `f(1)`을 부르면 int가 Integer로 박싱되어 Object로 간다. 오류가 아니다.
- 24-3처럼 Integer/Object/Number 버전이 같이 있으면 Integer 인자는 Integer 버전, int 인자는 (int 버전이 없다면) Integer 버전으로 간다.
- 문자열을 받는 오버로딩이 재귀로 이어지는 문제(25-1)는 매 호출에서 인자 타입이 무엇인지 적으면서 내려간다.

## static 멤버와 static 메서드 hiding
출제: 25-2, 25-1, 23-3, 23-1, 21-2
함정: static 메서드는 오버라이딩되지 않는다. 참조 변수의 타입으로 결정된다(hiding). 인스턴스 메서드와 반대다.

### 개념
| 항목 | 내용 |
|---|---|
| static 필드 | 클래스에 하나. 모든 객체가 공유. 생성자에서 `count++`하면 객체 수가 된다 |
| static 메서드 | 객체 없이 `클래스.메서드()`. 안에서 `this`·인스턴스 변수 사용 불가(23-3 오류 라인 문제) |
| static 메서드 hiding | 부모와 자식이 같은 static 메서드를 가지면 `Parent p = new Child(); p.who()`는 Parent의 것 |
| 인스턴스 메서드 오버라이딩 | 같은 상황에서 `p.me()`는 Child의 것 |
| 자식 클래스로 static 접근 | `Child.count`는 Parent의 static 필드와 같은 것 |
| static 블록 | 클래스가 처음 쓰일 때 한 번 실행 |

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
Child 생성 시에도 Parent() 생성자가 돌아 count가 는다. 객체 3개라 3. `p.who()`는 변수 타입 Parent의 static → "Parent", `p.me()`는 객체 타입 Child의 인스턴스 메서드 → "child1".

### 자주 틀리는 포인트
- 같은 `Parent p = new Child()`에서 static 메서드는 Parent 것, 인스턴스 메서드는 Child 것이다. 메서드 선언에 `static`이 있는지부터 본다.
- static 메서드 안에서 인스턴스 변수를 쓰면 컴파일 오류다. "오류가 나는 줄 번호"를 묻는 문제의 답이다.
- static 빈칸(21-2): 객체 없이 `클래스.메서드()`로 부르거나 모든 객체가 공유하는 카운터면 `static`이다.
- 싱글톤 패턴(별도 주제)은 static 필드 + private 생성자 + static getInstance()다.

## 재귀
출제: 26-2, 25-1, 24-2, 23-3, 20-4
함정: 문자열 재귀는 `substring(1)`로 한 글자씩 줄여 가고, 결과를 "재귀 결과 + 첫 글자"로 붙이면 뒤집힌다.

### 개념
| 유형 | 모양 | 결과 |
|---|---|---|
| 합·팩토리얼 | `n == 0 ? 0 : n + sum(n-1)` | sum(10) = 55 |
| 문자열 뒤집기 | `rev(s.substring(1)) + s.charAt(0)` | "abcd" → "dcba" |
| 연속 중복 제거 | 첫 두 글자가 같으면 첫 글자 버리고 재귀 | "aabbbca" → "abca" |
| 분할(이진 탐색) | 중간 비교 후 왼쪽/오른쪽 절반으로 재귀 | 인덱스 또는 -1 |
| 상속+재귀(20-4) | 부모의 재귀 메서드를 자식이 오버라이딩하면 재귀 호출도 자식 것으로 간다 | |

추적은 [코딩 공통] 재귀 추적법대로 호출 트리를 그린다. 문자열은 매 호출에서 s가 무엇인지 적는다.

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
rev("abcd") = rev("bcd") + 'a' = (rev("cd") + 'b') + 'a' = ... = "dcba". dedup은 "aa"에서 a 하나 버림, "bbb"에서 두 개 버림. bin(9): mid=2(5) → 오른쪽 lo=3, mid=4(9) → 4. bin(4): mid=2(5) → 왼쪽 hi=1, mid=0(1) → 오른쪽 lo=1, mid=1(3) → 오른쪽 lo=2 > hi=1 → -1.

### 자주 틀리는 포인트
- `s.charAt(0) + dedup(...)`에서 charAt은 char이고 뒤가 String이라 문자열 연결이 된다. 둘 다 char면 숫자 덧셈이 되므로 타입을 본다.
- 이진 탐색의 `mid = (lo + hi) / 2`는 정수 나눗셈이다. 매 호출의 lo, hi, mid를 표로 적는다.
- 재귀 안에서 오버라이딩된 메서드를 부르면(20-4) 자식 객체일 때 자식 메서드가 재귀된다.

## 배열 (length, 2차원 가변, 반환, 순위 계산)
출제: 22-3, 22-2, 21-1, 20-4, 20-1
함정: `length`는 배열의 필드(괄호 없음), `length()`는 String의 메서드, `size()`는 컬렉션이다.

### 개념
| 식 | 뜻 |
|---|---|
| `int[] a = new int[4]` | 0으로 채워진 길이 4. `a.length` = 4 |
| `int[][] m = new int[3][]` | 행만 3개. 각 행은 null. `m[i] = new int[i+1]`로 길이가 다른 행 |
| `m.length`, `m[2].length` | 행 수, 2행의 열 수 |
| `for (int v : a)` | 향상된 for. 원소 값만 읽는다 |
| 배열 반환 | 메서드가 `int[]`를 돌려주면 호출 쪽 변수가 같은 배열을 가리킨다 |
| 순위 계산 | `rank[i] = 1; for j: if (sc[j] > sc[i]) rank[i]++;` 자기보다 큰 수의 개수 + 1 |
| `Arrays.toString(a)` | `[70, 90, 80]` 모양 |

객체 배열 `Box[] arr = new Box[3]`은 원소가 null이다. `arr[i] = new Box()`를 해야 쓸 수 있다.

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
make(4)는 {0,1,4,9}. 가변 배열은 행 길이가 1,2,3이라 m[2].length = 3. 순위: 70보다 큰 수 2개 → 3등, 90은 1등, 80은 2등.

### 자주 틀리는 포인트
- `System.out.println(a)`로 배열을 직접 찍으면 `[I@...` 같은 주소가 나온다. 출력 문제는 반복문이나 `Arrays.toString`을 쓴다.
- 2차원 가변 배열 크기 빈칸(20-4, 21-1): 행 수는 `m.length`, 열 수는 `m[i].length`. 행마다 다를 수 있으므로 i를 붙인다.
- 순위 계산에서 비교 부호가 `>=`이면 자기 자신도 세어 모두 1씩 커진다.
- 배열 `==` 비교는 참조 비교다(==와 equals 주제).

## ==와 equals, String 메서드
출제: 24-3, 24-2, 23-2
함정: 리터럴 `"java"`끼리는 `==`가 true(같은 상수 풀), `new String("java")`는 false. 내용 비교는 항상 `equals`.

### 개념
| 식 | 결과 | 이유 |
|---|---|---|
| `"java" == "java"` | true | 리터럴은 상수 풀에서 공유 |
| `"java" == new String("java")` | false | new는 새 객체 |
| `"ja" + "va" == "java"` | true | 상수끼리의 연결은 컴파일 때 합쳐진다 |
| `a.substring(0,2) + "va" == a` | false | 실행 중에 만든 새 문자열 |
| `a.equals(b)` | 내용 비교 | 항상 이것으로 비교 |
| `int[] x == int[] y` | false | 배열도 참조 비교. 내용은 `Arrays.equals(x, y)` |

자주 나오는 String 메서드: `length()`, `charAt(i)`, `substring(a, b)`(b 미포함), `indexOf("c")`(없으면 -1), `split(",")`, `toUpperCase()`, `equals`, `isEmpty()`, `trim()`, `contains`. `split`은 연속된 구분자 사이에 빈 문자열이 생기고, 끝의 빈 문자열은 버린다.

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
"a,b,,c"를 ","로 나누면 "a", "b", "", "c" 네 조각이고 t[2]는 빈 문자열. charAt(2)는 'b', indexOf("c")는 5.

### 자주 틀리는 포인트
- 문자열 배열을 `for`로 돌며 `==`로 비교하는 문제(24-3)는 원소가 리터럴이면 true, `new String`이나 입력으로 만든 것이면 false다.
- `equals`는 대소문자를 구분한다. `equalsIgnoreCase`가 아니면 "Java"와 "java"는 다르다.
- `substring(0, 2)`는 인덱스 0, 1 두 글자다. 끝 인덱스는 포함하지 않는다.
- `split` 결과 길이(24-2)는 구분자 사이의 빈 조각을 포함하되 맨 끝 빈 조각은 제외한다. "a,b,"는 길이 2.

## 예외 흐름 (try-catch-finally)
출제: 25-2, 25-1, 24-3
함정: `finally`는 `return`이 있어도, 예외가 catch되지 않아도 실행된다. 출력 순서에서 finally가 return 값보다 먼저 찍힌다.

### 개념
| 상황 | 흐름 |
|---|---|
| try에서 예외 없음 | try 끝까지 → finally → 다음 문장 |
| try에서 예외, 맞는 catch 있음 | 예외 지점에서 try를 빠져나옴 → catch → finally → 다음 문장 |
| try에서 예외, 맞는 catch 없음 | finally 실행 후 호출한 쪽으로 예외가 올라간다 |
| try나 catch에 return | return 값을 정한 뒤 finally를 실행하고 반환한다 |
| 여러 catch | 위에서부터 처음 맞는 하나만. 부모 예외 타입(Exception)을 위에 두면 아래는 컴파일 오류 |

자주 나오는 예외: `ArithmeticException`(0으로 나눔), `ArrayIndexOutOfBoundsException`, `NullPointerException`, `NumberFormatException`(`Integer.parseInt("a")`). 이들은 모두 `RuntimeException`의 자식이라 `catch (RuntimeException e)` 또는 `catch (Exception e)`에 잡힌다. `throw new X("msg")`로 직접 던지고 `e.getMessage()`로 꺼낸다.

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
f(5): return 2가 정해지고 finally가 "F "를 찍은 뒤 println이 2를 찍는다. f(0): catch가 "A:zero "를 찍고 return -1, finally "F ", 그 뒤 -1. f(-1): IllegalArgumentException은 f 안의 catch에 안 맞으므로 finally "F "만 찍고 main의 catch로 올라가 "main:neg". f(-1)의 println은 실행되지 않는다.

### 자주 틀리는 포인트
- `System.out.println(f(0))`에서 f 안의 출력이 먼저, 반환값 출력이 나중이다. 한 줄에 "A:zero F -1"처럼 섞여 나온다.
- 예외가 catch되지 않은 호출의 println은 실행되지 않는다. 그 줄 대신 바깥 catch의 출력이 나온다.
- finally에 return이 있으면 try·catch의 return을 덮어쓴다. 기출에서는 드물지만 보이면 finally 값이 답이다.
- 인터페이스 구현 메서드 안에서 던진 예외(25-2)도 같은 규칙으로 호출자 쪽 catch를 찾아 올라간다.

## 반복문과 출력 형식
출제: 22-3, 22-2, 21-1, 20-3
함정: `print`와 `println`의 차이, 그리고 `+`로 이어붙일 때 숫자 덧셈이 되는 구간이 어디까지인지.

### 개념
| 항목 | 내용 |
|---|---|
| `print` / `println` | 줄바꿈 없음 / 있음. `println()`만 쓰면 빈 줄 |
| `printf("%d %5.2f %s%n", ...)` | C와 같은 서식. `%n`이 줄바꿈 |
| `while (i <= 100) { sum += i; i += 2; }` | 홀수 합. 끝난 뒤 i 값도 자주 묻는다 |
| 별 찍기 | 바깥 r행, 안쪽 r개. 안쪽이 `c <= r`인지 `c <= n - r`인지로 모양이 정해진다 |
| 최대 배수 | 조건을 만족할 때마다 덮어쓰면 마지막(최대) 값이 남는다 |
| `1 + 2 + "=" + 1 + 2` | "3=12". 문자열을 만나기 전까지만 덧셈 |

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
1~99 홀수 합은 50개 × 평균 50 = 2500이고, 99 다음 i = 101에서 조건이 깨진다. 50 미만 7의 배수 중 마지막은 49. `%5.2f`는 "3.14" 앞에 공백 1칸.

### 자주 틀리는 포인트
- `sum + " " + i`는 앞의 sum이 숫자 하나뿐이라 그대로 "2500 101"이다. `sum + i + " "`였다면 2601이 먼저 계산된다.
- 반복이 끝난 뒤의 i는 조건을 처음 깬 값이다(홀수 증가라 100이 아니라 101).
- 객체 배열을 반복하며 생성자에서 출력하는 문제(22-2)는 생성자 호출 횟수만큼 출력이 반복된다.

## 참조 전달 (객체 필드·배열·String)
출제: 25-2, 22-1
함정: 객체를 넘기면 필드 변경은 반영되고, 매개변수에 `new`로 다른 객체를 넣는 것은 반영되지 않는다. String은 불변이라 바뀌지 않는다.

### 개념
| 매개변수에 한 일 | 호출자에게 보이는가 |
|---|---|
| `b.v = 100` (객체 필드 변경) | 보인다 |
| `arr[0] = 100` (배열 원소 변경) | 보인다 |
| `n = 100` (기본형 재대입) | 안 보인다 |
| `b = new Box(7)` (참조 재대입) | 안 보인다. 그 뒤 `b.v = 999`도 새 객체에만 |
| `s = s + "!"` (String) | 안 보인다. 새 문자열이 만들어져 지역 변수만 바뀜 |

Java는 항상 "값 전달"이고 그 값이 참조(주소)일 뿐이다. 두 변수가 같은 객체를 가리키면(`Box other = box`) 한쪽으로 바꾼 필드가 다른 쪽에도 보인다.

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
객체 필드와 배열 원소만 바뀌고 int와 String은 그대로. replace는 지역 변수 b만 새 객체로 바꿔 box.v는 100 유지. other와 box는 같은 객체라 5.

### 자주 틀리는 포인트
- String 배열을 넘겨 `arr[0] = "x"`로 바꾸는 것(25-2)은 배열 원소 재대입이므로 반영된다. String 자체를 바꾸는 것과 구분한다.
- 메서드 안에서 `b = null`이나 `b = new ...`를 한 뒤의 변경은 호출자와 무관하다.
- `box == other`는 같은 객체를 가리키므로 true. 내용이 같은 다른 객체면 false.

## 싱글톤 패턴 코드
출제: 24-1, 21-3
함정: `getInstance()`를 몇 번 불러도 객체는 하나다. 카운터 필드가 인스턴스 필드여도 공유되는 것처럼 보인다.

### 개념
| 구성 | 코드 | 역할 |
|---|---|---|
| static 인스턴스 필드 | `private static Printer instance;` | 유일한 객체를 보관 |
| private 생성자 | `private Printer() { }` | 밖에서 `new` 금지 |
| static 접근 메서드 | `static Printer getInstance() { if (instance == null) instance = new Printer(); return instance; }` | 처음 한 번만 생성, 이후 같은 객체 반환 |

빈칸은 `static`, `instance == null`, `new Printer()`, `return instance` 자리에 나온다. 디자인 패턴 이름을 같이 묻기도 한다(Singleton, 생성 패턴).

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
a와 b는 같은 객체라 `==`가 true이고, print()를 합쳐 3번 불렀으므로 둘 다 3.

### 자주 틀리는 포인트
- count는 static이 아니지만 객체가 하나라서 결과가 공유된다. "static이 아니니 따로 센다"고 보면 틀린다.
- `getInstance()`가 static이어야 객체 없이 부를 수 있다. 빈칸 1순위.
- 생성자가 private이므로 `new Printer()`는 클래스 밖에서 컴파일 오류다.

## switch fall-through
출제: 22-2, 20-1
함정: `break`가 없으면 아래 case로 떨어진다. 반복문 안에서 `continue`를 만나면 switch 뒤의 문장을 건너뛰고 다음 반복으로 간다.

### 개념
C와 규칙이 같다([C] switch 주제). Java 고유 사항: `switch`에 String과 enum을 쓸 수 있다. 화살표 문법(`case 1 -> ...`)은 떨어지지 않지만 기출은 콜론 문법이다.

| 상황 | 결과 |
|---|---|
| `case 2: s += 2; case 3: s += 3; break;` 에 n=2 | 2와 3 둘 다 더해짐 |
| default가 마지막이고 그 위 case에 break 없음 | default도 실행 |
| 반복문 안 `case 0: ...; continue;` | switch와 그 아래를 건너뛰고 다음 반복 |

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
n=2는 case 2부터 break까지 2 + 3 = 5. "B"는 good, ok, end까지 떨어진다. i=0과 2는 "e"를 찍고 continue라 "|"를 건너뛰고, i=1만 "o|".

### 자주 틀리는 포인트
- `break`가 switch를 빠져나가는 것이지 for를 끝내는 것은 아니다. `continue`는 for의 다음 반복으로 간다.
- String switch는 `equals`로 비교한다. 대소문자가 다르면 default로 간다.

## 필드 hiding
출제: 24-3
함정: 필드는 오버라이딩되지 않는다. `P p = new C(); p.v`는 P의 v, `p.get()`은 C의 get()이 돌려주는 C의 v다.

### 개념
| 식 (`P p = new C()`, 둘 다 `int v`) | 결과 | 이유 |
|---|---|---|
| `p.v` | P의 v | 필드는 변수 타입으로 결정 |
| `p.get()` | C의 v | 메서드는 객체 타입으로 결정되고, C.get()은 C.v를 읽는다 |
| `((C) p).v` | C의 v | 캐스팅하면 C 타입으로 접근 |
| `super.v` (C 안에서) | P의 v | 가려진 부모 필드 접근 |
| `p.v = 99` | P의 v만 바뀜 | C의 v는 별개 변수 |

한 객체 안에 v가 두 개(P의 v, C의 v) 들어 있다고 그리면 된다.

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
p와 c는 같은 객체다. `p.v`는 P 쪽 변수 10, `p.get()`은 C.get()이라 20. `p.v = 99`는 P 쪽 변수만 바꿔 `((P) c).v`가 99이고 `c.v`는 20 그대로.

### 자주 틀리는 포인트
- 오버라이딩(메서드)과 hiding(필드·static 메서드)을 구분한다. 메서드만 객체 타입을 따른다.
- C의 get()이 없고 P의 get()만 있으면 `p.get()`은 P.get()이 P의 v(10)를 돌려준다. 메서드가 어느 클래스에 정의됐는지에 따라 읽는 v가 달라진다.

## 접근 제어자
출제: 26-2
함정: `private` 멤버는 자식 클래스에서도 직접 못 쓴다. 오버라이딩할 때 자식은 부모보다 접근 범위를 좁힐 수 없다.

### 개념
| 제어자 | 같은 클래스 | 같은 패키지 | 자식 클래스(다른 패키지) | 전체 |
|---|---|---|---|---|
| `private` | O | | | |
| (없음, default) | O | O | | |
| `protected` | O | O | O | |
| `public` | O | O | O | O |

오버라이딩 규칙: 부모가 `public`이면 자식도 `public`. 부모가 `protected`면 자식은 `protected` 또는 `public`. 좁히면 컴파일 오류. 인터페이스 메서드 구현은 반드시 `public`.

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
Saving은 private balance에 직접 접근할 수 없어 `super.getBalance()`를 거친다. audit()은 Account에 정의돼 있으므로 private log()와 balance를 쓸 수 있다. 같은 파일(같은 패키지)이라 protected와 default 필드는 Saving에서 보인다. `a.balance`나 `a.log()`를 main에 쓰면 컴파일 오류다.

### 자주 틀리는 포인트
- "오류가 나는 줄"을 묻는 문제에서 private 멤버를 클래스 밖에서 쓰는 줄, 또는 오버라이딩에서 접근 범위를 좁힌 줄이 답이다.
- private 메서드는 오버라이딩 대상이 아니다. 자식에 같은 이름의 메서드가 있어도 부모 쪽 코드에서는 부모 것이 실행된다.
- 기출 코드는 한 파일 안에 있으므로 default(패키지) 접근이 항상 허용된다.

## enum
출제: 25-3
함정: `values()`는 선언 순서 배열, `ordinal()`은 0부터 시작하는 순번, `name()`과 `toString()`은 상수 이름 문자열이다.

### 개념
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
values()를 돌며 이름과 순번을 찍는다. `Level.HIGH.w + Level.LOW.w`는 3 + 1 = 4(둘 다 int라 숫자 덧셈).

### 자주 틀리는 포인트
- `ordinal()`은 0부터다. 선언 순서를 바꾸면 값이 바뀐다.
- enum 상수에 붙은 괄호 값은 생성자 인자다. 필드에 저장된 값을 꺼내려면 `.w`처럼 필드 이름으로 접근한다.

## 스레드 (Runnable)
출제: 22-1
함정: `Thread`에 넘기는 것은 `Runnable` 구현 객체다. 빈칸은 `new Thread(new Car())`의 모양이고, 실행은 `start()`이지 `run()`이 아니다.

### 개념
| 방법 | 코드 | 실행 |
|---|---|---|
| Runnable 구현 | `class Car implements Runnable { public void run() {...} }` | `new Thread(new Car()).start()` |
| Thread 상속 | `class Bike extends Thread { public void run() {...} }` | `new Bike().start()` |
| `start()` | 새 스레드에서 run()을 실행 | 순서는 보장되지 않음 |
| `run()` 직접 호출 | 그냥 메서드 호출. 새 스레드 없음 | 호출한 자리에서 순서대로 |
| `join()` | 그 스레드가 끝날 때까지 기다림 | 출력 순서를 고정할 때 |

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
각 start() 뒤에 join()으로 기다리므로 순서가 고정된다. 마지막 `run()`은 스레드를 만들지 않고 main에서 바로 실행된다.

### 자주 틀리는 포인트
- 빈칸 문제(22-1)의 답은 `new Thread(new Car())` 또는 `Runnable`·`implements`·`start`다. 클래스가 `implements Runnable`이면 `new Car().start()`는 컴파일 오류다(Car는 Thread가 아니다).
- join() 없이 start()만 여러 번 하면 출력 순서는 정해지지 않는다. 기출은 순서가 결정되는 코드만 낸다.

## 람다와 함수형 인터페이스
출제: 없음
함정: 람다는 추상 메서드가 하나뿐인 인터페이스에만 대입할 수 있다. `(a, b) -> a + b`는 그 메서드의 본문이다.

### 개념
| 항목 | 내용 |
|---|---|
| 함수형 인터페이스 | 추상 메서드가 하나인 인터페이스. `interface Calc { int run(int a, int b); }` |
| 람다 대입 | `Calc add = (a, b) -> a + b;` 호출은 `add.run(2, 3)` |
| 표준 인터페이스 | `Function<T,R>` apply, `Predicate<T>` test, `Runnable` run, `Supplier<T>` get, `Consumer<T>` accept |
| 스트림 | `Arrays.stream(arr).map(x -> x * 10).sum()` |

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
람다 본문이 인터페이스 메서드가 되므로 run(2, 3)은 각각 덧셈·곱셈. 스트림은 (30+10+20).

### 자주 틀리는 포인트
- 람다 변수의 타입(Calc)과 호출 메서드 이름(run)을 짝지어 읽는다. 람다 자체에는 이름이 없다.
- 익명 클래스 `new Calc() { public int run(int a, int b) { return a + b; } }`와 같은 뜻이다.
