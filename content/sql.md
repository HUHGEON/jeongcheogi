# SQL

SQL은 21개 회차 420문제 중 38문제(9%)를 차지하고, 회차마다 0~4문제씩 나온다. 그런데 문제 형태를 모아 보면 딱 두 가지뿐이다. 하나는 **실행 결과 쓰기**(17문제)로, 작은 표 2~3개와 쿼리를 주고 COUNT 값이나 결과 표를 쓰게 한다. 다른 하나는 **키워드 빈칸**(21문제)으로, SQL 문장에서 예약어 자리를 비워 둔다. 2024년 이후 14문제 중 10문제가 실행 결과 쓰기였고, 2023년 이전에는 키워드 빈칸이 더 많았다.

아래 예제는 모두 `sqlite3 :memory:`로 직접 실행해 출력을 확인한 것이다. 다만 외래키 예제는 sqlite에서 `PRAGMA foreign_keys = ON;`을 먼저 실행해 줘야 동작한다.

## 키워드 빈칸: SELECT 문
출제: 26-2, 23-1, 22-2, 22-1, 21-2, 20-4, 20-3, 20-2
형태: SELECT 문장의 예약어 자리 1~3개 빈칸

### 개념

SELECT 문은 절을 쓰는 순서가 `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY`로 정해져 있다. 여기서 많이들 헷갈리는 게, 쓰는 순서와 실제로 처리되는 순서가 다르다는 점이다. 실제로는 `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY` 순서로 처리된다. 그래서 WHERE에서는 집계 함수를 쓸 수 없고 HAVING에서 써야 한다. WHERE가 처리될 때는 아직 GROUP BY로 묶기 전이기 때문이다. 반대로 ORDER BY는 맨 마지막에 처리되니 SELECT에서 붙인 별칭을 그대로 쓸 수 있다.

| 자리 | 완성 문장 | 단서 |
|---|---|---|
| 그룹별 집계 | `SELECT 부서번호, COUNT(*) FROM 사원 GROUP BY 부서번호;` | "부서별", "과목별" |
| 그룹 조건 | `SELECT 부서번호, SUM(급여) FROM 사원 GROUP BY 부서번호 HAVING COUNT(*) >= 2;` | 집계 결과에 조건 → **HAVING** |
| 그룹 최대·최소 | `SELECT 부서번호, MAX(급여), MIN(급여) FROM 사원 GROUP BY 부서번호 HAVING AVG(급여) >= 2000;` | |
| 패턴 | `SELECT 이름 FROM 학생 WHERE 이름 LIKE '이%';` | "이로 시작" → `'이%'`, "두 번째 글자가 철" → `'_철%'` |
| 정렬 | `SELECT 이름, 점수 FROM 학생 ORDER BY 점수 DESC;` | "내림차순" → **DESC**, 오름차순은 **ASC**(생략 가능) |
| 다중 정렬 | `SELECT 부서번호, 이름, 급여 FROM 사원 ORDER BY 부서번호 DESC, 급여 DESC;` | 앞 컬럼으로 먼저 정렬하고, 값이 같을 때만 뒤 컬럼으로 정렬 |
| 별칭 | `SELECT 부서번호, SUM(급여) AS 합계 FROM 사원 GROUP BY 부서번호;` | **AS**는 생략할 수 있다 (`SUM(급여) 합계`) |
| 목록 | `SELECT * FROM 학생 WHERE 학과 IN ('컴공', '전자');` | "중 하나" → **IN** |
| 범위 | `SELECT * FROM 학생 WHERE 점수 BETWEEN 70 AND 85;` | 양 끝 포함 |
| 조인 | `SELECT 이름, 부서명 FROM 사원 JOIN 부서 ON 사원.부서번호 = 부서.부서번호;` | 명시적 조인 조건 → **ON** |
| 모두 비교 | `SELECT 이름 FROM 사원 WHERE 급여 > ALL (SELECT 급여 FROM 사원 WHERE 부서번호 = 20);` | "모든 ~보다 크다" → **ALL** |
| 중복 제거 | `SELECT DISTINCT 학과 FROM 학생;` | |
| NULL 검사 | `SELECT 이름 FROM 학생 WHERE 점수 IS NULL;` | `= NULL`은 결과가 UNKNOWN이라 어떤 행도 고르지 못한다 |

표는 빈칸에 들어갈 완성 문장을 모아 둔 것이다. 이제 사원 테이블 하나에 절을 하나씩 붙여 가며 결과가 어떻게 바뀌는지 직접 확인한다. 사원 예제는 모두 이 5행으로 돌린 것이고(5단계만 학생 테이블을 쓴다), 부서번호가 NULL인 유관순이 하나 끼어 있다.

| 사번 | 이름 | 부서번호 | 급여 |
|---|---|---|---|
| 101 | 홍길동 | 10 | 3000 |
| 102 | 이순신 | 20 | 1000 |
| 103 | 강감찬 | 10 | 4000 |
| 104 | 유관순 | NULL | 2000 |
| 105 | 김유신 | 20 | 2500 |

#### 1. 행 고르기: SELECT ... FROM ... WHERE
급여가 2500 이상인 사람의 이름과 급여만 보고 싶다면 어떻게 쓸까? 어느 테이블에서(FROM) 어떤 행을(WHERE) 골라 어떤 열을(SELECT) 보여 줄지를 적으면 된다.

```sql
SELECT 이름, 급여 FROM 사원 WHERE 급여 >= 2500;
```
출력:

| 이름 | 급여 |
|---|---|
| 홍길동 | 3000 |
| 강감찬 | 4000 |
| 김유신 | 2500 |

WHERE는 행 하나하나에 조건을 대 보고 참인 행만 남긴다. 그렇게 남은 3행에서 SELECT에 적은 이름·급여 두 열만 나온다.

#### 2. 묶기: GROUP BY
그럼 부서별 인원은 어떻게 셀까? "부서별"이라는 말이 나오면 GROUP BY 자리다.

```sql
SELECT 부서번호, COUNT(*) FROM 사원 GROUP BY 부서번호;
```
출력:

| 부서번호 | COUNT(*) |
|---|---|
| NULL | 1 |
| 10 | 2 |
| 20 | 2 |

부서번호가 같은 행끼리 한 그룹이 되고, COUNT(*)는 그룹마다 따로 센다. 5행이 3행으로 줄어든 이유다. 부서번호가 NULL인 유관순도 버려지지 않고 NULL 그룹 하나로 나온다.

#### 3. 그룹에 조건 걸기: HAVING
이번에는 인원이 2명 이상인 부서만 남기고 싶다. 조건이니까 WHERE에 `COUNT(*) >= 2`를 쓰면 될 것 같지만, 아래 문장은 실행되지 않는다.

```sql
SELECT 부서번호, SUM(급여) FROM 사원 WHERE COUNT(*) >= 2 GROUP BY 부서번호;
```

sqlite는 이 문장을 실행하지 않고 `misuse of aggregate: COUNT()` 오류를 낸다. 위에서 처리 순서를 봤었다. WHERE가 처리될 때는 아직 그룹이 없으니 그룹마다 세는 COUNT를 쓸 수가 없는 것이다. 그래서 그룹을 만든 뒤에 거르는 HAVING에 써야 한다.

```sql
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT 부서번호, COUNT(*) AS 인원, SUM(급여) AS 합계 FROM 사원
GROUP BY 부서번호 HAVING COUNT(*) >= 2;
```
출력: `10 | 2 | 7000`, `20 | 2 | 3500` (부서번호 NULL인 유관순은 인원 1이라 HAVING에서 걸러진다)

2단계의 결과 3행 중에서 인원이 2인 두 그룹만 남았다. 정리하자면 WHERE는 묶기 전의 **행**을 거르고, HAVING은 묶은 뒤의 **그룹**을 거른다. 지문에 "집계 결과에 조건"이 보이면 HAVING이다.

#### 4. 정렬하기: ORDER BY
남은 행을 급여가 높은 순으로 보고 싶다면 맨 끝에 ORDER BY를 붙인다.

```sql
SELECT 이름, 급여 FROM 사원 WHERE 급여 >= 2500 ORDER BY 급여 DESC;
```
출력: `강감찬 4000`, `홍길동 3000`, `김유신 2500`

1단계와 같은 3행인데 순서만 바뀌었다. DESC가 내림차순이고, 아무것도 안 쓰면 오름차순(ASC)이다. 그럼 정렬 기준을 두 개 쓰면 어떻게 될까?

```sql
SELECT 부서번호, 이름, 급여 FROM 사원 WHERE 부서번호 IS NOT NULL ORDER BY 부서번호 DESC, 급여 DESC;
```
출력:

| 부서번호 | 이름 | 급여 |
|---|---|---|
| 20 | 김유신 | 2500 |
| 20 | 이순신 | 1000 |
| 10 | 강감찬 | 4000 |
| 10 | 홍길동 | 3000 |

먼저 부서번호로 20, 10 순서를 정하고, 부서번호가 같은 두 사람끼리만 급여로 다시 정렬했다. 강감찬(4000)이 김유신(2500)보다 급여가 높은데도 아래에 있는 건 앞 컬럼이 우선이기 때문이다.

ORDER BY는 맨 마지막에 처리되니 SELECT에서 붙인 별칭으로도 정렬할 수 있다.

```sql
SELECT 부서번호, SUM(급여) 합계 FROM 사원 GROUP BY 부서번호 ORDER BY 합계 DESC;
```
출력:

| 부서번호 | 합계 |
|---|---|
| 10 | 7000 |
| 20 | 3500 |
| NULL | 2000 |

`SUM(급여) 합계`처럼 AS를 빼고 써도 별칭이 붙는다.

#### 5. 패턴·목록·범위: LIKE, IN, BETWEEN
WHERE 조건에는 `=`, `>=` 말고도 자주 나오는 연산자가 셋 있다. 예제 테이블은 학생으로 바꾼다.

| 학번 | 이름 | 학과 | 점수 |
|---|---|---|---|
| 1 | 김철수 | 컴공 | 80 |
| 2 | 이영희 | 컴공 | NULL |
| 3 | 박민수 | 전자 | 70 |
| 4 | 최지우 | 전자 | NULL |
| 5 | 정하늘 | 기계 | 90 |

먼저 "이로 시작하는 이름"과 "두 번째 글자가 철인 이름"이다.

```sql
SELECT 이름 FROM 학생 WHERE 이름 LIKE '이%';
SELECT 이름 FROM 학생 WHERE 이름 LIKE '_철%';
```
출력:
```
이영희
김철수
```

`LIKE`에서 `%`는 0글자 이상을, `_`는 정확히 한 글자를 뜻한다. 그래서 `'%철%'`은 "철이 포함된" 값이고, `'_철'`은 "철 앞에 딱 한 글자"가 오는 값이다. 위의 `'_철%'`은 앞에 딱 한 글자(김), 그다음 철, 뒤는 몇 글자든 상관없다는 뜻이라 김철수가 걸렸다.

다음은 "컴공이나 전자 중 하나"와 "70점부터 85점까지"다.

```sql
SELECT 이름, 학과 FROM 학생 WHERE 학과 IN ('컴공', '전자');
SELECT 이름, 점수 FROM 학생 WHERE 점수 BETWEEN 70 AND 85;
```
출력:

| 이름 | 학과 |
|---|---|
| 김철수 | 컴공 |
| 이영희 | 컴공 |
| 박민수 | 전자 |
| 최지우 | 전자 |

| 이름 | 점수 |
|---|---|
| 김철수 | 80 |
| 박민수 | 70 |

IN은 괄호 안 목록 중 하나와 같으면 참이다. BETWEEN은 양 끝을 포함하기 때문에 딱 70점인 박민수도 들어왔다.

#### 6. 서브쿼리와 비교: ALL, ANY
"20번 부서의 모든 사람보다 급여가 많은 사원"처럼 비교 대상이 값 여러 개라면 어떻게 할까? 먼저 비교 대상부터 꺼낸다.

```sql
SELECT 급여 FROM 사원 WHERE 부서번호 = 20;
```
출력:
```
1000
2500
```

서브쿼리 앞에 붙는 비교 연산자는 이 목록을 어떻게 볼지 정한다. `> ALL`은 서브쿼리 결과의 모든 값보다 커야 하니 결국 **최댓값보다** 커야 참이다. 반면 `> ANY`는 그중 하나보다만 크면 되니 **최솟값보다** 크기만 하면 참이다. 방향을 뒤집으면 `< ALL`은 최솟값보다 작아야 참이고, `< ANY`는 최댓값보다 작으면 참이다. 이름이 달라 혼동하기 쉬운데 **ANY와 SOME은 같은 뜻**이다. 같은 식으로 `IN`은 `= ANY`와, `NOT IN`은 `<> ALL`과 같다. `EXISTS`는 성격이 조금 다르다. 값을 비교하지 않고, 서브쿼리가 행을 하나라도 돌려주는지만 본다.

참고로 sqlite는 `ALL`과 `ANY`를 지원하지 않아서 아래 예제에서는 `MAX`/`MIN` 비교로 바꿔 실행했다. 서브쿼리 결과가 비어 있지 않고 NULL이 없으면 두 방식의 결과는 같고, PostgreSQL에서 `> ALL`/`> ANY` 원문으로 실행해도 같은 결과가 나온다.

```sql
-- 급여 > ALL (SELECT 급여 FROM 사원 WHERE 부서번호 = 20) 과 결과가 같다 (서브쿼리가 비어 있지 않고 NULL이 없을 때)
SELECT 이름, 급여 FROM 사원 WHERE 급여 > (SELECT MAX(급여) FROM 사원 WHERE 부서번호 = 20);
```
출력: `홍길동 3000`, `강감찬 4000` (ANY였다면 MIN과 비교하므로 유관순·김유신도 포함되어 4명)

ALL은 최댓값 2500보다 커야 하니 3000, 4000 두 명만 남는다. ANY로 바꾸면 비교 기준이 최솟값 1000으로 내려간다. 직접 돌려 보면 이렇다.

```sql
-- 급여 > ANY (SELECT 급여 FROM 사원 WHERE 부서번호 = 20) 과 결과가 같다
SELECT 이름, 급여 FROM 사원 WHERE 급여 > (SELECT MIN(급여) FROM 사원 WHERE 부서번호 = 20);
```
출력:

| 이름 | 급여 |
|---|---|
| 홍길동 | 3000 |
| 강감찬 | 4000 |
| 유관순 | 2000 |
| 김유신 | 2500 |

