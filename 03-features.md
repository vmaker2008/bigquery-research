# 3. BigQuery 주요 기능

[← 목차로 돌아가기](index.md)

---

## 1. 쿼리 (Query)

### Google SQL (Standard SQL)
BigQuery는 ANSI SQL 표준을 준수하는 **GoogleSQL**을 사용합니다.

```sql
-- 기본 쿼리 예시
SELECT
  product_name,
  SUM(revenue) AS total_revenue,
  COUNT(DISTINCT customer_id) AS unique_customers
FROM `project.dataset.orders`
WHERE DATE(order_date) >= '2024-01-01'
GROUP BY product_name
ORDER BY total_revenue DESC
LIMIT 10;
```

### 고급 SQL 기능
- **Window Functions**: 순위, 이동 평균, 누적 합계 계산
- **JSON 처리**: 반정형 데이터를 SQL로 처리
- **지리공간 함수**: 지도 기반 분석 (`ST_DISTANCE`, `ST_CONTAINS` 등)
- **배열 및 구조체**: 중첩된 반복 데이터 처리
- **UDF(사용자 정의 함수)**: JavaScript, Python으로 커스텀 함수 작성

### 쿼리 최적화 도구
- **쿼리 설명(Explain)**: 실행 계획 사전 확인
- **쿼리 성능 인사이트**: 병목 지점 자동 감지
- **쿼리 캐싱**: 동일 쿼리 재실행 시 무료 캐시 결과 반환

---

## 2. 스토리지 (Storage)

### 열 기반 스토리지 (Columnar Storage)
BigQuery는 행(Row) 대신 열(Column) 단위로 데이터를 저장합니다.

```
행 기반 (전통 DB):     열 기반 (BigQuery):
━━━━━━━━━━━━━━━━      ━━━━━━━━━━━━━━━━━
[이름|나이|성별]       [이름열: 홍길동, 김철수, ...]
[홍길동|25|남]        [나이열: 25, 30, ...]
[김철수|30|남]        [성별열: 남, 남, ...]

→ 전체 행을 읽음       → 필요한 열만 읽음
→ 분석 쿼리에 비효율   → 분석 쿼리에 최적
```

### 데이터 타입
- **기본 타입**: INTEGER, FLOAT, NUMERIC, BOOLEAN, STRING, BYTES
- **날짜/시간**: DATE, TIME, DATETIME, TIMESTAMP
- **복합 타입**: ARRAY, STRUCT, JSON
- **지리공간**: GEOGRAPHY

### ACID 트랜잭션
- INSERT, UPDATE, DELETE, MERGE 지원
- 다중 테이블 ACID 트랜잭션 지원 (2023년 이후)
- 동시 수정에 대한 충돌 없는 처리

---

## 3. 스트리밍 (Streaming Ingestion)

### 실시간 데이터 수집

```
실시간 데이터 소스
        │
        ▼
   [Pub/Sub]  ←── IoT 센서, 앱 이벤트, 로그
        │
        ▼
  [Dataflow]  ←── 변환, 필터링 (선택적)
        │
        ▼
  [BigQuery]  ←── 수초 이내 쿼리 가능
```

### Streaming Insert 방식
- **API 직접 호출**: `tabledata.insertAll` API로 행 단위 삽입
- **지연시간**: 데이터 삽입 후 수 초 내 쿼리 가능
- **처리량**: 초당 수백만 행 수집 가능

### 연속 쿼리 (Continuous Queries)
- 스트리밍 데이터를 실시간으로 분석
- 이상 탐지, 알람 시스템에 활용
- Pub/Sub, Bigtable에 결과 실시간 발행

---

## 4. 데이터 로드 (Batch Loading)

### 지원 형식
| 형식 | 설명 |
|------|------|
| CSV | 일반 텍스트 |
| JSON | 반정형 데이터 |
| Avro | 바이너리, 스키마 포함 |
| Parquet | 컬럼 기반, 고압축 |
| ORC | 컬럼 기반 |
| Datastore Backup | Cloud Datastore 내보내기 |

### 데이터 소스
- **Cloud Storage**: GCS 버킷에서 직접 로드
- **로컬 파일**: 최대 10GB 직접 업로드
- **다른 Google 서비스**: Google Sheets, Drive
- **외부 서비스**: Data Transfer Service로 자동화

---

## 5. 연합 쿼리 (Federated Queries)

BigQuery 내부에 데이터를 복사하지 않고 **외부 데이터를 직접 쿼리**합니다.

### 지원 외부 소스
- **Cloud Storage**: CSV, Parquet, JSON 파일
- **Cloud Spanner**: 운영 DB와 분석 DB 동시 쿼리
- **Cloud SQL**: MySQL, PostgreSQL 실시간 조인
- **Bigtable**: NoSQL 데이터와 조인
- **AWS S3**: BigQuery Omni를 통해 쿼리
- **Azure Blob**: 멀티클라우드 분석

```sql
-- 외부 테이블과 BigQuery 테이블 조인 예시
SELECT
  bq.user_id,
  bq.purchase_amount,
  ext.user_profile
FROM `project.dataset.purchases` AS bq
JOIN EXTERNAL_QUERY(
  'connection_id',
  'SELECT * FROM users'
) AS ext ON bq.user_id = ext.id;
```

---

## 6. BigLake

**BigLake**는 BigQuery와 오픈 형식 데이터(Iceberg, Delta, Hudi)를 통합하는 스토리지 엔진입니다.

- Cloud Storage, Amazon S3, ADLS의 오픈 형식 파일 쿼리
- BigQuery의 보안 정책(행/열 수준)을 외부 데이터에도 적용
- 세밀한 접근 제어로 데이터 레이크 거버넌스 강화

---

## 7. BI Engine

**BI Engine**은 BigQuery의 인메모리 분석 가속 서비스입니다.

- 자주 사용하는 데이터를 메모리에 캐싱
- Looker, Tableau, Power BI 등 BI 도구 성능 향상
- 서브초(sub-second) 응답 시간 실현
- GB 단위 예약 (예: 1GB, 2GB, 10GB)

---

[← 이전: 아키텍처](02-architecture.md) | [다음: 요금 체계 →](04-pricing.md)
