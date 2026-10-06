# SQL

21개 회차 420문제 중 SQL은 38문제(9%)로, 회차당 0~4문제가 나온다. 형태는 둘뿐이다. **실행 결과 쓰기**(17문제)는 작은 표 2~3개와 쿼리를 주고 COUNT 값이나 결과 표를 쓰게 하고, **키워드 빈칸**(21문제)은 SQL 문장의 예약어 자리를 비워 둔다. 2024년 이후 14문제 중 10문제가 실행 결과 쓰기이고, 2023년 이전은 키워드 빈칸이 더 많았다. 아래 예제는 모두 `sqlite3 :memory:`로 실행해 출력을 확인한 것이다. 외래키 예제는 sqlite에서 `PRAGMA foreign_keys = ON;`을 먼저 실행해야 동작한다.

## 키워드 빈칸: SELECT 문
출제: 26-2, 23-1, 22-2, 22-1, 21-2, 20-4, 20-3, 20-2
형태: SELECT 문장의 예약어 자리 1~3개 빈칸

### 개념

SELECT 문의 절은 쓰는 순서가 정해져 있다. `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY`. 실제 처리 순서는 `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`라서, WHERE에서는 집계 함수를 쓸 수 없고 HAVING에서 쓴다. ORDER BY는 마지막이라 SELECT의 별칭을 쓸 수 있다.

| 자리 | 완성 문장 | 단서 |
|---|---|---|
| 그룹별 집계 | `SELECT 부서번호, COUNT(*) FROM 사원 GROUP BY 부서번호;` | "부서별", "과목별" |
| 그룹 조건 | `SELECT 부서번호, SUM(급여) FROM 사원 GROUP BY 부서번호 HAVING COUNT(*) >= 2;` | 집계 결과에 조건 → **HAVING** |
| 그룹 최대·최소 | `SELECT 부서번호, MAX(급여), MIN(급여) FROM 사원 GROUP BY 부서번호 HAVING AVG(급여) >= 2000;` | |
| 패턴 | `SELECT 이름 FROM 학생 WHERE 이름 LIKE '이%';` | "이로 시작" → `'이%'`, "두 번째 글자가 철" → `'_철%'` |
| 정렬 | `SELECT 이름, 점수 FROM 학생 ORDER BY 점수 DESC;` | "내림차순" → **DESC**, 오름차순은 **ASC**(생략 가능) |
| 목록 | `SELECT * FROM 학생 WHERE 학과 IN ('컴공', '전자');` | "중 하나" → **IN** |
| 범위 | `SELECT * FROM 학생 WHERE 점수 BETWEEN 70 AND 85;` | 양 끝 포함 |
| 조인 | `SELECT 이름, 부서명 FROM 사원 JOIN 부서 ON 사원.부서번호 = 부서.부서번호;` | 명시적 조인 조건 → **ON** |
| 모두 비교 | `SELECT 이름 FROM 사원 WHERE 급여 > ALL (SELECT 급여 FROM 사원 WHERE 부서번호 = 20);` | "모든 ~보다 크다" → **ALL** |
| 중복 제거 | `SELECT DISTINCT 학과 FROM 학생;` | |
| NULL 검사 | `SELECT 이름 FROM 학생 WHERE 점수 IS NULL;` | `= NULL`은 항상 거짓 |

`LIKE`의 `%`는 0글자 이상, `_`는 정확히 한 글자다. `ALL`과 `ANY`는 sqlite가 지원하지 않아 아래 예제에서는 같은 뜻의 `MAX`/`MIN`으로 바꿔 실행했다.

```sql
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT 부서번호, COUNT(*) AS 인원, SUM(급여) AS 합계 FROM 사원
GROUP BY 부서번호 HAVING COUNT(*) >= 2;
```
출력: `10 | 2 | 7000`, `20 | 2 | 3500` (부서번호 NULL인 유관순은 인원 1이라 HAVING에서 걸러진다)

```sql
SELECT 이름, 급여 FROM 사원 WHERE 급여 >= 2500 ORDER BY 급여 DESC;
```
출력: `강감찬 4000`, `홍길동 3000`, `김유신 2500`