1000보다 큰 사람은 1000을 받는 이순신 본인을 빼고 모두다.

#### 7. NULL 고르기: IS NULL
마지막은 부서번호가 비어 있는 사람 찾기다. `= NULL`로 쓰면 될 것 같지만 결과를 비교해 보면 다르다.

```sql
SELECT COUNT(*) FROM 사원 WHERE 부서번호 = NULL;
SELECT COUNT(*) FROM 사원 WHERE 부서번호 IS NULL;
```
출력: `0`, `1`

`= NULL`은 결과가 참도 거짓도 아닌 UNKNOWN이라 어떤 행도 WHERE를 통과하지 못한다. 유관순을 찾으려면 `IS NULL`로 써야 한다. 왜 그런지는 아래 「집계 함수와 NULL」 묶음에서 다시 다룬다.

### 기출에서 이렇게 나왔다

- 26-2, 21-2: '이'로 시작하는 이름을 내림차순 → **`'이%'` / `DESC`**
- 23-1: SUM과 그룹 조건 → **GROUP BY / HAVING**; MIN·MAX와 평균 조건 → **GROUP BY / HAVING AVG**
- 20-3: MAX·MIN, 그룹 평균 조건 → **GROUP BY / HAVING / AVG** (23-1과 같은 유형)
- 22-2: 서브쿼리의 모든 값보다 큰 → **ALL**
- 22-1: 점수 내림차순 → **ORDER BY 점수 DESC**
- 21-2: 명시적 조인 → **JOIN ... ON**
- 20-4: 그룹별 COUNT → **GROUP BY**
- 20-2: 목록에 포함 → **IN**

### 답 쓸 때

대소문자는 구분하지 않지만, 답안에는 예약어를 대문자로 쓰는 쪽이 안전하다. 문자열은 `'이%'`처럼 작은따옴표 안에 넣는다. DESC와 ASC를 뒤집어 쓰지 않으려면 "내림차순 = DESC = Descending"으로 묶어서 기억해 두면 된다.

## 결과 예측: DISTINCT 튜플 수, UNION, ON DELETE CASCADE
출제: 26-1, 23-3, 22-3, 20-1
형태: 쿼리 2~3개의 결과 행 수 / UNION 결과 표 / 삭제 후 COUNT

### 개념

이 묶음의 문제는 모두 "결과가 몇 행인가"를 묻는다. 그런데 같은 표라도 중복을 지우느냐, 두 결과를 합칠 때 중복을 남기느냐, 부모 행을 지울 때 자식 행이 같이 지워지느냐에 따라 행 수가 달라진다. 규칙부터 표로 모으면 이렇다.

| 함정 | 규칙 |
|---|---|
| `COUNT(*)` | 전체 행 수. 중복이든 NULL이든 다 센다 |
| `COUNT(DISTINCT 컬럼)` | 그 컬럼의 서로 다른 값 개수. NULL은 빼고 센다 |
| `GROUP BY 컬럼` | 그 컬럼이 NULL인 행은 버려지지 않고 **NULL끼리 한 그룹**이 된다(집계 함수의 NULL 무시와 다르다) |
| `SELECT DISTINCT A, B` | (A, B) **조합**이 같은 행만 하나로. A 하나만 같은 것은 중복이 아니다 |
| `UNION` | 두 결과를 합치고 **중복 행 제거**. `ORDER BY`는 맨 뒤에 한 번만, 전체 결과에 적용 |
| `UNION ALL` | 중복을 남긴다 |
| `ON DELETE CASCADE` | 부모 행을 지우면 그 행을 참조하던 자식 행도 **같이 삭제** |
| `ON DELETE SET NULL` | 부모 행을 지우면 자식의 외래키를 **NULL로** |
| `ON DELETE SET DEFAULT` | 부모 행을 지우면 자식의 외래키를 그 컬럼의 **DEFAULT 값으로** |
| `ON DELETE RESTRICT` | 참조하는 자식이 있으면 부모 삭제를 **즉시 거부** |
| `ON DELETE NO ACTION` · 옵션 없음 | 참조하는 자식이 있으면 부모 삭제가 **거부**된다 (RESTRICT와 결과는 같고, 검사 시점만 문장 끝으로 미룰 수 있다) |

ON UPDATE도 같은 옵션을 쓰는데, 이쪽은 부모 행을 지울 때가 아니라 부모의 키 값이 바뀔 때 자식을 어떻게 할지 정한다.

#### 1. 몇 개를 세나: COUNT(*)와 COUNT(DISTINCT)
아래 수강 테이블(6행)로 시작하자. 학번 1과 3은 두 과목씩 들었다.

| 학번 | 과목코드 | 성적 |
|---|---|---|
| 1 | C1 | A |
| 1 | C2 | B |
| 2 | C1 | A |
| 3 | C1 | A |
| 3 | C3 | C |
| 5 | C2 | A |

`COUNT(*)`는 행을 그대로 세니 6이다. 그럼 "수강한 학생 수"는? 학번 1과 3이 두 번씩 나와서 6이 아니다. 중복을 지운 학번은 이렇다.

```sql
SELECT DISTINCT 학번 FROM 수강;
```
출력:
```
1
2
3
5
```

서로 다른 학번은 네 개다. `COUNT(DISTINCT 학번)`은 이렇게 중복을 지운 뒤에 세는 것이라 4가 된다.

#### 2. 열이 둘이면 조합으로 본다: SELECT DISTINCT A, B
DISTINCT 뒤에 열이 둘 이상이면 어떻게 될까? 과목코드 하나만 쓸 때와 성적까지 같이 쓸 때를 나란히 돌리면 이렇다.

```sql
SELECT DISTINCT 과목코드 FROM 수강;
SELECT DISTINCT 과목코드, 성적 FROM 수강;
```
출력:

| 과목코드 |
|---|
| C1 |
| C2 |
| C3 |

| 과목코드 | 성적 |
|---|---|
| C1 | A |
| C2 | B |
| C3 | C |
| C2 | A |

과목코드만 보면 C2가 하나로 합쳐지지만, (C2, B)와 (C2, A)는 성적이 달라서 따로 남는다. 반대로 C1 세 행은 성적까지 A로 같아서 하나가 됐다. DISTINCT는 SELECT에 적은 열 전체의 **조합**이 같아야 중복으로 본다. 비슷하게 `GROUP BY`도 값이 같은 행을 하나로 묶는데, 앞의 SELECT 문 묶음 2단계에서 봤듯이 NULL인 행은 버리지 않고 NULL 그룹 하나로 남긴다.

여기까지를 쿼리 네 개로 한 번에 확인하면 이렇다.

```sql
CREATE TABLE 수강(학번 INT, 과목코드 TEXT, 성적 TEXT);
INSERT INTO 수강 VALUES (1,'C1','A'),(1,'C2','B'),(2,'C1','A'),
  (3,'C1','A'),(3,'C3','C'),(5,'C2','A');

SELECT COUNT(*) FROM 수강;
SELECT COUNT(DISTINCT 학번) FROM 수강;
SELECT COUNT(DISTINCT 성적) FROM 수강;
SELECT COUNT(*) FROM (SELECT DISTINCT 과목코드, 성적 FROM 수강);
```
출력: `6`, `4`, `3`, `4` (마지막은 (C1,A)·(C2,B)·(C3,C)·(C2,A) 네 조합. 과목코드 C1 세 행은 성적까지 같아 하나로 합쳐진다)

#### 3. 두 결과 합치기: UNION과 UNION ALL
이번에는 테이블 A의 값 {1, 2, 4, 5}와 B의 값 {2, 5, 8}을 하나로 합친다. 2와 5는 양쪽에 다 있다.

```sql
CREATE TABLE A(값 INT);  CREATE TABLE B(값 INT);
INSERT INTO A VALUES (1),(2),(4),(5);
INSERT INTO B VALUES (2),(5),(8);

SELECT 값 FROM A UNION SELECT 값 FROM B ORDER BY 값 DESC;
```
출력: `8, 5, 4, 2, 1` (2와 5는 한 번만)

```sql
SELECT 값 FROM A UNION ALL SELECT 값 FROM B ORDER BY 값 DESC;
```
출력: `8, 5, 5, 4, 2, 2, 1`

UNION은 합친 뒤 중복을 지우니 5행, UNION ALL은 중복을 그대로 두니 4 + 3 = 7행이다. 두 쿼리 모두 `ORDER BY`는 두 번째 SELECT 뒤에 한 번만 붙였는데, 이 ORDER BY는 B만이 아니라 합쳐진 전체 결과를 정렬한다. 그래서 A의 값과 B의 값이 섞여서 8부터 내려온다.

#### 4. 부모를 지우면 자식도: ON DELETE CASCADE
여기서부터는 외래키 옵션이다. 학과(부모)와 학생2(자식)를 만들고, 학생2의 학과코드가 학과를 참조하게 한 뒤 D1 학과를 지워 보자. D1에는 김·이 두 학생이 있다.

```sql
PRAGMA foreign_keys = ON;
CREATE TABLE 학과(학과코드 TEXT PRIMARY KEY, 학과명 TEXT);
CREATE TABLE 학생2(학번 INT PRIMARY KEY, 이름 TEXT,
  학과코드 TEXT REFERENCES 학과(학과코드) ON DELETE CASCADE);
INSERT INTO 학과 VALUES ('D1','컴공'),('D2','전자'),('D3','기계');
INSERT INTO 학생2 VALUES (1,'김','D1'),(2,'이','D1'),(3,'박','D2'),(4,'최','D2'),(5,'정','D3');

DELETE FROM 학과 WHERE 학과코드 = 'D1';
SELECT COUNT(*) FROM 학생2;
SELECT COUNT(DISTINCT 학과코드) FROM 학생2;
```
출력: `3`, `2` (D1 소속 2명이 같이 지워졌다)

학과에서 한 행을 지웠을 뿐인데 학생2가 5행에서 3행이 됐다. 남은 행을 직접 보면 D1 학생만 사라진 걸 확인할 수 있다.

```sql
SELECT * FROM 학생2;
```
출력:

| 학번 | 이름 | 학과코드 |
|---|---|---|
| 3 | 박 | D2 |
| 4 | 최 | D2 |
| 5 | 정 | D3 |

학과코드도 D2·D3 두 종류만 남아서 `COUNT(DISTINCT 학과코드)`가 2였던 것이다.

#### 5. 자식은 남기고 외래키만 비우기: ON DELETE SET NULL
CASCADE가 자식 행까지 지운다면, SET NULL은 자식 행은 그대로 두고 외래키 칸만 비운다. 개발 부서(10)를 지우면 거기 속한 사원 1, 2는 어떻게 될까?

```sql
CREATE TABLE 부서3(부서번호 INT PRIMARY KEY, 부서명 TEXT);
CREATE TABLE 사원3(사번 INT PRIMARY KEY,
  부서번호 INT REFERENCES 부서3(부서번호) ON DELETE SET NULL);
INSERT INTO 부서3 VALUES (10,'개발'),(20,'영업');
INSERT INTO 사원3 VALUES (1,10),(2,10),(3,20);
DELETE FROM 부서3 WHERE 부서번호 = 10;
SELECT * FROM 사원3;
```
출력: `1 | NULL`, `2 | NULL`, `3 | 20` (행은 그대로 3개, 외래키만 비었다)

그래서 같은 "부모 삭제 뒤 COUNT(*)" 문제라도 CASCADE면 줄고, SET NULL이면 그대로다. 옵션 이름부터 확인해야 하는 이유다.

#### 6. 기본값으로 바꾸기, 아예 막기: SET DEFAULT와 RESTRICT
SET DEFAULT는 외래키를 비우는 대신 그 컬럼의 DEFAULT 값으로 바꾼다. RESTRICT는 반대로 참조하는 자식이 있으면 부모 삭제 자체를 거부한다. 아래는 두 가지를 한 번에 돌린 것이다. 사원4의 부서번호 DEFAULT는 0(미배정)이다.

```sql
CREATE TABLE 부서4(부서번호 INT PRIMARY KEY, 부서명 TEXT);
INSERT INTO 부서4 VALUES (0,'미배정'),(10,'개발'),(20,'영업');
CREATE TABLE 사원4(사번 INT PRIMARY KEY,
  부서번호 INT DEFAULT 0 REFERENCES 부서4(부서번호) ON DELETE SET DEFAULT);
INSERT INTO 사원4 VALUES (1,10),(2,20);
DELETE FROM 부서4 WHERE 부서번호 = 10;
SELECT * FROM 사원4;

CREATE TABLE 사원5(사번 INT PRIMARY KEY,
  부서번호 INT REFERENCES 부서4(부서번호) ON DELETE RESTRICT);
INSERT INTO 사원5 VALUES (1,20);
DELETE FROM 부서4 WHERE 부서번호 = 20;   -- 사원5가 참조 중
```
출력: `1 | 0`, `2 | 20` (외래키가 DEFAULT 값 0으로 바뀌었다) / 마지막 DELETE는 `FOREIGN KEY constraint failed`로 거부된다

옵션을 아예 안 쓰면 어떻게 될까? 표에서 본 것처럼 NO ACTION과 같아서, 참조하는 자식이 있으면 부모 삭제가 거부된다. 자식이 없는 부모는 그냥 지워진다.

```sql
CREATE TABLE 부서6(부서번호 INT PRIMARY KEY, 부서명 TEXT);
CREATE TABLE 사원6(사번 INT PRIMARY KEY, 부서번호 INT REFERENCES 부서6(부서번호));
INSERT INTO 부서6 VALUES (10,'개발'),(20,'영업');
INSERT INTO 사원6 VALUES (1,10);
DELETE FROM 부서6 WHERE 부서번호 = 10;   -- 사원6이 참조 중
DELETE FROM 부서6 WHERE 부서번호 = 20;   -- 참조하는 사원 없음
SELECT * FROM 부서6;
```
출력: 첫 DELETE는 `FOREIGN KEY constraint failed`로 거부되고, 두 번째 DELETE는 수행된다.

| 부서번호 | 부서명 |
|---|---|
| 10 | 개발 |

정리하자면 부모 행을 지울 때 자식이 지워지는 건 CASCADE, 남는 건 나머지 전부다. 그중 SET NULL과 SET DEFAULT는 외래키 값이 바뀌고, RESTRICT·NO ACTION은 부모 삭제가 실패한다.

### 기출에서 이렇게 나왔다

