# Python
Python은 2023년 이후 회차마다 0~3문제, 대개 1~2문제씩 나온다. 그중 슬라이싱이 7개 회차로 압도적으로 많고, 그 뒤를 문자열 메서드(split·join), 딕셔너리·집합 조작, 리스트 메서드, 컴프리헨션, 얕은 복사, 클래스가 잇는다. 코드 자체는 짧은데 `[::-1]` 같은 한 글자 차이로 답이 갈린다. 출력도 모양 그대로 옮겨 적어야 한다. 리스트는 `[1, 2]`, 튜플은 `(1, 2)`처럼 괄호까지 쓰고, 문자열을 print로 직접 찍으면 따옴표 없이 쓰지만 리스트 안에 든 문자열은 `'a'`처럼 따옴표가 붙은 채로 쓴다. 나눗셈, 비트, 거짓 값, 2차원 리스트 같은 공통 규칙은 [코딩 공통] 페이지에 따로 정리해 두었다.

## 슬라이싱 [a:b:c]와 음수 인덱스
출제: 26-2, 26-1, 24-3, 24-1, 23-2, 22-2, 21-1
함정: 끝 인덱스 b는 포함하지 않는다. step이 음수면 방향이 거꾸로이고 a와 b의 기본값도 뒤집힌다.

### 개념
**슬라이싱(Slicing)**이란 문자열이나 리스트에서 일부를 잘라 내는 문법이다. 모양은 `s[시작:끝:간격]` 하나뿐이고, 세 가지만 기억하면 거의 모든 문제가 풀린다.

1. **끝은 포함하지 않는다.** 끝 번호에 닿기 바로 앞에서 멈춘다.
2. **비우면 기본값이 들어간다.** 시작을 비우면 처음부터, 끝을 비우면 마지막까지, 간격을 비우면 1칸씩이다.
3. **간격이 음수면 거꾸로 간다.** 이때는 시작과 끝의 기본값도 같이 뒤집혀서, 시작을 비우면 마지막 글자부터 출발한다.

말로만 보면 헷갈리니 `s = "programming"`의 번호를 먼저 적어 보자. 글자 위가 앞에서 센 번호(0부터), 아래가 뒤에서 센 번호(-1부터)다. 한 글자에 번호가 두 개씩 붙어 있는 셈이라, `s[2]`와 `s[-9]`는 같은 글자 o를 가리킨다.

```idx
제목: s = "programming"의 두 가지 번호
programming
```

읽는 법은 늘 똑같다. 시작 번호에서 출발해 간격만큼 건너뛰며 글자를 줍고, 끝 번호에 닿기 직전에 멈춘다. 위 그림을 옆에 두고 하나씩 따라가 보면 이렇다.

| 식 | 이렇게 읽는다 | 결과 |
|---|---|---|
| `s[2:5]` | 2번 o에서 출발해 5번 a 앞에서 멈춘다. 2, 3, 4번 세 글자 | ogr |
| `s[:4]` | 시작을 비웠으니 0번부터. 4번 앞까지라 0~3번 네 글자 | prog |
| `s[-3:]` | -3번 i에서 출발하고, 끝을 비웠으니 마지막 글자까지 간다 | ing |
| `s[-5:-2]` | -5번 m에서 출발해 -2번 n 앞에서 멈춘다. -5, -4, -3번 | mmi |
| `s[::2]` | 처음부터 끝까지 2칸씩 건너뛴다. 0, 2, 4, 6, 8, 10번 | pormig |
| `s[::-1]` | 간격이 -1이라 마지막 글자에서 출발해 한 칸씩 거꾸로 간다 | gnimmargorp |
| `s[::-2]` | 마지막 글자(10번)에서 출발해 2칸씩 거꾸로. 10, 8, 6, 4, 2, 0번 | gimrop |
| `s[5:1:-1]` | 5번 a에서 거꾸로 출발해 1번 r 앞에서 멈춘다. 5, 4, 3, 2번 | argo |
| `s[3:100]` | 끝이 범위를 넘어도 오류가 나지 않고 마지막 글자에서 멈춘다 | gramming |

여기서 가장 헷갈리는 게 간격이 음수일 때다. "끝 앞에서 멈춘다"는 규칙은 그대로인데 방향이 거꾸로라서, `s[5:1:-1]`의 "1 앞"은 1보다 큰 쪽인 2가 된다. 거꾸로 갈 때는 그림에서 오른쪽에서 왼쪽으로 손가락을 옮긴다고 생각하면 덜 틀린다.

리스트도 똑같이 잘린다. 다른 점은 리스트는 바꿀 수 있어서 `a[1:3] = [99]`처럼 잘라 낸 자리에 다른 값을 넣을 수 있다는 것이다. 이때 넣는 값의 개수가 잘라 낸 개수와 달라도 된다.

### 예제
```python
s = "programming"
print(s[2:5], s[-3:], s[:4], s[::2])
print(s[::-1], s[::-2])
print(s[5:1:-1], s[-1], s[-5:-2])
a = [10, 20, 30, 40, 50]
print(a[1:4], a[:-1], a[-2:])
a[1:3] = [99]
print(a)
print(a[10:], s[3:100])
```
출력:
```
ogr ing prog pormig
gnimmargorp gimrop
argo g mmi
[20, 30, 40] [10, 20, 30, 40] [40, 50]
[10, 99, 40, 50]
[] gramming
```
`s[::2]`는 인덱스 0,2,4,6,8,10을 차례로 집으니 p,o,r,m,i,g가 된다. `a[1:3] = [99]`는 그 자리에 있던 20, 30 두 개를 빼고 99 하나를 넣는 것이라 리스트 길이가 4가 된다. `a[10:]`은 범위 밖이지만 오류가 아니라 빈 리스트가 나온다.