```sql
-- 급여 > ALL (SELECT 급여 FROM 사원 WHERE 부서번호 = 20) 과 같은 뜻
SELECT 이름, 급여 FROM 사원 WHERE 급여 > (SELECT MAX(급여) FROM 사원 WHERE 부서번호 = 20);
```
출력: `홍길동 3000`, `강감찬 4000` (ANY였다면 MIN과 비교하므로 유관순·김유신도 포함되어 4명)

```sql
SELECT COUNT(*) FROM 사원 WHERE 부서번호 = NULL;
SELECT COUNT(*) FROM 사원 WHERE 부서번호 IS NULL;
```
출력: `0`, `1`

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

대소문자는 구분하지 않지만 예약어는 대문자로 쓰는 쪽이 안전하다. `'이%'`처럼 문자열은 작은따옴표 안에 넣는다. DESC와 ASC를 뒤집어 쓰지 않도록 "내림차순 = DESC = Descending"으로 기억한다.

## 결과 예측: DISTINCT 튜플 수, UNION, ON DELETE CASCADE
출제: 26-1, 23-3, 22-3, 20-1
형태: 쿼리 2~3개의 결과 행 수 / UNION 결과 표 / 삭제 후 COUNT

### 개념

| 함정 | 규칙 |
|---|---|
| `COUNT(*)` | 전체 행 수. 중복이든 NULL이든 다 센다 |
| `COUNT(DISTINCT 컬럼)` | 그 컬럼의 서로 다른 값 개수. NULL은 빼고 센다 |
| `SELECT DISTINCT A, B` | (A, B) **조합**이 같은 행만 하나로. A 하나만 같은 것은 중복이 아니다 |
| `UNION` | 두 결과를 합치고 **중복 행 제거**. `ORDER BY`는 맨 뒤에 한 번만, 전체 결과에 적용 |
| `UNION ALL` | 중복을 남긴다 |
| `ON DELETE CASCADE` | 부모 행을 지우면 그 행을 참조하던 자식 행도 **같이 삭제** |
| `ON DELETE SET NULL` | 부모 행을 지우면 자식의 외래키를 **NULL로** |
| 옵션 없음 | 참조하는 자식이 있으면 부모 삭제가 **거부** |

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

### 기출에서 이렇게 나왔다

- 26-1, 22-3, 20-1: DISTINCT가 있는 문과 없는 문 3개의 결과 행 수 → **200 / 3 / 1** (22-3은 **90 / 3 / 1**). 같은 유형이 세 번 나왔다
- 23-3: UNION 뒤 ORDER BY DESC 결과 표 → **8, 5, 4, 2, 1**
- 22-3: ON DELETE CASCADE가 걸린 테이블에서 부모 삭제 뒤 COUNT와 COUNT DISTINCT → **3 / 4**

### 답 쓸 때

UNION 결과를 쓸 때 중복을 먼저 지우고 정렬한다. CASCADE 문제는 "부모에서 지운 값을 외래키로 가진 자식 행"을 표에서 지운 뒤 센다.

## 키워드 빈칸: INSERT·UPDATE·DELETE
출제: 24-2, 23-2, 23-1, 21-2, 20-3
형태: DML 문장의 예약어 빈칸

### 개념

| 문장 | 완성 문장 | 주의 |
|---|---|---|
| 행 추가 | `INSERT INTO 교수(교수번호, 이름, 나이) VALUES (1, '김교수', 45);` | 컬럼 목록을 생략하면 모든 컬럼을 순서대로 |
| 조회 결과 추가 | `INSERT INTO 교수백업 SELECT 교수번호, 이름 FROM 교수 WHERE 나이 >= 46;` | VALUES 대신 SELECT. 괄호 없음 |
| 수정 | `UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1;` | WHERE가 없으면 전체 행이 바뀐다 |
| 삭제 | `DELETE FROM 교수 WHERE 교수번호 = 3;` | WHERE가 없으면 전체 행 삭제. 테이블은 남는다 |

DELETE는 행만 지우고 로그를 남겨 ROLLBACK이 되는 DML이고, DROP은 테이블 자체를 없애는 DDL이다. TRUNCATE는 행을 모두 지우되 DDL로 분류된다.