- 26-1, 22-3, 20-1: DISTINCT가 있는 문과 없는 문 3개의 결과 행 수 → **200 / 3 / 1** (22-3은 **90 / 3 / 1**). 같은 유형이 세 번 나왔다
- 23-3: UNION 뒤 ORDER BY DESC 결과 표 6칸 → 첫 칸은 헤더 **C1**, 이어서 **8, 5, 4, 2, 1**
- 22-3: ON DELETE CASCADE가 걸린 테이블에서 부모 삭제 뒤 COUNT와 COUNT DISTINCT → **3 / 4**

### 답 쓸 때

UNION 결과를 쓸 때는 중복부터 지우고 그다음에 정렬한다. 결과 표의 빈칸 수가 값 개수보다 하나 많다면 첫 칸은 값이 아니라 열 이름 자리다. 이때 열 이름은 첫 SELECT의 열 이름을 쓴다. CASCADE 문제는 "부모에서 지운 값을 외래키로 가진 자식 행"을 표에서 먼저 지워 놓고 센다.

## 키워드 빈칸: INSERT·UPDATE·DELETE
출제: 24-2, 23-2, 23-1, 21-2, 20-3
형태: DML 문장의 예약어 빈칸

### 개념

SELECT가 데이터를 보기만 한다면, INSERT·UPDATE·DELETE는 데이터를 실제로 넣고 고치고 지운다. 문장마다 짝이 되는 예약어가 정해져 있어서 빈칸 문제는 대부분 그 짝을 묻는다.

| 문장 | 완성 문장 | 주의 |
|---|---|---|
| 행 추가 | `INSERT INTO 교수(교수번호, 이름, 나이) VALUES (1, '김교수', 45);` | 컬럼 목록을 생략하면 모든 컬럼을 순서대로 |
| 조회 결과 추가 | `INSERT INTO 교수백업 SELECT 교수번호, 이름 FROM 교수 WHERE 나이 >= 46;` | VALUES 대신 SELECT. 괄호 없음 |
| 수정 | `UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1;` | WHERE가 없으면 전체 행이 바뀐다 |
| 삭제 | `DELETE FROM 교수 WHERE 교수번호 = 3;` | WHERE가 없으면 전체 행 삭제. 테이블은 남는다 |

여기서 DELETE, DROP, TRUNCATE를 혼동하기 쉽다. DELETE는 행만 지우는 DML이다. 로그를 남기기 때문에 ROLLBACK으로 되돌릴 수 있다. 반면 DROP은 테이블 자체를 없애는 DDL이다. 애매한 건 TRUNCATE인데, 행을 모두 지우지만 분류는 DDL이다.

#### 1. 행 넣기: INSERT INTO ... VALUES
교수 테이블을 만들고 세 명을 넣어 보자. 첫 번째는 컬럼 목록을 일부만, 두 번째는 컬럼 목록 없이, 세 번째는 교수번호와 이름만 넣는다.

```sql
CREATE TABLE 교수(교수번호 INT PRIMARY KEY, 이름 TEXT NOT NULL, 이메일 TEXT UNIQUE,
  나이 INT CHECK (나이 >= 20), 학과코드 TEXT DEFAULT 'D1');
INSERT INTO 교수(교수번호, 이름, 이메일, 나이) VALUES (1,'김교수','kim@x.kr',45);
INSERT INTO 교수 VALUES (2,'이교수','lee@x.kr',50,'D2');
INSERT INTO 교수(교수번호, 이름) VALUES (3,'박교수');
SELECT * FROM 교수;
```
출력: `1 김교수 kim@x.kr 45 D1`, `2 이교수 lee@x.kr 50 D2`, `3 박교수 NULL NULL D1` (안 넣은 컬럼은 NULL, DEFAULT가 있으면 그 값)

박교수는 이메일·나이를 안 넣었으니 NULL이고, 학과코드는 DEFAULT 'D1'이 들어갔다. 컬럼 목록을 생략한 이교수는 다섯 컬럼 값을 순서대로 다 적었다. 그럼 컬럼 목록 없이 값을 두 개만 주면 어떻게 될까?

```sql
INSERT INTO 교수 VALUES (4,'최교수');
```

sqlite는 `table 교수 has 5 columns but 2 values were supplied` 오류를 낸다. 컬럼 목록을 생략하면 모든 컬럼을 순서대로 채워야 하기 때문이다.

#### 2. 고치고 지우기: UPDATE ... SET, DELETE FROM
다음은 1번 교수의 나이를 46으로 고치고 3번 교수를 지우는 문장이다.

```sql
UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1;
SELECT 교수번호, 나이 FROM 교수 WHERE 교수번호 = 1;
DELETE FROM 교수 WHERE 교수번호 = 3;
SELECT COUNT(*) FROM 교수;
```
출력: `1 | 46`, `2`

두 문장 모두 WHERE로 대상 행을 하나로 좁혔기 때문에 1번 교수만 바뀌고 3번 교수만 지워졌다. WHERE가 없으면 어떻게 되는지는 4단계에서 본다.

#### 3. 조회 결과를 통째로 넣기: INSERT INTO ... SELECT
값을 하나씩 적는 대신 다른 테이블을 조회한 결과를 그대로 넣을 수도 있다. 이때는 VALUES 자리에 SELECT가 온다.

```sql
CREATE TABLE 교수백업(교수번호 INT, 이름 TEXT);
INSERT INTO 교수백업 SELECT 교수번호, 이름 FROM 교수 WHERE 나이 >= 46;
SELECT * FROM 교수백업;
```
출력: `1 김교수`, `2 이교수`

나이가 46 이상인 두 명(46으로 고친 김교수, 50인 이교수)이 한 번에 들어갔다. `VALUES (SELECT ...)`처럼 쓰지 않고 SELECT를 괄호 없이 바로 쓴다는 점을 기억해 두자.

#### 4. WHERE가 없으면, 그리고 DELETE와 DROP
UPDATE와 DELETE에서 WHERE를 빼면 테이블의 모든 행이 대상이 된다. 교수백업으로 확인한다.

```sql
UPDATE 교수백업 SET 이름 = '무명';
SELECT * FROM 교수백업;
DELETE FROM 교수백업;
SELECT COUNT(*) FROM 교수백업;
```
출력:

| 교수번호 | 이름 |
|---|---|
| 1 | 무명 |
| 2 | 무명 |

```
0
```

UPDATE는 두 행의 이름을 모두 바꿨고, DELETE는 모든 행을 지웠다. 그런데 `COUNT(*)`가 0으로 나온다는 건 테이블 자체는 아직 남아 있다는 뜻이다. 테이블까지 없애는 건 DROP이다.

```sql
DROP TABLE 교수백업;
SELECT COUNT(*) FROM 교수백업;
```

이번에는 `no such table: 교수백업` 오류가 난다. 위에서 말한 차이가 바로 이것이다. DELETE는 행만 지우는 DML이고, DROP은 테이블을 없애는 DDL이다. TRUNCATE는 sqlite에 없어 실행하지 않았다.

### 기출에서 이렇게 나왔다

- 24-2: INSERT ... VALUES / INSERT ... SELECT ... FROM / UPDATE ... SET 빈칸 한 문제
- 23-2: `INSERT INTO ... VALUES` 빈칸
- 23-1, 20-3: `DELETE FROM ... WHERE` 빈칸 (같은 유형)
- 21-2: `UPDATE ... SET` 빈칸

### 답 쓸 때

INSERT는 `INTO`, UPDATE는 `SET`, DELETE는 `FROM`이 각각 짝이라고 외워 두면 된다. 다만 `INSERT INTO ... SELECT`처럼 조회 결과를 넣을 때는 VALUES를 쓰지 않는다.

## 키워드 빈칸: DDL (CREATE·ALTER·DROP과 제약조건)
출제: 26-2, 26-1, 23-2, 20-3, 20-2
형태: CREATE TABLE 제약조건 빈칸 / ALTER·INDEX·VIEW·DROP 문장 빈칸

### 개념

DML이 행을 다룬다면 DDL은 테이블이라는 그릇 자체를 만들고(CREATE), 고치고(ALTER), 없앤다(DROP). 그릇을 만들 때 "이 칸에는 어떤 값만 들어올 수 있다"는 규칙을 같이 걸어 두는데, 이것이 제약조건이다. 시험은 이 제약조건 이름과 ALTER·DROP 문장의 예약어를 빈칸으로 낸다.

#### 1. 테이블 만들며 제약조건 걸기: CREATE TABLE
아래 교수 테이블에는 제약조건이 전부 걸려 있다. 주석에 적은 것처럼 제약조건마다 지키는 무결성이 다르다.

```sql
PRAGMA foreign_keys = ON;
CREATE TABLE 학과(학과코드 TEXT PRIMARY KEY, 학과명 TEXT);
INSERT INTO 학과 VALUES ('D1','컴공'),('D2','전자');

CREATE TABLE 교수(
  교수번호 INT PRIMARY KEY,                 -- 개체 무결성: NULL·중복 불가
  이름 TEXT NOT NULL,
  이메일 TEXT UNIQUE,                       -- 중복 불가, NULL은 허용
  나이 INT CHECK (나이 >= 20),              -- 도메인 무결성
  학과코드 TEXT DEFAULT 'D1',
  CONSTRAINT fk_학과 FOREIGN KEY (학과코드)
    REFERENCES 학과(학과코드) ON DELETE SET NULL   -- 참조 무결성
);
INSERT INTO 교수(교수번호, 이름, 이메일, 나이) VALUES (1,'김교수','kim@x.kr',45);
INSERT INTO 교수 VALUES (2,'이교수','lee@x.kr',50,'D2');
```

위에서부터 읽으면 PRIMARY KEY는 NULL도 중복도 안 되고, NOT NULL은 비울 수 없고, UNIQUE는 중복만 막고 NULL은 허용한다. CHECK는 값이 조건식을 만족해야 하고, DEFAULT는 값을 안 주면 넣을 기본값이다. 마지막 줄의 외래키는 `CONSTRAINT 이름 FOREIGN KEY (컬럼) REFERENCES 부모테이블(컬럼)` 순서로 쓰고, 26-1은 이 문장을 5칸 빈칸으로 냈다. 제약조건마다 실제로 어떤 값을 막는지는 3단계에서 하나씩 어겨 보며 확인한다.

#### 2. 만든 뒤 고치고 지우기: ALTER·INDEX·VIEW·DROP
테이블을 만든 뒤에 컬럼을 더하거나 빼고, 인덱스와 뷰를 만들고, 다 쓴 객체를 지우는 문장은 아래와 같다.

| 문장 | 완성 문장 |
|---|---|
| 컬럼 추가 | `ALTER TABLE 교수 ADD 전화 TEXT;` |
| 컬럼 변경·삭제 | `ALTER TABLE 교수 ALTER 나이 SET DEFAULT 30;` (표준 SQL, 기본값 변경) / 자료형 변경은 Oracle·MySQL `ALTER TABLE 교수 MODIFY 나이 INT;`, 표준 SQL `ALTER TABLE 교수 ALTER 나이 SET DATA TYPE INT;` / `ALTER TABLE 교수 DROP COLUMN 전화;` (DROP COLUMN은 sqlite에서, ALTER는 PostgreSQL에서 실행 확인. MODIFY는 실행 확인 안 함) |
| 인덱스 | `CREATE INDEX idx_교수_이름 ON 교수(이름);` |
| 뷰 | `CREATE VIEW 고령교수 AS SELECT 이름, 나이 FROM 교수 WHERE 나이 >= 45;` |
| 도메인 | `CREATE DOMAIN 성별 CHAR(1) DEFAULT '남' CONSTRAINT 성별제약 CHECK (VALUE IN ('남', '여'));` (표준 문법, sqlite 미지원. PostgreSQL에서 실행 확인) |
| 삭제 | `DROP TABLE 교수 CASCADE;` / `DROP VIEW 고령교수;` / `DROP INDEX idx_교수_이름;` |

DROP 뒤에 붙는 옵션도 짚고 가자. **DROP ... CASCADE**는 지우려는 객체만이 아니라 그 객체를 참조하는 다른 객체(뷰·외래키)까지 함께 삭제한다. 반대로 **RESTRICT**는 참조하는 객체가 있으면 삭제를 막는다. 참고로 sqlite는 CASCADE 옵션을 받지 않아서 아래에서는 옵션 없이 실행했고, `DROP TABLE ... CASCADE`의 동작은 PostgreSQL에서 실행해 확인했다.

#### 3. 제약조건을 어기면
그럼 제약조건을 어기면 실제로 어떤 오류가 날까? 위 교수 테이블에 1·2번 교수가 들어 있고 학과는 D1·D2만 있는 상태에서 하나씩 어겨 보면 이렇다.

```sql
INSERT INTO 교수(교수번호, 이름, 나이) VALUES (4,'최교수',19);         -- CHECK 위반
INSERT INTO 교수(교수번호, 이름, 이메일) VALUES (5,'정교수','kim@x.kr'); -- UNIQUE 위반
INSERT INTO 교수(교수번호, 이름, 학과코드) VALUES (6,'한교수','D9');     -- FOREIGN KEY 위반
INSERT INTO 교수(교수번호, 이름) VALUES (1,'중복');                     -- PRIMARY KEY 위반
INSERT INTO 교수(교수번호) VALUES (7);                                  -- NOT NULL 위반
```
출력: 다섯 문장 모두 오류로 거부된다 (`CHECK constraint failed`, `UNIQUE constraint failed: 교수.이메일`, `FOREIGN KEY constraint failed`, `UNIQUE constraint failed: 교수.교수번호`, `NOT NULL constraint failed: 교수.이름`) (참고: sqlite는 호환성 때문에 `INT PRIMARY KEY` 컬럼에 NULL을 넣어도 거부하지 않는다. 표준 SQL과 다른 DBMS에서는 기본키 NULL이 개체 무결성 위반으로 거부된다.)

메시지를 하나씩 맞춰 보면, 19세는 `나이 >= 20` 조건에 걸리고, kim@x.kr는 김교수가 이미 쓰고 있는 이메일이다. D9는 학과 테이블에 없는 코드라 참조할 부모가 없다. 기본키 위반은 sqlite가 `UNIQUE`라는 이름으로 알려 주는데, 이미 있는 교수번호 1을 또 넣으려 했기 때문이다. 마지막 문장은 이름을 비워서 NOT NULL에 걸렸다.

