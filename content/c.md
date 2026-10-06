# C
C는 Java와 함께 코딩 140문제의 대부분을 차지하는 언어로(색인 기준 두 언어가 각각 60문제 안팎), 2023년 이후 회차마다 2~7문제(대개 3~4문제)다. 포인터와 배열, 문자열 포인터, 구조체(특히 연결 리스트. 24-2부터 25-3까지 5회 연속), 재귀, static이 반복된다. 출력 결과 쓰기가 대부분이고 정렬·진법 변환·잔돈 계산 같은 짧은 알고리즘의 빈칸 채우기가 섞여 나온다. 공통 규칙(추적 표, 비트, 우선순위, 나눗셈, 형변환, 2차원 배열)은 [코딩 공통] 페이지에 있다.

## 문자열과 '\0', 문자열 포인터
출제: 25-3, 24-2, 24-1, 23-3, 23-2, 23-1, 22-2, 20-4
함정: 문자열 길이와 배열 크기는 다르다. `"hello"`는 길이 5, 저장에는 6바이트('\0' 포함)가 든다.

### 개념
| 규칙 | 내용 |
|---|---|
| 끝 표시 | 문자열은 `'\0'`(값 0)으로 끝난다. `while (*s)`, `while (s[i] != '\0')`은 끝까지 도는 관용구 |
| `%s` 출력 | 그 주소부터 '\0' 직전까지 찍는다. `p + 1`을 넘기면 두 번째 문자부터 |
| 중간에 '\0' 넣기 | `s[2] = '\0'`이면 그 뒤가 있어도 문자열은 거기서 끝난다 |
| `sizeof` vs `strlen` | `char s[10] = "hello"`에서 sizeof는 10, strlen은 5 |
| `*s++` | 현재 문자를 쓰고 포인터를 다음으로. `*s++ + 1`은 문자 값 + 1 |
| 리터럴 포인터 | `char *q = "abc"`는 읽기 전용. 내용을 바꾸는 코드는 `char s[] = "abc"` 쪽 |
| `<string.h>` 함수 | `strlen(s)` 길이('\0' 제외), `strcpy(d, s)` 복사, `strcat(d, s)` d 뒤에 이어 붙이기, `strcmp(a, b)` 같으면 0, a가 사전순으로 앞이면 음수, 뒤면 양수 |

### 예제
```c
#include <stdio.h>
int len(char *s) {
    int n = 0;
    while (*s++) n++;
    return n;
}
int main() {
    char s[10] = "hello";
    char *p = s;
    printf("%d %d\n", len(s), (int)sizeof(s));
    printf("%s %s\n", p + 1, &s[3]);
    printf("%c %c\n", *p, *(p + 4));
    s[2] = '\0';
    printf("%s %d\n", s, len(s));
    char *q = "abc";
    while (*q) printf("%c", *q++ + 1);
    printf("\n");
    return 0;
}
```
출력:
```
5 10
ello lo
h o
he 2
bcd
```
`while (*s++)`는 '\0'을 만나면 거짓이 되어 멈추고, 그 전까지 n이 5번 증가한다. `p + 1`은 'e'의 주소라 "ello", `&s[3]`은 "lo". s[2]에 '\0'을 넣으면 "he"가 된다. `*q++ + 1`은 각 문자의 다음 문자다.

```c
#include <stdio.h>
void copy(char *d, char *s) {
    while ((*d++ = *s++) != '\0');
}
void rev(char *s) {
    char *e = s, t;
    while (*e) e++;
    e--;
    while (s < e) { t = *s; *s = *e; *e = t; s++; e--; }
}
int main() {
    char a[20], b[] = "level", c[] = "stack";
    copy(a, c);
    rev(a);
    printf("%s %s\n", a, c);
    rev(b);
    printf("%s\n", b);
    int i, cnt = 0;
    for (i = 0; c[i] != '\0'; i++)
        if (c[i] == 'a' || c[i] == 'c') cnt++;
    printf("%d\n", cnt);
    return 0;
}
```
출력:
```
kcats stack
level
2
```
`copy`는 '\0'까지 복사한 뒤 멈춘다(대입식의 값이 0이 되는 순간). `rev`는 e를 끝('\0')까지 보낸 뒤 한 칸 물러서 마지막 문자를 가리키게 하고, 양끝을 교환하며 가운데로 모은다. 원본 c는 복사본만 뒤집었으므로 그대로다.

### 자주 틀리는 포인트
- `while (*s++)`에서 n은 '\0'을 세지 않는다. 길이 함수의 반환값은 strlen과 같다.
- `printf("%s", s + 2)`처럼 포인터 산술로 시작점을 옮긴 출력이 자주 나온다. 몇 번째 문자부터인지 0부터 센다.
- 뒤집기 함수에서 `e--`가 빠지면 '\0'이 앞으로 와서 빈 문자열이 출력된다. 빈칸 문제에서 이 줄을 묻는다.
- 전역 `char buf[]`에 `gets`·`scanf`로 입력받고 포인터로 조작하는 문제(23-2)는 입력 문자열을 코드 옆에 적어 두고 추적한다.

## 구조체 (. 과 ->, 구조체 배열·포인터)
출제: 26-2, 25-3, 25-1, 24-1, 23-3, 21-3, 21-1
함정: 구조체 변수는 `.`, 구조체 포인터는 `->`. `(*p).name`과 `p->name`은 같다.

### 개념
| 식 | 뜻 |
|---|---|
| `s.score` | 구조체 변수 s의 멤버 |
| `p->score` | 포인터 p가 가리키는 구조체의 멤버. `(*p).score`와 같다 |
| `s[1].score` | 구조체 배열의 두 번째 원소 |
| `(p + 1)->score`, `p[1].score` | 구조체 배열을 포인터로 접근. 둘 다 s[1].score |
| `p++` | 다음 구조체로 이동(구조체 크기만큼 주소가 증가) |
| 자기 참조 구조체 | `struct Node { int v; struct Node *next; }` 연결 리스트·트리의 기본 모양 |

