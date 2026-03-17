# 10. 최신 기능 (2024-2025)

[← 목차로 돌아가기](index.md)

---

## BigQuery Omni (멀티클라우드 분석)

### 개요
BigQuery Omni는 **AWS, Azure에서도 BigQuery를 사용**할 수 있게 해주는 멀티클라우드 분석 솔루션입니다.

```
멀티클라우드 아키텍처:
┌─────────────────────────────────────────────────┐
│                  Google Cloud                   │
│  ┌──────────┐    BigQuery     ┌──────────────┐  │
│  │ BigQuery │ ◄─ 크로스클라우드 ─► │  Looker     │  │
│  │  (메인)  │    조인 지원       │  (시각화)   │  │
│  └──────────┘                 └──────────────┘  │
└──────────┬────────────────────────┬─────────────┘
           │                        │
    ┌──────▼──────┐          ┌──────▼──────┐
    │     AWS     │          │    Azure    │
    │  Amazon S3  │          │ ADLS Gen2   │
    │ BigQuery    │          │ BigQuery    │
    │  Omni      │          │  Omni      │
    └─────────────┘          └─────────────┘
```

### 주요 기능
- **크로스클라우드 JOIN**: Google Cloud와 AWS/Azure 데이터를 하나의 쿼리로 조인
- **데이터 이동 불필요**: 원본 위치에서 직접 분석 (데이터 전송 비용 없음)
- **통합 보안**: Google Cloud IAM 보안 정책 적용
- **구체화된 뷰**: Omni 지역의 데이터를 지속적으로 복제하여 성능 향상

```sql
-- AWS S3 데이터와 BigQuery 데이터 크로스클라우드 JOIN
SELECT
  gcp.customer_id,
  gcp.purchase_amount,
  aws.crm_segment
FROM `project.dataset.gcp_purchases` gcp
JOIN `project.aws-dataset.crm_data` aws  -- AWS 데이터
  ON gcp.customer_id = aws.customer_id;
```

---

## BigQuery Advanced Runtime

### 개요
2025년 9월부터 모든 프로젝트의 **기본 쿼리 실행 엔진**으로 전환됩니다.

### 주요 개선사항
- 더 빠른 쿼리 실행 속도
- 향상된 메모리 관리
- 개선된 복잡한 쿼리 처리
- 더 넓은 SQL 기능 지원

---

## BigQuery Editions (2024)

### 새로운 에디션 체계

| Edition | 핵심 기능 | 추천 대상 |
|---------|----------|----------|
| **Standard** | 기본 BigQuery 기능 | 소규모 팀, PoC |
| **Enterprise** | 고급 보안, CMEK, SLA | 중/대규모 기업 |
| **Enterprise Plus** | 크로스 리전 재해 복구, 최고 SLA | 미션 크리티컬 |

### 에디션별 차별화 기능

```
Standard:
├── 온디맨드 및 자동 확장 슬롯
├── 기본 ML 기능
└── 1년/3년 약정 없음

Enterprise:
├── Standard 모든 기능 포함
├── CMEK (고객 관리 암호화 키)
├── BigQuery Omni
├── 고급 보안 기능
└── 엔터프라이즈 SLA

Enterprise Plus:
├── Enterprise 모든 기능 포함
├── 크로스 리전 재해 복구
├── 최고 수준 가용성 SLA
└── 우선 지원
```

---

## Analytics Hub (데이터 공유)

### 개요
BigQuery 데이터를 **복제 없이 안전하게 공유**하는 플랫폼입니다.

```
데이터 공유 모델:
┌─────────────────┐         ┌─────────────────┐
│  Publisher      │         │  Subscriber      │
│  (데이터 제공자) │         │  (데이터 소비자)  │
│                 │         │                 │
│  ┌───────────┐  │ Listing │  ┌───────────┐  │
│  │ 원본 데이터│  │───────► │  │공유 데이터│  │
│  └───────────┘  │  공개   │  └───────────┘  │
│  (복사 없음)     │         │  (실시간 접근)   │
└─────────────────┘         └─────────────────┘
```

### 활용 사례
- **내부 데이터 공유**: 부서 간 데이터 공유
- **파트너 데이터 교환**: B2B 데이터 공유
- **공공 데이터 배포**: 오픈 데이터셋 공유
- **유료 데이터 판매**: Google Cloud Marketplace를 통한 데이터 상품화

---

## BigQuery Studio (개발 환경)

### Git 통합