```sql
CREATE TABLE 교수(교수번호 INT PRIMARY KEY, 이름 TEXT NOT NULL, 이메일 TEXT UNIQUE,
  나이 INT CHECK (나이 >= 20), 학과코드 TEXT DEFAULT 'D1');
INSERT INTO 교수(교수번호, 이름, 이메일, 나이) VALUES (1,'김교수','kim@x.kr',45);
INSERT INTO 교수 VALUES (2,'이교수','lee@x.kr',50,'D2');
INSERT INTO 교수(교수번호, 이름) VALUES (3,'박교수');
SELECT * FROM 교수;
```
출력: `1 김교수 kim@x.kr 45 D1`, `2 이교수 lee@x.kr 50 D2`, `3 박교수 NULL NULL D1` (안 넣은 컬럼은 NULL, DEFAULT가 있으면 그 값)

```sql
UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1;
SELECT 교수번호, 나이 FROM 교수 WHERE 교수번호 = 1;
DELETE FROM 교수 WHERE 교수번호 = 3;
SELECT COUNT(*) FROM 교수;
```
출력: `1 | 46`, `2`

```sql
CREATE TABLE 교수백업(교수번호 INT, 이름 TEXT);
INSERT INTO 교수백업 SELECT 교수번호, 이름 FROM 교수 WHERE 나이 >= 46;
SELECT * FROM 교수백업;
```
출력: `1 김교수`, `2 이교수`

### 기출에서 이렇게 나왔다

- 24-2: INSERT ... VALUES / INSERT ... SELECT ... FROM / UPDATE ... SET 빈칸 한 문제
- 23-2: `INSERT INTO ... VALUES` 빈칸
- 23-1, 20-3: `DELETE FROM ... WHERE` 빈칸 (같은 유형)
- 21-2: `UPDATE ... SET` 빈칸

### 답 쓸 때

INSERT는 `INTO`, UPDATE는 `SET`, DELETE는 `FROM`이 각각 짝이다. `INSERT INTO ... SELECT`에는 VALUES를 쓰지 않는다.

## 키워드 빈칸: DDL (CREATE·ALTER·DROP과 제약조건)
출제: 26-2, 26-1, 23-2, 20-3, 20-2
형태: CREATE TABLE 제약조건 빈칸 / ALTER·INDEX·VIEW·DROP 문장 빈칸

### 개념

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

| 문장 | 완성 문장 |
|---|---|
| 컬럼 추가 | `ALTER TABLE 교수 ADD 전화 TEXT;` |
| 컬럼 변경·삭제 | `ALTER TABLE 교수 MODIFY 나이 INT;` / `ALTER TABLE 교수 DROP COLUMN 전화;` (표준 문법, sqlite에 MODIFY 없음) |
| 인덱스 | `CREATE INDEX idx_교수_이름 ON 교수(이름);` |
| 뷰 | `CREATE VIEW 고령교수 AS SELECT 이름, 나이 FROM 교수 WHERE 나이 >= 45;` |
| 도메인 | `CREATE DOMAIN 성별 CHAR(1) DEFAULT '남' CONSTRAINT 성별제약 CHECK (VALUE IN ('남', '여'));` (표준 문법, sqlite 미지원) |
| 삭제 | `DROP TABLE 교수 CASCADE;` / `DROP VIEW 고령교수;` / `DROP INDEX idx_교수_이름;` |

**DROP ... CASCADE**는 그 객체를 참조하는 다른 객체(뷰·외래키)까지 함께 삭제하고, **RESTRICT**는 참조하는 객체가 있으면 삭제를 막는다. (sqlite는 CASCADE 옵션을 받지 않아 아래에서는 옵션 없이 실행했다.)

제약조건 위반 시 어떤 오류가 나는지 확인한 것 (위 교수 테이블에 1·2번 교수가 있고, 학과는 D1·D2만 존재):