23-3에 "구조체 포인터로 멤버에 접근하는 연산자"를 단답으로 묻는 문제가 나왔다. 답은 `->`.

### 예제
```c
#include <stdio.h>
struct Student {
    char name[10];
    int score;
};
struct Node {
    int v;
    struct Node *left, *right;
};
void post(struct Node *n) {
    if (n == NULL) return;
    post(n->left);
    post(n->right);
    printf("%d ", n->v);
}
int main() {
    struct Student s[3] = {{"kim", 80}, {"lee", 95}, {"park", 70}};
    struct Student *p = s;
    int i, sum = 0;
    for (i = 0; i < 3; i++) sum += (p + i)->score;
    printf("%d %s\n", sum / 3, (*(p + 1)).name);
    p++;
    p->score += 5;
    printf("%d %c\n", s[1].score, p->name[0]);
    struct Node n4 = {4, NULL, NULL}, n5 = {5, NULL, NULL};
    struct Node n2 = {2, &n4, &n5}, n3 = {3, NULL, NULL};
    struct Node n1 = {1, &n2, &n3};
    post(&n1);
    printf("\n");
    return 0;
}
```
출력:
```
81 lee
100 l
4 5 2 3 1 
```
245 / 3은 정수 나눗셈이라 81. `p++` 뒤 p는 s[1]을 가리키므로 `p->score += 5`는 s[1].score를 100으로 만든다. 트리는 1(2(4,5),3) 모양이고 후위 순회는 왼쪽·오른쪽·자기 순이라 4 5 2 3 1.

### 자주 틀리는 포인트
- `p->score += 5`처럼 포인터로 바꾼 값은 원본 배열 s에도 반영된다. 복사가 아니라 같은 메모리다.
- 트리 순회 출력 순서: 전위 = 자기·왼·오, 중위 = 왼·자기·오, 후위 = 왼·오·자기. printf가 재귀 호출의 앞인지 뒤인지로 구분한다.
- 복리·평균 계산(24-1)처럼 구조체 멤버로 산술을 하는 문제는 정수 나눗셈과 실수 서식(`%.2f`)에 주의한다.
- 구조체 안의 문자열 멤버는 `p->name`이 배열(주소)이고 `p->name[0]`이 첫 글자다.

## 연결 리스트 순회·삽입·교환
출제: 25-3, 25-2, 25-1, 24-3, 24-2
함정: `next`를 바꾸는 순서가 틀리면 리스트가 끊긴다. 삽입은 "새 노드의 next를 먼저, 앞 노드의 next를 나중에".

### 개념
| 동작 | 코드 모양 | 추적 요령 |
|---|---|---|
| 순회 | `for (p = head; p != NULL; p = p->next)` | 노드를 상자로 그리고 화살표를 따라간다 |
| 끝에 추가 | `while (t->next != NULL) t = t->next; t->next = p;` | 빈 리스트일 때 head가 바뀌는 분기를 확인 |
| 중간 삽입 | `n->next = prev->next; prev->next = n;` | 이 두 줄의 순서가 핵심 |
| 삭제 | `prev->next = cur->next;` | cur가 가리키던 노드가 빠진다 |
| 역순 | `nx = cur->next; cur->next = prev; prev = cur; cur = nx;` | 세 포인터(prev, cur, nx)를 매 반복 표로 적는다 |
| 값 교환 | `t = p->v; p->v = p->next->v; p->next->v = t;` | 노드는 그대로, 값만 바뀐다 |
| 노드 교환 | prev→a→b를 prev→b→a로: `prev->next = b; a->next = b->next; b->next = a;` | next 세 개가 바뀐다. 1→2→3이 2→1→3이 된다 |

그림을 그릴 때는 노드마다 `[값 | next]` 상자를 쓰고, 포인터 변수(head, p, prev)를 상자 위에 이름표로 붙인다. 한 문장이 실행될 때마다 화살표 하나만 바꾼다.

```mermaid
제목: 노드는 값과 다음 노드의 주소(next)를 가진다. 마지막 노드의 next는 NULL
flowchart LR
  H(["head"]) --> A("10 | next")
  A --> B("20 | next")
  B --> N(["NULL"])
```

### 예제
```c
#include <stdio.h>
#include <stdlib.h>
typedef struct Node {
    int data;
    struct Node *next;
} Node;
Node *make(int d, Node *n) {
    Node *p = malloc(sizeof(Node));
    p->data = d;
    p->next = n;
    return p;
}
int main() {
    Node *head = make(1, make(2, make(3, NULL)));
    Node *cur = head, *prev = NULL;
    while (cur != NULL) {
        Node *nx = cur->next;
        cur->next = prev;
        prev = cur;
        cur = nx;
    }
    head = prev;
    Node *n = make(9, head->next);
    head->next = n;
    for (cur = head; cur != NULL; cur = cur->next) printf("%d ", cur->data);
    printf("\n%d\n", head->next->next->data);
    return 0;
}
```
출력:
```
3 9 2 1 
2
```
처음 리스트는 1→2→3. 역순 반복을 표로 적으면 (prev, cur) = (NULL,1) → (1,2) → (2,3) → (3,NULL)이고, 마지막 prev가 새 head라 3→2→1. 그 뒤 9를 head 다음에 끼우면 3→9→2→1. `head->next->next`는 세 번째 노드 2.

