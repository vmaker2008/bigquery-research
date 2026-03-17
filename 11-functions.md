---
layout: default
title: 11. BigQuery 자주 쓰는 함수
nav_order: 12
---

# 11. BigQuery 자주 쓰는 함수

[← 목차로 돌아가기](index)

---

## 목차
- [날짜/시간 함수](#날짜시간-함수)
- [문자열 함수](#문자열-함수)
- [집계 함수](#집계-함수)
- [윈도우 함수](#윈도우-함수)
- [조건/분기 함수](#조건분기-함수)
- [배열 함수](#배열-함수)
- [JSON 함수](#json-함수)
- [수학 함수](#수학-함수)
- [타입 변환 함수](#타입-변환-함수)
- [NULL 처리 함수](#null-처리-함수)

---

## 날짜/시간 함수

### CURRENT_DATE / CURRENT_TIMESTAMP
```sql
-- 오늘 날짜
SELECT CURRENT_DATE();                      -- 2025-03-17
SELECT CURRENT_TIMESTAMP();                -- 2025-03-17 13:00:00 UTC
SELECT CURRENT_DATETIME('Asia/Seoul');     -- 2025-03-17T22:00:00
```

### DATE / DATETIME / TIMESTAMP 생성
```sql
-- 날짜 생성
SELECT DATE(2025, 3, 17);                  -- 2025-03-17
SELECT DATE(timestamp_col, 'Asia/Seoul');  -- 타임스탬프 → 한국 날짜 변환

-- 타임스탬프 생성
SELECT TIMESTAMP('2025-03-17 09:00:00', 'Asia/Seoul');
```

### DATE_ADD / DATE_SUB — 날짜 계산
```sql
-- 7일 후
SELECT DATE_ADD(CURRENT_DATE(), INTERVAL 7 DAY);

-- 3개월 전
SELECT DATE_SUB(CURRENT_DATE(), INTERVAL 3 MONTH);

-- 타임스탬프 1시간 후
SELECT TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR);
```

### DATE_DIFF — 날짜 차이 계산
```sql
-- 가입일로부터 며칠이 지났는지
SELECT
  user_id,
  DATE_DIFF(CURRENT_DATE(), signup_date, DAY) AS days_since_signup,
  DATE_DIFF(CURRENT_DATE(), signup_date, MONTH) AS months_since_signup
FROM users;
```

### DATE_TRUNC — 날짜 잘라내기 (집계에 유용)
```sql
-- 월별 매출 집계
SELECT
  DATE_TRUNC(order_date, MONTH) AS month,
  SUM(amount) AS monthly_revenue
FROM orders
GROUP BY month
ORDER BY month;

-- 주간 집계
SELECT DATE_TRUNC(event_date, WEEK) AS week_start, COUNT(*) AS events
FROM events
GROUP BY week_start;
```

### FORMAT_DATE — 날짜 포맷 변환
```sql
SELECT FORMAT_DATE('%Y년 %m월 %d일', CURRENT_DATE());  -- 2025년 03월 17일
SELECT FORMAT_DATE('%Y-%m', order_date) AS year_month  -- 2025-03
FROM orders;
```

### EXTRACT — 날짜 요소 추출
```sql
SELECT
  EXTRACT(YEAR FROM order_date)    AS year,   -- 2025
  EXTRACT(MONTH FROM order_date)   AS month,  -- 3
  EXTRACT(DAY FROM order_date)     AS day,    -- 17
  EXTRACT(DAYOFWEEK FROM order_date) AS dow,  -- 1=일요일 ~ 7=토요일
  EXTRACT(HOUR FROM created_at)    AS hour
FROM orders;
```

### PARSE_DATE / PARSE_TIMESTAMP — 문자열 → 날짜 변환
```sql
-- 문자열을 날짜로 파싱
SELECT PARSE_DATE('%Y%m%d', '20250317');          -- 2025-03-17
SELECT PARSE_TIMESTAMP('%Y-%m-%d %H:%M:%S', '2025-03-17 09:00:00');
```

---

## 문자열 함수

### CONCAT — 문자열 이어 붙이기
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
SELECT CONCAT('[', category, '] ', product_name) AS label FROM products;
```

### SUBSTR / SUBSTRING — 문자열 일부 추출
```sql
-- SUBSTR(문자열, 시작위치, 길이)
SELECT SUBSTR('Hello BigQuery', 7, 8);  -- BigQuery
SELECT SUBSTR(phone_number, 1, 3) AS area_code FROM users;
```

### UPPER / LOWER — 대소문자 변환
```sql
SELECT UPPER('bigquery');   -- BIGQUERY
SELECT LOWER('BIGQUERY');   -- bigquery
SELECT UPPER(country_code) FROM orders;
```

### TRIM / LTRIM / RTRIM — 공백 제거
```sql
SELECT TRIM('  Hello  ');   -- 'Hello'
SELECT LTRIM('  Hello  ');  -- 'Hello  '
SELECT RTRIM('  Hello  ');  -- '  Hello'
SELECT TRIM(user_input)     -- 입력값 공백 제거
FROM form_submissions;
```

### REPLACE — 문자열 치환
```sql
SELECT REPLACE('2025-03-17', '-', '/');     -- 2025/03/17
SELECT REPLACE(phone, '-', '') AS clean_phone FROM users;
```

### REGEXP_REPLACE — 정규식으로 치환
```sql
-- 숫자 제외 모든 문자 제거
SELECT REGEXP_REPLACE(phone_number, r'[^0-9]', '');

-- 이메일 도메인 마스킹
SELECT REGEXP_REPLACE(email, r'@.*', '@***.com');
```

### LIKE / REGEXP_CONTAINS — 패턴 매칭
```sql
-- 와일드카드 검색
SELECT * FROM products WHERE name LIKE '%노트북%';

-- 정규식 검색 (REGEXP_CONTAINS)
SELECT * FROM users
WHERE REGEXP_CONTAINS(email, r'^[a-zA-Z0-9._%+-]+@gmail\.com$');

-- 여러 패턴 중 하나
SELECT * FROM logs
WHERE REGEXP_CONTAINS(message, r'ERROR|WARN|CRITICAL');
```

### SPLIT — 문자열 분리 → 배열
```sql
-- 쉼표로 분리
SELECT SPLIT('a,b,c,d', ',');  -- ['a', 'b', 'c', 'd']

-- 태그 분리 후 개수 세기
SELECT
  product_id,
  ARRAY_LENGTH(SPLIT(tags, ',')) AS tag_count
FROM products;
```

### STRING_AGG — 문자열 집계
```sql
-- 사용자별 구매 상품 목록을 하나의 문자열로
SELECT
  user_id,
  STRING_AGG(product_name, ', ' ORDER BY order_date) AS purchased_items
FROM orders
GROUP BY user_id;
```

### LENGTH / CHAR_LENGTH — 문자열 길이
```sql
SELECT LENGTH('BigQuery');       -- 8 (바이트)
SELECT CHAR_LENGTH('빅쿼리');    -- 3 (문자 수)
```

---

## 집계 함수

### 기본 집계
```sql
SELECT
  category,
  COUNT(*)                    AS total_count,      -- 전체 행 수
  COUNT(DISTINCT user_id)     AS unique_users,     -- 고유 사용자 수
  SUM(revenue)                AS total_revenue,    -- 합계
  AVG(revenue)                AS avg_revenue,      -- 평균
  MIN(revenue)                AS min_revenue,      -- 최솟값
  MAX(revenue)                AS max_revenue,      -- 최댓값
  STDDEV(revenue)             AS std_revenue       -- 표준편차
FROM orders
GROUP BY category;
```

### COUNTIF — 조건부 카운트
```sql
SELECT
  DATE(order_date) AS date,
  COUNT(*) AS total_orders,
  COUNTIF(status = 'completed') AS completed_orders,
  COUNTIF(amount > 100000) AS high_value_orders
FROM orders
GROUP BY date;
```

### APPROX_COUNT_DISTINCT — 빠른 근사 고유값 수
```sql
-- 정확한 COUNT DISTINCT보다 훨씬 빠름 (대용량 데이터에 유용)
SELECT
  DATE(event_date) AS date,
  APPROX_COUNT_DISTINCT(user_id) AS approx_dau  -- 일간 활성 사용자 근사값
FROM events
GROUP BY date;
```

### ANY_VALUE — 임의 값 하나 반환
```sql
-- GROUP BY 시 집계하지 않아도 되는 컬럼에 활용
SELECT
  user_id,
  ANY_VALUE(user_name) AS user_name,  -- 같은 user_id면 이름이 동일
  SUM(amount) AS total_spent
FROM orders
GROUP BY user_id;
```

---

## 윈도우 함수

BigQuery에서 가장 강력한 기능 중 하나입니다.

### ROW_NUMBER / RANK / DENSE_RANK — 순위
```sql
-- 카테고리별 매출 순위
SELECT
  product_name,
  category,
  revenue,
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS row_num,
  RANK()       OVER (PARTITION BY category ORDER BY revenue DESC) AS rank,
  DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS dense_rank
FROM products;

-- 각 카테고리에서 1위 상품만 추출
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
  FROM products
)
WHERE rn = 1;
```

### SUM / AVG (누적/이동)
```sql
-- 누적 매출
SELECT
  order_date,
  daily_revenue,
  SUM(daily_revenue) OVER (ORDER BY order_date) AS cumulative_revenue
FROM daily_sales;

-- 7일 이동 평균
SELECT
  order_date,
  daily_revenue,
  AVG(daily_revenue) OVER (
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS moving_avg_7d
FROM daily_sales;
```

### LAG / LEAD — 이전/다음 행 값 참조
```sql
-- 전일 대비 매출 증감
SELECT
  order_date,
  daily_revenue,
  LAG(daily_revenue, 1) OVER (ORDER BY order_date) AS prev_day_revenue,
  daily_revenue - LAG(daily_revenue, 1) OVER (ORDER BY order_date) AS revenue_diff,
  LEAD(daily_revenue, 1) OVER (ORDER BY order_date) AS next_day_revenue
FROM daily_sales;
```

### FIRST_VALUE / LAST_VALUE — 첫/마지막 값
```sql
-- 사용자의 첫 번째 구매 상품
SELECT
  user_id,
  order_date,
  product_name,
  FIRST_VALUE(product_name) OVER (
    PARTITION BY user_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
  ) AS first_purchase_item
FROM orders;
```

### NTILE — 분위수 나누기
```sql
-- 매출 기준으로 고객을 4분위로 분류
SELECT
  user_id,
  total_spent,
  NTILE(4) OVER (ORDER BY total_spent DESC) AS quartile
  -- 1: 상위 25%, 2: 26~50%, 3: 51~75%, 4: 하위 25%
FROM customer_spending;
```

### PERCENT_RANK / CUME_DIST — 백분위 순위
```sql
SELECT
  user_id,
  total_spent,
  ROUND(PERCENT_RANK() OVER (ORDER BY total_spent) * 100, 1) AS percentile
FROM customer_spending;
```

---

## 조건/분기 함수

### CASE WHEN — 조건 분기
```sql
-- 점수 → 등급 변환
SELECT
  student_id,
  score,
  CASE
    WHEN score >= 90 THEN 'A'
    WHEN score >= 80 THEN 'B'
    WHEN score >= 70 THEN 'C'
    WHEN score >= 60 THEN 'D'
    ELSE 'F'
  END AS grade
FROM exam_results;

-- 매출 구간 분류
SELECT
  order_id,
  amount,
  CASE
    WHEN amount >= 1000000 THEN '100만원 이상'
    WHEN amount >= 500000  THEN '50만원 이상'
    WHEN amount >= 100000  THEN '10만원 이상'
    ELSE '10만원 미만'
  END AS amount_range
FROM orders;
```

### IF — 간단한 조건
```sql
SELECT
  order_id,
  IF(amount > 50000, '고가', '일반') AS price_tier,
  IF(is_member, amount * 0.9, amount) AS final_price  -- 회원 10% 할인
FROM orders;
```

### IIF (= IF의 별칭)
```sql
SELECT IIF(status = 'active', '활성', '비활성') AS status_label FROM users;
```

### IFNULL / NULLIF / COALESCE — NULL 처리
```sql
-- IFNULL: NULL이면 대체값
SELECT IFNULL(discount, 0) AS discount FROM orders;

-- NULLIF: 두 값이 같으면 NULL 반환 (0으로 나누기 방지)
SELECT revenue / NULLIF(cost, 0) AS roi FROM campaigns;

-- COALESCE: 여러 값 중 첫 번째 non-NULL 반환
SELECT COALESCE(mobile_phone, home_phone, office_phone, '번호 없음') AS contact
FROM users;
```

---

## 배열 함수

### ARRAY_AGG — 행 → 배열 집계
```sql
-- 사용자별 구매 상품 배열로
SELECT
  user_id,
  ARRAY_AGG(product_id ORDER BY order_date) AS purchased_products,
  ARRAY_AGG(DISTINCT category) AS categories
FROM orders
GROUP BY user_id;
```

### UNNEST — 배열 → 행으로 펼치기
```sql
-- 배열을 행으로 펼쳐서 분석
SELECT user_id, tag
FROM users, UNNEST(tags) AS tag;

-- 인덱스와 함께
SELECT
  user_id,
  tag,
  idx
FROM users, UNNEST(tags) AS tag WITH OFFSET AS idx;
```

### ARRAY_LENGTH — 배열 길이
```sql
SELECT
  user_id,
  ARRAY_LENGTH(purchased_products) AS purchase_count
FROM user_purchases;
```

### ARRAY_CONTAINS / IN UNNEST — 배열 원소 포함 여부
```sql
-- 특정 상품을 구매한 사용자
SELECT user_id
FROM user_purchases
WHERE 'PROD_001' IN UNNEST(purchased_products);
```

---

## JSON 함수

### JSON_VALUE — JSON 문자열에서 값 추출
```sql
-- 단일 값 추출 (문자열 반환)
SELECT
  JSON_VALUE(event_params, '$.user_id')     AS user_id,
  JSON_VALUE(event_params, '$.page_title')  AS page_title
FROM events;
```

### JSON_QUERY — JSON 객체/배열 추출
```sql
-- 중첩 JSON 객체 추출
SELECT
  JSON_QUERY(metadata, '$.address')         AS address_json,
  JSON_QUERY(metadata, '$.tags')            AS tags_array
FROM users;
```

### JSON_VALUE_ARRAY / JSON_QUERY_ARRAY
```sql
-- JSON 배열 → BigQuery 배열로
SELECT
  user_id,
  tag
FROM users,
UNNEST(JSON_VALUE_ARRAY(metadata, '$.tags')) AS tag;
```

### TO_JSON_STRING — 값 → JSON 문자열 변환
```sql
SELECT TO_JSON_STRING(STRUCT(
  user_id AS id,
  user_name AS name,
  CURRENT_DATE() AS date
)) AS json_output;
-- {"id":"U001","name":"홍길동","date":"2025-03-17"}
```

---

## 수학 함수

```sql
SELECT
  ROUND(3.14159, 2),       -- 3.14 (반올림)
  FLOOR(3.9),              -- 3    (내림)
  CEIL(3.1),               -- 4    (올림)
  TRUNC(3.9),              -- 3    (소수점 버림)
  ABS(-42),                -- 42   (절댓값)
  MOD(17, 5),              -- 2    (나머지)
  POWER(2, 10),            -- 1024 (거듭제곱)
  SQRT(144),               -- 12   (제곱근)
  LOG(100, 10),            -- 2    (로그)
  LN(2.718),               -- ~1   (자연로그)
  SAFE_DIVIDE(10, 0);      -- NULL (0으로 나눠도 에러 없음)
```

### SAFE_DIVIDE — 안전한 나눗셈
```sql
-- 전환율 계산 (분모가 0일 때 에러 대신 NULL 반환)
SELECT
  campaign_id,
  clicks,
  conversions,
  SAFE_DIVIDE(conversions, clicks) AS conversion_rate
FROM campaigns;
```

---

## 타입 변환 함수

### CAST — 명시적 타입 변환
```sql
SELECT
  CAST('123' AS INT64),                    -- 문자→정수
  CAST(3.14 AS STRING),                    -- 숫자→문자
  CAST('2025-03-17' AS DATE),              -- 문자→날짜
  CAST(order_id AS STRING) AS order_str
FROM orders;
```

### SAFE_CAST — 실패해도 NULL 반환
```sql
-- 변환 실패 시 에러 대신 NULL 반환
SELECT
  SAFE_CAST(user_input AS INT64) AS numeric_value,  -- 변환 불가면 NULL
  SAFE_CAST(date_str AS DATE) AS parsed_date
FROM form_submissions;
```

### PARSE_NUMERIC / FORMAT 함수
```sql
-- 숫자 포맷팅
SELECT FORMAT('%,.0f', revenue) AS formatted_revenue;  -- 1,234,567

-- 문자 → 숫자 파싱
SELECT PARSE_NUMERIC('1,234.56');  -- 1234.56
```

---

## NULL 처리 함수

```sql
SELECT
  -- NULL 체크
  ISNULL(value),              -- NULL이면 TRUE
  value IS NULL,              -- NULL이면 TRUE
  value IS NOT NULL,          -- NULL이 아니면 TRUE

  -- NULL 대체
  IFNULL(value, 0),           -- NULL이면 0
  COALESCE(a, b, c),          -- 첫 번째 non-NULL 값

  -- NULL 생성
  NULLIF(value, ''),          -- 빈 문자열이면 NULL로

  -- 집계 시 NULL 제외 (자동)
  COUNT(nullable_col),        -- NULL 제외 카운트
  AVG(nullable_col)           -- NULL 제외 평균
FROM table_with_nulls;
```

---

## 함수 조합 실전 예시

### 예시 1: 월별 사용자 코호트 분석
```sql
SELECT
  FORMAT_DATE('%Y-%m', signup_date) AS cohort_month,
  DATE_DIFF(
    DATE_TRUNC(order_date, MONTH),
    DATE_TRUNC(signup_date, MONTH),
    MONTH
  ) AS months_after_signup,
  COUNT(DISTINCT user_id) AS active_users
FROM orders
JOIN users USING (user_id)
GROUP BY cohort_month, months_after_signup
ORDER BY cohort_month, months_after_signup;
```

### 예시 2: 구매 패턴 분석
```sql
SELECT
  user_id,
  COUNT(*) AS total_orders,
  ROUND(AVG(amount), 0) AS avg_order_value,
  MAX(order_date) AS last_order_date,
  DATE_DIFF(CURRENT_DATE(), MAX(order_date), DAY) AS days_since_last_order,
  STRING_AGG(DISTINCT category ORDER BY category) AS categories_purchased,
  CASE
    WHEN DATE_DIFF(CURRENT_DATE(), MAX(order_date), DAY) <= 30 THEN '활성'
    WHEN DATE_DIFF(CURRENT_DATE(), MAX(order_date), DAY) <= 90 THEN '위험'
    ELSE '이탈'
  END AS user_status
FROM orders
GROUP BY user_id;
```

### 예시 3: 이동 평균 + 전주 대비 증감
```sql
WITH weekly_sales AS (
  SELECT
    DATE_TRUNC(order_date, WEEK) AS week,
    SUM(amount) AS weekly_revenue
  FROM orders
  GROUP BY week
)
SELECT
  week,
  weekly_revenue,
  LAG(weekly_revenue) OVER (ORDER BY week) AS prev_week_revenue,
  ROUND(
    SAFE_DIVIDE(
      weekly_revenue - LAG(weekly_revenue) OVER (ORDER BY week),
      LAG(weekly_revenue) OVER (ORDER BY week)
    ) * 100, 1
  ) AS wow_growth_pct,
  AVG(weekly_revenue) OVER (
    ORDER BY week ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
  ) AS moving_avg_4w
FROM weekly_sales
ORDER BY week;
```

---

[← 이전: 최신 기능](10-latest-features) | [← 목차로 돌아가기](index)
