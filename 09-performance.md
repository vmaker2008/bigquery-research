---
layout: default
title: 9. 성능 최적화
nav_order: 10
---

# 9. 성능 최적화

[← 목차로 돌아가기](index)

---

## 성능 최적화 원칙

BigQuery 쿼리 성능은 세 가지 핵심 요소로 결정됩니다.

```
성능 결정 요소
┌────────────────────────────────────────────┐
│ 1. I/O (입력/출력)                          │
│    - 스캔하는 데이터 양                      │
│    - 불필요한 스캔 최소화 → 파티션, 클러스터링 │
├────────────────────────────────────────────┤
│ 2. 셔플 (노드 간 통신)                       │
│    - JOIN, GROUP BY 시 데이터 이동           │
│    - 최소화 → 쿼리 최적화                    │
├────────────────────────────────────────────┤
│ 3. 슬롯 용량 (Slot Capacity)                 │
│    - 동시 실행 가능한 쿼리 수                 │
│    - 슬롯 예약으로 성능 보장                  │
└────────────────────────────────────────────┘
```

---

## 1. 파티션 테이블 (Partitioned Tables)

가장 중요한 최적화 기법입니다. 날짜/시간 기반으로 데이터를 분할하여 쿼리 시 해당 파티션만 스캔합니다.

### 파티션 유형

#### 날짜/시간 파티션 (가장 일반적)
```sql
-- 날짜 컬럼으로 파티션
CREATE TABLE `project.dataset.events`
PARTITION BY DATE(event_timestamp)
AS SELECT * FROM `project.staging.raw_events`;

-- 파티션 필터 적용 (중요!)
SELECT *
FROM `project.dataset.events`
WHERE DATE(event_timestamp) = '2024-03-01';
-- 3월 1일 파티션만 스캔 → 비용/시간 대폭 절감
```

#### 정수 범위 파티션
```sql
-- 고객 ID 범위로 파티션
CREATE TABLE `project.dataset.customers`
PARTITION BY RANGE_BUCKET(customer_id, GENERATE_ARRAY(0, 1000000, 10000))
AS SELECT * FROM source_table;
```

#### 수집 시간 파티션
```sql
-- _PARTITIONTIME 자동 활용
SELECT *
FROM `project.dataset.logs`
WHERE _PARTITIONTIME = '2024-03-01 00:00:00 UTC';
```

### 파티션 효과
```
파티션 없음:                    파티션 있음:
전체 1TB 테이블 스캔           2024-03-01 파티션만 스캔
비용: $6.25                   비용: $0.02 (1/365)
시간: 30초                    시간: 0.1초
```

---

## 2. 클러스터링 (Clustering)

파티션 내에서 특정 컬럼으로 데이터를 물리적으로 정렬하여 추가 최적화합니다.

```sql
-- 파티션 + 클러스터링 조합
CREATE TABLE `project.dataset.orders`
PARTITION BY DATE(order_date)
CLUSTER BY customer_id, product_category
AS SELECT * FROM source;

-- 클러스터링 키로 쿼리 시 최적화
SELECT *
FROM `project.dataset.orders`
WHERE DATE(order_date) = '2024-03-01'
  AND customer_id = 'CUST_001'  -- 클러스터링 키
  AND product_category = 'Electronics';  -- 클러스터링 키
```

### 클러스터링 선택 기준
- **고카디널리티 컬럼**: 많은 고유값 (customer_id, product_id 등)
- **자주 필터링하는 컬럼**: WHERE, JOIN 조건에 자주 사용
- **최대 4개 컬럼** 지정 가능

---

## 3. 구체화된 뷰 (Materialized Views)

자주 실행하는 집계 쿼리 결과를 미리 계산하여 저장합니다.

```sql
-- 구체화된 뷰 생성 (일별 매출 집계)
CREATE MATERIALIZED VIEW `project.dataset.daily_revenue_mv`
PARTITION BY report_date
CLUSTER BY product_category
AS
SELECT
  DATE(order_timestamp) AS report_date,
  product_category,
  SUM(revenue) AS total_revenue,
  COUNT(*) AS order_count
FROM `project.dataset.orders`
GROUP BY report_date, product_category;

-- 쿼리 시 자동으로 구체화된 뷰 활용
-- BigQuery가 자동으로 최적 실행 경로 선택
```