### 자주 틀리는 포인트
- 음수 인덱스는 -1이 마지막 글자다. 그래서 `s[-5:-2]`는 끝 -2가 빠지므로 -5, -4, -3 세 글자만 나온다.
- `s[::-2]`를 "앞에서부터 2칸씩 뒤집기"로 많이들 착각하는데, step이 음수라 출발점은 마지막 글자다.
- 리스트를 슬라이스하면 새 리스트가 만들어진다. `a[:]`를 얕은 복사로 쓰는 것도 이 때문이다. 반면 `a[0]`처럼 인덱스 하나로 꺼내면 원소 자체가 나온다.
- `dict.items()`는 슬라이싱이 안 되니 `list(d.items())[1:]`처럼 리스트로 바꾼 뒤 자른다. 대신 반복은 바로 된다. 26-2가 그런 문제였는데, `for key, value in locations.items()`로 돌면서 key의 마지막 글자와 value의 첫 글자를 이어 붙이면 **CNNLRPYT**다. 딕셔너리는 3.7부터 넣은 순서를 지키기 때문에 넣은 순서대로 이어 붙이면 된다.
- 문자열은 불변이라 `s[0] = 'x'`처럼 한 글자를 바꾸려 하면 오류가 난다. 리스트는 된다.

## 문자열 메서드 (split·join·strip·replace·find·포맷)
출제: 26-1, 24-2, 23-3, 22-2
함정: `split()`은 인자가 없으면 공백(여러 개 포함)으로 나누고 양끝 공백도 버린다. `split(",")`은 쉼표 하나마다 나눈다. `split(" ")`는 공백 하나마다 나눠 빈 문자열이 생긴다(`" a  b ".split(" ")` → `['', 'a', '', 'b', '']`). `join`은 구분자가 앞에 온다.

### 개념
`t = "Hello, World"`

| 메서드 | 결과 | 비고 |
|---|---|---|
| `t.split(",")` | `['Hello', ' World']` | 공백이 남는다 |
| `t.split()` | `['Hello,', 'World']` | 공백 기준, 쉼표는 붙어 있다 |
| `"-".join(["a","b","c"])` | a-b-c | 리스트 원소는 문자열이어야 함 |
| `" x ".strip()` | x | 양쪽 공백 제거 |
| `t.replace("l", "L", 2)` | HeLLo, World | 앞에서 2개만 |
| `t.find("o")`, `t.find("z")` | 4, -1 | 없으면 -1. `index`는 없으면 오류 |
| `t.count("l")` | 3 | |
| `t.upper()`, `t.lower()` | | 원본은 안 바뀜 |
| `t.startswith("He")`, `"12".isdigit()`, `"ab".isalpha()` | True | |
| `"%d:%s:%.1f" % (3, "x", 3.14159)` | 3:x:3.1 | C 서식 |
| `"{}-{:>3}".format("a", 7)` | `a-  7` | `>3`은 오른쪽 정렬 3칸 |
| `f"{10 // 3:02d}"` | 03 | f-string |
| `"Hello".center(9, "*")` | `**Hello**` | |

### 예제
```python
s = " Hello, World "
t = s.strip()
print(t.upper(), t.lower(), len(s), len(t))
print(t.split(","), t.split(), t.replace("l", "L", 2))
print(t.find("o"), t.find("z"), t.count("l"), t.index("W"))
print("-".join(["a", "b", "c"]), "".join(reversed("abc")))
print(t.startswith("He"), t[7:].isalpha(), "12".isdigit())
print("%d:%s:%.1f" % (3, "x", 3.14159), "{}-{:>3}".format("a", 7), f"{10 // 3:02d}")
w = "the cat the dog".split()
print(w.count("the"), " ".join(w[::-1]))
print("Hello".center(9, "*"))
```
출력:
```
HELLO, WORLD hello, world 14 12
['Hello', ' World'] ['Hello,', 'World'] HeLLo, World
4 -1 3 7
a-b-c cba
True True True
3:x:3.1 a-  7 03
2 dog the cat the
**Hello**
```
strip 전 길이는 14, 후는 12다. 둘째 줄에서 보듯 리스트를 print하면 그 안의 문자열 원소에 따옴표가 붙는다. 단어 순서를 뒤집을 때는 split으로 나누고, `[::-1]`로 뒤집고, join으로 다시 붙이는 세 단계를 거친다.

### 자주 틀리는 포인트
- `split(",")`로 나눈 결과 `' World'`에는 앞 공백이 그대로 남아 있다. 쉼표만 기준으로 잘랐기 때문이다. 출력을 적을 때 이 공백을 빠뜨리면 틀린다.
- `join`을 부르는 쪽은 구분자 문자열이다. 그래서 `"-".join(리스트)`로 쓰지, `리스트.join("-")`으로 쓰지 않는다.
- 24-2는 `count`를 쓰지 않고 `for i in range(len(x))`로 돌면서 `x[i:i+len(y)]`를 잘라, y와 같으면 1을 더하는 방식이었다. 이렇게 세면 겹치는 위치도 센다. 예를 들어 "aaa"에서 "aa"를 찾으면 2가 나온다. 반면 `count`는 겹치지 않게 세기 때문에 `"aaa".count("aa")`는 1이다. 끝 근처에서 잘린 조각은 길이가 짧아져 y와 같지 않을 뿐, 오류는 나지 않는다. 위에서 본 것처럼 슬라이싱은 범위를 넘어도 오류가 없기 때문이다. 결국 24-2는 "ab"가 0·4·7번, "ca"가 3·6·9번에서 맞아 둘 다 3이다.
- 따옴표 문자열 두 개를 쉼표 없이 나란히 쓰면 하나로 이어 붙는다. 24-2의 `print(f'ab{3}' f'ca{3}')`가 이 경우라 공백 없이 ab3ca3이 찍힌다. 쉼표로 나눠 넘기면 print가 사이에 공백을 넣으니, 쉼표가 있는지부터 본다.
- `replace`, `upper` 같은 메서드는 새 문자열을 돌려 줄 뿐 원본은 그대로 둔다. 그러니 결과를 변수에 다시 넣었는지 확인한다.
- 23-3처럼 split 자리가 빈칸으로 나오면, 공백으로 나눌 때는 `split()`, 특정 문자로 나눌 때는 `split(문자)`를 쓴다.
- `for c in 문자열`에서 c는 한 글자다. 그래서 `c not in "ong"`은 c가 o, n, g 가운데 어느 것도 아닌지를 보는 것이지, 부분 문자열 "ong"을 통째로 지우는 것이 아니다. 26-1이 바로 이 경우다. HumanDev를 `split()`하고 단어마다 `[::-1]`로 뒤집으면 veDnamuH가 되고, 여기서 o·n·g 글자를 하나씩 거르면 n만 빠져 **veDamuH**가 된다. 참고로 `input("입력 = ")`의 안내 문구는 출력 답에 넣지 않는다.