#### 4. 실제로 고치고 지워 보기
이번에는 학과 D2를 지워 ON DELETE SET NULL을 확인하고, 2단계 표의 문장들을 차례로 실행한다.

```sql
DELETE FROM 학과 WHERE 학과코드 = 'D2';     -- 이교수가 참조 중, ON DELETE SET NULL
SELECT 교수번호, 학과코드 FROM 교수;
ALTER TABLE 교수 ADD 전화 TEXT;
CREATE INDEX idx_교수_이름 ON 교수(이름);
CREATE VIEW 고령교수 AS SELECT 이름, 나이 FROM 교수 WHERE 나이 >= 45;
SELECT * FROM 고령교수;
DROP VIEW 고령교수;
DROP INDEX idx_교수_이름;
```
출력: `1 | D1`, `2 | NULL` / 뷰 조회 `김교수 45`, `이교수 50` / 나머지는 오류 없이 수행

이교수의 학과코드가 NULL이 된 건 외래키에 ON DELETE SET NULL을 걸어 뒀기 때문이다. ADD로 붙인 전화 컬럼은 DROP COLUMN으로 다시 뺄 수 있다.

```sql
ALTER TABLE 교수 DROP COLUMN 전화;
SELECT * FROM 교수;
SELECT * FROM 고령교수;
```
출력:

| 교수번호 | 이름 | 이메일 | 나이 | 학과코드 |
|---|---|---|---|---|
| 1 | 김교수 | kim@x.kr | 45 | D1 |
| 2 | 이교수 | lee@x.kr | 50 | NULL |

마지막 줄은 `no such table: 고령교수` 오류다. 앞에서 DROP VIEW로 뷰를 지웠으니 더는 조회할 수 없다.

#### 5. 값의 범위를 이름으로 만들기: CREATE DOMAIN
도메인은 "이 자료형에 이런 기본값과 CHECK 조건"을 묶어 이름을 붙여 둔 것이다. 2단계 표의 문장을 조각내면 세 부분이다.

```sql
CREATE DOMAIN 성별 CHAR(1)                           -- 이름과 자료형
  DEFAULT '남'                                       -- 기본값
  CONSTRAINT 성별제약 CHECK (VALUE IN ('남', '여'));  -- 허용 값
```

sqlite는 도메인을 지원하지 않아 여기서는 실행 결과를 붙이지 않았다(표에 적은 대로 PostgreSQL에서 실행을 확인한 문장이다). 조건식 안의 `VALUE`는 이 도메인에 들어오는 값 자체를 가리킨다. 26-2에서 비운 칸이 바로 CHECK 자리였다. 테이블의 `CHECK (나이 >= 20)`과 같은 예약어를 쓴다고 기억하면 된다.

### 기출에서 이렇게 나왔다

- 26-2: `CREATE DOMAIN ... ( ) (VALUE IN ...)` → **CHECK**
- 26-1: `CONSTRAINT ... FOREIGN KEY ... REFERENCES` 5칸
- 23-2: `DROP VIEW ... ( )` 참조 객체까지 삭제 → **CASCADE**
- 20-3: 컬럼 추가 → **ALTER TABLE ... ADD**
- 20-2: 인덱스 생성 → **CREATE INDEX ... ON**

### 답 쓸 때

외래키 제약은 `CONSTRAINT 이름 FOREIGN KEY (컬럼) REFERENCES 부모테이블(컬럼)` 순서로 쓴다. 헷갈리는 건 REFERENCES 뒤인데, 여기에는 테이블명이 오고 그 괄호 안에 부모의 컬럼이 온다. CASCADE와 RESTRICT도 혼동하지 않도록 한다. 참조하는 객체까지 같이 지우는 쪽이 CASCADE, 삭제를 막는 쪽이 RESTRICT다.

## 결과 예측: 집계 함수와 NULL, AND/OR 우선순위
출제: 25-3, 24-1, 22-2, 21-1
형태: 표 + 쿼리 → COUNT 값

### 개념

표에 NULL이 한두 칸 끼어 있으면 COUNT 값이 달라진다. 집계 함수는 NULL을 건너뛰고, NULL과 비교한 조건은 참도 거짓도 아닌 UNKNOWN이 되기 때문이다. 여기에 AND가 OR보다 먼저 묶인다는 규칙까지 겹치면 실수하기 쉽다. 규칙을 표로 먼저 보고, 학생 테이블로 하나씩 확인하자.

| 함정 | 규칙 |
|---|---|
| `COUNT(*)` | NULL이 있어도 행을 다 센다 |
| `COUNT(컬럼)` | 그 컬럼이 **NULL인 행은 빼고** 센다 |
| `SUM·AVG·MAX·MIN` | NULL을 무시한다. AVG는 NULL이 아닌 값만으로 평균 |
| `AND`와 `OR` | **AND가 먼저** 묶인다. `A OR B AND C`는 `A OR (B AND C)` |
| `NULL` 비교 | `= NULL`, `<> NULL`의 결과는 거짓이 아니라 UNKNOWN이라 WHERE를 통과하는 행이 없다(`NOT (컬럼 = NULL)`도 0행). NULL 검사는 `IS NULL`·`IS NOT NULL`로만 한다. `<>` 조건 하나만 있으면 NULL 행은 빠진다(OR로 다른 참 조건이 붙으면 통과한다) |
| NULL이 낀 OR·AND | NULL과의 비교(`=`, `IN`, `>` 등)는 UNKNOWN이지만 그 행을 바로 버리지 않는다. **UNKNOWN OR 참 = 참**, UNKNOWN OR 거짓 = UNKNOWN, UNKNOWN AND 참 = UNKNOWN, UNKNOWN AND 거짓 = 거짓이다. WHERE는 최종 결과가 참인 행만 남긴다 |

#### 1. 무엇을 세나: COUNT(*)와 COUNT(컬럼)
학생 테이블은 5행이고, 그중 이영희·최지우 두 명의 점수가 NULL이다. 같은 테이블을 세 가지 방법으로 세면 결과가 다 다르다.

```sql
CREATE TABLE 학생(학번 INT, 이름 TEXT, 학과 TEXT, 점수 INT);
INSERT INTO 학생 VALUES (1,'김철수','컴공',80),(2,'이영희','컴공',NULL),
  (3,'박민수','전자',70),(4,'최지우','전자',NULL),(5,'정하늘','기계',90);

SELECT COUNT(*), COUNT(점수), COUNT(학과) FROM 학생;
```
출력: `5 | 3 | 5`

`COUNT(*)`는 행 자체를 세니 5다. `COUNT(점수)`는 점수 칸이 NULL인 두 행을 빼고 세서 3이 되고, 학과에는 NULL이 없으니 `COUNT(학과)`는 다시 5다. 괄호 안에 무엇을 썼는지가 답을 가른다.

#### 2. 행을 고른 다음에 센다: WHERE + COUNT(컬럼)
25-3·22-2 기출은 WHERE로 행을 먼저 고른 뒤 `COUNT(컬럼)`으로 세는 문제였다. 같은 모양의 아래 쿼리도 WHERE를 통과하는 행부터 확인한다.

```sql
SELECT * FROM 학생 WHERE 학과 IN ('컴공','전자') OR 점수 >= 90;
```
출력:

| 학번 | 이름 | 학과 | 점수 |
|---|---|---|---|
| 1 | 김철수 | 컴공 | 80 |
| 2 | 이영희 | 컴공 | NULL |
| 3 | 박민수 | 전자 | 70 |
| 4 | 최지우 | 전자 | NULL |
| 5 | 정하늘 | 기계 | 90 |

컴공·전자 네 명은 IN 조건으로, 기계과 정하늘은 `점수 >= 90`으로 통과해 5행이 다 남았다. 이제 이 5행을 `COUNT(점수)`로 센다.

```sql
SELECT COUNT(점수) FROM 학생 WHERE 학과 IN ('컴공','전자') OR 점수 >= 90;
```
출력: `3` (조건을 만족하는 행은 5개지만 점수가 NULL인 2행은 COUNT(점수)에서 빠진다)

#### 3. SUM·AVG·MAX·MIN도 NULL을 건너뛴다
COUNT만 그런 게 아니다. 나머지 집계 함수도 NULL 칸은 없는 셈 치고 계산한다.

```sql
SELECT SUM(점수), AVG(점수), MAX(점수), MIN(점수) FROM 학생;
```
출력: `240 | 80.0 | 90 | 70` (AVG는 240/3이지 240/5가 아니다)

합계 240은 80 + 70 + 90이다. 평균을 낼 때도 NULL이 아닌 세 값만 쓰기 때문에 240을 3으로 나눈 80.0이 나온다. NULL을 0으로 쳤다면 48.0이 되었을 것이다.

#### 4. AND가 OR보다 먼저: 괄호가 있을 때와 없을 때
같은 조건에 괄호만 다르게 친 두 쿼리다.

```sql
SELECT COUNT(*) FROM 학생 WHERE 학과='컴공' OR 학과='전자' AND 점수 >= 70;
SELECT COUNT(*) FROM 학생 WHERE (학과='컴공' OR 학과='전자') AND 점수 >= 70;
```
출력: `3`, `2` (괄호가 없으면 "컴공 전부" + "전자 중 70 이상" = 2 + 1. 괄호가 있으면 두 학과 중 70 이상인 2명)

어떤 행이 남았는지 직접 보면 차이가 더 분명하다.

```sql
SELECT 이름, 학과, 점수 FROM 학생 WHERE 학과='컴공' OR 학과='전자' AND 점수 >= 70;
SELECT 이름, 학과, 점수 FROM 학생 WHERE (학과='컴공' OR 학과='전자') AND 점수 >= 70;
```
출력:

| 이름 | 학과 | 점수 |
|---|---|---|
| 김철수 | 컴공 | 80 |
| 이영희 | 컴공 | NULL |
| 박민수 | 전자 | 70 |

| 이름 | 학과 | 점수 |
|---|---|---|
| 김철수 | 컴공 | 80 |
| 박민수 | 전자 | 70 |

괄호가 없는 첫 쿼리는 `학과='컴공' OR (학과='전자' AND 점수 >= 70)`으로 읽힌다. 그래서 점수가 NULL인 이영희도 컴공이라는 이유만으로 들어왔다. 괄호를 친 두 번째 쿼리는 모든 행에 `점수 >= 70`이 걸리니 이영희가 빠진다.

#### 5. NULL과 비교하면 UNKNOWN
이번에는 사원 테이블에서 "10번 부서가 아닌 사람"을 센다. 부서번호가 NULL인 유관순은 10번 부서가 아니니 들어갈 것 같다.

```sql
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);
SELECT COUNT(*) FROM 사원 WHERE 부서번호 <> 10;
```
출력: `2` (부서번호가 NULL인 유관순은 `<> 10`에도 잡히지 않는다)

결과는 이순신·김유신 두 명이다. `NULL <> 10`은 거짓이 아니라 UNKNOWN이고, WHERE는 참인 행만 남기기 때문에 유관순이 빠졌다. 조건을 뒤집어 `NOT (부서번호 = NULL)`로 써도 마찬가지다.

```sql
SELECT COUNT(*) FROM 사원 WHERE NOT (부서번호 = NULL);
```
출력:
```
0
```

UNKNOWN을 NOT으로 뒤집어도 여전히 UNKNOWN이라 한 행도 통과하지 못한다. 그런데 UNKNOWN이 나왔다고 그 행이 무조건 버려지는 건 아니다. 다른 조건과 OR로 붙어 있으면 이야기가 달라진다.

```sql
SELECT 이름, 부서번호, 급여 FROM 사원 WHERE 부서번호 <> 10 OR 급여 >= 2000;
SELECT 이름, 부서번호, 급여 FROM 사원 WHERE 부서번호 <> 10 AND 급여 >= 2000;
```
출력:

| 이름 | 부서번호 | 급여 |
|---|---|---|
| 홍길동 | 10 | 3000 |
| 이순신 | 20 | 1000 |
| 강감찬 | 10 | 4000 |
| 유관순 | NULL | 2000 |
| 김유신 | 20 | 2500 |

| 이름 | 부서번호 | 급여 |
|---|---|---|
| 김유신 | 20 | 2500 |

유관순 행에서 `부서번호 <> 10`은 UNKNOWN, `급여 >= 2000`은 참이다. OR에서는 UNKNOWN OR 참 = 참이라 첫 쿼리에서 살아남고, AND에서는 UNKNOWN AND 참 = UNKNOWN이라 두 번째 쿼리에서 빠진다. 25-3 기출의 (NULL,5) 행이 바로 이 OR 경우다.

### 기출에서 이렇게 나왔다

- 25-3, 22-2: IN과 OR이 섞인 WHERE 뒤 `COUNT(컬럼)`, 그 컬럼에 NULL 포함 → **4** (25-3), **3** (22-2). 같은 유형
- 25-3 풀이: 행마다 따져 보면 (2,NULL)·(3,6)·(2,3)은 `C1 IN(2,3)`이 참이고, (4,5)는 `C2 IN(3,5)`가 참이다. 가장 헷갈리는 건 (NULL,5)다. `C1 IN(2,3)`은 UNKNOWN이지만 OR로 붙은 `C2 IN(3,5)`가 참이라서 통과한다. 이렇게 통과한 5행 중 C2가 NULL인 (2,NULL)은 `COUNT(C2)`가 빼고 세므로 답은 **4**다. 여기서 (NULL,5)까지 빼 버리면 3이라는 오답이 나온다.
- 24-1, 21-1: 괄호 없는 `OR ... AND` 조건의 COUNT → **1** (같은 유형)

### 답 쓸 때

쿼리는 두 단계로 읽는다. 먼저 WHERE 조건으로 행을 고르는데, 이때 AND가 먼저 묶인다는 걸 잊으면 안 된다. 그다음에 COUNT 괄호 안을 본다. `COUNT(*)`가 아니라면 고른 행에서 해당 컬럼이 NULL인 행을 지운 뒤 센다.

## 결과 예측: 조인
출제: 26-2, 25-3, 25-1, 21-3
형태: 표 2개 + 조인 쿼리 → COUNT 또는 결과 표(헤더 포함)

### 개념

조인은 두 테이블을 옆으로 붙여 한 결과로 만드는 것이다. 붙이는 기준이 같아도 "짝이 없는 행을 남기느냐, 버리느냐"에 따라 종류가 갈리고, 그래서 결과 행 수가 달라진다. 종류별 차이는 아래 표와 같다.

