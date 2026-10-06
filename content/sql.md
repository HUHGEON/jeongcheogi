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

`LIKE`에서 `%`는 0글자 이상을, `_`는 정확히 한 글자를 뜻한다. 그래서 `'%철%'`은 "철이 포함된" 값이고, `'_철'`은 "철 앞에 딱 한 글자"가 오는 값이다.

서브쿼리 앞에 붙는 비교 연산자도 짚고 가자. `> ALL`은 서브쿼리 결과의 모든 값보다 커야 하니 결국 **최댓값보다** 커야 참이다. 반면 `> ANY`는 그중 하나보다만 크면 되니 **최솟값보다** 크기만 하면 참이다. 방향을 뒤집으면 `< ALL`은 최솟값보다 작아야 참이고, `< ANY`는 최댓값보다 작으면 참이다. 이름이 달라 혼동하기 쉬운데 **ANY와 SOME은 같은 뜻**이다. 같은 식으로 `IN`은 `= ANY`와, `NOT IN`은 `<> ALL`과 같다. `EXISTS`는 성격이 조금 다르다. 값을 비교하지 않고, 서브쿼리가 행을 하나라도 돌려주는지만 본다.

참고로 sqlite는 `ALL`과 `ANY`를 지원하지 않아서 아래 예제에서는 `MAX`/`MIN` 비교로 바꿔 실행했다. 서브쿼리 결과가 비어 있지 않고 NULL이 없으면 두 방식의 결과는 같고, PostgreSQL에서 `> ALL`/`> ANY` 원문으로 실행해도 같은 결과가 나온다.

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
-- 급여 > ALL (SELECT 급여 FROM 사원 WHERE 부서번호 = 20) 과 결과가 같다 (서브쿼리가 비어 있지 않고 NULL이 없을 때)
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

대소문자는 구분하지 않지만, 답안에는 예약어를 대문자로 쓰는 쪽이 안전하다. 문자열은 `'이%'`처럼 작은따옴표 안에 넣는다. DESC와 ASC를 뒤집어 쓰지 않으려면 "내림차순 = DESC = Descending"으로 묶어서 기억해 두면 된다.

## 결과 예측: DISTINCT 튜플 수, UNION, ON DELETE CASCADE
출제: 26-1, 23-3, 22-3, 20-1
형태: 쿼리 2~3개의 결과 행 수 / UNION 결과 표 / 삭제 후 COUNT

### 개념

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

| 문장 | 완성 문장 | 주의 |
|---|---|---|
| 행 추가 | `INSERT INTO 교수(교수번호, 이름, 나이) VALUES (1, '김교수', 45);` | 컬럼 목록을 생략하면 모든 컬럼을 순서대로 |
| 조회 결과 추가 | `INSERT INTO 교수백업 SELECT 교수번호, 이름 FROM 교수 WHERE 나이 >= 46;` | VALUES 대신 SELECT. 괄호 없음 |
| 수정 | `UPDATE 교수 SET 나이 = 46 WHERE 교수번호 = 1;` | WHERE가 없으면 전체 행이 바뀐다 |
| 삭제 | `DELETE FROM 교수 WHERE 교수번호 = 3;` | WHERE가 없으면 전체 행 삭제. 테이블은 남는다 |

여기서 DELETE, DROP, TRUNCATE를 혼동하기 쉽다. DELETE는 행만 지우는 DML이다. 로그를 남기기 때문에 ROLLBACK으로 되돌릴 수 있다. 반면 DROP은 테이블 자체를 없애는 DDL이다. 애매한 건 TRUNCATE인데, 행을 모두 지우지만 분류는 DDL이다.

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

INSERT는 `INTO`, UPDATE는 `SET`, DELETE는 `FROM`이 각각 짝이라고 외워 두면 된다. 다만 `INSERT INTO ... SELECT`처럼 조회 결과를 넣을 때는 VALUES를 쓰지 않는다.

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
| 컬럼 변경·삭제 | `ALTER TABLE 교수 ALTER 나이 SET DEFAULT 30;` (표준 SQL, 기본값 변경) / 자료형 변경은 Oracle·MySQL `ALTER TABLE 교수 MODIFY 나이 INT;`, 표준 SQL `ALTER TABLE 교수 ALTER 나이 SET DATA TYPE INT;` / `ALTER TABLE 교수 DROP COLUMN 전화;` (DROP COLUMN은 sqlite에서, ALTER는 PostgreSQL에서 실행 확인. MODIFY는 실행 확인 안 함) |
| 인덱스 | `CREATE INDEX idx_교수_이름 ON 교수(이름);` |
| 뷰 | `CREATE VIEW 고령교수 AS SELECT 이름, 나이 FROM 교수 WHERE 나이 >= 45;` |
| 도메인 | `CREATE DOMAIN 성별 CHAR(1) DEFAULT '남' CONSTRAINT 성별제약 CHECK (VALUE IN ('남', '여'));` (표준 문법, sqlite 미지원. PostgreSQL에서 실행 확인) |
| 삭제 | `DROP TABLE 교수 CASCADE;` / `DROP VIEW 고령교수;` / `DROP INDEX idx_교수_이름;` |