```
BigQuery Studio ← → GitHub/GitLab
        │
   SQL 쿼리 파일
   Python 노트북
   Dataform 파이프라인
        │
   버전 관리, PR, 코드 리뷰
```

### 협업 기능
- **Colab Enterprise 통합**: Python 노트북과 BigQuery 원클릭 연동
- **공유 개발 환경**: 팀원과 실시간 협업
- **스케줄된 쿼리 관리**: 정기 실행 쿼리 관리
- **Dataform 통합**: SQL 기반 데이터 변환 파이프라인 버전 관리

---

## Dataplex Universal Catalog (데이터 거버넌스)

### 기능 개요

```
┌──────────────────────────────────────────────────────┐
│              Dataplex Universal Catalog              │
├───────────────┬──────────────┬─────────────────────┤
│ 자동 메타데이터  │  데이터 품질   │    데이터 계보        │
│   수집         │   관리         │    추적               │
│               │               │                      │
│ BigQuery,     │ 자동 프로파일링 │ 데이터 출처 추적       │
│ Cloud Storage,│ 품질 점수      │ 어디서 왔고            │
│ Spanner 등    │ 이상값 탐지    │ 어디로 갔는지          │
└───────────────┴──────────────┴─────────────────────┘
```

### 핵심 기능
- **자동 메타데이터 수집**: BigQuery, Cloud Storage 등에서 자동 추출
- **데이터 품질 점수**: 자동으로 데이터 품질 평가
- **데이터 계보 추적**: 데이터의 출처와 변환 이력 추적
- **민감 데이터 자동 분류**: 개인정보, 금융정보 등 자동 탐지

---

## 벡터 검색 & AI 검색 (2024)

### VECTOR_SEARCH
```sql
-- 의미론적 유사도 검색
SELECT query.id, base.id, distance
FROM VECTOR_SEARCH(
  TABLE `project.dataset.document_embeddings`,
  'embedding',
  (SELECT embedding
   FROM `project.dataset.query_embeddings`
   WHERE query_id = 'Q001'),
  top_k => 5,
  distance_type => 'COSINE'
);
```

### AI.SEARCH (자연어 검색)
```sql
-- 자연어로 데이터 검색
SELECT *
FROM AI.SEARCH(
  TABLE `project.dataset.products`,
  'search_index',
  'lightweight portable laptop for students'
)
LIMIT 10;
```

---

## BigQuery Continuous Queries (실시간 쿼리)

스트리밍 데이터를 **실시간으로 지속적으로 분석**합니다.

```sql
-- 실시간 이상 탐지 쿼리
-- Pub/Sub에서 데이터 지속 수신 → 이상 탐지 → 결과를 Pub/Sub으로 발행
EXPORT DATA
  OPTIONS (uri = 'pubsub://projects/PROJECT/topics/ALERTS')
AS (
  SELECT transaction_id, amount, timestamp, 'FRAUD_SUSPECTED' AS alert_type
  FROM `project.streaming_dataset.transactions`
  WHERE amount > 10000
    AND timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 5 MINUTE)
);
```

---

## 2024-2025 주요 업데이트 타임라인

| 시기 | 기능 |
|------|------|
| 2024 Q1 | Analytics Hub 데이터 구독 모델 강화 |
| 2024 Q2 | BigQuery ML - TimesFM 시계열 모델 지원 |
| 2024 Q3 | Dataplex Universal Catalog 출시 |
| 2024 Q4 | BigQuery Editions 전면 개편 |
| 2025 Q1 | Advanced Runtime Preview |
| 2025 Q2 | 크로스클라우드 JOIN 정식 지원 |
| 2025 Q3 | BigQuery Advanced Runtime 기본 엔진 전환 |
| 2025 Q4 | Enterprise Plus 크로스 리전 DR 강화 |

---

## 참고 자료

### 공식 문서
- [BigQuery 개요](https://cloud.google.com/bigquery/docs/introduction)
- [BigQuery Omni](https://cloud.google.com/bigquery/docs/omni-introduction)
- [Analytics Hub](https://cloud.google.com/bigquery/docs/analytics-hub-introduction)
- [BigQuery ML](https://cloud.google.com/bigquery/docs/bqml-introduction)
- [Dataplex](https://cloud.google.com/dataplex/docs/introduction)
- [벡터 검색](https://cloud.google.com/bigquery/docs/vector-search-intro)
- [BigQuery Editions](https://cloud.google.com/bigquery/docs/editions-intro)

---

[← 이전: 성능 최적화](09-performance.md) | [← 목차로 돌아가기](index.md)