## 딕셔너리 (items·keys·values·get)
출제: 26-2, 25-3, 25-2
함정: `d.get(k)`는 키가 없으면 None(또는 기본값)이고, `d[k]`는 없으면 오류다. 빈도 세기 관용구는 `d[k] = d.get(k, 0) + 1`.

### 개념
| 식 | 뜻 |
|---|---|
| `d["c"] = 3` | 추가 또는 덮어쓰기 |
| `d["a"] += 10` | 기존 값 갱신 |
| `d.get("z")`, `d.get("z", 0)` | None, 0 |
| `"b" in d` | 키 존재 여부(값이 아니라 키) |
| `d.keys()`, `d.values()`, `d.items()` | 키, 값, (키, 값) 튜플. 출력하려면 `list()`로 감싼다 |
| `for k, v in d.items()` | 키와 값 동시 순회 |
| `del d["b"]`, `d.pop("c")` | 삭제. pop은 값을 돌려 준다 |
| 순서 | 넣은 순서가 유지된다 |
| 출력 모양 | `{'a': 11, 'b': 2}` 키에 따옴표, 콜론 뒤 공백 |

### 예제
```python
d = {"a": 1, "b": 2}
d["c"] = 3
d["a"] += 10
print(d, len(d))
print(d.get("z"), d.get("z", 0), "b" in d)
print(list(d.keys()), list(d.values()))
print(list(d.items())[1:])
for k, v in d.items():
    if v % 2 == 0:
        print(k, end=" ")
print()
del d["b"]
print(d.pop("c"), d)
cnt = {}
for ch in "banana":
    cnt[ch] = cnt.get(ch, 0) + 1
print(cnt)
```
출력:
```
{'a': 11, 'b': 2, 'c': 3} 3
None 0 True
['a', 'b', 'c'] [11, 2, 3]
[('b', 2), ('c', 3)]
b 
3 {'a': 11}
{'b': 1, 'a': 3, 'n': 2}
```
items()는 그대로는 못 자르지만, 리스트로 바꾸면 튜플의 리스트가 되므로 `[1:]`로 자를 수 있다. for 문에서 값이 짝수인 키는 b뿐이다. 여기서 헷갈리기 쉬운 게 `print(d.pop("c"), d)`다. 인자는 왼쪽부터 평가하므로 pop이 먼저 실행되고, 그 뒤의 d가 찍힌다. 마지막 빈도 딕셔너리는 글자가 처음 나온 순서인 b, a, n 순으로 쌓인다.

### 자주 틀리는 포인트
- `for k in d`는 키만 돈다. 값까지 쓰려면 `d[k]`로 꺼내거나 `items()`로 돌아야 한다.
- `print(a.pop(), a)`처럼 한 print 안에서 값을 바꾸고 출력도 같이 하면, 왼쪽부터 순서대로 평가된 결과가 찍힌다. 위 예제의 pop과 같은 원리다.
- 딕셔너리 컴프리헨션 `{k: v for ...}`은 컴프리헨션 주제에서 다룬다.
- 25-3처럼 `enumerate`로 인덱스를 키로 쓰는 문제는 `enumerate(리스트, 시작값)`에서 시작값부터 확인한다. 안 주면 기본이 0이다.

## 리스트 메서드 (append·extend·insert·pop·remove·sort·reverse)
출제: 24-3, 22-1, 20-4
함정: `append([7, 8])`은 리스트 하나를 원소로 넣고, `extend([7, 8])`은 원소를 풀어 넣는다. `sort()`는 원본을 바꾸고 None을 돌려 주며, `sorted()`는 새 리스트를 돌려 준다.

### 개념
| 메서드 | 동작 | 반환 |
|---|---|---|
| `append(x)` | 끝에 x 추가 | None |
| `extend(iter)` | 원소들을 끝에 이어 붙임 | None |
| `insert(i, x)` | i 자리에 삽입 | None |
| `pop()`, `pop(0)` | 마지막 / i번째를 빼서 돌려 줌 | 그 값 |
| `remove(x)` | 값 x를 처음 하나만 삭제 | None |
| `index(x)`, `count(x)` | 위치 / 개수 | int |
| `sort()`, `sort(reverse=True)` | 원본 정렬 | None |
| `reverse()` | 원본 뒤집기 | None |
| `sorted(a)`, `a[::-1]` | 새 리스트 | 리스트 |
| `a[0], a[-1] = a[-1], a[0]` | 교환 | |
| `[[0] * 2 for _ in range(2)]` | 독립된 행을 가진 2차원 | |

메서드 이름 자체를 묻기도 한다. 22-1에는 extend·pop·reverse의 설명을 주고 이름을 쓰는 단답이 나왔다.

### 예제
```python
a = [3, 1, 2]
a.append(5)
a.extend([7, 8])
a.insert(1, 9)
print(a)
print(a.pop(), a.pop(0), a)
a.remove(2)
a.sort()
print(a, a.index(7), a.count(9))
a.reverse()
print(a)
b = sorted(a)
print(a, b)
a[0], a[-1] = a[-1], a[0]
print(a)
m = [[0] * 2 for _ in range(2)]
m[0][0] = 1
print(m)
```
출력:
```
[3, 9, 1, 2, 5, 7, 8]
8 3 [9, 1, 2, 5, 7]
[1, 5, 7, 9] 2 1
[9, 7, 5, 1]
[9, 7, 5, 1] [1, 5, 7, 9]
[1, 7, 5, 9]
[[1, 0], [0, 0]]
```
둘째 줄은 pop()이 8을, pop(0)이 3을 먼저 빼고 나서 a가 찍힌다. 인자를 왼쪽부터 평가하기 때문이다. 그다음 remove(2) 뒤 정렬하면 [1, 5, 7, 9]가 된다. sorted는 새 리스트를 돌려 줄 뿐 a를 바꾸지 않는다. 마지막 교환은 양끝 원소를 맞바꾼 것이다.

### 자주 틀리는 포인트
- `b = a.sort()`라고 쓰면 b는 None이다. sort는 원본을 바꾸고 None을 돌려 주기 때문이다. `print(a.sort())`도 같은 이유로 None이다.
- 많이들 헷갈리는 게 `remove`와 `pop`인데, `remove`는 값을 받고 `pop`은 인덱스를 받는다. `del a[i]`도 인덱스다.
- `a.index(x)`는 찾는 값이 없으면 오류가 나고, `count`는 없으면 0이다.
- `[[0] * 2] * 2`는 같은 행을 두 번 참조하는 것이라, 한 칸만 바꿔도 두 행이 같이 바뀐다. 자세한 건 얕은 복사 주제에서 본다.