DROP 뒤에 붙는 옵션도 짚고 가자. **DROP ... CASCADE**는 지우려는 객체만이 아니라 그 객체를 참조하는 다른 객체(뷰·외래키)까지 함께 삭제한다. 반대로 **RESTRICT**는 참조하는 객체가 있으면 삭제를 막는다. 참고로 sqlite는 CASCADE 옵션을 받지 않아서 아래에서는 옵션 없이 실행했고, `DROP TABLE ... CASCADE`의 동작은 PostgreSQL에서 실행해 확인했다.

그럼 제약조건을 어기면 실제로 어떤 오류가 날까? 위 교수 테이블에 1·2번 교수가 들어 있고 학과는 D1·D2만 있는 상태에서 하나씩 어겨 보면 이렇다.

```sql
INSERT INTO 교수(교수번호, 이름, 나이) VALUES (4,'최교수',19);         -- CHECK 위반
INSERT INTO 교수(교수번호, 이름, 이메일) VALUES (5,'정교수','kim@x.kr'); -- UNIQUE 위반
INSERT INTO 교수(교수번호, 이름, 학과코드) VALUES (6,'한교수','D9');     -- FOREIGN KEY 위반
INSERT INTO 교수(교수번호, 이름) VALUES (1,'중복');                     -- PRIMARY KEY 위반
INSERT INTO 교수(교수번호) VALUES (7);                                  -- NOT NULL 위반
```
출력: 다섯 문장 모두 오류로 거부된다 (`CHECK constraint failed`, `UNIQUE constraint failed: 교수.이메일`, `FOREIGN KEY constraint failed`, `UNIQUE constraint failed: 교수.교수번호`, `NOT NULL constraint failed: 교수.이름`) (참고: sqlite는 호환성 때문에 `INT PRIMARY KEY` 컬럼에 NULL을 넣어도 거부하지 않는다. 표준 SQL과 다른 DBMS에서는 기본키 NULL이 개체 무결성 위반으로 거부된다.)

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

외래키 제약은 `CONSTRAINT 이름 FOREIGN KEY (컬럼) REFERENCES 부모테이블(컬럼)` 순서로 쓴다. 헷갈리는 건 REFERENCES 뒤인데, 여기에는 테이블명이 오고 그 괄호 안에 부모의 컬럼이 온다. CASCADE와 RESTRICT도 혼동하지 않도록 한다. 참조하는 객체까지 같이 지우는 쪽이 CASCADE, 삭제를 막는 쪽이 RESTRICT다.

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
| `NULL` 비교 | `= NULL`, `<> NULL`의 결과는 거짓이 아니라 UNKNOWN이라 WHERE를 통과하는 행이 없다(`NOT (컬럼 = NULL)`도 0행). NULL 검사는 `IS NULL`·`IS NOT NULL`로만 한다. `<>` 조건 하나만 있으면 NULL 행은 빠진다(OR로 다른 참 조건이 붙으면 통과한다) |
| NULL이 낀 OR·AND | NULL과의 비교(`=`, `IN`, `>` 등)는 UNKNOWN이지만 그 행을 바로 버리지 않는다. **UNKNOWN OR 참 = 참**, UNKNOWN OR 거짓 = UNKNOWN, UNKNOWN AND 참 = UNKNOWN, UNKNOWN AND 거짓 = 거짓이다. WHERE는 최종 결과가 참인 행만 남긴다 |

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
- 25-3 풀이: 행마다 따져 보면 (2,NULL)·(3,6)·(2,3)은 `C1 IN(2,3)`이 참이고, (4,5)는 `C2 IN(3,5)`가 참이다. 가장 헷갈리는 건 (NULL,5)다. `C1 IN(2,3)`은 UNKNOWN이지만 OR로 붙은 `C2 IN(3,5)`가 참이라서 통과한다. 이렇게 통과한 5행 중 C2가 NULL인 (2,NULL)은 `COUNT(C2)`가 빼고 세므로 답은 **4**다. 여기서 (NULL,5)까지 빼 버리면 3이라는 오답이 나온다.
- 24-1, 21-1: 괄호 없는 `OR ... AND` 조건의 COUNT → **1** (같은 유형)