```sql
INSERT INTO 교수(교수번호, 이름, 나이) VALUES (4,'최교수',19);         -- CHECK 위반
INSERT INTO 교수(교수번호, 이름, 이메일) VALUES (5,'정교수','kim@x.kr'); -- UNIQUE 위반
INSERT INTO 교수(교수번호, 이름, 학과코드) VALUES (6,'한교수','D9');     -- FOREIGN KEY 위반
INSERT INTO 교수(교수번호, 이름) VALUES (1,'중복');                     -- PRIMARY KEY 위반
INSERT INTO 교수(교수번호) VALUES (7);                                  -- NOT NULL 위반
```
출력: 다섯 문장 모두 오류로 거부된다 (`CHECK constraint failed`, `UNIQUE constraint failed: 교수.이메일`, `FOREIGN KEY constraint failed`, `UNIQUE constraint failed: 교수.교수번호`, `NOT NULL constraint failed: 교수.이름`)

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

### 기출에서 이렇게 나왔다

- 26-2: `CREATE DOMAIN ... ( ) (VALUE IN ...)` → **CHECK**
- 26-1: `CONSTRAINT ... FOREIGN KEY ... REFERENCES` 5칸
- 23-2: `DROP VIEW ... ( )` 참조 객체까지 삭제 → **CASCADE**
- 20-3: 컬럼 추가 → **ALTER TABLE ... ADD**
- 20-2: 인덱스 생성 → **CREATE INDEX ... ON**

### 답 쓸 때

외래키 제약은 `CONSTRAINT 이름 FOREIGN KEY (컬럼) REFERENCES 부모테이블(컬럼)` 순서다. REFERENCES 뒤에는 테이블명이 오고 괄호 안에 부모의 컬럼이 온다. CASCADE와 RESTRICT를 혼동하지 않는다.

## 결과 예측: 집계 함수와 NULL, AND/OR 우선순위
출제: 25-3, 24-1, 22-2, 21-1
형태: 표 + 쿼리 → COUNT 값

### 개념

| 함정 | 규칙 |
|---|---|
| `COUNT(*)` | NULL이 있어도 행을 다 센다 |
| `COUNT(컬럼)` | 그 컬럼이 **NULL인 행은 빼고** 센다 |
| `SUM·AVG·MAX·MIN` | NULL을 무시한다. AVG는 NULL이 아닌 값만으로 평균 |
| `AND`와 `OR` | **AND가 먼저** 묶인다. `A OR B AND C`는 `A OR (B AND C)` |
| `NULL` 비교 | `= NULL`, `<> NULL`은 모두 거짓. `IS NULL`만 참이 된다. `<>` 조건도 NULL 행은 빠진다 |

```sql
CREATE TABLE 학생(학번 INT, 이름 TEXT, 학과 TEXT, 점수 INT);
INSERT INTO 학생 VALUES (1,'김철수','컴공',80),(2,'이영희','컴공',NULL),
  (3,'박민수','전자',70),(4,'최지우','전자',NULL),(5,'정하늘','기계',90);

SELECT COUNT(*), COUNT(점수), COUNT(학과) FROM 학생;
```
출력: `5 | 3 | 5`

```sql
SELECT COUNT(점수) FROM 학생 WHERE 학과 IN ('컴공','전자') OR 점수 >= 90;
```
출력: `3` (조건을 만족하는 행은 5개지만 점수가 NULL인 2행은 COUNT(점수)에서 빠진다)

```sql
SELECT SUM(점수), AVG(점수), MAX(점수), MIN(점수) FROM 학생;
```
출력: `240 | 80.0 | 90 | 70` (AVG는 240/3이지 240/5가 아니다)

```sql
SELECT COUNT(*) FROM 학생 WHERE 학과='컴공' OR 학과='전자' AND 점수 >= 70;
SELECT COUNT(*) FROM 학생 WHERE (학과='컴공' OR 학과='전자') AND 점수 >= 70;
```
출력: `3`, `2` (괄호가 없으면 "컴공 전부" + "전자 중 70 이상" = 2 + 1. 괄호가 있으면 두 학과 중 70 이상인 2명)

```sql
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);
SELECT COUNT(*) FROM 사원 WHERE 부서번호 <> 10;
```
출력: `2` (부서번호가 NULL인 유관순은 `<> 10`에도 잡히지 않는다)

### 기출에서 이렇게 나왔다