```c
#include <stdio.h>
#include <stdlib.h>
struct Node { int v; struct Node *next; };
struct Node *add(struct Node *head, int v) {
    struct Node *p = malloc(sizeof(struct Node)), *t = head;
    p->v = v; p->next = NULL;
    if (head == NULL) return p;
    while (t->next != NULL) t = t->next;
    t->next = p;
    return head;
}
int main() {
    struct Node *head = NULL, *p;
    int i;
    for (i = 1; i <= 5; i++) head = add(head, i * 10);
    for (p = head; p != NULL && p->next != NULL; p = p->next->next) {
        int t = p->v; p->v = p->next->v; p->next->v = t;
    }
    int cnt = 0;
    for (p = head; p != NULL; p = p->next) { printf("%d ", p->v); cnt++; }
    printf("\n%d\n", cnt);
    return 0;
}
```
출력:
```
20 10 40 30 50 
5
```
10→20→30→40→50을 만든 뒤 두 칸씩 건너뛰며 이웃한 값을 바꾼다. (10,20)→(20,10), (30,40)→(40,30), 50은 짝이 없어 `p->next != NULL` 조건에서 멈춘다.

### 자주 틀리는 포인트
- 역순 코드에서 `nx = cur->next`를 먼저 저장하지 않으면 다음 노드를 잃는다. 빈칸이 이 줄인 경우가 많다.
- `head = prev`를 빠뜨리면 head는 여전히 옛 첫 노드(이제 마지막 노드)를 가리켜 1만 출력된다.
- 값 교환과 노드 교환은 다르다. 값 교환은 next가 그대로이고, 노드 교환은 next를 여러 개 바꾼다. 어느 쪽인지 코드에서 `->v`를 바꾸는지 `->next`를 바꾸는지로 본다.
- 문자열을 한 글자씩 노드로 넣고 역순 출력하는 유형(25-2)은 "앞에 삽입"(`p->next = head; head = p;`)이면 자동으로 역순이 된다.

## 숫자 알고리즘 패턴 (완전수·소수·자릿수 뒤집기·잔돈)
출제: 23-3, 23-2, 23-1, 22-3, 22-1, 21-2, 20-3
함정: 소수 판별에서 `break`로 빠져나왔는지 끝까지 돌았는지를 `i == n`으로 구분한다.

### 개념
| 패턴 | 코드 핵심 | 결과 |
|---|---|---|
| 완전수 | `for (i = 1; i < n; i++) if (n % i == 0) s += i;` 뒤 `s == n` | 6, 28 |
| 소수 | `for (i = 2; i < n; i++) if (n % i == 0) break;` 뒤 `i == n`이면 소수 | 2 3 5 7 11 13 17 19 |
| 자릿수 뒤집기 | `rev = rev * 10 + x % 10; x /= 10;` | 1234 → 4321 |
| 자릿수 합 | `s += x % 10; x /= 10;` | 1234 → 10 |
| 잔돈(큰 단위부터) | `cnt = m / coin; m %= coin;` | 1270 → 500×2, 100×2, 50×1, 10×2 |
| 거듭제곱·팩토리얼 | `r *= b` n번 / `r *= i` | 반복 또는 재귀 |

### 예제
```c
#include <stdio.h>
int main() {
    int n, i, s, cnt = 0, rev = 0, x = 1234;
    for (n = 2; n <= 30; n++) {
        s = 0;
        for (i = 1; i < n; i++) if (n % i == 0) s += i;
        if (s == n) printf("%d ", n);
    }
    printf("\n");
    for (n = 2; n <= 20; n++) {
        for (i = 2; i < n; i++) if (n % i == 0) break;
        if (i == n) cnt++;
    }
    printf("%d\n", cnt);
    while (x > 0) { rev = rev * 10 + x % 10; x /= 10; }
    printf("%d\n", rev);
    int coin[] = {500, 100, 50, 10}, m = 1270;
    for (i = 0; i < 4; i++) { printf("%d ", m / coin[i]); m %= coin[i]; }
    printf("\n");
    return 0;
}
```
출력:
```
6 28 
8
4321
2 2 1 2 
```
30 이하 완전수는 6(1+2+3)과 28. 20 이하 소수는 8개. 뒤집기는 x%10으로 끝자리를 떼어 rev의 끝에 붙인다. 잔돈은 몫을 찍고 나머지로 m을 갱신한다.

### 자주 틀리는 포인트
- 완전수 판별에서 `s = 0` 초기화가 안쪽 반복 바깥에 있어야 한다. 빈칸 문제에서 이 자리가 비면 `s = 0`이 답이다.
- 소수 판별 반복이 `i <= n / 2`나 `i * i <= n`으로 바뀌면 "끝까지 돌았다"의 조건도 바뀐다. 조건식을 그대로 옮겨 적는다.
- 반복 곱에서 초기값이 0이면 결과는 항상 0이다(20-3). 초기값을 먼저 본다.
- 자릿수 뒤집기 빈칸은 `% 10`, `/ 10`, `* 10` 세 개가 돌아가며 나온다.

## 배열과 포인터 연산
출제: 26-2, 26-1, 21-2
함정: `p + 1`은 주소가 1 느는 것이 아니라 "다음 원소"다. `*(p + i)`와 `p[i]`와 `a[i]`는 같다.

### 개념
| 식 | 뜻 |
|---|---|
| `int *p = a;` | p는 a[0]의 주소. `p = &a[0]`과 같다 |
| `*(p + i)` = `p[i]` | i번째 원소 |
| `p++` | 다음 원소를 가리킴. 배열 이름 a는 `a++` 불가 |
| `*p++` | `*(p++)`: 현재 값을 쓰고 p를 이동 |
| `(*p)++`, `++*p` | 가리키는 값을 증가. p는 그대로 |
| `&a[3] - p` | 두 주소 사이의 원소 개수 |
| `*(a + 4) - *a` | a[4] - a[0] (값의 뺄셈) |