| 조인 | 결과 행 | 함정 |
|---|---|---|
| 묵시적 조인 `FROM A, B WHERE A.k = B.k` | INNER JOIN과 같다 | WHERE에 조인 조건과 검색 조건이 섞여 있다 |
| `INNER JOIN ... ON` | 양쪽에 짝이 있는 행만 | 외래키가 NULL인 행은 빠진다 |
| `LEFT OUTER JOIN` | 왼쪽은 다 남기고 짝 없는 오른쪽은 NULL | |
| `RIGHT OUTER JOIN` | 오른쪽은 다 남기고 짝 없는 왼쪽은 NULL | `WHERE 왼쪽.키 IS NULL`이면 "짝 없는 오른쪽 행"만 |
| `FULL OUTER JOIN` | 양쪽 다 남긴다 | INNER 행 수 + 왼쪽만 + 오른쪽만 |
| `CROSS JOIN` | 모든 조합. 행 수 = m × n | WHERE의 LIKE로 걸러진 조합 수를 세야 한다 |

아래부터는 같은 두 테이블에 조인 종류만 바꿔 건 결과를 비교한다. 부서는 4개이고 사원은 5명이다.

| 부서번호 | 부서명 |
|---|---|
| 10 | 개발 |
| 20 | 영업 |
| 30 | 인사 |
| 40 | 총무 |

| 사번 | 이름 | 부서번호 | 급여 |
|---|---|---|---|
| 101 | 홍길동 | 10 | 3000 |
| 102 | 이순신 | 20 | 1000 |
| 103 | 강감찬 | 10 | 4000 |
| 104 | 유관순 | NULL | 2000 |
| 105 | 김유신 | 20 | 2500 |

눈여겨볼 곳은 두 군데다. 사원 쪽의 유관순은 부서번호가 NULL이라 짝이 되는 부서가 없고, 부서 쪽의 인사(30)·총무(40)는 소속 사원이 없다.

#### 1. 묵시적 조인: FROM A, B WHERE
JOIN이라는 단어 없이 FROM에 테이블 두 개를 쉼표로 나열하고, 조인 조건을 WHERE에 적는 방식이다. 아래는 25-1과 같은 형태로 "영업 부서에서 급여가 2000 미만인 사원"을 찾는 쿼리다.

```sql
CREATE TABLE 부서(부서번호 INT, 부서명 TEXT);
INSERT INTO 부서 VALUES (10,'개발'),(20,'영업'),(30,'인사'),(40,'총무');
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT 이름, 급여 FROM 사원, 부서
WHERE 사원.부서번호 = 부서.부서번호 AND 부서명 = '영업' AND 급여 < 2000;
```
출력:

| 이름 | 급여 |
|---|---|
| 이순신 | 1000 |

WHERE에 조건이 세 개 있는데, 성격이 다르다. `사원.부서번호 = 부서.부서번호`는 두 테이블을 잇는 조인 조건이고, `부서명 = '영업'`과 `급여 < 2000`은 행을 거르는 검색 조건이다. 조인 조건으로 짝을 지은 4행 중에서 영업 부서는 이순신·김유신이고, 그중 급여가 2000 미만인 사람은 이순신 하나다. 결과 표를 쓸 때는 SELECT에 적은 이름·급여가 헤더가 된다.

#### 2. 짝이 있는 행만: INNER JOIN
같은 조인 조건을 `INNER JOIN ... ON`으로 쓰면 묵시적 조인과 결과가 같다. 검색 조건 없이 짝지어진 행을 전부 찍으면 이렇다.

```sql
SELECT 사원.이름, 부서.부서명 FROM 사원 INNER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력:

| 이름 | 부서명 |
|---|---|
| 홍길동 | 개발 |
| 이순신 | 영업 |
| 강감찬 | 개발 |
| 김유신 | 영업 |

4행이 나왔다. 유관순은 부서번호가 NULL이라 어느 부서와도 짝이 맞지 않아 빠졌고, 사원이 없는 인사·총무도 빠졌다.

#### 3. 왼쪽은 다 남기기: LEFT OUTER JOIN
그럼 짝이 없어도 사원은 전부 보고 싶다면? FROM에 먼저 적은 왼쪽 테이블(사원)을 다 살리는 LEFT OUTER JOIN을 쓴다.

```sql
SELECT 사원.이름, 부서.부서명 FROM 사원 LEFT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력:

| 이름 | 부서명 |
|---|---|
| 홍길동 | 개발 |
| 이순신 | 영업 |
| 강감찬 | 개발 |
| 유관순 | NULL |
| 김유신 | 영업 |

INNER 결과에 유관순 한 행이 더 붙었다. 짝이 될 부서가 없으니 오른쪽 칸(부서명)은 NULL로 채워진다.

#### 4. 양쪽 다 남기기: FULL OUTER JOIN
FULL OUTER JOIN은 왼쪽에서 짝 없는 행과 오른쪽에서 짝 없는 행을 모두 남긴다.

```sql
SELECT 사원.이름, 부서.부서명 FROM 사원 FULL OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력:

| 이름 | 부서명 |
|---|---|
| 홍길동 | 개발 |
| 이순신 | 영업 |
| 강감찬 | 개발 |
| 유관순 | NULL |
| 김유신 | 영업 |
| NULL | 인사 |
| NULL | 총무 |

LEFT 결과 5행에 사원 없는 부서 2행이 더해져 7행이다. 행 수만 물을 때는 INNER 행 수에 양쪽의 짝 없는 행을 더하면 된다.

```sql
SELECT COUNT(*) FROM 사원 INNER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
SELECT COUNT(*) FROM 사원 FULL OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력: `4`, `7` (INNER 4 + 부서 없는 사원 1 + 사원 없는 부서 2)

#### 5. 오른쪽은 다 남기기: RIGHT OUTER JOIN
RIGHT OUTER JOIN은 LEFT의 반대로, 뒤에 적은 오른쪽 테이블(부서)을 다 남긴다.

```sql
SELECT 부서.부서명, 사원.이름 FROM 사원 RIGHT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력: `개발 홍길동`, `영업 이순신`, `개발 강감찬`, `영업 김유신`, `인사 NULL`, `총무 NULL`

이번에는 유관순이 빠지고 인사·총무가 사원 칸이 NULL인 채로 남았다. 26-2는 여기에 `IS NULL` 조건을 하나 더 붙였다. 사원 쪽 칸이 NULL인 행, 즉 짝 없는 부서만 고르는 조건이다.

```sql
SELECT COUNT(*) FROM 사원 RIGHT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호
WHERE 사원.사번 IS NULL;
```
출력: `2` (사원이 한 명도 없는 부서 수)

COUNT 대신 부서명을 찍어 보면 그 두 부서가 누구인지 보인다.

```sql
SELECT 부서.부서명 FROM 사원 RIGHT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호 WHERE 사원.사번 IS NULL;
```
출력:
```
인사
총무
```

#### 6. 모든 조합: CROSS JOIN
CROSS JOIN은 조인 조건 없이 왼쪽 행 하나하나에 오른쪽 행을 전부 붙인다. 학생 5명과 과목 3개라면 몇 행이 될까?

```sql
CREATE TABLE 학생(학번 INT, 이름 TEXT, 학과 TEXT, 점수 INT);
INSERT INTO 학생 VALUES (1,'김철수','컴공',80),(2,'이영희','컴공',NULL),
  (3,'박민수','전자',70),(4,'최지우','전자',NULL),(5,'정하늘','기계',90);
CREATE TABLE 과목(과목코드 TEXT, 과목명 TEXT);
INSERT INTO 과목 VALUES ('C1','데이터베이스'),('C2','운영체제'),('C3','데이터통신');

SELECT COUNT(*) FROM 학생 CROSS JOIN 과목;
SELECT COUNT(*) FROM 학생 CROSS JOIN 과목 WHERE 과목명 LIKE '데이터%';
SELECT COUNT(*) FROM 학생 CROSS JOIN 과목 WHERE 이름 LIKE '%수' AND 과목명 LIKE '데이터%';
```
출력: `15`, `10`, `4` (5 × 3 = 15. 과목 쪽만 걸러지면 5 × 2 = 10. 양쪽 다 걸러지면 "수"로 끝나는 2명 × 2과목 = 4)

`과목명 LIKE '데이터%'`에 맞는 과목은 데이터베이스·데이터통신 둘이다. 이름이 "수"로 끝나는 학생은 몇 명일까?

```sql
SELECT 이름 FROM 학생 WHERE 이름 LIKE '%수';
```
출력:
```
김철수
박민수
```

두 명이다. 조건이 학생 쪽에 하나, 과목 쪽에 하나씩 따로 걸려 있으니 걸러진 행 수끼리 곱하면 2 × 2 = 4가 된다. 조건이 두 테이블을 함께 쓰는 경우는 곱셈이 통하지 않는데, 이건 아래 「답 쓸 때」의 21-3 풀이에서 다룬다.

### 기출에서 이렇게 나왔다

- 26-2: RIGHT OUTER JOIN 뒤 `IS NULL` 조건 COUNT → **2**
- 25-3, 21-3: CROSS JOIN 뒤 LIKE 조건 COUNT → **4** (25-3), **5** (21-3). 같은 유형
- 25-1: 묵시적 조인 + 조건 결과 표(헤더 포함) → 헤더 **이름, 급여**, 행 **이순신, 1000**

### 답 쓸 때

CROSS JOIN 문제는 WHERE 조건이 어디에 걸리는지부터 본다. 조건이 `과목명 LIKE '데이터%'`처럼 **한쪽 테이블에만** 걸리면 "조건에 맞는 왼쪽 행 수 × 조건에 맞는 오른쪽 행 수"로 세면 된다. 그런데 조건이 **두 테이블을 함께** 쓰면 곱하면 안 된다. `GAMJA.NAME LIKE A.RULE`처럼 패턴이 다른 테이블의 컬럼에 들어 있는 경우인데, 이때는 오른쪽 행(패턴)마다 맞는 왼쪽 행 수를 세어 더해야 한다. 21-3이 바로 이 경우다. `S%`에 맞는 건 SIGAMJA·SEAGAMJA 2개, `%A%`에는 세 이름이 모두 맞아 3개이니 합은 **5**다. 곱셈으로 풀면 3 × 2 = 6이 되어 틀린다.

```sql
CREATE TABLE GAMJA(NAME TEXT); INSERT INTO GAMJA VALUES ('SIGAMJA'),('WANGGAMJA'),('SEAGAMJA');
CREATE TABLE A(RULE TEXT); INSERT INTO A VALUES ('S%'),('%A%');
SELECT COUNT(*) CNT FROM GAMJA CROSS JOIN A WHERE GAMJA.NAME LIKE A.RULE;
```
출력:
```
5
```
결과 표를 요구하는 문제라면 헤더 이름은 SELECT 절에 적힌 그대로 쓴다.

## 결과 예측: 서브쿼리
출제: 26-2, 26-1, 24-3, 24-1
형태: 표 + 서브쿼리가 든 쿼리 → COUNT 또는 값

### 개념

서브쿼리는 쿼리 안에 괄호로 들어간 또 하나의 SELECT다. 풀 때 중요한 건 "안쪽을 언제, 몇 번 계산하느냐"이고, 이것은 서브쿼리가 무엇을 돌려주는지와 바깥 행을 참조하는지에 따라 정해진다.

| 종류 | 읽는 순서 |
|---|---|
| 단일 행 서브쿼리 `WHERE 급여 >= (SELECT AVG(급여) FROM 사원)` | 안쪽을 먼저 계산해 **값 하나**로 바꾼 뒤 바깥을 읽는다 |
| 다중 행 서브쿼리 `WHERE 학번 IN (SELECT ...)` | 안쪽을 **목록**으로 바꾼 뒤 바깥을 읽는다. ANY·ALL·EXISTS도 이 부류 |
| 상관 서브쿼리 `WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = S.부서번호)` | 안쪽이 바깥 행을 참조하므로 **바깥 행마다** 안쪽을 다시 계산한다 |
| 중첩 (서브쿼리 안에 GROUP BY·HAVING) | 가장 안쪽부터 결과 집합을 적어 두고 한 단계씩 바깥으로 |

예제는 조인 묶음과 같은 부서(4개)·사원(5명) 테이블과, 학생·수강 테이블을 쓴다.

#### 1. 값 하나로 바뀌는 서브쿼리: 단일 행
"평균 급여 이상인 사원"을 세려면 평균부터 알아야 한다. 안쪽 `SELECT AVG(급여) FROM 사원`을 먼저 따로 돌린 뒤, 그 값을 바깥에 넣는다.

```sql
CREATE TABLE 부서(부서번호 INT, 부서명 TEXT);
INSERT INTO 부서 VALUES (10,'개발'),(20,'영업'),(30,'인사'),(40,'총무');
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT AVG(급여) FROM 사원;
SELECT COUNT(*) FROM 사원 JOIN 부서 ON 사원.부서번호 = 부서.부서번호
WHERE 급여 >= (SELECT AVG(급여) FROM 사원);
```
출력: `2500.0`, `3` (평균 이상은 홍길동·강감찬·김유신 3명. 전체 평균은 조인과 무관하게 5명으로 계산한다)

안쪽이 2500.0 하나로 바뀌고 나면 바깥은 `WHERE 급여 >= 2500`과 다를 게 없다. 바깥에서는 조인 때문에 유관순이 빠지지만, 괄호 안의 평균은 사원 테이블 전체를 따로 계산한다는 점에 주의하자.

#### 2. 바깥 행마다 다시 계산: 상관 서브쿼리
이번에는 "자기 부서 평균보다 급여가 높은 사원"이다. 비교 기준이 사람마다 다르니 안쪽을 한 번만 계산해서는 안 된다. 아래는 부서별 평균과 상관 서브쿼리 결과를 함께 돌린 것이다.

```sql
SELECT 부서번호, AVG(급여) FROM 사원 GROUP BY 부서번호;
SELECT 이름, 부서번호, 급여 FROM 사원 S
WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = S.부서번호);
```
출력: 부서별 평균 `NULL 2000.0`, `10 3500.0`, `20 1750.0` / 상관 서브쿼리 결과 `강감찬 10 4000`, `김유신 20 2500` (각자 **자기 부서** 평균보다 높은 사람. 유관순은 부서번호가 NULL이라 `부서번호 = NULL` 비교가 거짓이 되어 평균이 NULL, 비교도 거짓)

안쪽의 `S.부서번호`가 바깥 행을 가리키는 부분이다. 바깥에서 홍길동 행을 읽을 때는 안쪽이 10번 부서 평균 3500.0이 되고, 3000은 그보다 작아 탈락한다. 강감찬 행에서도 기준은 같은 3500.0인데 4000이라 통과한다. 이순신·김유신 행에서는 기준이 20번 부서 평균 1750.0으로 바뀌어 이순신(1000)은 탈락, 김유신(2500)은 통과다. 이렇게 행마다 기준을 다시 구해 표 옆에 적어 가며 푸는 것이 상관 서브쿼리 문제의 정석이다.

#### 3. 목록과 비교: IN, NOT IN
서브쿼리가 값 여러 개를 돌려주면 그 목록과 비교한다. "C2 과목을 들은 학생"이라면 먼저 C2를 들은 학번 목록이 필요하다.

```sql
SELECT 학번 FROM 수강 WHERE 과목코드 = 'C2';
```
출력:
```
1
5
```

목록은 {1, 5}다. 바깥 쿼리는 학번이 이 목록에 들어 있는 학생을 고른다. NOT IN은 반대로, 수강 테이블에 한 번도 나오지 않은 학번을 고른다.

```sql
CREATE TABLE 학생(학번 INT, 이름 TEXT, 학과 TEXT, 점수 INT);
INSERT INTO 학생 VALUES (1,'김철수','컴공',80),(2,'이영희','컴공',NULL),
  (3,'박민수','전자',70),(4,'최지우','전자',NULL),(5,'정하늘','기계',90);