### 답 쓸 때

쿼리는 두 단계로 읽는다. 먼저 WHERE 조건으로 행을 고르는데, 이때 AND가 먼저 묶인다는 걸 잊으면 안 된다. 그다음에 COUNT 괄호 안을 본다. `COUNT(*)`가 아니라면 고른 행에서 해당 컬럼이 NULL인 행을 지운 뒤 센다.

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

| 종류 | 읽는 순서 |
|---|---|
| 단일 행 서브쿼리 `WHERE 급여 >= (SELECT AVG(급여) FROM 사원)` | 안쪽을 먼저 계산해 **값 하나**로 바꾼 뒤 바깥을 읽는다 |
| 다중 행 서브쿼리 `WHERE 학번 IN (SELECT ...)` | 안쪽을 **목록**으로 바꾼 뒤 바깥을 읽는다. ANY·ALL·EXISTS도 이 부류 |
| 상관 서브쿼리 `WHERE 급여 > (SELECT AVG(급여) FROM 사원 WHERE 부서번호 = S.부서번호)` | 안쪽이 바깥 행을 참조하므로 **바깥 행마다** 안쪽을 다시 계산한다 |
| 중첩 (서브쿼리 안에 GROUP BY·HAVING) | 가장 안쪽부터 결과 집합을 적어 두고 한 단계씩 바깥으로 |

여기서 많이들 놓치는 게 빈 서브쿼리다. 서브쿼리 안에서 집계할 행이 하나도 없으면 COUNT는 0을 돌려주지만, SUM·AVG·MAX·MIN은 0이 아니라 NULL을 돌려준다. 그러면 바깥의 `x > NULL`은 UNKNOWN이 되고, 그 행은 결과에서 빠진다.

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

```sql
SELECT COUNT(*) FROM 부서 WHERE 부서번호 NOT IN (SELECT 부서번호 FROM 사원);
SELECT COUNT(*) FROM 부서 D WHERE NOT EXISTS (SELECT 1 FROM 사원 S WHERE S.부서번호 = D.부서번호);
```
출력: `0`, `2` (사원 부서번호에 NULL(유관순)이 섞여 있으면 `NOT IN`은 `<> NULL` 비교가 UNKNOWN이 되어 어떤 행도 남지 않는다. NOT EXISTS는 NULL과 상관없이 사원 없는 부서 2개를 센다)

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

### 기출에서 이렇게 나왔다

- 21-3: `GRANT SELECT ON 학생 TO USER1` 빈칸

### 답 쓸 때

문장 뼈대는 `GRANT ... ON ... TO`, `REVOKE ... ON ... FROM` 두 개만 기억하면 된다. CASCADE는 REVOKE 쪽에만 붙는다. 위 표에서 본 것처럼 USER1이 남에게 준 권한까지 함께 회수할 때 쓰는 옵션이다.

## SQL 분류와 SELECT 처리 순서
출제: 없음

### 개념

| 분류 | 명령 | 특징 |
|---|---|---|
| **DDL(Data Definition Language)** | CREATE·ALTER·DROP·TRUNCATE | 스키마 정의. 자동 COMMIT |
| **DML(Data Manipulation Language)** | SELECT·INSERT·UPDATE·DELETE | 데이터 조작. 트랜잭션 단위로 되돌릴 수 있다. NCS 학습모듈은 SELECT를 **DQL(Data Query Language, 데이터 질의어)**로 따로 부르기도 한다 |
| **DCL(Data Control Language)** | GRANT·REVOKE (국내 시험은 COMMIT·ROLLBACK도 포함) | 권한, 그리고 무결성·병행 제어·회복 |
| **TCL(Transaction Control Language)** | COMMIT·ROLLBACK·SAVEPOINT | 트랜잭션 제어. **국내 시험(NCS·필기)은 COMMIT·ROLLBACK을 DCL로 분류한다.** "DCL 명령어"를 물으면 GRANT·REVOKE·COMMIT·ROLLBACK |