### 예제
```c
#include <stdio.h>
int main() {
    int a[5] = {10, 20, 30, 40, 50};
    int *p = a;
    printf("%d %d\n", *p, *(p + 2));
    p++;
    printf("%d %d\n", *p, p[2]);
    printf("%d\n", *(a + 4) - *a);
    printf("%d\n", (int)(&a[3] - p));
    int s = 0, i;
    for (i = 0; i < 5; i++) s += *(a + i);
    printf("%d\n", s / 5);
    ++*p;
    printf("%d %d\n", *p, a[1]);
    return 0;
}
```
출력:
```
10 30
20 40
40
2
30
21 21
```
`p++` 뒤 p는 a[1]을 가리키므로 `p[2]`는 a[3] = 40. `&a[3] - p`는 a[3]과 a[1] 사이 거리 2. `++*p`는 a[1]을 21로 바꾼다.

### 자주 틀리는 포인트
- `p++` 이후의 `p[i]`는 원래 배열 기준으로 한 칸 밀려 있다. p가 지금 어디를 가리키는지 그림에 표시한다.
- `*p++`와 `(*p)++`를 구분한다. 괄호가 없으면 `++`가 p에 붙는다.
- 평균 계산(26-1)은 합이 int면 `/ 5`도 정수 나눗셈이다. `%.1f`로 찍는다면 `(double)` 형변환이 어디 있는지 본다.
- 배열을 함수에 넘기면 포인터가 되므로 함수 안의 `sizeof(arr)`는 배열 크기가 아니라 포인터 크기다.

## 포인터 배열과 배열 포인터
출제: 24-2, 22-2, 21-3
함정: `int *pa[2]`는 포인터가 2개 들어 있는 배열이고, `int (*ap)[3]`은 "길이 3 배열"을 가리키는 포인터 하나다.

### 개념
| 선언 | 뜻 | 쓰임 |
|---|---|---|
| `int *pa[2]` | int 포인터 2개짜리 배열 | `pa[i] = a[i]`로 각 행의 시작 주소를 담아 2차원처럼 사용 |
| `int (*ap)[3]` | int[3]을 가리키는 포인터 | `ap = a`(a는 `int a[?][3]`). `ap[1][2]`, `(*(ap+1))[2]` |
| `char *names[]` | 문자열 포인터 배열 | `names[2]`는 "park", `*names[1]`은 'l' |
| `**(a + 1)` | `*(a + 1)`은 1행, 그것의 `*`는 1행 0열 | `a[1][0]` |

포인터 배열에서 `pa[i] + j`는 i행 j열의 주소, `*(pa[i] + j)`는 그 값이다.

### 예제
```c
#include <stdio.h>
int main() {
    int a[2][3] = {{1, 2, 3}, {4, 5, 6}};
    int *pa[2] = {a[0], a[1]};
    int (*ap)[3] = a;
    printf("%d %d\n", pa[1][2], *(pa[0] + 1));
    printf("%d %d\n", ap[1][0], (*(ap + 1))[2]);
    printf("%d\n", **(a + 1));
    char *names[] = {"kim", "lee", "park"};
    printf("%s %c\n", names[2], *names[1]);
    printf("%c\n", *(names[0] + 1));
    int i, s = 0;
    for (i = 0; i < 2; i++) s += *(pa[i] + i);
    printf("%d\n", s);
    return 0;
}
```
출력:
```
6 2
4 6
4
park l
i
6
```
`pa[1][2]`는 a[1][2] = 6, `*(pa[0] + 1)`은 a[0][1] = 2. `*names[1]`은 "lee"의 첫 글자, `*(names[0] + 1)`은 "kim"의 두 번째 글자. 마지막 합은 a[0][0] + a[1][1] = 1 + 5.

### 자주 틀리는 포인트
- `*names[1]`은 `*(names[1])`이다(`[]`가 `*`보다 먼저). 문자열의 첫 글자가 나온다.
- 포인터 배열 합 문제(22-2)는 `*(pa[i] + j)`에서 i와 j 중 무엇이 고정인지 반복문에서 확인한다.
- `int (*ap)[3]`에서 `ap + 1`은 3개 원소(한 행)만큼 건너뛴다.

## 이중 포인터
출제: 25-2, 25-1, 24-3
함정: `*pp = &b`는 p를 바꾸고, `**pp = 5`는 p가 가리키는 변수를 바꾼다. 별 개수를 세어 어느 층을 건드리는지 본다.

### 개념
| 식 | 층 | 뜻 |
|---|---|---|
| `pp` | 2층 | p의 주소 |
| `*pp` | 1층 | p 자체(어떤 int의 주소). `*pp = &b`는 p를 b로 돌림 |
| `**pp` | 0층 | p가 가리키는 int 값 |
| 함수 인자 `int **q` | | 함수 안에서 `*q = ...`로 호출자의 포인터 변수를 바꿀 수 있다 |
| `int **m = malloc(...)` | | 행 포인터 배열. `m[i] = malloc(...)`로 각 행을 만들면 `m[i][j]` 사용 |

구조체 이중 포인터(`struct Node **head`)는 함수 안에서 head 자체를 바꾸기 위해 쓴다. `*head = newNode;`가 호출자의 head를 바꾼다.

### 예제
```c
#include <stdio.h>
#include <stdlib.h>
void set(int **q) { **q = 100; }
int main() {
    int a = 1, b = 2;
    int *p = &a;
    int **pp = &p;
    **pp = 10;
    printf("%d %d\n", a, *p);
    *pp = &b;
    **pp += 5;
    printf("%d %d %d\n", a, b, *p);
    set(pp);
    printf("%d %d\n", a, b);
    int **m = malloc(2 * sizeof(int *));
    int i, j;
    for (i = 0; i < 2; i++) {
        m[i] = malloc(3 * sizeof(int));
        for (j = 0; j < 3; j++) m[i][j] = i * 3 + j;
    }
    printf("%d %d\n", m[1][2], *(*(m + 1) + 0));
    return 0;
}
```
출력:
```
10 10
10 7 7
10 100
5 3
```
`**pp = 10`은 a를 바꾼다. `*pp = &b`로 p가 b를 가리키게 되므로 `**pp += 5`는 b를 7로, 이후 `set(pp)`는 b를 100으로 만든다. a는 10 그대로. m은 2×3 배열처럼 쓰이고 m[1][2] = 1*3+2 = 5, m[1][0] = 3.

