---
layout: default
title: 8. 보안 & 접근 제어
nav_order: 9
---

# 8. 보안 & 접근 제어

[← 목차로 돌아가기](index)

---

## 보안 레이어 개요

```
BigQuery 보안 아키텍처
┌──────────────────────────────────────────┐
│ 1. 네트워크 보안                          │
│    VPC Service Controls, Private IP      │
├──────────────────────────────────────────┤
│ 2. 인증 (Authentication)                 │
│    Google Identity, Service Account      │
├──────────────────────────────────────────┤
│ 3. 권한 부여 (Authorization)              │
│    IAM Roles & Permissions               │
├──────────────────────────────────────────┤
│ 4. 세분화된 접근 제어                     │
│    행 수준 보안 + 열 수준 보안            │
├──────────────────────────────────────────┤
│ 5. 암호화                                │
│    저장 중 + 전송 중 암호화               │
├──────────────────────────────────────────┤
│ 6. 감사 & 모니터링                        │
│    Cloud Audit Logs                      │
└──────────────────────────────────────────┘
```

---

## 1. IAM (Identity and Access Management)

### 최소 권한 원칙
사용자에게 필요한 최소한의 권한만 부여합니다.

### 주요 IAM 역할

#### 관리자 역할
| 역할 | 권한 범위 |
|------|----------|
| **BigQuery Admin** | 프로젝트 내 모든 BigQuery 리소스 완전 제어 |
| **BigQuery Studio Admin** | 데이터, Dataform, 노트북 완전 제어 |

#### 데이터 접근 역할
| 역할 | 권한 |
|------|------|
| **Data Owner** | 데이터셋, 테이블, 모델 완전 제어 + 권한 부여 |
| **Data Editor** | 데이터 생성, 수정, 삭제, 조회 |
| **Data Viewer** | 데이터 조회 및 내보내기만 가능 |
| **Metadata Viewer** | 메타데이터 조회만 가능 |

#### 쿼리 역할
| 역할 | 권한 |
|------|------|
| **Job User** | 쿼리 실행 (프로젝트 레벨) |
| **User** | 쿼리 실행 + 데이터셋 생성 |
| **Read Session User** | Storage Read API 사용 |

### IAM 적용 예시 (gcloud)
```bash
# 특정 사용자에게 데이터셋 조회 권한 부여
bq add-iam-policy-binding \
  --member="user:analyst@company.com" \
  --role="roles/bigquery.dataViewer" \
  project:dataset

# 서비스 계정에 쿼리 실행 권한 부여
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:sa@project.iam.gserviceaccount.com" \
  --role="roles/bigquery.jobUser"
```

---

## 2. 행 수준 보안 (Row-Level Security)

특정 사용자에게 테이블의 특정 행만 보이도록 제한합니다.

### 행 수준 접근 정책 생성
```sql
-- 영업 지역별 접근 제한
CREATE ROW ACCESS POLICY sales_region_policy
ON `project.dataset.sales`
GRANT TO ("group:us-team@company.com")
FILTER USING (region = 'US');

CREATE ROW ACCESS POLICY sales_region_policy_apac
ON `project.dataset.sales`
GRANT TO ("group:apac-team@company.com")
FILTER USING (region = 'APAC');

-- 관리자는 모든 행 접근
CREATE ROW ACCESS POLICY admin_all_access
ON `project.dataset.sales`
GRANT TO ("group:admin@company.com")
FILTER USING (TRUE);
```

```
결과:
┌──────────────────────────────────────────┐
│ 실제 데이터          │ 사용자가 보는 데이터  │
│ 지역=US, 매출=100    │ US팀: 지역=US만 보임  │
│ 지역=APAC, 매출=200  │ APAC팀: APAC만 보임  │
│ 지역=EU, 매출=150    │ 관리자: 모두 보임     │
└──────────────────────────────────────────┘
```

---

## 3. 열 수준 보안 (Column-Level Security)

민감한 열(컬럼)을 특정 사용자에게 숨기거나 마스킹합니다.

