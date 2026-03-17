---
layout: default
title: 7. BigQuery ML & AI
nav_order: 8
---

# 7. BigQuery ML & AI 기능

[← 목차로 돌아가기](index)

---

## BigQuery ML (BQML) 개요

**BigQuery ML**은 SQL 쿼리만으로 머신러닝 모델을 생성, 훈련, 평가, 예측할 수 있는 서비스입니다.

### 핵심 장점

```
전통적 ML 파이프라인:
데이터 추출 → Python/R 환경 → 모델 훈련 → 배포 → 예측

BigQuery ML:
데이터 있는 곳에서 바로 SQL로 → 모델 생성 → 예측
(데이터 이동 없음, Python 불필요)
```

---

## 지원 모델 유형

### 1. 내장 지도학습 모델

#### 선형 회귀 (Linear Regression)
```sql
-- 판매량 예측 모델 생성
CREATE OR REPLACE MODEL `project.dataset.sales_forecast`
OPTIONS(
  model_type = 'linear_reg',
  input_label_cols = ['revenue']
) AS
SELECT
  season,
  marketing_spend,
  product_category,
  revenue
FROM `project.dataset.historical_sales`;
```

#### 로지스틱 회귀 (Logistic Regression)
```sql
-- 이탈 예측 분류 모델
CREATE OR REPLACE MODEL `project.dataset.churn_model`
OPTIONS(
  model_type = 'logistic_reg',
  input_label_cols = ['churned']
) AS
SELECT
  days_since_last_purchase,
  total_purchases,
  avg_order_value,
  churned
FROM `project.dataset.customer_features`;
```

### 2. 비지도학습 모델

#### K-평균 클러스터링 (K-Means)
```sql
-- 고객 세분화
CREATE OR REPLACE MODEL `project.dataset.customer_segments`
OPTIONS(
  model_type = 'kmeans',
  num_clusters = 5
) AS
SELECT
  annual_spend,
  visit_frequency,
  avg_session_duration,
  product_diversity
FROM `project.dataset.customer_metrics`;

-- 세그먼트 확인
SELECT * FROM ML.PREDICT(MODEL `project.dataset.customer_segments`,
  TABLE `project.dataset.new_customers`);
```

### 3. 시계열 예측 (ARIMA_PLUS)
```sql
-- 다음 30일 매출 예측
CREATE OR REPLACE MODEL `project.dataset.revenue_forecast`
OPTIONS(
  model_type = 'ARIMA_PLUS',
  time_series_timestamp_col = 'order_date',
  time_series_data_col = 'daily_revenue',
  horizon = 30
) AS
SELECT order_date, SUM(amount) AS daily_revenue
FROM `project.dataset.orders`
GROUP BY order_date;

-- 예측 실행
SELECT * FROM ML.FORECAST(
  MODEL `project.dataset.revenue_forecast`,
  STRUCT(30 AS horizon, 0.9 AS confidence_level)
);
```

### 4. 추천 시스템 (Matrix Factorization)
```sql
-- 상품 추천 모델
CREATE OR REPLACE MODEL `project.dataset.product_recommender`
OPTIONS(
  model_type = 'matrix_factorization',
  user_col = 'user_id',
  item_col = 'product_id',
  rating_col = 'rating'
) AS
SELECT user_id, product_id, rating
FROM `project.dataset.user_ratings`;
```

---

## ML 워크플로우

```
1. 데이터 탐색 및 준비
   SELECT 로 피처 확인

2. 모델 생성
   CREATE MODEL

3. 모델 평가
   ML.EVALUATE(MODEL ...)

4. 예측
   ML.PREDICT(MODEL ...)

5. 피처 중요도 확인
   ML.FEATURE_IMPORTANCE(MODEL ...)
```

### 모델 평가 예시
```sql
-- 분류 모델 성능 평가
SELECT *
FROM ML.EVALUATE(MODEL `project.dataset.churn_model`,
  (SELECT * FROM `project.dataset.test_data`)
);
-- precision, recall, f1_score, roc_auc 등 반환
```