### 자주 틀리는 포인트
- `*pp = &b` 한 줄 뒤로는 `*p`가 b를 뜻한다. p의 방향이 바뀐 시점을 표에 기록한다.
- 함수가 `int *p`를 받아 `p = ...`로 재대입해도 호출자의 포인터는 안 바뀐다. 바꾸려면 `int **`로 받아 `*p = ...`해야 한다. 연결 리스트 삽입 함수에서 head가 안 바뀌는 이유가 이것이다.
- malloc으로 만든 2차원은 `m[i][j]`와 `*(*(m + i) + j)`가 같다. 행 i, 열 j 순서다.

## 비트·삼항·시프트 (마스크 연산)
출제: 25-3, 25-1, 24-1
함정: 결과를 `%x`로 찍으면 16진 소문자다. 10진으로 쓰면 틀린다.

### 개념
| 연산 | 뜻 | 0xA5(1010 0101) 기준 |
|---|---|---|
| `v & 0x0F` | 하위 4비트만 남김 | 0101 = 5 |
| `v \| 0x0F` | 하위 4비트를 전부 1로 | 1010 1111 = af |
| `v ^ 0xFF` | 8비트 반전 | 0101 1010 = 5a |
| `v >> 4` | 상위 4비트를 아래로 | 1010 = 10 |
| `(v << 1) & 0xFF` | 왼쪽 시프트 후 8비트만 유지 | 0100 1010 = 74 |
| `n & (1 << i)` | i번째 비트 검사 | 1인 비트 개수 세기에 사용 |

삼항 `a > b ? a : b`는 최댓값, `(a & 1) ? "odd" : "even"`은 홀짝이다. 비트 연산자 전체 표는 [코딩 공통] 비트 연산 주제에 있다.

### 예제
```c
#include <stdio.h>
int main() {
    unsigned char v = 0xA5;
    printf("%x %x %x\n", v & 0x0F, v | 0x0F, v ^ 0xFF);
    printf("%d %d\n", v >> 4, (v << 1) & 0xFF);
    int n = 37, i, bits = 0;
    for (i = 0; i < 8; i++) if (n & (1 << i)) bits++;
    printf("%d\n", bits);
    int a = 7, b = 12;
    int max = a > b ? a : b;
    printf("%d %s\n", max, (a & 1) ? "odd" : "even");
    printf("%d\n", 1 << 3 | 1 << 1);
    return 0;
}
```
출력:
```
5 af 5a
10 74
3
12 odd
10
```
0xA5를 2진 1010 0101로 펼쳐 놓고 자리별로 계산한다. 37 = 100101이라 1인 비트가 3개. `1 << 3 | 1 << 1`은 8 | 2 = 10.

### 자주 틀리는 포인트
- 구조체 멤버에 `& 0xA5` 같은 마스크를 거는 문제(25-1)는 멤버 값을 먼저 2진으로 바꾼 다음 자리를 맞춰 AND한다.
- `%x`는 소문자(af), `%X`는 대문자(AF)다. 서식 문자를 그대로 따른다.
- 시프트는 `|`보다 우선순위가 높지만 `+`보다는 낮다. `1 << 2 + 1`은 `1 << 3`이다.

## 정렬 코드 빈칸 (버블·선택)
출제: 25-1, 23-2, 20-1
함정: 빈칸은 대개 반복 범위(`n - 1 - i`), 비교 부호(`>` 오름차순 / `<` 내림차순), 교환 세 줄 중 하나다.

### 개념
| 정렬 | 모양 | 빈칸이 자주 나오는 자리 |
|---|---|---|
| 버블 | 이웃한 `a[j]`와 `a[j+1]`을 비교·교환. 바깥 i 1회전마다 뒤에서부터 하나씩 확정 | 안쪽 범위 `j < n - 1 - i`, 비교 `a[j] > a[j+1]` |
| 선택 | i 자리에 올 최솟값(최댓값)의 인덱스 k를 찾은 뒤 한 번만 교환 | `k = i` 초기화, 비교 `b[j] < b[k]`, 교환 `b[i]↔b[k]` |
| 교환 | `t = a[j]; a[j] = a[j+1]; a[j+1] = t;` | 임시 변수 줄 |

"몇 회전 후의 배열 상태"를 묻는 문제는 i 값마다 배열을 한 줄씩 적는다.

### 예제
```c
#include <stdio.h>
int main() {
    int a[5] = {5, 3, 8, 1, 4};
    int i, j, t, n = 5;
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - 1 - i; j++)
            if (a[j] > a[j + 1]) { t = a[j]; a[j] = a[j + 1]; a[j + 1] = t; }
        if (i == 0) {
            for (j = 0; j < n; j++) printf("%d ", a[j]);
            printf("\n");
        }
    }
    int b[5] = {5, 3, 8, 1, 4}, k;
    for (i = 0; i < n - 1; i++) {
        k = i;
        for (j = i + 1; j < n; j++) if (b[j] > b[k]) k = j;
        t = b[i]; b[i] = b[k]; b[k] = t;
    }
    for (i = 0; i < n; i++) printf("%d ", b[i]);
    printf("\n");
    return 0;
}
```
출력:
```
3 5 1 4 8 
8 5 4 3 1 
```
버블 1회전(i=0): 5,3 교환 → 3,5,8,1,4; 8,1 교환 → 3,5,1,8,4; 8,4 교환 → 3,5,1,4,8. 가장 큰 8이 맨 뒤로 갔다. 선택 정렬은 비교가 `>`라 내림차순이다.