**장점**:
- 복잡한 집계를 미리 계산
- 원본 테이블 업데이트 시 자동 갱신
- 쿼리 응답 시간 10~100배 개선

---

## 4. 쿼리 최적화 기법

### 필요한 컬럼만 SELECT
```sql
-- ❌ 전체 컬럼 스캔
SELECT * FROM `project.dataset.large_table`;

-- ✅ 필요한 컬럼만 (비용 절감)
SELECT user_id, event_type, event_timestamp
FROM `project.dataset.large_table`;
```

### JOIN 최적화

#### 큰 테이블을 먼저 (왼쪽에)
```sql
-- ✅ 큰 테이블 LEFT, 작은 테이블 RIGHT
SELECT l.order_id, r.product_name
FROM `large_orders_table` l  -- 수억 행
JOIN `small_products_table` r  -- 수천 행
ON l.product_id = r.product_id;
```

#### 브로드캐스트 조인 힌트
```sql
-- 작은 테이블을 모든 노드에 복사 (셔플 제거)
SELECT /*+ BROADCAST(small_table) */
  l.*, r.category
FROM large_table l
JOIN small_table r ON l.id = r.id;
```

### 필터 먼저 적용
```sql
-- ✅ 서브쿼리에서 먼저 필터링
SELECT *
FROM (
  SELECT *
  FROM `project.dataset.events`
  WHERE event_date >= '2024-01-01'  -- 먼저 필터
) filtered
JOIN `project.dataset.users` u USING (user_id);
```

### GROUP BY 최적화
```sql
-- ❌ 고카디널리티 컬럼 GROUP BY (셔플 많음)
GROUP BY user_id, session_id, event_id;

-- ✅ 사전 집계 후 GROUP BY
SELECT date, COUNT(DISTINCT user_id)
FROM (
  SELECT DATE(event_timestamp) AS date, user_id
  FROM events
  GROUP BY date, user_id
)
GROUP BY date;
```

---

## 5. BI Engine (인메모리 캐싱)

자주 사용하는 데이터를 메모리에 캐싱하여 서브초 응답을 실현합니다.

```
설정 방법:
BigQuery 콘솔 → BI Engine → 메모리 예약 (GB 단위)

예약 크기:
1GB → $45/월
10GB → $450/월
100GB → $4,500/월

효과:
평균 쿼리 응답: 30초 → 0.3초 (100배 개선)
```

---

## 6. 쿼리 성능 분석 도구

### 실행 계획 확인
```sql
-- 쿼리 드라이 런 (실행 없이 비용만 확인)
-- bq CLI:
bq query --dry_run --use_legacy_sql=false \
  "SELECT * FROM project.dataset.table"

-- 출력: Query successfully validated.
-- Assuming the tables are not modified,
-- running this query will process 1.5 GB bytes.
```

### INFORMATION_SCHEMA으로 성능 분석
```sql
-- 최근 24시간 느린 쿼리 찾기
SELECT
  job_id,
  query,
  total_bytes_processed,
  total_slot_ms,
  TIMESTAMP_DIFF(end_time, start_time, SECOND) AS duration_seconds
FROM `project.region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT`
WHERE creation_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR)
  AND state = 'DONE'
ORDER BY total_slot_ms DESC
LIMIT 10;
```

---

## 7. 성능 최적화 체크리스트

```
성능 최적화 체크리스트
━━━━━━━━━━━━━━━━━━━━━━
테이블 설계:
☐ 날짜/시간 파티션 테이블 사용
☐ 자주 필터링하는 컬럼으로 클러스터링
☐ 중첩/반복 컬럼으로 조인 최소화
☐ 자주 사용하는 집계에 구체화된 뷰 생성

쿼리 작성:
☐ SELECT * 대신 필요한 컬럼만 명시
☐ 파티션 필터 반드시 적용
☐ 클러스터링 컬럼으로 필터링
☐ 큰 테이블을 JOIN 왼쪽에 배치
☐ 서브쿼리에서 먼저 데이터 필터링

인프라:
☐ BI Engine으로 자주 사용 데이터 캐싱
☐ 슬롯 예약으로 성능 보장
☐ 슬롯 사용률 모니터링
```

---

[← 이전: 보안](08-security) | [다음: 최신 기능 →](10-latest-features)