- 25-3, 22-2: IN과 OR이 섞인 WHERE 뒤 `COUNT(컬럼)`, 그 컬럼에 NULL 포함 → **4** (25-3), **3** (22-2). 같은 유형
- 24-1, 21-1: 괄호 없는 `OR ... AND` 조건의 COUNT → **1** (같은 유형)

### 답 쓸 때

쿼리를 읽을 때 먼저 WHERE 조건으로 행을 고르고(AND 먼저), 그다음 COUNT 괄호 안을 본다. `COUNT(*)`가 아니면 해당 컬럼이 NULL인 행을 지운 뒤 센다.

## 결과 예측: 조인
출제: 26-2, 25-3, 25-1, 21-3
형태: 표 2개 + 조인 쿼리 → COUNT 또는 결과 표(헤더 포함)

### 개념

| 조인 | 결과 행 | 함정 |
|---|---|---|
| 묵시적 조인 `FROM A, B WHERE A.k = B.k` | INNER JOIN과 같다 | WHERE에 조인 조건과 검색 조건이 섞여 있다 |
| `INNER JOIN ... ON` | 양쪽에 짝이 있는 행만 | 외래키가 NULL인 행은 빠진다 |
| `LEFT OUTER JOIN` | 왼쪽은 다 남기고 짝 없는 오른쪽은 NULL | |
| `RIGHT OUTER JOIN` | 오른쪽은 다 남기고 짝 없는 왼쪽은 NULL | `WHERE 왼쪽.키 IS NULL`이면 "짝 없는 오른쪽 행"만 |
| `FULL OUTER JOIN` | 양쪽 다 남긴다 | INNER 행 수 + 왼쪽만 + 오른쪽만 |
| `CROSS JOIN` | 모든 조합. 행 수 = m × n | WHERE의 LIKE로 걸러진 조합 수를 세야 한다 |

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

