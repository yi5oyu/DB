# SQL(Structured Query Language)

    관계형 데이터베이스를 관리하고 조작하기 위한 표준 프로그래밍 언어

## 주요 기능

### DDL(Data Definition Language)
 
    데이터 정의어: 데이터베이스 구조를 정의하고 수정하는 명령어
    
    CREATE: 테이블, 뷰, 인덱스 등을 생성
    ALTER: 기존 구조를 변경
    DROP: 객체를 삭제
    TRUNCATE: 테이블의 모든 데이터를 삭제

### DML(Data Manipulation Language)

    데이터 조작어: 데이터를 조작하는 명령어

    SELECT: 데이터 조회
    INSERT: 새로운 데이터 삽입
    UPDATE: 기존 데이터 수정
    DELETE: 데이터 삭제

### DCL(Data Control Language)

    데이터 제어어: 데이터베이스 접근 권한을 관리

    GRANT: 권한 부여
    REVOKE: 권한 회수

### TCL(Transaction Control Language)

    트랜잭션 제어어: 트랜잭션을 관리

    COMMIT: 변경사항 확정
    ROLLBACK: 변경사항 취소
    SAVEPOINT: 트랜잭션 내 저장점 설정

## SELECT

```sql
-- 모든 컬럼 조회
SELECT * FROM 테이블명;

-- 특정 칼럼 조회
SELECT 컬럼명1, 컬럼명2, ...
FROM 테이블명;

-- AS 키워드(별칭)
SELECT 
    emp_name AS 사원명,
    salary AS 급여,
    dept_name AS 부서명
FROM employees;


```