## 클래스 (__init__·self·클래스 변수·상속)
출제: 25-1, 21-1
함정: `self.v`는 객체마다 따로, 클래스 바로 아래 `count = 0`은 모든 객체가 공유한다. `Node.count += 1`은 공유 변수를 올린다.

### 개념
| 항목 | 내용 |
|---|---|
| `__init__(self, ...)` | 생성자. `Node(1)`을 부르면 자동 실행 |
| `self` | 그 객체 자신. 메서드 첫 매개변수 |
| 인스턴스 변수 | `self.v = v` 객체마다 별개 |
| 클래스 변수 | 클래스 몸체의 `count = 0`. `Node.count`로도, `root.count`로도 읽힌다 |
| 상속 | `class Leaf(Node)`. `super().__init__(v)`로 부모 생성자 호출 |
| 오버라이딩 | 자식이 같은 이름 메서드를 정의하면 자식 것이 실행된다 |
| `__str__` | `print(obj)`나 `str(obj)`의 모양 |
| 기본 인자 `left=None` | 안 넘기면 None. `if self.left:`로 존재 확인 |

### 예제
```python
class Node:
    count = 0
    def __init__(self, v, left=None, right=None):
        self.v = v
        self.left = left
        self.right = right
        Node.count += 1
    def total(self):
        s = self.v
        if self.left:
            s += self.left.total()
        if self.right:
            s += self.right.total()
        return s
    def __str__(self):
        return f"Node({self.v})"
class Leaf(Node):
    def __init__(self, v):
        super().__init__(v)
        self.tag = "leaf"
    def total(self):
        return self.v * 2
root = Node(1, Node(2, Leaf(3)), Leaf(4))
print(Node.count, root.total())
print(root.left, root.right.tag)
root.v = 10
print(root.total(), Node.count, root.count)
```
출력:
```
4 17
Node(2) leaf
26 4 4
```
객체가 4개 생성되니 count는 4다. Leaf는 total을 오버라이딩해서 자기 것이 실행되므로, total은 1 + (2 + 3×2) + 4×2 = 17이 된다. root.v를 10으로 바꾸면 26이다. `root.count`는 객체를 통해 읽었지만 결국 클래스 변수를 읽는 것이라 역시 4다.

### 자주 틀리는 포인트
- 여기서 많이들 헷갈리는 게 `self.count += 1`이다. 이렇게 쓰면 클래스 변수가 오르는 게 아니라, 그 객체에만 인스턴스 변수 count가 새로 생기고 클래스 변수는 그대로다. 그러니 `Node.count`인지 `self.count`인지부터 본다.
- 인자 평가 순서도 챙겨야 한다. 바깥 객체를 만들려면 인자부터 평가해야 하므로, `Node(1, Node(2, Leaf(3)), Leaf(4))`는 안쪽 Leaf(3) → Node(2) → Leaf(4) → Node(1) 순으로 생성된다. 생성자에서 출력하는 문제라면 이 순서대로 찍힌다.
- 21-1은 클래스 변수에 문자열을 두고 인덱싱하는 문제였다. 클래스 변수는 `Cls.name[0]`처럼 클래스 이름으로도, 객체로도 접근된다.
- 25-1은 리스트로 트리를 만들었다. `n[(i-1)//2].c.append(n[i])`는 i번 원소를 (i-1)//2번 원소의 자식으로 붙인다. 이걸 따라가 보면 인덱스 0이 루트, 1·2가 0의 자식, 3·4가 1의 자식, 5·6이 2의 자식이 되니 완전 이진 트리다. 데이터가 [3, 5, 8, 12, 15, 17, 21]이면 레벨 0은 3, 레벨 1은 5·8, 레벨 2는 12·15·17·21이다.
- 아마 25-1에서 가장 헷갈리는 부분이 조건 표현식일 것이다. `A if 조건 else B`는 `+`보다 우선순위가 낮고 오른쪽부터 묶인다. 그래서 `n.v if level % 2 else 0 + sum(...) if n else 0`은 `n.v if level % 2 else ((0 + sum(...)) if n else 0)`으로 읽어야 한다. 풀어 보면 홀수 레벨은 자기 값을, 짝수 레벨은 자식들의 calc 합을 돌려준다. 루트는 레벨 0이니 레벨 1 자식 5와 8의 값을 더해 **13**이 되고, 레벨 1에서 재귀가 멈추므로 레벨 2 노드는 더해지지 않는다. `(n.v if level % 2 else 0) + sum(...)`로 잘못 묶어도 이 데이터에서는 우연히 13으로 같지만, 다른 데이터에서는 답이 갈린다.

## 집합 (add·remove·update·& | - ^)
출제: 25-2, 20-2
함정: 중복이 자동으로 사라지고 순서가 없다. `add(2)`를 이미 있는 원소에 해도 변화가 없다.

### 개념
| 식 | 뜻 |
|---|---|
| `a.add(x)` | 원소 하나 추가 |
| `a.update([6, 7])` | 여러 개 추가 |
| `a.remove(x)` | 삭제. 없으면 오류 |
| `a.discard(x)` | 삭제. 없어도 조용히 넘어감 |
| `a & b`, `a \| b`, `a - b`, `a ^ b` | 교집합, 합집합, 차집합, 대칭 차집합 |
| `set("hello")` | 문자 집합 {h, e, l, o}. 순서는 정해지지 않으므로 보통 `sorted`로 출력 |
| `len(set(리스트))` | 중복 제거 후 개수 |
| `{1, 2} <= a` | 부분집합 |
| 출력 모양 | `{1, 2, 3}`. 빈 집합은 `set()` (`{}`는 빈 딕셔너리) |

### 예제
```python
a = {1, 2, 3}
b = {3, 4}
a.add(2)
a.add(5)
a.update([6, 7])
a.remove(7)
a.discard(100)
print(a, len(a))
print(a & b, a | b, a - b, a ^ b)
print(sorted(set("hello")), len(set([1, 1, 2])))
print(sorted(set([3, 1, 2, 3])))
print(3 in b, {1, 2} <= a)
```
출력:
```
{1, 2, 3, 5, 6} 5
{3} {1, 2, 3, 4, 5, 6} {1, 2, 5, 6} {1, 2, 4, 5, 6}
['e', 'h', 'l', 'o'] 2
[1, 2, 3]
True True
```
add(2)는 2가 이미 있어서 무시되고, 7은 넣었다가 다시 뺐다. 대칭 차집합은 양쪽 중 한쪽에만 있는 원소를 모은 것이다.