```sql
SELECT COUNT(*) FROM 사원 INNER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
SELECT COUNT(*) FROM 사원 FULL OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력: `4`, `7` (INNER 4 + 부서 없는 사원 1 + 사원 없는 부서 2)

```sql
SELECT 부서.부서명, 사원.이름 FROM 사원 RIGHT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호;
```
출력: `개발 홍길동`, `영업 이순신`, `개발 강감찬`, `영업 김유신`, `인사 NULL`, `총무 NULL`

```sql
SELECT COUNT(*) FROM 사원 RIGHT OUTER JOIN 부서 ON 사원.부서번호 = 부서.부서번호
WHERE 사원.사번 IS NULL;
```
출력: `2` (사원이 한 명도 없는 부서 수)

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

### 기출에서 이렇게 나왔다

- 26-2: RIGHT OUTER JOIN 뒤 `IS NULL` 조건 COUNT → **2**
- 25-3, 21-3: CROSS JOIN 뒤 LIKE 조건 COUNT → **4** (25-3), **5** (21-3). 같은 유형
- 25-1: 묵시적 조인 + 조건 결과 표(헤더 포함) → 헤더 **이름, 급여**, 행 **이순신, 1000**

### 답 쓸 때

CROSS JOIN은 조합을 전부 나열하지 말고 "조건에 맞는 왼쪽 행 수 × 조건에 맞는 오른쪽 행 수"로 센다. 결과 표를 요구하면 헤더 이름을 SELECT 절에 적힌 그대로 쓴다.

## 결과 예측: 서브쿼리
출제: 26-2, 26-1, 24-3, 24-1
형태: 표 + 서브쿼리가 든 쿼리 → COUNT 또는 값

### 개념

| 종류 | 읽는 순서 |
|---|---|
| 단일 행 서브쿼리 `WHERE 급여 >= (SELECT AVG(급여) FROM 사원)` | 안쪽을 먼저 계산해 **값 하나**로 바꾼 뒤 바깥을 읽는다 |
| 다중 행 서브쿼리 `WHERE 학번 IN (SELECT ...)` | 안쪽을 **목록**으로 바꾼 뒤 바깥을 읽는다. ANY·ALL·EXISTS도 이 부류 |
| 상관 서브쿼리 `WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = S.부서번호)` | 안쪽이 바깥 행을 참조하므로 **바깥 행마다** 안쪽을 다시 계산한다 |
| 중첩 (서브쿼리 안에 GROUP BY·HAVING) | 가장 안쪽부터 결과 집합을 적어 두고 한 단계씩 바깥으로 |

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

```sql
SELECT 부서번호, AVG(급여) FROM 사원 GROUP BY 부서번호;
SELECT 이름, 부서번호, 급여 FROM 사원 S
WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = S.부서번호);
```
출력: 부서별 평균 `NULL 2000.0`, `10 3500.0`, `20 1750.0` / 상관 서브쿼리 결과 `강감찬 10 4000`, `김유신 20 2500` (각자 **자기 부서** 평균보다 높은 사람. 유관순은 부서번호가 NULL이라 `부서번호 = NULL` 비교가 거짓이 되어 평균이 NULL, 비교도 거짓)

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

```sql
SELECT 이름 FROM 학생
WHERE EXISTS (SELECT * FROM 수강 WHERE 수강.학번 = 학생.학번 AND 과목코드 = 'C1');
```
출력: `김철수`, `이영희`, `박민수`

### 기출에서 이렇게 나왔다

- 26-2: 상관 서브쿼리(AVG)와 IN이 섞인 COUNT → **3**
- 26-1: JOIN 뒤 AVG 서브쿼리 조건 COUNT → **2**
- 24-3: JOIN + GROUP BY·HAVING이 든 중첩 서브쿼리 COUNT → **1**
- 24-1: IN 서브쿼리 결과 값 → **a, b**

### 답 쓸 때

상관 서브쿼리는 바깥 표의 행마다 "이 행의 값으로 안쪽을 돌리면?"을 표 옆에 적어 가며 푼다. 단일 행 서브쿼리는 값을 먼저 구해 쿼리에 숫자로 바꿔 적어 두면 실수가 준다.

## 키워드 빈칸: DCL (GRANT·REVOKE)
출제: 21-3
형태: GRANT 문장의 예약어 빈칸

### 개념

DCL은 권한을 다루는 명령이다. sqlite는 사용자 개념이 없어 GRANT·REVOKE를 지원하지 않으므로 아래는 표준 문법만 적는다(실행 확인 안 함).

| 문장 | 완성 문장 | 뜻 |
|---|---|---|
| 권한 부여 | `GRANT SELECT ON 학생 TO USER1;` | USER1에게 학생 테이블 조회 권한 |
| 재부여 허용 | `GRANT UPDATE ON 학생 TO USER1 WITH GRANT OPTION;` | USER1이 그 권한을 남에게 다시 줄 수 있다 |
| 권한 회수 | `REVOKE SELECT ON 학생 FROM USER1;` | |
| 연쇄 회수 | `REVOKE UPDATE ON 학생 FROM USER1 CASCADE;` | USER1이 남에게 준 권한까지 함께 회수 |
| 재부여 권한만 회수 | `REVOKE GRANT OPTION FOR UPDATE ON 학생 FROM USER1;` | |

GRANT는 `TO`, REVOKE는 `FROM`이 짝이다. 권한 종류는 SELECT·INSERT·UPDATE·DELETE·ALL이다.

### 기출에서 이렇게 나왔다

- 21-3: `GRANT SELECT ON 학생 TO USER1` 빈칸

### 답 쓸 때

`GRANT ... ON ... TO`, `REVOKE ... ON ... FROM`. CASCADE는 REVOKE 쪽에만 붙는다.

## SQL 분류와 SELECT 처리 순서
출제: 없음

### 개념

| 분류 | 명령 | 특징 |
|---|---|---|
| **DDL(Data Definition Language)** | CREATE·ALTER·DROP·TRUNCATE | 스키마 정의. 자동 COMMIT |
| **DML(Data Manipulation Language)** | SELECT·INSERT·UPDATE·DELETE | 데이터 조작. 트랜잭션 단위로 되돌릴 수 있다 |
| **DCL(Data Control Language)** | GRANT·REVOKE | 권한 |
| **TCL(Transaction Control Language)** | COMMIT·ROLLBACK·SAVEPOINT | 트랜잭션 제어. DCL에 포함시키기도 한다 |

SELECT 문은 `SELECT [DISTINCT] 컬럼 FROM 테이블 [WHERE 조건] [GROUP BY 컬럼] [HAVING 그룹조건] [ORDER BY 컬럼 ASC|DESC]` 순서로 쓴다. 처리는 FROM(테이블 확보) → WHERE(행 선택) → GROUP BY(묶기) → HAVING(그룹 선택) → SELECT(열 선택·집계 계산) → ORDER BY(정렬) 순이다. 이 순서 때문에 WHERE에서는 그룹 함수를 쓸 수 없고, HAVING은 GROUP BY 없이는 뜻이 없다.

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

```sql
CREATE TABLE 직원(사번 INT, 이름 TEXT, 상사사번 INT);
INSERT INTO 직원 VALUES (1,'대표',NULL),(2,'팀장A',1),(3,'팀장B',1),(4,'사원A',2);
SELECT E.이름 AS 직원, M.이름 AS 상사 FROM 직원 E JOIN 직원 M ON E.상사사번 = M.사번;
```
출력: `팀장A 대표`, `팀장B 대표`, `사원A 팀장A` (상사가 없는 대표는 빠진다)

```sql
CREATE TABLE 부서(부서번호 INT, 부서명 TEXT);
INSERT INTO 부서 VALUES (10,'개발'),(20,'영업'),(30,'인사'),(40,'총무');
CREATE TABLE 사원(사번 INT, 이름 TEXT, 부서번호 INT, 급여 INT);
INSERT INTO 사원 VALUES (101,'홍길동',10,3000),(102,'이순신',20,1000),
  (103,'강감찬',10,4000),(104,'유관순',NULL,2000),(105,'김유신',20,2500);

