# 이상적인 출력 예시 — 에러 로그 분석 보고서

> 위 input-example.md 입력에 대해 "이렇게 나오면 완벽하다"는 기준 보고서.
> 형식·구조·톤·판단 기준이 이 파일을 그대로 따른다.

---

## 에러 로그 분석 보고서

**분석 범위:** 2026-05-15 02:14 ~ 09:14 (운영 환경)
**대상 서비스:** api-gateway, payment-service, notification-service, auth-service, data-pipeline
**총 에러 건수:** 9건 ERROR + 2건 WARN

---

## 1. 에러 분류표

| # | 에러 유형 | 서비스 | 발생 횟수 | 심각도 | 상태 |
|---|---|---|---|---|---|
| E-01 | NullPointerException — AuthFilter.java:87 | api-gateway | 4회 | 🔴 HIGH | 반복 발생 |
| E-02 | ConnectionPoolExhaustedException | payment-service | 2회 | 🔴 HIGH | 반복 발생 |
| E-03 | ExternalApiException — SMTP 502 | notification-service | 1회 | 🟡 MEDIUM | 재시도 소진, 큐 대기 |
| E-04 | DataCorruptionException — checksum mismatch | data-pipeline | 1회 | 🔴 HIGH | 원인 불명 |
| E-05 | ClassCastException — ResponseMapper.java:201 | api-gateway | 1회 | 🟡 MEDIUM | 최근 배포 후 첫 발생 |
| W-01 | Slow query — orders 테이블 풀스캔 | payment-service | 1회 | 🟡 MEDIUM | 인덱스 누락 의심 |
| W-02 | RefreshTokenExpiredException 빈도 급증 | auth-service | 다수 | 🟡 MEDIUM | 정상 만료이나 3배 급증 |

---

## 2. 근본원인 추정

**E-01 (NullPointerException):**
토큰 검증 로직이 null을 반환하는 케이스 미처리. 02:14 ~ 02:22 사이 4건 연속 발생 → 특정 조건에서 반복 트리거되는 버그 가능성 높음.

**E-02 (ConnectionPoolExhausted):**
W-01 슬로우 쿼리(4823ms)가 커넥션을 점유한 채 반환하지 않아 풀(20/20) 소진. W-01 → E-02 인과관계 의심.

**E-04 (DataCorruption):**
직전 job-4420은 정상 완료 → 파일 전송 중 손상 또는 쓰기 충돌 가능성. 원인 확정 불가 — ⚠️ 사람 검토 필요.

**E-05 (ClassCastException):**
v2.3.1 배포(2026-05-14 23:00) 이후 첫 발생 → 배포 변경사항과 ResponseMapper 호환성 문제 의심.

**W-02 (RefreshToken 급증):**
빈도 3배 급증 원인 불명 — 외부 공격(토큰 브루트포스) 또는 클라이언트 버그 가능성. ⚠️ 사람 검토 필요.

---

## 3. 우선순위별 액션 아이템

| 우선순위 | 항목 | 담당 | 기한 |
|---|---|---|---|
| P0 | E-01: AuthFilter null 체크 추가 및 핫픽스 검토 | 개발팀 | 즉시 |
| P0 | E-02: orders 테이블 인덱스 추가 → 커넥션 풀 회복 | DB팀 | 즉시 |
| P1 | E-04: batch_20260515_0800.csv 무결성 재확인 + 재처리 여부 결정 | ⚠️ 사람 판단 필요 | 당일 |
| P1 | E-05: v2.3.1 ResponseMapper 변경사항 검토 + 롤백 여부 결정 | ⚠️ 사람 판단 필요 | 당일 |
| P2 | W-02: RefreshToken 급증 원인 조사 (공격 or 버그) | 보안/개발팀 | 금일 내 |
| P3 | E-03: SMTP 공급사 장애 여부 확인 + 큐 메시지 재발송 | 운영팀 | 금일 내 |

---

## 4. ⚠️ 사람 검토 필요 항목

아래 항목은 AI가 판단할 수 없으므로 담당자가 직접 확인해야 합니다.

- **E-04 DataCorruption:** 데이터 손실 규모 확인 + 재처리 or 폐기 결정
- **E-05 ClassCastException:** v2.3.1 롤백 여부 — 비즈니스 영향도 판단 필요
- **W-02 토큰 급증:** 보안 위협 여부 — 외부 공격 로그와 대조 필요

---

## 5. 한 줄 요약

> 핵심 원인 2개: AuthFilter null 미처리(E-01)와 슬로우 쿼리발 커넥션 풀 소진(E-02). 즉시 핫픽스 후 E-04·E-05·W-02는 사람이 판단.