CREATE TABLE 수강(학번 INT, 과목코드 TEXT, 성적 TEXT);
INSERT INTO 수강 VALUES (1,'C1','A'),(1,'C2','B'),(2,'C1','A'),
  (3,'C1','A'),(3,'C3','C'),(5,'C2','A');

SELECT 이름 FROM 학생 WHERE 학번 IN (SELECT 학번 FROM 수강 WHERE 과목코드 = 'C2');
SELECT 이름 FROM 학생 WHERE 학번 NOT IN (SELECT 학번 FROM 수강);
```
출력: `김철수`, `정하늘` / `최지우`

수강 테이블의 학번은 1, 2, 3, 5뿐이라 NOT IN을 통과하는 건 학번 4인 최지우 하나다.

#### 4. 있는지만 보기: EXISTS
EXISTS는 값을 비교하지 않고, 안쪽이 행을 하나라도 돌려주는지만 본다. 보통 바깥 행을 참조하는 상관 서브쿼리 모양으로 쓴다.

```sql
SELECT 이름 FROM 학생
WHERE EXISTS (SELECT * FROM 수강 WHERE 수강.학번 = 학생.학번 AND 과목코드 = 'C1');
```
출력: `김철수`, `이영희`, `박민수`

학생 행마다 "이 학번으로 C1을 수강한 행이 있는가"를 묻는다. 학번 1·2·3은 C1 수강 행이 있어 참이고, 4·5는 없어 거짓이다. 안쪽에서 `SELECT *`를 쓰든 `SELECT 1`을 쓰든 결과가 같은 것도 값을 보지 않기 때문이다.

#### 5. NOT IN과 NULL: 결과가 0이 되는 함정
이번에는 "사원이 한 명도 없는 부서"를 센다. 조인 묶음에서 답이 인사·총무 2개라는 걸 이미 봤는데, NOT IN으로 쓰면 다른 답이 나온다. 먼저 안쪽 목록이다.

```sql
SELECT DISTINCT 부서번호 FROM 사원;
```
출력:
```
10
20
NULL
```

목록에 NULL이 섞여 있다. 이 상태로 NOT IN과 NOT EXISTS를 나란히 돌린 결과다.

```sql
SELECT COUNT(*) FROM 부서 WHERE 부서번호 NOT IN (SELECT 부서번호 FROM 사원);
SELECT COUNT(*) FROM 부서 D WHERE NOT EXISTS (SELECT 1 FROM 사원 S WHERE S.부서번호 = D.부서번호);
```
출력: `0`, `2` (사원 부서번호에 NULL(유관순)이 섞여 있으면 `NOT IN`은 `<> NULL` 비교가 UNKNOWN이 되어 어떤 행도 남지 않는다. NOT EXISTS는 NULL과 상관없이 사원 없는 부서 2개를 센다)

부서 30을 예로 들면 `30 NOT IN (10, 20, NULL)`은 `30 <> 10 AND 30 <> 20 AND 30 <> NULL`과 같다. 마지막 비교가 UNKNOWN이니 전체가 참이 될 수 없다. 반면 NOT EXISTS는 "부서번호가 30인 사원 행이 있는가"만 묻기 때문에 NULL 행은 상관이 없다.

#### 6. 빈 서브쿼리는 NULL을 돌려준다
여기서 많이들 놓치는 게 빈 서브쿼리다. 서브쿼리 안에서 집계할 행이 하나도 없으면 COUNT는 0을 돌려주지만, SUM·AVG·MAX·MIN은 0이 아니라 NULL을 돌려준다. 그러면 바깥의 `x > NULL`은 UNKNOWN이 되고, 그 행은 결과에서 빠진다.

사원이 없는 30번 부서로 확인하면 이렇다.

```sql
SELECT COUNT(급여), AVG(급여), MAX(급여) FROM 사원 WHERE 부서번호 = 30;
SELECT COUNT(*) FROM 사원 WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = 30);
```
출력:

| COUNT(급여) | AVG(급여) | MAX(급여) |
|---|---|---|
| 0 | NULL | NULL |

```
0
```

COUNT만 0이고 AVG와 MAX는 NULL이다. 그래서 바깥의 `급여 > NULL`은 사원 다섯 명 모두에게 UNKNOWN이 되어 한 명도 남지 않는다. 26-2 기출에서 x=10 행이 빠진 이유가 바로 이것이다.

### 기출에서 이렇게 나왔다

- 26-2: 상관 서브쿼리(AVG)와 IN이 섞인 COUNT → **3**. 상관 서브쿼리라서 A 행마다 안쪽을 다시 계산해야 한다. 먼저 x=10이면 `A2.x < 10`인 행이 없어 IN 목록이 비는데, 빈 집합의 AVG는 NULL이라 `10 > NULL`이 UNKNOWN이 되어 빠진다. 나머지는 x=20은 id {1} → AVG(5, 15) = 10, x=30은 id {1, 2} → AVG(5, 15, 20) = 13.33, x=40은 id {1, 2, 3} → AVG(5, 15, 20, 35) = 18.75로 셋 다 참이다. 여기서 빈 집합의 AVG를 0으로 놓으면 4라는 오답이 나온다
- 26-1: JOIN 뒤 AVG 서브쿼리 조건 COUNT → **2**
- 24-3: JOIN + GROUP BY·HAVING이 든 중첩 서브쿼리 COUNT → **1**
- 24-1: IN 서브쿼리 결과 값 → **a, b**

### 답 쓸 때

상관 서브쿼리는 바깥 행마다 안쪽을 다시 계산하기 때문에, 바깥 표의 행마다 "이 행의 값으로 안쪽을 돌리면?"을 표 옆에 적어 가며 푼다. 단일 행 서브쿼리는 더 간단하다. 값을 먼저 구한 뒤 쿼리에 숫자로 바꿔 적어 두면 실수가 준다.

## 키워드 빈칸: DCL (GRANT·REVOKE)
출제: 21-3
형태: GRANT 문장의 예약어 빈칸

### 개념

DCL(데이터 제어어)이란 데이터 보안·무결성 유지·병행 제어·회복을 위한 명령을 말한다. 여기서 헷갈리는 게 COMMIT·ROLLBACK의 소속이다. COMMIT·ROLLBACK을 TCL로 따로 떼기도 하지만, 국내 시험은 권한을 다루는 GRANT·REVOKE와 트랜잭션을 끝내는 COMMIT·ROLLBACK을 모두 DCL로 분류한다. 실제로 2025년 3회 필기에서 "DCL 명령어가 아닌 것"을 물었을 때 보기는 COMMIT·ROLLBACK·GRANT·SELECT였고, 정답은 SELECT였다. sqlite는 사용자 개념이 없어 GRANT·REVOKE를 지원하지 않는다. 그래서 아래는 표준 문법만 적었고, 다섯 문장 모두 PostgreSQL에서 실행해 확인했다.

| 문장 | 완성 문장 | 뜻 |
|---|---|---|
| 권한 부여 | `GRANT SELECT ON 학생 TO USER1;` | USER1에게 학생 테이블 조회 권한 |
| 재부여 허용 | `GRANT UPDATE ON 학생 TO USER1 WITH GRANT OPTION;` | USER1이 그 권한을 남에게 다시 줄 수 있다 |
| 권한 회수 | `REVOKE SELECT ON 학생 FROM USER1;` | |
| 연쇄 회수 | `REVOKE UPDATE ON 학생 FROM USER1 CASCADE;` | USER1이 남에게 준 권한까지 함께 회수 |
| 재부여 권한만 회수 | `REVOKE GRANT OPTION FOR UPDATE ON 학생 FROM USER1;` | |

GRANT는 `TO`, REVOKE는 `FROM`이 짝이다. 권한은 누구"에게" 주고, 누구"로부터" 거둬들인다고 생각하면 외우기 쉽다. 대표적인 권한은 SELECT·INSERT·UPDATE·DELETE·ALL이고, 그 밖에 ALTER·INDEX·REFERENCES나 DDL 권한(CREATE 등)도 줄 수 있다. 모든 사용자에게 줄 때는 `TO PUBLIC`이라 쓴다. 그리고 여러 권한을 묶어 사용자 그룹에 주는 단위가 있는데, 이것을 **롤(ROLE)**이라 한다.

아래는 USER1 한 명에게 권한을 주고 거두는 순서다. sqlite에서 돌릴 수 없는 문장이라 실행 결과 없이 문법만 적었다.

#### 1. 권한 주기: GRANT ... ON ... TO
- 형식: `GRANT 권한 ON 테이블 TO 사용자`
- 권한 자리: SELECT·INSERT·UPDATE·DELETE·ALL 등
- 모든 사용자 대상: `TO PUBLIC`

```sql
GRANT SELECT ON 학생 TO USER1;
```

"무엇을(SELECT) 어디에(ON 학생) 누구에게(TO USER1)" 순서로 읽힌다. 21-3에서 빈칸이 난 문장이 바로 이 모양이다.

#### 2. 받은 권한을 남에게 다시 주기: WITH GRANT OPTION
- 형식: GRANT 문장 끝에 `WITH GRANT OPTION`
- 효과: 받은 사람이 같은 권한을 다른 사용자에게 재부여 가능

```sql
GRANT UPDATE ON 학생 TO USER1 WITH GRANT OPTION;
```

이 옵션 없이 받은 권한은 USER1 혼자만 쓸 수 있다. 옵션을 붙여 주면 USER1이 그 UPDATE 권한을 또 다른 사용자에게 넘길 수 있게 된다.

#### 3. 권한 거두기: REVOKE ... ON ... FROM
- 형식: `REVOKE 권한 ON 테이블 FROM 사용자`
- `CASCADE`: 그 사용자가 남에게 준 권한까지 연쇄 회수
- `GRANT OPTION FOR`: 권한은 두고 재부여 권한만 회수

```sql
REVOKE SELECT ON 학생 FROM USER1;
REVOKE UPDATE ON 학생 FROM USER1 CASCADE;
REVOKE GRANT OPTION FOR UPDATE ON 학생 FROM USER1;
```

2단계에서 USER1이 UPDATE 권한을 다른 사람에게 넘긴 상태라면, 두 번째 문장처럼 CASCADE를 붙이면 USER1에게서 거둘 때 USER1이 넘겨준 권한까지 함께 사라진다. 세 번째 문장은 USER1의 UPDATE 권한 자체는 남겨 두고, 남에게 다시 줄 수 있는 자격만 뺏는다.

### 기출에서 이렇게 나왔다

- 21-3: `GRANT SELECT ON 학생 TO USER1` 빈칸

### 답 쓸 때

문장 뼈대는 `GRANT ... ON ... TO`, `REVOKE ... ON ... FROM` 두 개만 기억하면 된다. CASCADE는 REVOKE 쪽에만 붙는다. 위 표에서 본 것처럼 USER1이 남에게 준 권한까지 함께 회수할 때 쓰는 옵션이다.

## SQL 분류와 SELECT 처리 순서
출제: 없음

### 개념

SQL 명령은 하는 일에 따라 무리를 나눈다. 테이블 구조를 정하는 명령, 데이터를 다루는 명령, 권한과 트랜잭션을 다루는 명령이 각각 따로 이름을 갖고 있다.

#### 1. 명령의 네 무리: DDL·DML·DCL·TCL

| 분류 | 명령 | 특징 |
|---|---|---|
| **DDL(Data Definition Language)** | CREATE·ALTER·DROP·TRUNCATE | 스키마 정의. 자동 COMMIT |
| **DML(Data Manipulation Language)** | SELECT·INSERT·UPDATE·DELETE | 데이터 조작. 트랜잭션 단위로 되돌릴 수 있다. NCS 학습모듈은 SELECT를 **DQL(Data Query Language, 데이터 질의어)**로 따로 부르기도 한다 |
| **DCL(Data Control Language)** | GRANT·REVOKE (국내 시험은 COMMIT·ROLLBACK도 포함) | 권한, 그리고 무결성·병행 제어·회복 |
| **TCL(Transaction Control Language)** | COMMIT·ROLLBACK·SAVEPOINT | 트랜잭션 제어. **국내 시험(NCS·필기)은 COMMIT·ROLLBACK을 DCL로 분류한다.** "DCL 명령어"를 물으면 GRANT·REVOKE·COMMIT·ROLLBACK |

여기서 헷갈리는 칸은 두 군데다. TRUNCATE는 행을 지우는데도 DDL에 들어가고, COMMIT·ROLLBACK은 TCL로 따로 떼기도 하지만 국내 시험에서는 DCL로 본다.

#### 2. 쓰는 순서와 처리 순서
SELECT 문은 `SELECT [DISTINCT] 컬럼 FROM 테이블 [WHERE 조건] [GROUP BY 컬럼] [HAVING 그룹조건] [ORDER BY 컬럼 ASC|DESC]` 순서로 쓴다. 하지만 처리되는 순서는 다르다. 먼저 FROM으로 테이블을 확보하고 WHERE로 행을 고른 뒤, GROUP BY로 묶고 HAVING으로 그룹을 고른다. 그러고 나서야 SELECT에서 열을 고르고 집계를 계산하며, 마지막으로 ORDER BY로 정렬한다. WHERE에서 그룹 함수를 쓸 수 없는 것도 이 순서 때문이다. WHERE가 처리될 때는 아직 묶인 그룹이 없다.

처리 순서만 따로 적으면 이렇다.

1. FROM: 대상 테이블 확보
2. WHERE: 조건에 맞는 행 선택
3. GROUP BY: 같은 값끼리 그룹 생성
4. HAVING: 조건에 맞는 그룹 선택
5. SELECT: 열 선택, 집계 계산
6. ORDER BY: 정렬

SELECT 문 묶음에서 WHERE에 COUNT를 쓰면 오류가 나고, ORDER BY에서는 별칭이 통하는 걸 직접 봤었다. 둘 다 이 순서로 설명된다.

#### 3. GROUP BY 없이 쓰는 HAVING
한 가지 더, HAVING은 보통 GROUP BY와 함께 쓰지만 GROUP BY 없이 쓰면 테이블 전체를 한 그룹으로 본다.

```sql
-- 사원 테이블(5행)
SELECT COUNT(*) FROM 사원 HAVING COUNT(*) >= 5;
SELECT COUNT(*) FROM 사원 HAVING COUNT(*) >= 6;
```
출력: `5` / 행 없음 (전체 5행이 한 그룹)

전체 5행이 한 그룹이니 `COUNT(*)`는 5다. 조건이 `>= 5`면 그 그룹이 통과해 한 행이 나오고, `>= 6`이면 그룹이 탈락해 결과가 아예 비어 버린다. 0이 찍히는 게 아니라 행이 없다는 점이 다르다.

## 조인·서브쿼리·집합 연산의 나머지 문법
출제: 없음

### 개념

| 문법 | 완성 문장 | 뜻 |
|---|---|---|
| 셀프 조인 | `SELECT E.이름, M.이름 FROM 직원 E JOIN 직원 M ON E.상사사번 = M.사번;` | 같은 표를 별칭 둘로 조인 |
| NATURAL JOIN | `SELECT 이름, 부서명 FROM 사원 NATURAL JOIN 부서;` | 이름이 같은 컬럼으로 자동 동등 조인, 중복 컬럼 하나만 |
| USING | `SELECT 이름, 부서명 FROM 사원 JOIN 부서 USING (부서번호);` | 조인 컬럼명을 지정 |
| FROM 절 서브쿼리 | `SELECT 부서번호, 평균 FROM (SELECT 부서번호, AVG(급여) AS 평균 FROM 사원 GROUP BY 부서번호) T WHERE 평균 >= 2000;` | 인라인 뷰 |
| INTERSECT | `SELECT 값 FROM A INTERSECT SELECT 값 FROM B;` | 양쪽에 다 있는 행 |
| EXCEPT(MINUS) | `SELECT 값 FROM A EXCEPT SELECT 값 FROM B;` | A에만 있는 행. Oracle은 MINUS |

기출로 나온 적은 없는 문법들이지만, 앞에서 본 조인·서브쿼리·UNION의 연장선이다. 표의 순서대로 하나씩 돌려 본다.

#### 1. 같은 표끼리 조인: 셀프 조인
직원 테이블에는 각 직원의 상사 사번이 들어 있다. "직원 이름 옆에 상사 이름"을 붙이려면 직원 테이블을 두 번 써야 한다. 그래서 하나는 직원(E), 하나는 상사(M)라는 별칭을 붙여 서로 다른 테이블처럼 조인한다.

```sql
CREATE TABLE 직원(사번 INT, 이름 TEXT, 상사사번 INT);
INSERT INTO 직원 VALUES (1,'대표',NULL),(2,'팀장A',1),(3,'팀장B',1),(4,'사원A',2);
SELECT E.이름 AS 직원, M.이름 AS 상사 FROM 직원 E JOIN 직원 M ON E.상사사번 = M.사번;
```
출력: `팀장A 대표`, `팀장B 대표`, `사원A 팀장A` (상사가 없는 대표는 빠진다)

대표의 상사사번은 NULL이라 M 쪽에서 짝을 찾지 못한다. INNER 조인이니 짝 없는 행은 빠진다는, 조인 묶음의 규칙이 그대로 적용된 것이다.

#### 2. 조인 조건 줄여 쓰기: NATURAL JOIN과 USING
NATURAL JOIN은 ON 조건을 아예 쓰지 않는다. 두 테이블에서 이름이 같은 컬럼(여기서는 부서번호)을 찾아 알아서 동등 조인한다.

```sql
CREATE TABLE 부서(부서번호 INT, 부서명 TEXT);
INSERT INTO 부서 VALUES (10,'개발'),(20,'영업'),(30,'인사'),(40,'총무');
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT 이름, 부서명 FROM 사원 NATURAL JOIN 부서;
SELECT * FROM 사원 NATURAL JOIN 부서 LIMIT 1;
```
출력: `홍길동 개발`, `이순신 영업`, `강감찬 개발`, `김유신 영업` / 전체 컬럼은 5개이고 부서번호는 한 번만 나온다. 표준 SQL(Oracle·PostgreSQL)은 공통 컬럼을 맨 앞에 두어 `부서번호 사번 이름 급여 부서명` 순서다(sqlite만 `사번 이름 부서번호 급여 부서명` 순서로 보여 준다).

결과 4행은 INNER JOIN과 같다. 다른 점은 `SELECT *`로 볼 때 부서번호 컬럼이 두 번이 아니라 한 번만 나온다는 것이다. USING은 이름이 같은 컬럼 중 어느 것으로 조인할지를 직접 적어 주는 방식이다.

```sql
SELECT 이름, 부서명 FROM 사원 JOIN 부서 USING (부서번호);
```
출력:

| 이름 | 부서명 |
|---|---|
| 홍길동 | 개발 |
| 이순신 | 영업 |
| 강감찬 | 개발 |
| 김유신 | 영업 |

같은 4행이다. 공통 컬럼이 부서번호 하나뿐이라 NATURAL JOIN과 결과가 같다.

#### 3. FROM 자리의 서브쿼리: 인라인 뷰
서브쿼리는 WHERE뿐 아니라 FROM 자리에도 올 수 있다. 이때 괄호 안 결과는 잠깐 쓰고 버리는 테이블처럼 쓰이고, 이것을 인라인 뷰라고 부른다.

```sql
SELECT 부서번호, 평균 FROM (SELECT 부서번호, AVG(급여) AS 평균 FROM 사원 GROUP BY 부서번호) T WHERE 평균 >= 2000;
```
출력:

| 부서번호 | 평균 |
|---|---|
| NULL | 2000.0 |
| 10 | 3500.0 |

괄호 안에서 부서별 평균 표(NULL 2000.0, 10 3500.0, 20 1750.0)가 먼저 만들어지고, 바깥 WHERE가 그 표에서 평균 2000 이상인 두 행을 고른다. 안쪽에서 붙인 별칭 `평균`을 바깥에서 컬럼 이름처럼 쓸 수 있다는 점이 편하다. 같은 조건을 `HAVING AVG(급여) >= 2000`으로 써도 결과는 같다.

#### 4. 교집합과 차집합: INTERSECT, EXCEPT
UNION이 합집합이라면 INTERSECT는 양쪽에 다 있는 행, EXCEPT는 앞쪽에만 있는 행을 남긴다. 결과 예측 묶음의 A {1, 2, 4, 5}와 B {2, 5, 8}을 다시 쓰자.

```sql
CREATE TABLE A(값 INT);  CREATE TABLE B(값 INT);
INSERT INTO A VALUES (1),(2),(4),(5);
INSERT INTO B VALUES (2),(5),(8);

