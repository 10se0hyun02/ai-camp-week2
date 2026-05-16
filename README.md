# AI CAMP Week 2 — 조서현

**자동화 MVP:** 마케팅솔루션 에러 로그를 AI가 수집·분류·심각도 라벨링하여 일일 요약 초안을 생성하고, 담당자가 검토 후 Slack에 발송한다.

## AI / 사람 역할 경계

| 역할 | 담당 |
|---|---|
| 에러 로그 수집 · 분류 · 심각도 라벨링 · 요약 초안 생성 | AI |
| 원인 판단 · 조치 결정 · Slack 채널 실제 발송 | 사람 (담당자) |

## 입력 예시

마케팅솔루션 운영 중 발생하는 에러 로그 (익명화)

```
[2026-05-15 09:12:34] ERROR | service=ga4-connector | code=403 | msg="Access denied: property 123456789 not authorized"
[2026-05-15 09:13:01] WARN  | service=crm-sync | code=429 | msg="Rate limit exceeded: 1000 req/min cap hit"
[2026-05-15 09:13:45] ERROR | service=ad-platform-meta | code=500 | msg="Internal server error from Meta API"
[2026-05-15 09:22:50] ERROR | service=data-pipeline | code=504 | msg="Upstream timeout: BigQuery job exceeded 30s limit"
```

## 골든 출력 (이상적인 결과)

| 심각도 | 서비스 | 에러코드 | 건수 | 요약 |
|---|---|---|---|---|
| 🔴 긴급 | ga4-connector | 403 | 2건 | GA4 property 권한 만료 — 서비스 계정 재인증 필요 |
| 🔴 긴급 | data-pipeline | 504 | 1건 | BigQuery 쿼리 타임아웃 — 최적화 필요 |
| 🟡 주의 | crm-sync | 429 | 1건 | CRM API 속도 제한 도달 — 배치 간격 조정 권장 |
| 🟡 주의 | ad-platform-meta | 500 | 1건 | Meta API 서버 오류 — 재시도 후 지속 시 확인 |

## 첫 초안 vs 골든 출력 주요 격차

| 기준 | 초안 문제점 |
|---|---|
| 빠진 정보 | 심각도 없음, 조치 방향 없음, 건수 집계 없음 |
| 구조 차이 | 번호 목록 나열 vs 표 + 조치 섹션 분리 |
| 톤 차이 | "좋을 것 같습니다" 등 완곡 표현 → 운영 리포트에 부적합 |
| 사실 오류 | 시간 범위 임의 축소, 추측 표현 사용 |
| 역할 경계 | 담당자 액션 포인트 안내 없음 |

## 3주차 제약 조건 후보

1. 심각도 라벨 3단계 필수: 🔴 긴급 / 🟡 주의 / ⬜ 정상
2. 출력 형식 고정: `심각도 | 서비스 | 에러코드 | 건수 | 요약` 표 + 조치 필요 항목 목록
3. 추측성 표현 금지: "~인 것 같습니다", "~보입니다" 사용 불가
4. 반복 에러 집계 필수: 동일 service+code 2건 이상이면 자동으로 🔴 긴급 상향
5. 정상 항목 포함 필수: INFO 레벨도 ⬜ 정상으로 표에 포함

## 민감정보 처리

- 모든 입력 예시는 익명화됨 (실제 고객 연락처·내부 시스템 정보 미포함)
- 서비스 계정명·캠페인 ID·이메일 주소는 가상값으로 대체

---

*AI CAMP 1기 — 하네스 엔지니어링 주말반*