SELECT 문은 `SELECT [DISTINCT] 컬럼 FROM 테이블 [WHERE 조건] [GROUP BY 컬럼] [HAVING 그룹조건] [ORDER BY 컬럼 ASC|DESC]` 순서로 쓴다. 하지만 처리되는 순서는 다르다. 먼저 FROM으로 테이블을 확보하고 WHERE로 행을 고른 뒤, GROUP BY로 묶고 HAVING으로 그룹을 고른다. 그러고 나서야 SELECT에서 열을 고르고 집계를 계산하며, 마지막으로 ORDER BY로 정렬한다. WHERE에서 그룹 함수를 쓸 수 없는 것도 이 순서 때문이다. WHERE가 처리될 때는 아직 묶인 그룹이 없다. 한 가지 더, HAVING은 보통 GROUP BY와 함께 쓰지만 GROUP BY 없이 쓰면 테이블 전체를 한 그룹으로 본다.

```sql
-- 사원 테이블(5행)
SELECT COUNT(*) FROM 사원 HAVING COUNT(*) >= 5;
SELECT COUNT(*) FROM 사원 HAVING COUNT(*) >= 6;
```
출력: `5` / 행 없음 (전체 5행이 한 그룹)

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
출력: `홍길동 개발`, `이순신 영업`, `강감찬 개발`, `김유신 영업` / 전체 컬럼은 5개이고 부서번호는 한 번만 나온다. 표준 SQL(Oracle·PostgreSQL)은 공통 컬럼을 맨 앞에 두어 `부서번호 사번 이름 급여 부서명` 순서다(sqlite만 `사번 이름 부서번호 급여 부서명` 순서로 보여 준다).

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

**뷰(View)**란 하나 이상의 기본 테이블에서 유도한 SELECT 문에 이름을 붙여 저장해 둔 가상 테이블을 말한다. 쉽게 말해 데이터가 아니라 쿼리를 저장해 두는 것이다. 뷰는 물리적으로 데이터를 갖지 않고 조회할 때마다 원본에서 계산한다. 그래서 원본이 바뀌면 뷰 결과도 바로 바뀌고, 원본 테이블을 DROP하면 뷰도 쓸 수 없다. 뷰를 바탕으로 또 다른 뷰를 정의할 수도 있는데, 이때도 기초가 된 테이블이나 뷰를 지우면 그 위에 정의된 뷰는 쓸 수 없다.

| 장점 | 단점 |
|---|---|
| **논리적 데이터 독립성**: 기본 테이블 구조가 바뀌어도 뷰로 접근하는 응용은 영향이 적다 | 뷰에는 독립적인 **인덱스를 만들 수 없다** |
| **보안**: 필요한 열·행만 사용자에게 보인다 | **ALTER로 정의를 바꿀 수 없다**. DROP 뒤 다시 CREATE한다 |
| **편의**: 복잡한 조인·조건을 이름 하나로 조회한다 | **삽입·갱신·삭제 제약**: 조인·집계 함수·DISTINCT·GROUP BY가 든 뷰는 갱신할 수 없다 |

**WITH CHECK OPTION**은 뷰를 정의한 WHERE 조건을 벗어나는 INSERT·UPDATE를 거부하는 옵션이다. 예를 들어 `CREATE VIEW 개발부 AS SELECT * FROM 사원 WHERE 부서번호 = 10 WITH CHECK OPTION;`으로 뷰를 만들었다고 해보자. 그러면 이 뷰로는 부서번호 20인 행을 넣을 수도, 부서번호를 20으로 바꿀 수도 없다. sqlite는 이 옵션을 지원하지 않아 PostgreSQL에서 실행해 확인했다.

**인덱스(Index)**는 검색 속도를 높이려고 <키 값, 주소> 쌍을 따로 정렬해 둔 구조다. 대신 대가가 있다. 조회는 빨라지지만 INSERT·UPDATE·DELETE를 할 때마다 인덱스도 함께 갱신해야 하니 쓰기 비용이 늘고, 공간도 따로 차지한다. PRIMARY KEY와 UNIQUE에는 인덱스가 자동으로 만들어진다.

**트랜잭션 명령**도 정리해 두자. `COMMIT`은 확정하고, `ROLLBACK`은 마지막 COMMIT 이후를 모두 취소한다. `SAVEPOINT 이름`은 중간 지점을 만들어 두는 명령인데, `ROLLBACK TO 이름`을 쓰면 마지막 COMMIT까지가 아니라 그 지점까지만 되돌린다.

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