SELECT 값 FROM A INTERSECT SELECT 값 FROM B;
SELECT 값 FROM A EXCEPT SELECT 값 FROM B;
```
출력: `2, 5` / `1, 4`

2와 5는 양쪽에 다 있어 INTERSECT에 남고, A에서 그 둘을 빼면 1과 4가 남는다. EXCEPT는 앞뒤를 바꾸면 결과가 달라진다. `SELECT 값 FROM B EXCEPT SELECT 값 FROM A;`로 돌리면 B에만 있는 8 하나가 나온다. Oracle에서는 같은 연산을 MINUS라고 쓴다.

## 뷰·인덱스·트랜잭션 명령
출제: 없음

### 개념

테이블을 직접 다루는 것 말고도 DB에는 조회를 편하게 해 주는 객체(뷰), 빠르게 해 주는 객체(인덱스), 여러 변경을 한 덩어리로 확정하거나 취소하는 명령(트랜잭션 명령)이 있다.

#### 1. 쿼리에 이름 붙이기: 뷰
**뷰(View)**란 하나 이상의 기본 테이블에서 유도한 SELECT 문에 이름을 붙여 저장해 둔 가상 테이블을 말한다. 쉽게 말해 데이터가 아니라 쿼리를 저장해 두는 것이다. 뷰는 물리적으로 데이터를 갖지 않고 조회할 때마다 원본에서 계산한다. 그래서 원본이 바뀌면 뷰 결과도 바로 바뀌고, 원본 테이블을 DROP하면 뷰도 쓸 수 없다. 뷰를 바탕으로 또 다른 뷰를 정의할 수도 있는데, 이때도 기초가 된 테이블이나 뷰를 지우면 그 위에 정의된 뷰는 쓸 수 없다.

| 장점 | 단점 |
|---|---|
| **논리적 데이터 독립성**: 기본 테이블 구조가 바뀌어도 뷰로 접근하는 응용은 영향이 적다 | 뷰에는 독립적인 **인덱스를 만들 수 없다** |
| **보안**: 필요한 열·행만 사용자에게 보인다 | **ALTER로 정의를 바꿀 수 없다**. DROP 뒤 다시 CREATE한다 |
| **편의**: 복잡한 조인·조건을 이름 하나로 조회한다 | **삽입·갱신·삭제 제약**: 조인·집계 함수·DISTINCT·GROUP BY가 든 뷰는 갱신할 수 없다 |

뷰가 데이터를 갖지 않는다는 말이 실제로 어떤 뜻인지 sqlite로 확인해 보자. 10번 부서만 보여 주는 뷰를 만들고, 원본 테이블을 고친 뒤 다시 조회한다.

```sql
CREATE VIEW 개발부 AS SELECT 이름, 급여 FROM 사원 WHERE 부서번호 = 10;
SELECT * FROM 개발부;
UPDATE 사원 SET 급여 = 5000 WHERE 이름 = '홍길동';
SELECT * FROM 개발부;
```
출력:

| 이름 | 급여 |
|---|---|
| 홍길동 | 3000 |
| 강감찬 | 4000 |

| 이름 | 급여 |
|---|---|
| 홍길동 | 5000 |
| 강감찬 | 4000 |

뷰에는 손대지 않았는데 홍길동의 급여가 5000으로 바뀌어 나온다. 조회할 때마다 저장된 SELECT를 원본에 다시 돌리기 때문이다. 그럼 원본을 지우면?

```sql
DROP TABLE 사원;
SELECT * FROM 개발부;
```

뷰 정의는 남아 있지만 조회하면 `no such table: main.사원` 오류가 난다. 기초 테이블이 사라지면 뷰를 쓸 수 없다는 말이 이것이다.

#### 2. 뷰 조건을 벗어나는 변경 막기: WITH CHECK OPTION
**WITH CHECK OPTION**은 뷰를 정의한 WHERE 조건을 벗어나는 INSERT·UPDATE를 거부하는 옵션이다. 예를 들어 `CREATE VIEW 개발부 AS SELECT * FROM 사원 WHERE 부서번호 = 10 WITH CHECK OPTION;`으로 뷰를 만들었다고 해보자. 그러면 이 뷰로는 부서번호 20인 행을 넣을 수도, 부서번호를 20으로 바꿀 수도 없다. sqlite는 이 옵션을 지원하지 않아 PostgreSQL에서 실행해 확인했다.

#### 3. 검색을 빠르게: 인덱스
**인덱스(Index)**는 검색 속도를 높이려고 <키 값, 주소> 쌍을 따로 정렬해 둔 구조다. 대신 대가가 있다. 조회는 빨라지지만 INSERT·UPDATE·DELETE를 할 때마다 인덱스도 함께 갱신해야 하니 쓰기 비용이 늘고, 공간도 따로 차지한다. PRIMARY KEY와 UNIQUE에는 인덱스가 자동으로 만들어진다.

자동으로 만들어진다는 건 sqlite에서도 눈으로 볼 수 있다. CREATE INDEX를 한 번도 쓰지 않고 테이블만 만든 뒤 DB가 가진 객체 목록을 조회한 결과다.

```sql
CREATE TABLE 회원(회원번호 INT PRIMARY KEY, 이메일 TEXT UNIQUE, 이름 TEXT);
SELECT type, name FROM sqlite_master WHERE tbl_name = '회원';
```
출력:

| type | name |
|---|---|
| table | 회원 |
| index | sqlite_autoindex_회원_1 |
| index | sqlite_autoindex_회원_2 |

테이블 하나와 인덱스 두 개가 있다. 회원번호(PRIMARY KEY)와 이메일(UNIQUE)에 하나씩 자동으로 붙은 것이다.

#### 4. 확정과 취소: COMMIT·ROLLBACK·SAVEPOINT
**트랜잭션 명령**도 정리해 두자. `COMMIT`은 확정하고, `ROLLBACK`은 마지막 COMMIT 이후를 모두 취소한다. `SAVEPOINT 이름`은 중간 지점을 만들어 두는 명령인데, `ROLLBACK TO 이름`을 쓰면 마지막 COMMIT까지가 아니라 그 지점까지만 되돌린다.

아래는 101번 급여를 바꾼 뒤 SAVEPOINT를 찍고, 102번 급여를 바꾼 다음 그 지점으로 되돌린 예다.

```sql
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

