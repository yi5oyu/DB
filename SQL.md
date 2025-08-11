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
-- 상위 N개 결과만 조회
SELECT * FROM 테이블명 
ORDER BY 컬럼명1 DESC 
LIMIT 10;
```

## JOIN

`INNER JOIN`

```sql
-- 기본 구조
SELECT 컬럼명1, 컬럼명2, ...
FROM 테이블1 별칭1
INNER JOIN 테이블2 별칭2 ON 조인조건;

-- 일치하는 데이터, 추가 조건 조회
SELECT a.컬럼명1, b.컬럼명2
FROM 테이블1 a
JOIN 테이블2 b ON a.공통컬럼 = b.공통컬럼 -- INNER 생략 가능
WHERE a.컬럼명3 = '조건값';
```

`LEFT JOIN`

```sql
-- 왼쪽 테이블의 모든 데이터 + 오른쪽 테이블의 일치하는 데이터
SELECT a.컬럼명1, b.컬럼명2
FROM 테이블1 a
LEFT JOIN 테이블2 b ON a.공통컬럼 = b.공통컬럼;

-- 오른쪽 테이블에 없는 데이터만 조회
SELECT a.컬럼명1
FROM 테이블1 a
LEFT JOIN 테이블2 b ON a.공통컬럼 = b.공통컬럼
WHERE b.공통컬럼 IS NULL;
```

`RIGHT JOIN`

```sql
-- 오른쪽 테이블의 모든 데이터 + 왼쪽 테이블의 일치하는 데이터
SELECT a.컬럼명1, b.컬럼명2
FROM 테이블1 a
RIGHT JOIN 테이블2 b ON a.공통컬럼 = b.공통컬럼;

-- 왼쪽 테이블에 없는 데이터만 조회
SELECT b.컬럼명1
FROM 테이블1 a
RIGHT JOIN 테이블2 b ON a.공통컬럼 = b.공통컬럼
WHERE a.공통컬럼 IS NULL;
```

`다중 테이블 JOIN`

```sql
-- 3개 테이블 조인
SELECT a.컬럼명1, b.컬럼명2, c.컬럼명3
FROM 테이블1 a
JOIN 테이블2 b ON a.공통컬럼1 = b.공통컬럼1
JOIN 테이블3 c ON b.공통컬럼2 = c.공통컬럼2;

-- 혼합 조인
SELECT a.컬럼명1, b.컬럼명2, c.컬럼명3
FROM 테이블1 a
LEFT JOIN 테이블2 b ON a.공통컬럼1 = b.공통컬럼1
INNER JOIN 테이블3 c ON a.공통컬럼2 = c.공통컬럼2;
```

## 함수

`문자열 함수`

```sql
-- CONCAT: 문자열 연결
SELECT CONCAT(컬럼명1, ' ', 컬럼명2) AS 연결문자열
FROM 테이블명;

-- 대소문자 변환
-- UPPER: 대문자 변환
SELECT UPPER(컬럼명) AS 대문자 FROM 테이블명;
-- LOWER: 소문자 변환  
SELECT LOWER(컬럼명) AS 소문자 FROM 테이블명;

-- 문자열 추출
-- SUBSTRING: 부분 문자열 추출
SELECT SUBSTRING(컬럼명, 시작위치, 길이) AS 부분문자열
FROM 테이블명;
-- LEFT: 왼쪽부터 N글자
SELECT LEFT(컬럼명, 3) AS 왼쪽3글자 FROM 테이블명;
-- RIGHT: 오른쪽부터 N글자
SELECT RIGHT(컬럼명, 3) AS 오른쪽3글자 FROM 테이블명;

-- 문자열 검색
-- LOCATE: 문자열 위치 찾기
SELECT LOCATE('찾을문자', 컬럼명) AS 위치 FROM 테이블명;