### 자주 틀리는 포인트
- 비교 부호 하나로 오름차순·내림차순이 바뀐다. 출력 순서를 보고 부호를 거꾸로 추정하는 빈칸 문제가 있다.
- 버블 정렬 1회전 후에는 끝 한 자리만 확정이다. 전체가 정렬됐다고 생각하면 틀린다.
- 선택 정렬의 안쪽 반복은 `j = i + 1`부터 시작한다. `j = 0`이면 다른 알고리즘이 된다.
- Java 정렬 빈칸(23-1)도 같은 모양이다. swap 세 줄의 순서를 묻는다.

## 함수 호출 (값 전달과 주소 전달)
출제: 26-2, 24-2, 20-3
함정: `swap(a, b)`는 복사본을 바꾸므로 원본이 안 변한다. `swap(&a, &b)`와 포인터 매개변수여야 변한다.

### 개념
| 전달 방식 | 선언 | 호출 | 원본 변화 |
|---|---|---|---|
| 값 전달 | `void f(int x)` | `f(a)` | 없음 |
| 주소 전달 | `void f(int *x)` | `f(&a)` | `*x = ...`로 바뀜 |
| 배열 전달 | `void f(int arr[])` 또는 `int *arr` | `f(a)` | 배열은 항상 주소가 넘어가므로 바뀜 |
| 반환값 | `int f(int x) { x++; return x; }` | `r = f(a)` | a는 그대로, r만 새 값 |
| 중첩 호출 | `f(g(2))` | 안쪽 g부터 | g의 반환값이 f의 인자가 된다. f(x)=x*2, g(x)=x+3이면 f(g(2)) = 10 |

### 예제
```c
#include <stdio.h>
void swap1(int a, int b) { int t = a; a = b; b = t; }
void swap2(int *a, int *b) { int t = *a; *a = *b; *b = t; }
void fill(int arr[], int n) { int i; for (i = 0; i < n; i++) arr[i] *= 2; }
int inc(int x) { x++; return x; }
int main() {
    int x = 1, y = 2;
    swap1(x, y);
    printf("%d %d\n", x, y);
    swap2(&x, &y);
    printf("%d %d\n", x, y);
    int a[3] = {1, 2, 3};
    fill(a, 3);
    printf("%d %d %d\n", a[0], a[1], a[2]);
    int r = inc(x);
    printf("%d %d\n", x, r);
    return 0;
}
```
출력:
```
1 2
2 1
2 4 6
2 3
```
swap1은 복사본만 바꿔 1 2 그대로. swap2는 주소로 받아 2 1. 배열은 주소가 넘어가므로 fill 뒤 원본이 두 배. inc는 반환값만 3이고 x는 2다.

### 자주 틀리는 포인트
- 함수 안에서 매개변수 이름이 호출 쪽 변수와 같아도(`int a`와 `a`) 서로 다른 변수다. 이름이 같아서 헷갈리게 만든 문제가 있다.
- 값 전달 swap 뒤에 switch가 이어지는 문제(24-2)는 swap이 효과가 없다는 것을 먼저 확정하고 switch에 들어간다.
- 배열을 넘기고 함수 안에서 `arr = 다른배열`로 재대입하면 호출자의 배열은 안 바뀐다. 원소 대입(`arr[i] = `)만 반영된다.

## 재귀
출제: 23-3, 22-1
함정: 반환식에 `n *`이 붙어 있으면 팩토리얼, `+`이면 합, 두 번 호출하면 피보나치다. 호출 횟수는 트리 노드 수다.

### 개념
| 함수 | 모양 | 값 |
|---|---|---|
| 팩토리얼 | `n <= 1 ? 1 : n * fact(n-1)` | fact(5) = 120 |
| 거듭제곱 | `e == 0 ? 1 : b * power(b, e-1)` | power(2,10) = 1024 |
| 자릿수 합 | `n == 0 ? 0 : n % 10 + sumd(n/10)` | sumd(1234) = 10 |
| 피보나치 | `n < 2 ? n : fib(n-1) + fib(n-2)` | fib(5) = 5, 호출 15번 |

추적은 [코딩 공통]의 재귀 추적법(호출 트리)대로 한다. 전역 카운터가 있으면 함수에 들어갈 때마다 1씩 더하므로 호출 횟수가 된다.

### 예제
```c
#include <stdio.h>
int fact(int n) { return n <= 1 ? 1 : n * fact(n - 1); }
int power(int b, int e) { return e == 0 ? 1 : b * power(b, e - 1); }
int sumd(int n) { return n == 0 ? 0 : n % 10 + sumd(n / 10); }
int cnt = 0;
int fib(int n) {
    cnt++;
    if (n < 2) return n;
    return fib(n - 1) + fib(n - 2);
}
int main() {
    printf("%d %d %d\n", fact(5), power(2, 10), sumd(1234));
    int r = fib(5);
    printf("%d %d\n", r, cnt);
    return 0;
}
```
출력:
```
120 1024 10
5 15
```
fib 호출 횟수는 calls(n) = 1 + calls(n-1) + calls(n-2), calls(0) = calls(1) = 1이라 calls(2)=3, (3)=5, (4)=9, (5)=15.

### 자주 틀리는 포인트
- 종료 조건이 `n == 1`이면 fact(0)은 무한 재귀다. 기출은 `n <= 1`을 쓴다. 조건을 그대로 적는다.
- 피보나치 종료 조건 `n < 2`에서 `return n`은 fib(0)=0, fib(1)=1이다. `return 1`이면 값이 달라진다.
- 재귀 함수 안의 printf 위치(호출 전/후)에 따라 출력 순서가 뒤집힌다. [코딩 공통] 재귀 추적법 예제 참고.

## static 변수와 전역 변수
출제: 24-3, 23-2
함정: `static int s = 0;`은 처음 한 번만 0이 되고 이후 호출에서는 이전 값을 유지한다. 지역 변수는 매번 다시 0이다.