### 자주 틀리는 포인트
- 0~7처럼 작은 정수로 된 집합은 출력이 오름차순으로 보이는데, 그렇다고 보장되는 것은 아니다. 예를 들어 `{10, 2, 33, 4}`를 찍으면 `{33, 10, 2, 4}`가 나온다. 문자열 집합은 순서가 실행마다 다를 수 있다. 20-2(23-1 재출제)는 `sorted` 없이 문자열 집합을 그대로 출력했고, 정답표는 남은 원소를 `{'…'}` 형식으로 적었다. 순서는 정해져 있지 않으니, 원소가 빠지거나 겹치지 않았는지만 확인하면 된다.
- `a - b`와 `b - a`는 다르다. 차집합은 왼쪽에만 있는 원소를 남기기 때문이다.
- 집합 컴프리헨션 `{x % 3 for x in range(10)}`은 중복이 제거되니 {0, 1, 2}만 남는다.
- 25-2처럼 `s = set(dst.values())`로 만든 집합은 그 순간의 값을 복사해 온 새 집합이다. 그래서 뒤에서 `dst[2] = 7`로 딕셔너리를 바꿔도 s는 {2, 4, 6} 그대로다. 마찬가지로 컴프리헨션이 끝난 뒤 `lst[0] = 99`로 원본 리스트를 바꿔도 dst의 키·값은 바뀌지 않는다. 정리하면 s({2, 4, 6, 99}) & set(dst.values())({2, 7, 6})는 {2, 6}이고, len은 **2**다.

## 연산자 (==와 is, 시프트, //, %, **, 비교 연쇄)
출제: 21-3, 21-2
함정: `==`는 값 비교, `is`는 같은 객체인지 비교다. 리스트 두 개를 따로 만들면 `==`는 True, `is`는 False.

### 개념
| 식 | 결과 | 비고 |
|---|---|---|
| `[1,2] == [1,2]` / `is` | True / False | 서로 다른 객체 |
| `c = a; a is c` | True | 같은 객체 |
| `3 << 2`, `20 >> 2` | 12, 5 | ×4, ÷4 |
| `6 & 3`, `6 \| 3`, `6 ^ 3`, `~5` | 2, 7, 5, -6 | |
| `2 ** 10` | 1024 | 거듭제곱 |
| `7 // 2`, `-7 // 2`, `7.0 // 2` | 3, -4, 3.0 | 바닥 나눗셈. 실수가 끼면 실수 |
| `-7 % 3` | 2 | 결과 부호는 제수를 따름 |
| `not 0`, `1 and 2`, `0 or "x"` | True, 2, x | and/or는 피연산자를 돌려 줌 |
| `3 > 2 > 1` | True | 비교 연쇄 = `3 > 2 and 2 > 1` |
| `1 < 2 == True` | False | `1 < 2 and 2 == True` → 2 == True는 False |
| `x //= 2`, `x **= 3` | | 복합 대입 |
| `"a" * 3`, `[0] * 2 + [1]` | aaa, [0, 0, 1] | 반복과 연결 |
| `"b" < "ab"`, `(1, 2) < (1, 3)` | False, True | 사전순, 앞 원소부터 |
| `1 == 1.0`, `"1" == 1` | True, False | int·float·bool은 값이 같으면 같음, 문자열과 숫자는 다름 |

### 예제
```python
a = [1, 2]
b = [1, 2]
c = a
print(a == b, a is b, a is c)
print(3 << 2, 20 >> 2, 6 & 3, 6 | 3, 6 ^ 3, ~5)
print(2 ** 10, 7 // 2, -7 // 2, 7 % 3, -7 % 3, 7.0 // 2)
print(not 0, 1 and 2, 0 or "x", [] or {}, 3 > 2 > 1)
x = 5
x //= 2
x **= 3
print(x, 10 / 2, type(10 / 2))
print("a" * 3, [0] * 2 + [1], "b" < "ab", (1, 2) < (1, 3))
print(1 < 2 == True, 1 == 1.0, "1" == 1)
```
출력:
```
True False True
12 5 2 7 5 -6
1024 3 -4 1 2 3.0
True 2 x {} True
8 5.0 <class 'float'>
aaa [0, 0, 1] False True
False True False
```
x는 5에서 시작해 `5 // 2 = 2`가 되고, 다시 `2 ** 3 = 8`이 된다. `[] or {}`는 둘 다 거짓이라 마지막 값인 {}가 나온다. True/False가 아니라 {}가 그대로 나오는 건 and/or가 피연산자를 돌려 주기 때문이다. `10 / 2`는 항상 float라서 5.0이다.

### 자주 틀리는 포인트
- `print(10 / 2)`는 `5.0`이다. 정수로 받고 싶으면 `//`를 쓴다.
- `a == b`가 True여도 `a is b`는 False일 수 있다. `==`는 값을 보고 `is`는 같은 객체인지를 보기 때문이다. 21-3의 `==` 비교 문제는 값 비교로 풀면 된다.
- `~5`는 `-(5+1)`이라 -6이다. 시프트는 [코딩 공통]의 비트 연산 주제에서 다룬다.
- 비교 연쇄는 수학처럼 읽으면 되는데, 가운데 값이 양쪽 비교에 모두 쓰인다는 점을 잊으면 안 된다. 표의 `1 < 2 == True`가 False인 것도 가운데 2가 True와 한 번 더 비교되기 때문이다.

## 컴프리헨션 (리스트·딕셔너리·집합)
출제: 25-2
함정: `[식 for x in ... if 조건]`에서 if는 필터이고, `[A if 조건 else B for x in ...]`에서 if-else는 값 선택이다. 위치가 다르다.