BEGIN;
UPDATE 사원 SET 급여 = 9999 WHERE 사번 = 101;
SAVEPOINT sp1;
UPDATE 사원 SET 급여 = 1 WHERE 사번 = 102;
ROLLBACK TO sp1;
COMMIT;
SELECT 사번, 급여 FROM 사원 WHERE 사번 IN (101, 102);
```
출력: `101 | 9999`, `102 | 1000` (SAVEPOINT 뒤의 변경만 취소되고 앞의 변경은 COMMIT됐다)

이번에는 SAVEPOINT 없이 전체를 지웠다가 ROLLBACK해 보자. DML 묶음에서 DELETE는 ROLLBACK으로 되돌릴 수 있다고 했었다.

```sql
BEGIN;
DELETE FROM 사원;
ROLLBACK;
SELECT COUNT(*) FROM 사원;
```
출력: `5` (DELETE는 ROLLBACK으로 되돌아온다)

## 내장 함수·윈도우 함수
출제: 없음

### 개념

함수는 결과 행이 몇 개 나오느냐로 크게 갈린다. 행마다 값 하나를 돌려주는 함수, 여러 행을 하나로 줄이는 함수, 행은 그대로 두고 그룹 정보를 옆에 붙이는 함수다. 이 순서대로 정리한다.

#### 1. 행마다 하나 vs 여러 행을 하나로: 단일 행 함수와 집계 함수
**단일 행 함수**는 행 하나마다 계산해 값 하나를 돌려주기 때문에 입력 행 수와 출력 행 수가 같다. 반면 **집계(다중 행) 함수**(COUNT·SUM·AVG·MAX·MIN)는 여러 행을 묶어 값 하나로 줄인다. 학생 테이블(5행)에 하나씩 써 보면 차이가 바로 보인다.

```sql
SELECT 이름, LENGTH(이름) FROM 학생;
SELECT SUM(점수) FROM 학생;
```
출력:

| 이름 | LENGTH(이름) |
|---|---|
| 김철수 | 3 |
| 이영희 | 3 |
| 박민수 | 3 |
| 최지우 | 3 |
| 정하늘 | 3 |

```
240
```

LENGTH는 5행을 넣었더니 5행이 나왔고, SUM은 5행을 받아 한 행으로 줄였다.

#### 2. 자주 쓰는 단일 행 함수
종류별로 모으면 아래와 같다.

| 분류 | 함수 |
|---|---|
| 문자 | `UPPER`·`LOWER`(대·소문자), `SUBSTR(s, 시작, 길이)`(부분 추출), `LENGTH`(길이), `CONCAT`(연결. 연산자로는 세로줄 두 개), `TRIM`(양끝 공백 제거), `REPLACE(s, a, b)`(치환) |
| 숫자 | `ROUND(n, 자리)`(반올림), `TRUNC(n, 자리)`(버림), `MOD(a, b)`(나머지), `ABS`(절댓값), `CEIL`(올림), `FLOOR`(내림) |
| 날짜·변환 | `SYSDATE`(현재 시각), `ADD_MONTHS`, `TO_CHAR`(숫자·날짜 → 문자), `TO_DATE`(문자 → 날짜), `TO_NUMBER`(문자 → 숫자) |
| NULL 처리 | `NVL(a, b)`(a가 NULL이면 b), `NVL2(a, b, c)`(a가 NULL이 아니면 b, NULL이면 c), `COALESCE(a, b, ...)`(처음으로 NULL이 아닌 값), `NULLIF(a, b)`(같으면 NULL, 다르면 a) |

이 중 날짜·변환 함수와 NVL·NVL2·TRUNC(자릿수 지정)는 Oracle 함수라서 sqlite에서는 실행하지 않았고, 아래는 sqlite에 있는 함수만 골라 실행한 것이다.

```sql
SELECT UPPER('abc'), SUBSTR('database', 1, 4), LENGTH('hello'), REPLACE('aXbX', 'X', '-');
SELECT ROUND(3.146, 1), ROUND(3.156, 1), ABS(-7), 17 % 5;
SELECT COALESCE(NULL, NULL, 3), NULLIF(5, 5), NULLIF(5, 3);
```
출력: `ABC | data | 5 | a-b-` / `3.1 | 3.2 | 7 | 2` / `3 | NULL | 5`

`SUBSTR('database', 1, 4)`는 1번째 글자부터 4글자라 data다. SQL은 글자 위치를 1부터 센다. `ROUND(3.146, 1)`과 `ROUND(3.156, 1)`은 소수 둘째 자리(4와 5)에서 반올림해 3.1과 3.2로 갈린다. COALESCE는 앞에서부터 처음 만나는 NULL 아닌 값 3을, NULLIF는 두 값이 같으면 NULL을, 다르면 앞의 값 5를 돌려준다. NULL 처리 함수는 이런 식으로 NULL 칸을 다른 값으로 바꿀 때 쓴다.

```sql
SELECT 이름, COALESCE(점수, 0) FROM 학생;
```
출력:

| 이름 | COALESCE(점수, 0) |
|---|---|
| 김철수 | 80 |
| 이영희 | 0 |
| 박민수 | 70 |
| 최지우 | 0 |
| 정하늘 | 90 |

#### 3. 행을 남긴 채 순위 매기기: 윈도우 함수
**윈도우 함수**는 행을 줄이지 않고 각 행 옆에 그룹 안의 순위·합계를 붙여 주는 함수다. 범위는 `OVER (PARTITION BY 그룹 ORDER BY 정렬)`로 정한다. GROUP BY와 혼동하기 쉬운데, 차이는 남는 행에 있다. GROUP BY는 그룹당 한 행만 남기지만, 윈도우 함수는 원래 행이 그대로 남는다.

| 함수 | 같은 값 처리 | 80점이 둘일 때 |
|---|---|---|
| **RANK()** | 공동 순위를 주고 그 수만큼 건너뛴다 | 1, 2, 2, 4 |
| **DENSE_RANK()** | 공동 순위를 주되 건너뛰지 않는다 | 1, 2, 2, 3 |
| **ROW_NUMBER()** | 같아도 무조건 다른 번호 | 1, 2, 3, 4 |

윈도우 함수는 **OLAP 함수**라고도 부른다. 종류를 묶어 보면 집계(SUM·AVG… OVER), **순위**(RANK·DENSE_RANK·ROW_NUMBER), **행 순서**(FIRST_VALUE·LAST_VALUE, 이전 행 값을 가져오는 **LAG**와 다음 행 값을 가져오는 **LEAD**), **그룹 내 비율**(RATIO_TO_REPORT·PERCENT_RANK·CUME_DIST·NTILE)이 있다. PARTITION BY는 생략할 수도 있는데, 생략하면 전체가 하나의 윈도우가 된다.

```sql
CREATE TABLE 성적(이름 TEXT, 반 TEXT, 점수 INT);
INSERT INTO 성적 VALUES ('가','A',90),('나','A',80),('다','A',80),('라','A',70),('마','B',85),('바','B',60);

SELECT 이름, 점수,
  RANK()       OVER (ORDER BY 점수 DESC) AS 랭크,
  DENSE_RANK() OVER (ORDER BY 점수 DESC) AS 덴스,
  ROW_NUMBER() OVER (ORDER BY 점수 DESC, 이름) AS 번호
FROM 성적 WHERE 반 = 'A' ORDER BY 번호;
```
출력: `가 90 1 1 1`, `나 80 2 2 2`, `다 80 2 2 3`, `라 70 4 3 4`

A반 네 명 중 나·다가 80점으로 같다. RANK는 둘에게 2등을 주고 다음 사람(라)을 4등으로 건너뛰고, DENSE_RANK는 라에게 바로 3등을 준다. ROW_NUMBER는 같은 점수라도 번호를 나눠 줘야 하니, 위 쿼리에서는 이름을 두 번째 정렬 기준으로 넣어 나에게 2, 다에게 3을 줬다.

#### 4. 그룹 합계를 행 옆에 붙이기: PARTITION BY
OVER 안에 PARTITION BY를 쓰면 그 컬럼 값이 같은 행끼리 따로 계산한다. 반별 합계를 각 행 옆에 붙이는 쿼리다.

```sql
SELECT 이름, 반, 점수, SUM(점수) OVER (PARTITION BY 반) AS 반합계 FROM 성적 ORDER BY 반, 점수 DESC;
```
출력: `가 A 90 320`, `나 A 80 320`, `다 A 80 320`, `라 A 70 320`, `마 B 85 145`, `바 B 60 145` (6행이 그대로 남고 반별 합계가 옆에 붙는다. `GROUP BY 반`이었다면 `A 320`, `B 145` 두 행)

#### 5. 소계와 총계: ROLLUP·CUBE·GROUPING SETS
- **ROLLUP(A, B)**: (A, B)별 소계, A별 소계, 총계
- **CUBE(A, B)**: (A, B)·A·B·전체, 가능한 모든 조합의 소계
- **GROUPING SETS((A), (B))**: 지정한 묶음별 집계만

**그룹 함수**는 GROUP BY 뒤에 써서 소계·총계 행을 함께 만들어 주는 함수다. 세 가지가 헷갈리기 쉬우니 하나씩 보자. **ROLLUP(A, B)**는 (A, B)별 소계와 A별 소계, 그리고 총계를 만든다. 컬럼이 n개면 n+1단계가 나오고, 컬럼 순서가 바뀌면 결과도 바뀐다. **CUBE(A, B)**는 가능한 모든 조합인 (A, B)·A·B·전체의 소계를 만든다. **GROUPING SETS((A), (B))**는 지정한 묶음별 집계만 만들고, 컬럼 순서와도 상관없다. 예를 들어 위 성적 표에서 `SELECT 반, SUM(점수) FROM 성적 GROUP BY ROLLUP(반) ORDER BY 반;`을 실행하면 `A 320`, `B 145`, `NULL 465`가 나오는데, 마지막 행이 총계다. sqlite는 ROLLUP을 지원하지 않아 PostgreSQL에서 실행해 확인했다.

## 프로시저·트리거·사용자 정의 함수·커서
출제: 없음

### 개념

SQL 문장을 하나씩 직접 보내는 대신, 문장 묶음을 DB 안에 저장해 두고 이름으로 부르거나 어떤 일이 생기면 저절로 돌게 할 수 있다. 네 가지가 있는데, 누가 언제 실행하는지와 값을 돌려주는지가 서로 다르다.

| 객체 | 뜻 | 구별 단서 |
|---|---|---|
| **저장 프로시저(Stored Procedure)** | 자주 쓰는 SQL 묶음을 미리 컴파일해 DB에 저장해 두고 호출해 쓰는 것 | 반환값 없이 OUT 매개변수로 결과를 넘긴다. `EXECUTE`로 호출 |
| **사용자 정의 함수(UDF)** | 사용자가 만든 함수. 반드시 값 하나를 **RETURN**한다 | SELECT 문 안에서 호출할 수 있다 |
| **트리거(Trigger)** | INSERT·UPDATE·DELETE 같은 이벤트가 일어나면 **자동으로** 실행되는 프로시저 | 사용자 정의 무결성 구현, 변경 이력 기록 |
| **커서(Cursor)** | 여러 행으로 된 SELECT 결과를 한 행씩 가리키며 처리하는 포인터 | 명시적 커서는 **DECLARE → OPEN → FETCH → CLOSE** |

#### 1. 저장 프로시저와 사용자 정의 함수
- 저장 프로시저: 미리 컴파일해 DB에 저장한 SQL 묶음, `EXECUTE`로 호출
- 프로시저 매개변수 모드: **IN**(입력)·**OUT**(출력)·**INOUT**(입출력) 세 가지
- 사용자 정의 함수: 값 하나를 반드시 RETURN, SELECT 문 안에서 호출 가능

둘 다 SQL 묶음에 이름을 붙여 저장해 둔다는 점은 같다. 갈리는 지점은 결과를 넘기는 방식이다. 프로시저는 반환값 없이 OUT 매개변수로 결과를 내보내고, 함수는 RETURN으로 값 하나를 돌려준다. 함수가 SELECT 문 안에서 컬럼처럼 쓰일 수 있는 것도 값을 돌려주기 때문이다.

#### 2. 결과를 한 행씩: 커서
커서는 누가 만드느냐에 따라 둘로 나뉜다. DBMS가 일반 SQL을 실행할 때 알아서 만드는 것이 **묵시적 커서(Implicit)**이고, 개발자가 직접 선언·제어하는 것이 **명시적 커서(Explicit)**다. 프로시저·UDF·커서는 sqlite에 없어서 실행하지 않았고, 트리거만 sqlite로 확인했다.

명시적 커서는 네 단계를 차례로 밟는다.

1. DECLARE: 커서 이름과 SELECT 문 선언
2. OPEN: 선언한 SELECT 실행
3. FETCH: 한 행씩 꺼내 변수에 저장 (행이 끝날 때까지 반복)
4. CLOSE: 커서 해제

#### 3. 이벤트가 나면 저절로: 트리거
- 실행 시점: INSERT·UPDATE·DELETE 같은 이벤트 발생 시 자동 실행
- 정의 요소: 이벤트, 시점(BEFORE·AFTER), 행 단위 여부(FOR EACH ROW)
- 매개변수·반환값 없음, 내부에서 COMMIT·ROLLBACK 사용 불가

트리거는 프로시저와 달리 매개변수(IN·OUT)도 반환값도 없다. 대신 어떤 이벤트(INSERT·UPDATE·DELETE)에, 어느 시점(**BEFORE·AFTER**)에, 행마다 실행할지(**FOR EACH ROW**)로 정의한다. 그리고 트리거 안에서는 **COMMIT·ROLLBACK을 쓸 수 없다**는 점도 기억해 두자. 변경 전후 값은 Oracle에서는 `:OLD`·`:NEW`로, SQL Server에서는 `deleted`·`inserted`로 참조한다. INSERT라면 변경 전 값이 없으니 OLD가 NULL이고, DELETE라면 변경 후 값이 없으니 NEW가 NULL이다.

sqlite로 급여가 바뀔 때마다 이력을 남기는 트리거를 만들어 보자. sqlite는 OLD·NEW를 콜론 없이 쓴다.

```sql
CREATE TABLE 사원(사번 INT, 급여 INT);
CREATE TABLE 급여이력(사번 INT, 이전 INT, 이후 INT);
CREATE TRIGGER trg_급여 AFTER UPDATE OF 급여 ON 사원
BEGIN
  INSERT INTO 급여이력 VALUES (OLD.사번, OLD.급여, NEW.급여);
END;
INSERT INTO 사원 VALUES (1, 3000);
UPDATE 사원 SET 급여 = 3500 WHERE 사번 = 1;
SELECT * FROM 급여이력;
```
출력: `1 | 3000 | 3500` (UPDATE만 했는데 트리거가 이력 행을 자동으로 넣었다. OLD는 변경 전 행, NEW는 변경 후 행)