-- 문자열 변경
-- REPLACE: 문자열 치환
SELECT REPLACE(컬럼명, '기존문자', '새문자') AS 변경결과
FROM 테이블명;
-- TRIM: 공백 제거
SELECT TRIM(컬럼명) AS 공백제거 FROM 테이블명;

-- 문자열 길이
-- LENGTH: 문자열 길이
SELECT LENGTH(컬럼명) AS 길이 FROM 테이블명;
```

---

`숫자 함수`

```sql
-- ROUND: 반올림
SELECT ROUND(컬럼명) AS 반올림 FROM 테이블명;           -- 소수점 첫째자리
SELECT ROUND(컬럼명, 2) AS 반올림2자리 FROM 테이블명;    -- 소수점 둘째자리

-- CEIL/CEILING: 올림
SELECT CEIL(컬럼명) AS 올림 FROM 테이블명;
-- FLOOR: 내림
SELECT FLOOR(컬럼명) AS 내림 FROM 테이블명;

-- ABS: 절댓값
SELECT ABS(컬럼명) AS 절댓값 FROM 테이블명;

-- MOD: 나머지
SELECT MOD(컬럼명, 3) AS 나머지 FROM 테이블명;
SELECT 컬럼명 % 3 AS 나머지 FROM 테이블명;

-- POWER: 거듭제곱
SELECT POWER(컬럼명, 2) AS 제곱 FROM 테이블명;

-- SQRT: 제곱근
SELECT SQRT(컬럼명) AS 제곱근 FROM 테이블명;
```

---

`날짜 함수`

```sql
-- 현재 날짜/시간
SELECT CURRENT_TIMESTAMP AS 현재시간;
SELECT CURRENT_DATE AS 현재날짜;
SELECT CURRENT_TIME AS 현재시간만;

-- 날짜 추출
SELECT 
    EXTRACT(YEAR FROM 날짜컬럼) AS 연도,
    EXTRACT(MONTH FROM 날짜컬럼) AS 월,
    EXTRACT(DAY FROM 날짜컬럼) AS 일
FROM 테이블명;

-- 날짜 계산
SELECT 날짜컬럼 + INTERVAL 1 DAY AS 하루후 FROM 테이블명;
SELECT 날짜컬럼 - INTERVAL 1 MONTH AS 한달전 FROM 테이블명;
SELECT 날짜컬럼 + INTERVAL 1 YEAR AS 일년후 FROM 테이블명;
```

---

`조건 함수`

```sql
-- CASE WHEN: 다중 조건
SELECT 
    컬럼명1,
    CASE 
        WHEN 컬럼명2 > 100 AND 컬럼명3 = 'Y' THEN '1등급'
        WHEN 컬럼명2 > 50 OR 컬럼명3 = 'Y' THEN '2등급'
        ELSE '3등급'
    END AS 등급
FROM 테이블명;

-- 단순 CASE (특정 값과 비교)
SELECT 
    컬럼명1,
    CASE 컬럼명2
        WHEN 'A' THEN '우수'
        WHEN 'B' THEN '보통'
        WHEN 'C' THEN '미흡'
        ELSE '기타'
    END AS 평가
FROM 테이블명;

-- COALESCE: 첫 번째 NULL이 아닌 값 반환
SELECT 
    컬럼명1,
    COALESCE(컬럼명2, 컬럼명3, '기본값') AS 결과
FROM 테이블명;

-- NULL 처리
SELECT 
    컬럼명1,
    CASE WHEN 컬럼명2 IS NULL THEN '기본값' ELSE 컬럼명2 END AS 결과
FROM 테이블명;
```

---

`집계 함수`

```sql
-- COUNT: 개수
SELECT COUNT(*) AS 전체개수 FROM 테이블명;
SELECT COUNT(컬럼명) AS NULL제외개수 FROM 테이블명;
SELECT COUNT(DISTINCT 컬럼명) AS 중복제거개수 FROM 테이블명;

-- SUM: 합계
SELECT SUM(컬럼명) AS 합계 FROM 테이블명;

-- AVG: 평균
SELECT AVG(컬럼명) AS 평균 FROM 테이블명;