### Policy Tags 설정
```sql
-- 민감 데이터 분류 태그 적용 후 마스킹
-- (Google Cloud Console 또는 Terraform으로 설정)

-- 마스킹된 열 조회 시 결과:
-- 원본:  email = "user@example.com"
-- 마스킹: email = "u***@***.com"  (또는 NULL)
```

### 지원 마스킹 규칙
| 마스킹 규칙 | 결과 |
|------------|------|
| `SHA256` | 해시값으로 대체 |
| `Always NULL` | NULL 반환 |
| `Default Value` | 기본값 반환 (숫자=0, 문자="") |
| `Email Mask` | 이메일 마스킹 |
| `Last Four Characters` | 마지막 4자리만 표시 |

---

## 4. 암호화

### 기본 암호화 (자동 적용)
- **저장 데이터**: AES-256 자동 암호화
- **전송 중 데이터**: TLS 1.2+ 자동 암호화
- **키 관리**: Google이 자동으로 키 관리

### 고객 관리 암호화 키 (CMEK)
```sql
-- CMEK로 보호된 데이터셋 생성
CREATE SCHEMA `project.sensitive_dataset`
OPTIONS (
  location = 'US',
  default_kms_key_name = 'projects/PROJECT/locations/us/keyRings/RING/cryptoKeys/KEY'
);
```

### Cloud KMS 통합
```
Cloud KMS (키 관리)
       ↓ 키 제공
  BigQuery 데이터 암호화
       ↓ 키 취소 시
  데이터 접근 즉시 차단
```

---

## 5. VPC Service Controls

BigQuery 데이터를 특정 네트워크 경계 안에서만 접근 가능하도록 제한합니다.

```
인터넷 (외부)
      │
 ┌────▼────┐
 │ VPC SC  │ ← 서비스 경계
 │ 경계     │    (Access Policy)
 │  ┌──────┴──────┐
 │  │  BigQuery   │
 │  │  (보호됨)   │
 │  └─────────────┘
 └─────────┘
      ↑
 내부 네트워크만 접근 허용
```

---

## 6. 감사 로그 (Audit Logs)

모든 BigQuery 활동을 Cloud Audit Logs에 기록합니다.

### 로그 유형

| 로그 유형 | 기록 내용 |
|----------|----------|
| **Admin Activity** | 리소스 생성, 수정, 삭제 |
| **Data Access** | 데이터 읽기/쓰기 쿼리 |
| **System Event** | 시스템 자동 작업 |
| **Policy Denied** | 거부된 접근 시도 |

### 감사 로그 쿼리 예시
```sql
-- Cloud Logging에서 BigQuery 접근 로그 분석
SELECT
  timestamp,
  protopayload_auditlog.authenticationInfo.principalEmail AS user,
  protopayload_auditlog.methodName AS action,
  resource.labels.dataset_id AS dataset
FROM `project.logs.cloudaudit_googleapis_com_data_access_*`
WHERE resource.type = 'bigquery_dataset'
  AND DATE(timestamp) = CURRENT_DATE()
ORDER BY timestamp DESC;
```

---

## 7. 보안 체크리스트

```
BigQuery 보안 체크리스트
━━━━━━━━━━━━━━━━━━━━━━
☐ 최소 권한 원칙으로 IAM 역할 할당
☐ 서비스 계정 사용 (개인 계정 지양)
☐ 민감한 데이터 열에 Policy Tag 적용
☐ 개인정보 포함 테이블에 행 수준 보안 적용
☐ CMEK 적용 (규정 준수 필요 시)
☐ VPC Service Controls로 네트워크 경계 설정
☐ 감사 로그 활성화 및 모니터링
☐ 정기적인 IAM 권한 리뷰
☐ 사용하지 않는 서비스 계정 비활성화
☐ 예산 알림 설정 (이상 사용 탐지)
```

---

[← 이전: BigQuery ML & AI](07-ml-ai) | [다음: 성능 최적화 →](09-performance)