### 개념
| 모양 | 결과 |
|---|---|
| `[x * x for x in range(6) if x % 2 == 1]` | [1, 9, 25] 홀수만 제곱 |
| `{k: len(k) for k in ["aa", "b", "ccc"]}` | {'aa': 2, 'b': 1, 'ccc': 3} 딕셔너리 |
| `{x % 3 for x in range(10)}` | {0, 1, 2} 집합(중복 제거) |
| `[(i, j) for i in range(1, 3) for j in range(2)]` | 이중 for. 앞의 for가 바깥 |
| `[[i * j for j in range(1, 4)] for i in range(1, 3)]` | 2차원. 안쪽 대괄호가 한 행 |
| `sum(x for x in ... if ...)` | 제너레이터. 리스트 없이 합 |
| `["even" if x % 2 == 0 else "odd" for x in range(3)]` | 조건 값 선택 |

### 예제
```python
sq = [x * x for x in range(6) if x % 2 == 1]
print(sq)
d = {k: len(k) for k in ["aa", "b", "ccc"]}
print(d)
s = {x % 3 for x in range(10)}
print(s)
pairs = [(i, j) for i in range(1, 3) for j in range(2)]
print(pairs)
m = [[i * j for j in range(1, 4)] for i in range(1, 3)]
print(m)
print(sum(x for x in range(1, 11) if x % 3 == 0))
print(["even" if x % 2 == 0 else "odd" for x in range(3)])
```
출력:
```
[1, 9, 25]
{'aa': 2, 'b': 1, 'ccc': 3}
{0, 1, 2}
[(1, 0), (1, 1), (2, 0), (2, 1)]
[[1, 2, 3], [2, 4, 6]]
18
['even', 'odd', 'even']
```
이중 for는 `for i: for j:`를 펼쳐 쓴 것과 같은 순서로 돈다. 앞의 for가 바깥이라서 i 하나에 대해 j를 다 돈 다음 i가 넘어간다. 2차원에서는 i가 행(1, 2)을, j가 열(1~3)을 맡는다. 3의 배수 합은 3+6+9 = 18이다.

### 자주 틀리는 포인트
- 결과 타입은 바깥 괄호가 정한다. `[]`면 리스트, `{k: v}`면 딕셔너리, `{x}`면 집합이다. 헷갈리기 쉬운 게 `()`인데, 튜플이 아니라 제너레이터다.
- 필터로 쓰는 if에는 else를 못 붙인다. 그러니 else가 보이면 앞쪽에 오는 값 선택 문법이다.
- 집합 컴프리헨션은 중복이 사라지면서 개수가 줄어든다. 그래서 `len`을 묻는 문제에서 틀리기 쉽다.

## lambda·map·filter·sorted(key)
출제: 22-3
함정: `map`과 `filter`의 결과는 바로 출력하면 객체 주소가 나온다. `list()`로 감싸야 값이 보인다.

### 개념
| 식 | 결과 |
|---|---|
| `f = lambda x, y=2: x ** y` | f(3) = 9, f(2, 3) = 8 |
| `list(map(lambda x: x * 2, nums))` | 각 원소 변환 |
| `list(filter(lambda x: x % 2 == 0, nums))` | 조건 참인 원소만 |
| `list(map(str, nums))`, `sum(map(int, "123"))` | 내장 함수도 넘길 수 있다 |
| `sorted(words, key=len)` | 길이순. `key=lambda w: len(w)`와 같음 |
| `sorted(..., reverse=True)` | 내림차순 |
| `max(nums, key=lambda x: -x)` | key가 최대인 원소 → 실제로는 최솟값 1 |
| `reduce(lambda a, b: a * b, nums)` | 누적. `from functools import reduce` |

### 예제
```python
f = lambda x, y=2: x ** y
print(f(3), f(2, 3))
nums = [1, 2, 3, 4, 5]
print(list(map(lambda x: x * 2, nums)))
print(list(filter(lambda x: x % 2 == 0, nums)))
print(list(map(str, nums)), sum(map(int, "123")))
words = ["bb", "a", "ccc"]
print(sorted(words, key=lambda w: len(w)), sorted(words, key=len, reverse=True))
print(max(nums, key=lambda x: -x))
from functools import reduce
print(reduce(lambda a, b: a * b, nums))
print(list(zip(nums, map(lambda x: x % 2, nums)))[:2])
```
출력:
```
9 8
[2, 4, 6, 8, 10]
[2, 4]
['1', '2', '3', '4', '5'] 6
['a', 'bb', 'ccc'] ['ccc', 'bb', 'a']
1
120
[(1, 1), (2, 0)]
```
`map(str, nums)`의 결과는 문자열 리스트라서 따옴표가 붙는다. `max(key=-x)`는 원소 자체가 아니라 -x가 가장 큰 원소를 고르는 것이라 1이 나온다. reduce는 1×2×3×4×5를 차례로 누적한다.

### 자주 틀리는 포인트
- `filter`와 `map`이 헷갈리기 쉬운데, `filter`는 원소를 바꾸지 않고 고르기만 하고 `map`은 전부 바꾼다.
- `sorted(key=...)`는 key 값으로 정렬만 할 뿐, 돌려 주는 것은 원래 원소다. 예제에서 길이로 정렬했는데 길이가 아니라 단어가 나온 것을 보면 된다.
- 람다도 `y=2` 같은 기본 인자가 있으면 일반 함수와 같은 규칙으로 생략할 수 있다.

## 얕은 복사와 참조
출제: 26-1
함정: `b = a`는 복사가 아니라 같은 리스트에 이름을 하나 더 붙인 것이다. `a[:]`·`list(a)`·`copy()`는 한 겹만 복사한다(안쪽 리스트는 공유).

### 개념
| 식 | 바깥 리스트 | 안쪽 리스트 |
|---|---|---|
| `b = a` | 공유 | 공유 |
| `c = a[:]`, `list(a)`, `a.copy()` | 새로 만듦 | 공유 (얕은 복사) |
| `copy.deepcopy(a)` | 새로 만듦 | 새로 만듦 |
| `[[0] * 2] * 2` | | 같은 행을 2번 참조. 한 칸 바꾸면 두 행이 바뀜 |
| `[[0] * 2 for _ in range(2)]` | | 행마다 독립 |

함수 인자로 리스트를 넘길 때도 따져 봐야 한다. `lst.append(n)`은 호출자의 리스트를 직접 바꾸지만, `lst = lst + [n]`은 새 리스트를 지역 변수에 넣는 것이라 호출자에게는 안 보인다. 헷갈리는 건 `lst += [n]`인데, 이건 제자리 변경이라 호출자에게 보인다.

