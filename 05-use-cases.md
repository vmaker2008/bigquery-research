---
layout: default
title: 5. 사용 사례 & 모범 사례
nav_order: 6
---

# 5. 사용 사례 & 모범 사례

[← 목차로 돌아가기](index)

---

## 실제 사용 사례

### 1. 마케팅 & 광고 분석

**사례: Google Analytics + BigQuery 연동**

```sql
-- GA4 데이터와 CRM 데이터 결합 분석
SELECT
  g.user_pseudo_id,
  g.event_name,
  g.event_date,
  c.customer_tier,
  c.lifetime_value
FROM `project.analytics.events_*` g
JOIN `project.crm.customers` c
  ON g.user_pseudo_id = c.ga_user_id
WHERE g.event_name = 'purchase'
  AND g._TABLE_SUFFIX BETWEEN '20240101' AND '20241231'
ORDER BY c.lifetime_value DESC;
```

**활용 효과**:
- 광고 ROAS(투자 대비 수익) 정확한 측정
- 고객 세그먼트별 행동 패턴 분석
- 이탈 고객 예측 모델 구축

---

### 2. 이커머스 판매 분석

```sql
-- 상품별 월간 매출 추이 분석
SELECT
  product_category,
  FORMAT_DATE('%Y-%m', order_date) AS month,
  SUM(revenue) AS monthly_revenue,
  COUNT(*) AS order_count,
  AVG(revenue) AS avg_order_value
FROM `project.ecommerce.orders`
WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 12 MONTH)
GROUP BY product_category, month
ORDER BY month DESC, monthly_revenue DESC;
```

---

### 3. 실시간 사기 탐지

```
이벤트 스트림 → Pub/Sub → Dataflow → BigQuery
                                       ↓
                               실시간 ML 모델
                                       ↓
                              이상 거래 탐지 알림
```

```sql
-- 비정상 거래 패턴 탐지
SELECT
  user_id,
  COUNT(*) AS tx_count_1h,
  SUM(amount) AS total_amount_1h,
  STDDEV(amount) AS amount_variance
FROM `project.payments.transactions`
WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
GROUP BY user_id
HAVING tx_count_1h > 10 OR total_amount_1h > 1000000;
```

---

### 4. 로그 분석 & 모니터링

```sql
-- 애플리케이션 에러 분석
SELECT
  error_code,
  error_message,
  COUNT(*) AS occurrence_count,
  MIN(timestamp) AS first_seen,
  MAX(timestamp) AS last_seen
FROM `project.logs.app_errors`
WHERE DATE(timestamp) = CURRENT_DATE()
GROUP BY error_code, error_message
ORDER BY occurrence_count DESC
LIMIT 20;
```

---

## 모범 사례 (Best Practices)

### 1. 테이블 설계

#### 파티션 테이블 사용
```sql
-- 날짜 기반 파티션 테이블 생성
CREATE TABLE `project.dataset.sales`
PARTITION BY DATE(sale_date)
CLUSTER BY product_id, region
AS
SELECT * FROM `project.staging.sales_raw`;
```

**파티션 + 클러스터링 조합 효과**:
- 파티션: 날짜 범위로 스캔 데이터 축소
- 클러스터링: 해당 파티션 내 추가 최적화

#### 중첩/반복 구조 활용
```sql
-- 배열(ARRAY) 사용으로 중복 행 제거
SELECT
  order_id,
  ARRAY_AGG(STRUCT(product_id, quantity, price)) AS line_items
FROM orders
GROUP BY order_id;
```

---

### 2. 쿼리 작성 모범 사례

#### SELECT * 피하기
```sql
-- ❌ 나쁜 예: 전체 컬럼 스캔
SELECT * FROM `orders`;

-- ✅ 좋은 예: 필요한 컬럼만
SELECT order_id, customer_id, total_amount FROM `orders`;
```

#### 파티션 필터 활용
```sql
-- ✅ 파티션 열로 필터링 (비용 절감)
SELECT *
FROM `project.dataset.events`
WHERE event_date BETWEEN '2024-01-01' AND '2024-01-31';
```

#### 임시 테이블 대신 CTE 사용
```sql
-- ✅ CTE로 복잡한 쿼리 구조화
WITH monthly_sales AS (
  SELECT
    DATE_TRUNC(order_date, MONTH) AS month,
    SUM(amount) AS total
  FROM orders
  GROUP BY month
),
ranked_months AS (
  SELECT *, RANK() OVER (ORDER BY total DESC) AS rank
  FROM monthly_sales
)
SELECT * FROM ranked_months WHERE rank <= 3;
```

---

### 3. 데이터 수집 모범 사례

| 시나리오 | 권장 방법 |
|----------|----------|
| 대용량 배치 로드 | Cloud Storage → BigQuery Load Job |
| 실시간 이벤트 | Pub/Sub → Dataflow → BigQuery |
| SaaS 데이터 | BigQuery Data Transfer Service |
| 데이터 변환 | ELT 방식 (BigQuery 내부에서 변환) |
| CDC (변경 데이터 캡처) | Datastream → BigQuery |

---

### 4. 비용 관리 모범 사례

```
비용 관리 체크리스트
━━━━━━━━━━━━━━━━━
☐ 필요한 컬럼만 SELECT
☐ 파티션 테이블 + 파티션 필터 사용
☐ 쿼리 실행 전 드라이 런으로 비용 확인
☐ 쿼리 결과 캐싱 활용
☐ 예산 알림 설정
☐ 90일 이상 미사용 테이블 → 장기 스토리지 활용
☐ 슬롯 사용량 모니터링
```

---

### 5. 거버넌스 모범 사례

```
데이터 거버넌스 레이어
┌──────────────────────────────────┐
│ 1. 데이터 카탈로그                │
│    - Dataplex Universal Catalog  │
│    - 자동 메타데이터 수집          │
├──────────────────────────────────┤
│ 2. 접근 제어                      │
│    - IAM 최소 권한 원칙           │
│    - 행/열 수준 보안              │
├──────────────────────────────────┤
│ 3. 데이터 품질                    │
│    - 자동 프로파일링              │
│    - 품질 점수 모니터링           │
├──────────────────────────────────┤
│ 4. 감사 & 모니터링               │
│    - Cloud Audit Logs            │
│    - 접근 기록 추적               │
└──────────────────────────────────┘
```

---

[← 이전: 요금 체계](04-pricing) | [다음: 비교 →](06-comparison)