### 개념
| 종류 | 선언 위치 | 초기화 시점 | 수명 |
|---|---|---|---|
| 지역 변수 | 함수 안 | 호출마다 | 함수가 끝나면 사라짐 |
| static 지역 변수 | 함수 안 `static` | 프로그램 시작 시 한 번 | 프로그램 끝까지. 호출 사이에 값 유지 |
| 전역 변수 | 함수 밖 | 프로그램 시작 시 한 번(초기값 없으면 0) | 프로그램 끝까지. 모든 함수가 공유 |
| 같은 이름의 지역 변수 | 블록 안 | | 그 블록에서는 지역이 전역을 가린다 |

### 예제
```c
#include <stdio.h>
int g = 0;
int counter() {
    static int s = 0;
    int l = 0;
    s++; l++; g++;
    return s + l;
}
int main() {
    int i, r = 0;
    for (i = 0; i < 3; i++) r = counter();
    printf("%d %d\n", r, g);
    {
        int g = 100;
        printf("%d\n", g);
    }
    printf("%d\n", g);
    return 0;
}
```
출력:
```
4 3
100
3
```
세 번째 호출에서 s = 3(누적), l = 1(매번 새로) → 4. g는 3번 증가해 3. 안쪽 블록의 `int g = 100`은 별개 변수라 블록을 나오면 다시 3이다.

### 자주 틀리는 포인트
- `static int s = 0;` 줄은 매 호출마다 실행되는 것처럼 보이지만 초기화는 한 번뿐이다. 두 번째 호출부터는 이 줄을 건너뛴다고 생각하면 된다.
- 전역 변수는 초기값을 안 써도 0이다. 지역 변수는 쓰레기값이다(기출은 반드시 초기화한다).
- 전역 버퍼를 포인터로 조작하는 문제(23-2)는 함수가 끝나도 버퍼 내용이 남아 있다는 점을 쓴다.

## 스택·큐 코드 읽기
출제: 25-2, 23-2
함정: 원형 큐의 `rear = (rear + 1) % N`은 끝에서 0으로 돌아간다. front와 rear 값을 매 연산마다 적는다.

### 개념
| 자료구조 | 변수 | push/enqueue | pop/dequeue | 빈 상태 |
|---|---|---|---|---|
| 스택(배열) | `top = -1` | `st[++top] = v` | `return st[top--]` | `top == -1` |
| 큐(선형) | `front = rear = 0` | `q[rear++] = v` | `return q[front++]` | `front == rear` |
| 원형 큐 | `front = rear = 0`, 크기 N | `q[rear] = v; rear = (rear+1) % N` | `v = q[front]; front = (front+1) % N` | `front == rear` |

`(i + 1) % 5` 같은 식(23-2 빈칸)은 "다음 칸으로, 끝이면 0으로"다. 스택은 LIFO(마지막에 넣은 것이 먼저), 큐는 FIFO다.

### 예제
```c
#include <stdio.h>
#define N 4
int q[N], front = 0, rear = 0;
int st[N], top = -1;
void push(int v) { st[++top] = v; }
int pop() { return st[top--]; }
void enq(int v) { q[rear] = v; rear = (rear + 1) % N; }
int deq() { int v = q[front]; front = (front + 1) % N; return v; }
int main() {
    push(1); push(2); push(3);
    printf("%d ", pop());
    push(4);
    printf("%d ", pop());
    printf("%d\n", pop());
    enq(10); enq(20); enq(30);
    printf("%d ", deq());
    enq(40);
    enq(50);
    printf("%d %d\n", front, rear);
    printf("%d ", deq());
    printf("%d\n", q[0]);
    return 0;
}
```
출력:
```
3 4 2
10 1 1
20 50
```
스택: 1,2,3 넣고 3을 뺀 뒤 4를 넣으면 [1,2,4], 4와 2가 차례로 나온다. 큐: 10,20,30 넣으면 rear=3. 10을 빼 front=1. 40은 q[3]에 들어가고 rear는 (3+1)%4 = 0으로 돌아간다. 50은 q[0]에 들어가 rear=1.

### 자주 틀리는 포인트
- `st[++top]`과 `st[top++]`는 다르다. top 초기값이 -1이면 전위 `++top`이 맞다.
- 원형 큐에서 rear가 0으로 돌아간 뒤 q[0]에 덮어쓴다. 이미 뺀 자리라 데이터 손실은 없다.
- 큐의 원소 수는 `(rear - front + N) % N`이다. front == rear는 비어 있음(가득 참과 구분하기 위해 한 칸을 비워 두는 구현도 있다).

## switch fall-through
출제: 24-2, 23-2
함정: `break`가 없으면 일치한 case부터 아래 case들을 전부 실행한다. default도 지나간다.

### 개념
| 규칙 | 내용 |
|---|---|
| 진입 | 값과 같은 `case`로 점프한다. 없으면 `default` |
| 진행 | `break`를 만날 때까지 아래로 계속 실행. 다음 case 라벨은 무시 |
| default 위치 | 중간에 있어도 된다. 위에서 떨어져 내려오면 실행된다 |
| 반복문 안 | `break`는 switch만 빠져나간다. 반복문을 끝내지 않는다 |

### 예제
```c
#include <stdio.h>
int main() {
    int i, s = 0;
    for (i = 0; i < 4; i++) {
        switch (i) {
            case 0: s += 1;
            case 1: s += 10; break;
            case 2: s += 100;
            default: s += 1000;
        }
    }
    printf("%d\n", s);
    char g = 'B';
    switch (g) {
        case 'A': printf("A");
        case 'B': printf("B");
        case 'C': printf("C"); break;
        case 'D': printf("D");
    }
    printf("\n");
    return 0;
}
```
출력:
```
2121
BC
```
i=0: 1 + 10(떨어짐) = 11. i=1: +10 → 21. i=2: +100 + 1000(default로 떨어짐) → 1121. i=3: default만 → 2121. 'B'는 B, C를 찍고 break.