## 내장 함수·윈도우 함수
출제: 없음

### 개념

**단일 행 함수**는 행 하나마다 계산해 값 하나를 돌려주기 때문에 입력 행 수와 출력 행 수가 같다. 반면 **집계(다중 행) 함수**(COUNT·SUM·AVG·MAX·MIN)는 여러 행을 묶어 값 하나로 줄인다.

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

```sql
SELECT 이름, 반, 점수, SUM(점수) OVER (PARTITION BY 반) AS 반합계 FROM 성적 ORDER BY 반, 점수 DESC;
```
출력: `가 A 90 320`, `나 A 80 320`, `다 A 80 320`, `라 A 70 320`, `마 B 85 145`, `바 B 60 145` (6행이 그대로 남고 반별 합계가 옆에 붙는다. `GROUP BY 반`이었다면 `A 320`, `B 145` 두 행)

**그룹 함수**는 GROUP BY 뒤에 써서 소계·총계 행을 함께 만들어 주는 함수다. 세 가지가 헷갈리기 쉬우니 하나씩 보자. **ROLLUP(A, B)**는 (A, B)별 소계와 A별 소계, 그리고 총계를 만든다. 컬럼이 n개면 n+1단계가 나오고, 컬럼 순서가 바뀌면 결과도 바뀐다. **CUBE(A, B)**는 가능한 모든 조합인 (A, B)·A·B·전체의 소계를 만든다. **GROUPING SETS((A), (B))**는 지정한 묶음별 집계만 만들고, 컬럼 순서와도 상관없다. 예를 들어 위 성적 표에서 `SELECT 반, SUM(점수) FROM 성적 GROUP BY ROLLUP(반) ORDER BY 반;`을 실행하면 `A 320`, `B 145`, `NULL 465`가 나오는데, 마지막 행이 총계다. sqlite는 ROLLUP을 지원하지 않아 PostgreSQL에서 실행해 확인했다.

## 프로시저·트리거·사용자 정의 함수·커서
출제: 없음

### 개념

| 객체 | 뜻 | 구별 단서 |
|---|---|---|
| **저장 프로시저(Stored Procedure)** | 자주 쓰는 SQL 묶음을 미리 컴파일해 DB에 저장해 두고 호출해 쓰는 것 | 반환값 없이 OUT 매개변수로 결과를 넘긴다. `EXECUTE`로 호출 |
| **사용자 정의 함수(UDF)** | 사용자가 만든 함수. 반드시 값 하나를 **RETURN**한다 | SELECT 문 안에서 호출할 수 있다 |
| **트리거(Trigger)** | INSERT·UPDATE·DELETE 같은 이벤트가 일어나면 **자동으로** 실행되는 프로시저 | 사용자 정의 무결성 구현, 변경 이력 기록 |
| **커서(Cursor)** | 여러 행으로 된 SELECT 결과를 한 행씩 가리키며 처리하는 포인터 | 명시적 커서는 **DECLARE → OPEN → FETCH → CLOSE** |

커서는 누가 만드느냐에 따라 둘로 나뉜다. DBMS가 일반 SQL을 실행할 때 알아서 만드는 것이 **묵시적 커서(Implicit)**이고, 개발자가 직접 선언·제어하는 것이 **명시적 커서(Explicit)**다. 프로시저·UDF·커서는 sqlite에 없어서 실행하지 않았고, 트리거만 sqlite로 확인했다.

프로시저의 매개변수 모드는 **IN**(입력)·**OUT**(출력)·**INOUT**(입출력) 세 가지다. 트리거는 이와 달리 매개변수(IN·OUT)도 반환값도 없다. 대신 어떤 이벤트(INSERT·UPDATE·DELETE)에, 어느 시점(**BEFORE·AFTER**)에, 행마다 실행할지(**FOR EACH ROW**)로 정의한다. 그리고 트리거 안에서는 **COMMIT·ROLLBACK을 쓸 수 없다**는 점도 기억해 두자. 변경 전후 값은 Oracle에서는 `:OLD`·`:NEW`로, SQL Server에서는 `deleted`·`inserted`로 참조한다. INSERT라면 변경 전 값이 없으니 OLD가 NULL이고, DELETE라면 변경 후 값이 없으니 NEW가 NULL이다.

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
