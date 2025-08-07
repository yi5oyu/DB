# SQL(Structured Query Language)

    관계형 데이터베이스를 관리하고 조작하기 위한 표준 프로그래밍 언어

> MySQL 기준으로 작성

## SELECT

`SELECT 문`
```sql
-- 기본 구조
SELECT 컬럼명1, 컬럼명2, ...
FROM 테이블명
WHERE 조건
ORDER BY 정렬기준
LIMIT 개수;

-- 모든 컬럼 조회
SELECT * FROM 테이블명;

-- 특정 칼럼 조회
SELECT 컬럼명1, 컬럼명2, ...
FROM 테이블명;

-- AS 키워드(별칭)
SELECT 
    컬럼명1 AS 별칭1, -- 컬럼명1 별칭1, :생략가능
    컬럼명2 AS 별칭2, 
    컬럼명3 AS 별칭3  -- 컬럼명1 AS `별-칭 1`, :공백 특수문자는 백틱(`) 사용
FROM 테이블명;

-- 중복된 값 제거
SELECT DISTINCT 컬럼명1, 컬럼명2 
FROM 테이블명;
```

`WHERE 조건`

```sql
-- 기본 연산자 사용
SELECT * FROM 테이블명 WHERE 컬럼명1 = '값';
-- = != <> < > >= <=

-- 논리 연산자
SELECT * FROM 테이블명 
WHERE 컬럼명1 = '값1' AND 컬럼명2 > 100;
-- AND OR NOT

-- 범위
-- BETWEEN
SELECT * FROM 테이블명 
WHERE 컬럼명2 BETWEEN 100 AND 500;

-- IN/NOT IN
SELECT * FROM 테이블명 
WHERE 컬럼명1 IN ('값1', '값2', '값3');

-- 패턴 매칭
SELECT * FROM 테이블명 
WHERE 컬럼명1 LIKE '검색어%';
-- %검색어 검색어% %검색어% 검색_어(한글자 매칭)

-- NULL
SELECT * FROM 테이블명 
WHERE 컬럼명1 IS NULL;
-- IS IS NOT
```

`ORDER BY 정렬`

```sql
SELECT * FROM 테이블명 
ORDER BY 컬럼명1;
-- DESC ASC(기본값 오름차순)

-- 복합 정렬
SELECT * FROM 테이블명 
ORDER BY 컬럼명1 ASC, 컬럼명2 DESC, 컬럼명3 DESC;
-- 칼럼명1 오름차순 정렬 -> 칼럼명2 내림차순 정렬 -> 칼럼명3 내림차순 정렬

-- 별칭 사용 정렬
SELECT 
    컬럼명1,
    컬럼명2,
    컬럼명2 * 12 AS 계산컬럼
FROM 테이블명 
ORDER BY 계산컬럼 DESC;
```

`LIMIT 결과 제한`

```sql

```