-- MIN/MAX: 최솟값/최댓값
SELECT 
    MIN(컬럼명) AS 최솟값,
    MAX(컬럼명) AS 최댓값
FROM 테이블명;
```

## GROUP BY

```sql
-- 단일 컬럼 그룹화
SELECT 
    컬럼명1,
    COUNT(*) AS 개수
FROM 테이블명
GROUP BY 컬럼명1;

-- 다중 컬럼 그룹화
SELECT 
    컬럼명1,
    컬럼명2,
    COUNT(*) AS 개수,
    SUM(컬럼명3) AS 합계
FROM 테이블명
GROUP BY 컬럼명1, 컬럼명2;

-- 소계와 총계 자동 생성
SELECT 
    컬럼명1,
    컬럼명2,
    COUNT(*) AS 개수
FROM 테이블명
GROUP BY 컬럼명1, 컬럼명2 WITH ROLLUP;

-- GROUPING 함수로 구분
SELECT 
    CASE 
        WHEN GROUPING(컬럼명1) = 1 THEN '총계'
        WHEN GROUPING(컬럼명2) = 1 THEN CONCAT(컬럼명1, ' 소계')
        ELSE 컬럼명1
    END AS 구분,
    컬럼명2,
    COUNT(*) AS 개수
FROM 테이블명
GROUP BY 컬럼명1, 컬럼명2 WITH ROLLUP;
```

`HAVING`

```sql
-- 집계 결과 조건
SELECT 
    컬럼명1,
    COUNT(*) AS 개수
FROM 테이블명
GROUP BY 컬럼명1
HAVING COUNT(*) > 10;

-- 여러 조건
SELECT 
    컬럼명1,
    COUNT(*) AS 개수,
    AVG(컬럼명2) AS 평균
FROM 테이블명
GROUP BY 컬럼명1
HAVING COUNT(*) > 5 AND AVG(컬럼명2) > 100;

-- WHERE와 HAVING 함께 사용
SELECT 
    컬럼명1,
    COUNT(*) AS 개수
FROM 테이블명
WHERE 컬럼명2 > 1000  -- 그룹화 전 필터링
GROUP BY 컬럼명1
HAVING COUNT(*) > 5;  -- 그룹화 후 필터링
```

## 서브쿼리

```sql
-- 기본 구조
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1
WHERE 컬럼명3 = (SELECT 컬럼명3 FROM 테이블명2 WHERE 조건);

-- 여러 값과 비교
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1
WHERE 컬럼명3 IN (SELECT 컬럼명3 FROM 테이블명2 WHERE 조건);

-- 특정 조건에 해당하지 않는 데이터
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1
WHERE 컬럼명3 NOT IN (SELECT 컬럼명3 FROM 테이블명2 WHERE 컬럼명3 IS NOT NULL);

-- 존재하는 경우만 조회
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1 a
WHERE EXISTS (SELECT 1 FROM 테이블명2 b WHERE a.공통컬럼 = b.공통컬럼);

-- 존재하지 않는 경우만 조회
SELECT 컬럼명1, 컬럼명2
FROM 테이블명1 a
WHERE NOT EXISTS (SELECT 1 FROM 테이블명2 b WHERE a.공통컬럼 = b.공통컬럼);

-- 추가 정보 조회
SELECT 
    컬럼명1,
    컬럼명2,
    (SELECT 컬럼명3 FROM 테이블명2 WHERE 테이블명2.공통컬럼 = 테이블명1.공통컬럼) AS 추가정보
FROM 테이블명1;

-- 집계 결과를 테이블처럼 사용
SELECT 별칭.그룹컬럼, 별칭.총합계
FROM (
    SELECT 
        컬럼명1 AS 그룹컬럼,
        SUM(컬럼명2) AS 총합계
    FROM 테이블명1
    GROUP BY 컬럼명1
) 별칭
WHERE 별칭.총합계 > 1000;
```