---

## BigQuery AI 기능 (2024-2025)

### 1. 생성 AI 통합 (Generative AI)

#### AI 함수
```sql
-- 텍스트 감정 분석
SELECT
  review_text,
  ML.GENERATE_TEXT(
    MODEL `project.models.gemini_pro`,
    STRUCT(
      CONCAT('다음 리뷰의 감정을 분석해주세요: ', review_text) AS prompt,
      0.1 AS temperature
    )
  ).ml_generate_text_result AS sentiment
FROM `project.dataset.product_reviews`
LIMIT 10;
```

#### 지원 AI 작업
- **텍스트 생성**: 요약, 번역, Q&A
- **감정 분석**: 긍정/부정/중립 분류
- **엔티티 추출**: 텍스트에서 주요 정보 추출
- **이미지 분석**: Cloud Vision API 연동

---

### 2. 벡터 검색 (Vector Search)

```sql
-- 1. 텍스트 임베딩 생성
CREATE OR REPLACE TABLE `project.dataset.product_embeddings` AS
SELECT
  product_id,
  product_name,
  ML.GENERATE_EMBEDDING(
    MODEL `project.models.text_embedding`,
    STRUCT(product_description AS content)
  ).ml_generate_embedding_result AS embedding
FROM `project.dataset.products`;

-- 2. 벡터 인덱스 생성
CREATE VECTOR INDEX product_search_index
ON `project.dataset.product_embeddings`(embedding)
OPTIONS(distance_type = 'COSINE');

-- 3. 유사 상품 검색
SELECT base.product_id, base.product_name, distance
FROM VECTOR_SEARCH(
  TABLE `project.dataset.product_embeddings`,
  'embedding',
  (SELECT embedding FROM `project.dataset.product_embeddings`
   WHERE product_id = 'PROD_001'),
  top_k => 10
);
```

---

### 3. RAG (Retrieval-Augmented Generation)

문서에서 정보를 추출하여 LLM의 응답 품질을 향상시키는 패턴입니다.

```
문서 (PDF, 텍스트)
      ↓
   임베딩 생성
      ↓
BigQuery 벡터 저장
      ↓
  사용자 질문
      ↓
  유사 문서 검색 (VECTOR_SEARCH)
      ↓
LLM에 컨텍스트와 함께 전달 (ML.GENERATE_TEXT)
      ↓
   정확한 답변
```

---

### 4. Vertex AI 통합

```sql
-- Vertex AI에 배포된 커스텀 모델 사용
CREATE OR REPLACE MODEL `project.dataset.custom_model`
OPTIONS(
  model_type = 'tensorflow',
  model_registry = 'vertex_ai',
  vertex_ai_model_id = 'your_model_id',
  vertex_ai_model_version_id = '1'
);

-- 예측
SELECT * FROM ML.PREDICT(
  MODEL `project.dataset.custom_model`,
  TABLE `project.dataset.new_data`
);
```

---

## BigQuery ML 지원 모델 전체 목록

| 카테고리 | 모델 유형 | 활용 사례 |
|----------|----------|----------|
| **회귀** | linear_reg | 판매량, 가격 예측 |
| **분류** | logistic_reg, dnn_classifier, boosted_tree_classifier, random_forest_classifier | 이탈 예측, 스팸 분류 |
| **클러스터링** | kmeans | 고객 세분화, 이상 탐지 |
| **추천** | matrix_factorization | 상품 추천 |
| **시계열** | ARIMA_PLUS, TimesFM | 수요 예측, 이상 탐지 |
| **임포트** | tensorflow, onnx, xgboost | 기존 모델 활용 |
| **원격** | Vertex AI 모델 | 엔터프라이즈 ML |
| **생성 AI** | Gemini, PaLM 2 | 텍스트 생성, 분석 |

---

[← 이전: 비교](06-comparison) | [다음: 보안 →](08-security)
