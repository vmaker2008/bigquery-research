# 6. BigQuery vs Snowflake vs Redshift

[← 목차로 돌아가기](index.md)

---

## 종합 비교표

| 항목 | BigQuery | Snowflake | Amazon Redshift |
|------|----------|-----------|-----------------|
| **제공사** | Google Cloud | 독립 (멀티클라우드) | Amazon Web Services |
| **아키텍처** | 완전 서버리스 | 하이브리드 (멀티 클러스터) | 공유 없음(Shared-Nothing) |
| **컴퓨팅/스토리지** | 완전 분리 | 분리 | 결합 (Redshift Serverless는 분리) |
| **설정 필요** | 없음 (NoOps) | 최소화 | 클러스터 관리 필요 |
| **확장 방식** | 자동 무한 확장 | 웨어하우스 크기 조정 | 수동 노드 추가 |
| **동시성** | 매우 높음 | 매우 높음 | 중간~높음 |
| **SQL 표준** | GoogleSQL (ANSI 호환) | Snowflake SQL (ANSI 호환) | Redshift SQL (PostgreSQL 기반) |
| **ML 지원** | BigQuery ML (내장) | Snowpark ML | Redshift ML (SageMaker 연동) |
| **멀티클라우드** | BigQuery Omni (AWS, Azure) | 기본 지원 (AWS, Azure, GCP) | AWS 기본, 제한적 멀티클라우드 |
| **가격 모델** | 온디맨드 or 슬롯 예약 | 컴퓨팅 초당 + 스토리지 | 온디맨드 or RI(예약 인스턴스) |
| **무료 티어** | 월 1TB 쿼리 + 10GB 스토리지 | 30일 무료 트라이얼 | 2개월 무료 |
| **오픈 포맷** | Iceberg, Delta, Hudi 지원 | Iceberg 지원 | 제한적 지원 |

---

## 상세 비교

### 성능

#### BigQuery
- 페타바이트 규모에서 탁월한 성능
- Dremel 엔진으로 수천 개 노드 병렬 처리
- 쿼리당 자동으로 최적 슬롯 할당

#### Snowflake
- 가상 웨어하우스 크기로 성능 조절 (XS → 4XL)
- 동일 데이터에 여러 웨어하우스 동시 실행 가능
- 쿼리 캐싱이 매우 효율적

#### Redshift
- 인메모리 캐싱(AQUA)으로 성능 향상
- 분산 키와 정렬 키 최적화 필요
- Redshift Serverless로 자동 확장 지원

---

### 비용 구조

```
BigQuery (온디맨드):
쿼리 비용 = 스캔한 데이터 TB × $6.25
장점: 쿼리 안 하면 비용 없음
단점: 대용량 쿼리 시 예측 어려움

Snowflake:
비용 = 컴퓨팅 크레딧 × 시간 + 스토리지 GB
장점: 웨어하우스 일시 정지로 비용 절감
단점: 웨어하우스 관리 필요

Redshift:
비용 = 인스턴스 타입 × 시간 (또는 서버리스)
장점: RI 구매 시 최대 75% 할인
단점: 사용 안 해도 클러스터 유지 비용 발생
```

---

### 생태계 통합

#### BigQuery
- **강점**: Google Analytics, Looker, Vertex AI, Dataflow와 완벽 통합
- **약점**: Google Cloud 생태계 외에서는 제한적

#### Snowflake
- **강점**: 모든 주요 클라우드, BI 도구, 데이터 파이프라인 도구와 폭넓은 통합
- **약점**: 특정 클라우드에 깊이 통합된 서비스 활용 어려움

#### Redshift
- **강점**: AWS 서비스(S3, EMR, SageMaker, Glue)와 깊은 통합
- **약점**: AWS 외 환경에서 사용 어려움

---

### 사용 편의성

```
학습 곡선:
BigQuery    ████░░░░░░  낮음 (NoOps, 간단한 설정)
Snowflake   ██████░░░░  중간 (웨어하우스 개념 이해 필요)
Redshift    ████████░░  높음 (클러스터 관리, 분산 키 설정)
```

---

## 선택 가이드

### BigQuery를 선택해야 할 때
- ✅ Google Cloud를 주요 플랫폼으로 사용
- ✅ 완전 서버리스, 인프라 관리 불필요
- ✅ 페타바이트 규모 데이터 처리 필요
- ✅ Google Analytics, Looker와 통합 중요
- ✅ SQL로 ML 모델 직접 구축 필요
- ✅ 예측 불가능한 워크로드 (사용량 기반 과금)

### Snowflake를 선택해야 할 때
- ✅ 멀티클라우드 전략 필요
- ✅ 다양한 데이터 공유 기능 필요 (Data Marketplace)
- ✅ 클라우드에 중립적인 포지션 유지
- ✅ 동시 쿼리 성능이 가장 중요
- ✅ 폭넓은 파트너 생태계 활용

### Redshift를 선택해야 할 때
- ✅ AWS가 주요 클라우드 플랫폼
- ✅ S3, EMR, SageMaker와 깊은 통합 필요
- ✅ PostgreSQL 기반 SQL 친숙
- ✅ 예약 인스턴스로 비용 최적화 가능

---

## 2024-2025년 트렌드

1. **모든 서비스가 서버리스로 이동**: Redshift Serverless, Snowflake Serverless도 출시
2. **오픈 테이블 포맷 경쟁**: Iceberg 지원이 업계 표준으로 정착
3. **AI/ML 통합 강화**: 모든 서비스가 LLM, 벡터 검색 기능 추가
4. **멀티클라우드 필수**: BigQuery Omni, Snowflake가 멀티클라우드 강화
5. **데이터 공유 생태계**: Analytics Hub, Snowflake Marketplace 경쟁 심화

---

[← 이전: 사용 사례](05-use-cases.md) | [다음: BigQuery ML & AI →](07-ml-ai.md)