### 자주 틀리는 포인트
- i 값마다 "어느 case로 들어가서 어디서 break하는가"를 한 줄씩 적는다. 누적값만 적다가 떨어짐을 놓친다.
- `case 2: ... default: ...`처럼 default가 마지막이 아닐 수도 있다. 위치보다 "떨어지는 경로"로 읽는다.
- 24-2처럼 값 전달 swap 뒤에 switch가 오면, switch에 들어가는 값이 swap 전 값이라는 점부터 확정한다.

## printf 서식과 자료형
출제: 26-1, 24-1
함정: 같은 값도 서식에 따라 다르게 나온다. 97을 `%d`는 97, `%c`는 a, `%x`는 61.

### 개념
| 서식 | 뜻 | 예(255, 'a', 3.14159) |
|---|---|---|
| `%d` | 10진 정수 | 255 |
| `%u` / `%ld` | 부호 없는 정수 / long | (unsigned)-1은 4294967295 |
| `%x` / `%X` | 16진 소문자 / 대문자 | ff / FF |
| `%o` | 8진 | 377 |
| `%c` | 문자 | a (97을 넘겨도 a) |
| `%s` | 문자열 | |
| `%f` | 실수 소수점 6자리 | 3.141590 |
| `%lf` | double. printf에서는 `%f`와 같고, scanf로 double을 읽을 때는 반드시 `%lf` | 3.141590 |
| `%.2f` | 소수점 2자리 (반올림) | 3.14 |
| `%5.1f` | 전체 5칸, 소수 1자리, 오른쪽 정렬 | `  3.1` |
| `%-5d` | 왼쪽 정렬 5칸 | `255  ` |
| `%05d` | 0으로 채워 5칸 | 00255 |
| `%%` | % 문자 | % |

자료형 크기(일반적인 64비트 환경): char 1, int 4, double 8. `sizeof`는 바이트 수다. 대소문자 변환: `c - 32`(소→대), `c + 32`(대→소). `isupper`, `toupper`는 `<ctype.h>`.

### 예제
```c
#include <stdio.h>
int main() {
    int n = 255;
    char c = 'a';
    double d = 3.14159;
    printf("%d %x %o %c\n", n, n, n, c);
    printf("%c %d\n", c - 32, c);
    printf("%.2f %5.1f|%-5d|%05d\n", d, d, n, n);
    printf("%d %d %d\n", (int)sizeof(char), (int)sizeof(int), (int)sizeof(double));
    printf("%d\n", 'A' + 1);
    printf("%c\n", 'A' + 1);
    printf("%d%%\n", 50);
    return 0;
}
```
출력:
```
255 ff 377 a
A 97
3.14   3.1|255  |00255
1 4 8
66
B
50%
```
`c - 32`를 `%c`로 찍으면 대문자 A, c를 `%d`로 찍으면 97. `%5.1f`는 "3.1" 앞에 공백 2칸이 붙어 5칸이 된다.

### 자주 틀리는 포인트
- `%x` 출력은 접두사 0x 없이 소문자다. 답안에 "0xff"나 "FF"라고 쓰면 서식과 다르다.
- `%.2f`는 반올림한다. 3.145 같은 값은 이진 표현 때문에 기대와 다를 수 있으므로 기출은 깔끔한 값을 쓴다.
- `char`에 'A' + 1을 넣는 것은 가능하고, 출력 서식이 `%d`인지 `%c`인지가 답을 결정한다.
- 폭 지정(`%5d`)에서 자릿수가 폭보다 크면 잘리지 않고 그대로 나온다.

## 함수 포인터
출제: 26-1
함정: `int (*fp)(int, int)`는 "함수를 가리키는 포인터"다. `int *fp(int, int)`는 포인터를 반환하는 함수 선언이다.

### 개념
| 식 | 뜻 |
|---|---|
| `int (*fp)(int, int);` | int 둘을 받아 int를 돌려주는 함수를 가리키는 포인터 |
| `fp = add;` | 함수 이름이 곧 주소. `&add`도 같다 |
| `fp(2, 3)`, `(*fp)(2, 3)` | 둘 다 호출. 같은 결과 |
| 구조체 멤버 `int (*fn)(int, int);` | `ops[i].fn(4, 5)`로 호출. 이름 문자열과 짝지어 표로 쓰는 유형 |
| 매개변수 `int (*f)(int, int)` | 함수를 인자로 넘김. `apply(mul, 3, 7)` |

### 예제
```c
#include <stdio.h>
int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }
struct Op { char *name; int (*fn)(int, int); };
int apply(int (*f)(int, int), int x, int y) { return f(x, y); }
int main() {
    int (*fp)(int, int) = add;
    printf("%d\n", fp(2, 3));
    fp = mul;
    printf("%d\n", (*fp)(2, 3));
    struct Op ops[2] = {{"add", add}, {"mul", mul}};
    int i;
    for (i = 0; i < 2; i++) printf("%s=%d ", ops[i].name, ops[i].fn(4, 5));
    printf("\n%d\n", apply(ops[1].fn, 3, 7));
    return 0;
}
```
출력:
```
5
6
add=9 mul=20 
21
```
fp가 add일 때 5, mul로 바꾸면 6. 구조체 배열의 fn 멤버는 각각 add, mul이라 9와 20. `apply(ops[1].fn, 3, 7)`은 mul(3, 7).

### 자주 틀리는 포인트
- 함수 포인터가 지금 어느 함수를 가리키는지가 전부다. 대입 줄마다 표시한다.
- 26-1 문제처럼 결과를 `%x`로 찍으면 16진으로 바꿔야 한다. 20은 14, 21은 15.
- `(*fp)(2, 3)`의 괄호는 호출 문법일 뿐이다. 이중 포인터와 혼동하지 않는다.