SELECT 이름, 부서명 FROM 사원 NATURAL JOIN 부서;
SELECT * FROM 사원 NATURAL JOIN 부서 LIMIT 1;
```
출력: `홍길동 개발`, `이순신 영업`, `강감찬 개발`, `김유신 영업` / 전체 컬럼은 `사번 이름 부서번호 급여 부서명` 5개(부서번호가 한 번만)

```sql
CREATE TABLE A(값 INT);  CREATE TABLE B(값 INT);
INSERT INTO A VALUES (1),(2),(4),(5);
INSERT INTO B VALUES (2),(5),(8);

SELECT 값 FROM A INTERSECT SELECT 값 FROM B;
SELECT 값 FROM A EXCEPT SELECT 값 FROM B;
```
출력: `2, 5` / `1, 4`

## 뷰·인덱스·트랜잭션 명령
출제: 없음

### 개념

**뷰(View)**는 SELECT 문을 이름 붙여 저장한 가상 테이블이다. 물리적으로 데이터를 갖지 않고 조회할 때마다 원본에서 계산한다. 보안(필요한 열만 노출)과 편의(복잡한 조인을 단순 이름으로)가 목적이다. 원본 테이블을 DROP하면 뷰도 쓸 수 없고, 뷰를 통한 INSERT·UPDATE는 조인·집계가 없는 단순 뷰에서만 가능하다. 뷰는 ALTER로 바꿀 수 없고 DROP 뒤 다시 CREATE한다.

**인덱스(Index)**는 검색 속도를 위해 <키 값, 주소> 쌍을 따로 정렬해 둔 구조다. 조회는 빨라지지만 INSERT·UPDATE·DELETE마다 인덱스도 갱신해야 해서 쓰기 비용이 늘고 공간을 차지한다. PRIMARY KEY와 UNIQUE에는 자동으로 만들어진다.

**트랜잭션 명령**: `COMMIT`은 확정, `ROLLBACK`은 마지막 COMMIT 이후를 모두 취소, `SAVEPOINT 이름`은 중간 지점을 만들고 `ROLLBACK TO 이름`으로 그 지점까지만 되돌린다.

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

```sql
BEGIN;
DELETE FROM 사원;
ROLLBACK;
SELECT COUNT(*) FROM 사원;
```
출력: `5` (DELETE는 ROLLBACK으로 되돌아온다)