### 예제
```python
import copy
a = [1, 2, [3, 4]]
b = a
c = a[:]
d = list(a)
e = copy.deepcopy(a)
a[0] = 9
a[2].append(5)
print(a)
print(b, c is a, c)
print(d, e)
x = [0] * 3
y = [[0] * 2] * 2
y[0][0] = 7
print(x, y)
def add(lst, n):
    lst.append(n)
    lst = lst + [n]
    return lst
p = [1]
q = add(p, 2)
print(p, q)
```
출력:
```
[9, 2, [3, 4, 5]]
[9, 2, [3, 4, 5]] False [1, 2, [3, 4, 5]]
[1, 2, [3, 4, 5]] [1, 2, [3, 4]]
[0, 0, 0] [[7, 0], [7, 0]]
[1, 2] [1, 2, 2]
```
b는 a와 같은 객체이니 a의 변화를 전부 따라간다. c, d는 바깥 리스트가 따로라서 a[0]=9는 안 보이지만, 안쪽 [3, 4]는 공유하고 있어서 5가 붙은 것은 보인다. 완전히 독립인 건 e 하나뿐이다. y는 같은 행 두 개를 담은 것이라 둘 다 7이 됐다. add에서는 append가 p에 반영되고, `lst + [n]`은 새 리스트라서 q만 다르다.

### 자주 틀리는 포인트
- 문제를 풀 때는 "안쪽 리스트가 바뀌었는가, 바깥 원소가 바뀌었는가"를 나눠서 본다. 얕은 복사가 보호해 주는 건 바깥뿐이기 때문이다.
- `[0] * 3`은 원소가 정수라 공유 문제가 없다. 문제가 되는 건 리스트의 리스트일 때뿐이다.
- 함수 안에서 `lst = ...`로 재대입하고 나면, 그 뒤의 변경은 호출자와 무관하다. 반면 `lst.append`, `lst[i] = `, `lst += `는 호출자에게 반영된다.

## range·enumerate·zip
출제: 25-3
함정: `range(a, b)`는 b 미포함이고, `enumerate(x, 1)`의 두 번째 인자는 시작 번호다. `zip`은 짧은 쪽 길이에 맞춘다.

### 개념
| 식 | 결과 |
|---|---|
| `range(5)` | 0 1 2 3 4 |
| `range(2, 10, 3)` | 2 5 8 |
| `range(5, 0, -2)` | 5 3 1 |
| `enumerate("abc", 1)` | (1,'a') (2,'b') (3,'c') |
| `zip(["kim","lee","park"], [90, 80])` | ('kim', 90), ('lee', 80). park는 짝이 없어 버려짐 |
| `dict(zip(keys, values))` | 딕셔너리 만들기 |
| `sum(range(1, 11))` | 55 |
| `print(x, end=" ")` | 줄바꿈 대신 공백 |

range·zip·enumerate·map의 결과를 그대로 print하면 값이 아니라 `range(0, 5)`나 객체 주소가 나온다. 그래서 값을 보려면 `list()`로 감싸거나 for로 돌아야 한다.

### 예제
```python
print(list(range(5)), list(range(2, 10, 3)), list(range(5, 0, -2)))
for i, ch in enumerate("abc", 1):
    print(i, ch, end=" ")
print()
names = ["kim", "lee", "park"]
scores = [90, 80]
print(list(zip(names, scores)))
print(dict(zip(names, scores)))
d = {}
for i, n in enumerate(names):
    d[n] = i
print(d, sum(range(1, 11)))
for a, b in zip(range(3), range(10, 13)):
    print(a + b, end=",")
print()
```
출력:
```
[0, 1, 2, 3, 4] [2, 5, 8] [5, 3, 1]
1 a 2 b 3 c 
[('kim', 90), ('lee', 80)]
{'kim': 90, 'lee': 80}
{'kim': 0, 'lee': 1, 'park': 2} 55
10,12,14,
```
enumerate의 시작값을 1로 줬으니 번호가 1부터 붙는다. zip은 짧은 쪽 길이 2에 맞추므로 park가 빠진다. 마지막 줄은 `end=","`로 찍었기 때문에 맨 끝에도 쉼표가 붙는다.

### 자주 틀리는 포인트
- `range(5, 0, -2)`는 끝값 0을 포함하지 않으니 5, 3, 1에서 끝난다.
- `end=" "`로 찍은 줄은 마지막에도 공백이 남는다. 출력 답안에서는 눈에 보이지 않지만, 그 줄의 줄바꿈은 그다음 `print()`가 만든다는 점은 알아 둔다.
- enumerate의 시작값을 안 주면 0부터 센다(25-3 dict 키).

## 함수 기본 인자·가변 인자·nonlocal
출제: 22-1
함정: 기본 인자가 리스트(`acc=[]`)면 호출 사이에 같은 리스트가 재사용된다. `grow(1)` 뒤 `grow(2)`는 [1, 2]다.

### 개념
| 모양 | 뜻 |
|---|---|
| `def greet(name, msg="hi", times=1)` | 뒤 두 개는 생략 가능. `greet("lee", times=2)`처럼 이름으로 지정 |
| `*args` | 위치 인자를 튜플로. `sum(args)` |
| `**kwargs` | 이름 인자를 딕셔너리로. `kwargs.get("k")` |
| `acc=[]` 기본 인자 | 함수 정의 때 한 번만 만들어진다. 변경이 누적됨 |
| `b=None` 뒤 `b = [] if b is None else b` | 매번 새 리스트를 만드는 안전한 방법 |
| `nonlocal n` | 안쪽 함수에서 바깥 함수의 변수 변경 |
| `global g` | 함수 안에서 전역 변수 변경 |

### 예제
```python
def greet(name, msg="hi", times=1):
    return (msg + " " + name + " ") * times
print(greet("kim"))
print(greet("lee", times=2))
print(greet("park", "bye"))
def total(*args, **kwargs):
    return sum(args), len(kwargs), kwargs.get("k")
print(total(1, 2, 3, k=9, j=0))
print(total())
def grow(x, acc=[]):
    acc.append(x)
    return acc
print(grow(1), grow(2))
def f(a, b=None):
    b = [] if b is None else b
    b.append(a)
    return b
print(f(1), f(2))
def outer():
    n = 0
    def inner():
        nonlocal n
        n += 1
        return n
    inner()
    return inner()
print(outer())
```
출력:
```
hi kim 
hi lee hi lee 
bye park 
(6, 2, 9)
(0, 0, None)
[1, 2] [1, 2]
[1] [2]
2
```
`print(grow(1), grow(2))`는 두 호출이 같은 리스트를 돌려 주므로 둘 다 [1, 2]로 찍힌다. 반면 None 기본값 방식은 호출할 때마다 새 리스트를 만들어서 [1], [2]로 독립이다. outer는 inner를 두 번 부르니 2가 된다.

