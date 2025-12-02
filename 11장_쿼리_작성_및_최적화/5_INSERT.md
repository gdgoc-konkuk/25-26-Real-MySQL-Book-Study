# 6절. UPDATE 와 DELETE

- 기본적으로 **한 테이블의 여러 레코드를 변경/삭제**하는 문장.
- MySQL은 여러 테이블을 조인해서 한 번에 변경/삭제하는 **JOIN UPDATE / JOIN DELETE** 도 지원 → 잘못된 데이터 일괄 수정·삭제에 유용.

---

## 11.6.1 UPDATE … ORDER BY … LIMIT n

- WHERE 조건에 맞는 모든 레코드를 고치지 않고,

  **정렬 기준 + LIMIT 개수만큼만 순서대로 변경/삭제** 가능.


```sql
DELETE FROM employees
ORDER BY last_name
LIMIT 10;
```

- 너무 많은 레코드를 한 번에 바꾸면 부하가 큰데, 이 방식으로 **조금씩 잘라 처리**할 수 있음.
- 단, **STATEMENT 기반 복제**(binlog_format=STATEMENT)에서는 `ORDER BY ... LIMIT` 포함 문장이 “어떤 행이 지워질지 확정 못 한다”고 **unsafe 경고** 발생 → 가능하면 ROW 기반 복제를 쓰거나 조심해서 사용.

---

## 11.6.2 JOIN UPDATE

- **여러 테이블을 조인해서 한 번에 UPDATE** 하는 문장.

```sql
UPDATE tb_test1 t1, employees e
SET t1.first_name = e.first_name
WHERE e.emp_no = t1.emp_no;
```

- 웹 서비스 OLTP에서 잦게 쓰면
    - 조인되는 모든 테이블에 락·부하가 커질 수 있으니
    - **배치/통계성 작업** 위주로 사용하는 게 좋다.
- JOIN UPDATE도 결국 “조인 쿼리”이므로, **조인 순서에 따라 성능이 달라짐** → 실행 계획 확인 필수.

### GROUP BY가 섞인 JOIN UPDATE

- “부서별 사원 수를 departments.emp_count에 저장” 같은 집계-갱신은 서브쿼리(파생 테이블) 로 한 번 더 싸서 처리.

```sql
UPDATE departments d,
       ( SELECT dept_no, COUNT(*) AS emp_count
         FROM dept_emp
         GROUP BY dept_no ) dc
SET d.emp_count = dc.emp_count
WHERE d.dept_no = dc.dept_no;
```

- 조인 순서를 강제로 정하고 싶다면
    - `STRAIGHT_JOIN` 사용, 혹은
    - MySQL 8.0의 `JOIN_ORDER` 옵티마이저 힌트 사용.
- 파생 테이블 대신 **LATERAL JOIN** 으로도 구현 가능 (동일한 아이디어).

---

## 11.6.3 여러 레코드 UPDATE (Row Constructor)

- MySQL 8.0부터 **VALUES ROW(...)** 문법으로 한 쿼리 안에서 “임시 테이블”처럼 여러 값 묶음을 만들고 JOIN UPDATE 가능.

```sql
UPDATE user_level ul
INNER JOIN (
    ROW(1, 1),
    ROW(2, 4)
) new_user_level (user_id, user_lv)
ON new_user_level.user_id = ul.user_id
SET ul.user_lv = ul.user_lv + new_user_level.user_lv;
```

- 장점:
    - **각 레코드마다 다른 값**을 한 번에 갱신할 수 있음
    - 별도로 임시 테이블을 만들지 않아도 됨 → 간단한 “미니 배치 UPDATE” 용도.

---

## 11.6.4 JOIN DELETE

- 여러 테이블을 조인한 뒤, **그중 일부(또는 전부) 테이블의 레코드만 삭제**하는 문장.

### 한 테이블만 삭제

```sql
DELETE e
FROM employees e
JOIN dept_emp de  ON e.emp_no = de.emp_no
JOIN departments d ON d.dept_no = de.dept_no
WHERE d.dept_no = 'd001';
```

- `FROM` 뒤에는 조인에 참여하는 모든 테이블을 쓰고,

  `DELETE` 뒤에는 **실제로 지울 테이블의 별칭만** 적는다 (`e`).


### 여러 테이블 동시에 삭제

```sql
DELETE e, de, d
FROM employees e
JOIN dept_emp de  ON e.emp_no = de.emp_no
JOIN departments d ON d.dept_no = de.dept_no
WHERE d.dept_no = 'd001';
```

- employees / dept_emp / departments 세 테이블에서 관련 행을 한 번에 삭제.

### 조인 순서 제어

- JOIN UPDATE와 마찬가지로 실행 계획이 중요 → 옵티마이저가 마음에 안 들면:

```sql
DELETE /*+ JOIN_ORDER (d, de, e) */ e, de, d
FROM departments d
INNER JOIN dept_emp de  ON de.dept_no = d.dept_no
INNER JOIN employees e  ON e.emp_no  = de.emp_no
WHERE d.dept_no = 'd001';
```

- 또는 `STRAIGHT_JOIN` 으로 조인 순서를 고정할 수 있음.