### 자주 틀리는 포인트
- `print('a=', num1, 'b=', num2)`처럼 쉼표로 나열하면 인자 사이에 공백이 하나씩 들어간다. sep 기본값이 `' '`이기 때문이다. 22-1이 이 경우로, `func(20)`이 기본값 num2=2를 써서 `a= 20 b= 2`가 나오고, 답에는 `=` 뒤 공백까지 그대로 쓴다. 공백을 없애려면 `sep=''`를 준다.
- 함수가 여러 값을 return하면 튜플이 된다. 그래서 `(6, 2, 9)`처럼 괄호가 붙어 출력된다.
- 위치 매개변수에서는 기본 인자 뒤에 기본값 없는 인자를 둘 수 없다. 단, `*` 뒤에 오는 키워드 전용 인자는 예외다. 호출할 때도 순서가 있어서, 위치 인자가 이름 인자보다 앞에 온다.
- 함수 안에서 `n += 1`만 쓰면 지역 변수로 취급되어 오류가 난다. 그러니 `global`·`nonlocal` 선언이 있는지부터 본다.

## type()과 자료형 변환
출제: 24-3
함정: `type(x) == int`는 정확히 int일 때만 True이고, `isinstance(True, int)`는 True다(bool은 int의 자식). `/`의 결과는 (정수·실수끼리면) 항상 float.

### 개념
| 식 | 결과 |
|---|---|
| `type(1)`, `type(2.0)`, `type("3")` | int, float, str |
| `type([4])`, `type((5,))`, `type({6})`, `type({7: 8})` | list, tuple, set, dict |
| `type(None)` | NoneType |
| `type(1 / 1)`, `type(1 // 1)`, `type(2 ** -1)` | float, int, float |
| `int("12")`, `str(12)`, `int(3.9)`, `float("2.5")` | 12, "12", 3, 2.5 |
| `True + True`, `bool("False")` | 2, True (빈 문자열이 아니면 참) |
| `(1,)` vs `(1)` | 튜플 / 그냥 정수 1 |
| `t = (1, 2); t[0] = 9` | 오류(TypeError). 튜플은 순서가 있지만 변경 불가, 리스트는 변경 가능 |
| `10 ** 20` | 큰 정수도 그대로(오버플로 없음) |
| `0.1 + 0.2 == 0.3` | False (부동소수점) |

### 예제
```python
vals = [1, 2.0, "3", [4], (5,), {6}, {7: 8}, True, None]
print([type(v).__name__ for v in vals])
print(type(1 / 1), type(1 // 1), type(2 ** 3), type(2 ** -1))
print(int("12") + 3, str(12) + "3", int(3.9), float("2.5"))
print(True + True, isinstance(True, int), bool("False"))
def kind(x):
    if type(x) == int:
        return "int"
    elif isinstance(x, (list, tuple)):
        return "seq"
    return "other"
print(kind(3), kind([1]), kind((1,)), kind("s"))
print(10 ** 20, 2 ** 0.5 > 1.4, 0.1 + 0.2 == 0.3)
t = (1,)
print(t * 2, len(t), (1) == 1)
```
출력:
```
['int', 'float', 'str', 'list', 'tuple', 'set', 'dict', 'bool', 'NoneType']
<class 'float'> <class 'int'> <class 'int'> <class 'float'>
15 123 3 2.5
2 True True
int seq seq other
100000000000000000000 True False
(1, 1) 1 True
```
`type(x)`를 그대로 print하면 `<class 'int'>` 모양으로 나오고, `.__name__`을 붙이면 이름만 나온다. `int("12") + 3`은 숫자끼리 더해 15가 되고, `str(12) + "3"`은 문자열끼리 이어 붙여 "123"이 된다.

### 자주 틀리는 포인트
- 24-3 같은 type 분기 문제에서는 bool 입력을 조심한다. `type(x) == int`와 `isinstance(x, int)`는 bool이 들어오면 결과가 다르다. bool은 int의 자식이라서 그렇다.
- `int(3.9)`는 버림이라 3이지, 반올림해서 4가 되는 게 아니다. 4가 나오는 건 `round(3.9)`다.
- 원소 하나짜리 튜플은 `(1,)`처럼 쉼표가 필요하다. 쉼표가 없으면 그냥 정수 1이다.

## 입력 input()
출제: 26-1
함정: `input()`은 항상 문자열이다. 숫자로 쓰려면 `int()`가 필요하고, 한 줄에 여러 값은 `split()`으로 나눈다.

### 개념
| 모양 | 뜻 |
|---|---|
| `n = int(input())` | 한 줄을 읽어 정수로 |
| `nums = list(map(int, input().split()))` | 공백으로 나뉜 여러 정수 |
| `name, age = input().split()` | 두 문자열로 분리 대입 |
| `a, b = map(int, input().split(","))` | 쉼표 구분 두 정수 |
| `input("prompt: ")` | 안내 문구 출력 후 입력 |

### 예제
입력으로 `3`, `10 20 30`, `kim 25`, `4,5` 네 줄을 준다고 해보자.
```python
n = int(input())
nums = list(map(int, input().split()))
name, age = input().split()
print(n, nums, sum(nums) // n)
print(name, int(age) + 1, type(age).__name__)
a, b = map(int, input().split(","))
print(a * b)
```
출력:
```
3 [10, 20, 30] 20
kim 26 str
20
```
age는 split으로 나온 값이라 숫자처럼 보여도 문자열 "25"다. 그래서 `int(age) + 1`로 바꿔 더해야 26이 되고, 변환하지 않은 age 자체의 타입은 str로 찍힌다.

### 자주 틀리는 포인트
- `input() + 1`은 str에 int를 더하는 것이라 오류다. `int(input()) + 1`처럼 먼저 바꿔야 한다.
- `input().split()`의 결과도 문자열 리스트다. 숫자로 비교하거나 계산하기 전에 `map(int, ...)`이 있는지 본다.
- 기출 코드에서 입력값은 문제 지문에 주어진다. 그러니 지문의 값을 변수 옆에 적어 두고 시작한다.
