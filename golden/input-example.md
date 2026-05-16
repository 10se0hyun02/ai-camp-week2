# 실제 입력 예시 — 마케팅솔루션 에러 로그

> DX팀 마케팅솔루션 운영 중 발생하는 실제 에러 로그 (익명화)

```
[2026-05-15 09:12:34] ERROR | service=ga4-connector | code=403 | msg="Access denied: property 123456789 not authorized" | user_id=svc-account-dx | retry=3/3
[2026-05-15 09:13:01] WARN  | service=crm-sync | code=429 | msg="Rate limit exceeded: 1000 req/min cap hit" | endpoint=/contacts/batch | retry=1/3
[2026-05-15 09:13:45] ERROR | service=ad-platform-meta | code=500 | msg="Internal server error from Meta API" | campaign_id=9988001 | retry=3/3
[2026-05-15 09:14:02] INFO  | service=email-trigger | msg="Batch job completed" | sent=1842 | failed=0 | duration_ms=4201
[2026-05-15 09:15:17] ERROR | service=ga4-connector | code=403 | msg="Access denied: property 123456789 not authorized" | user_id=svc-account-dx | retry=3/3
[2026-05-15 09:22:50] ERROR | service=data-pipeline | code=504 | msg="Upstream timeout: BigQuery job exceeded 30s limit" | job_id=bq-job-20260515-0041 | retry=2/3
[2026-05-15 09:23:18] WARN  | service=crm-sync | code=409 | msg="Conflict: duplicate contact detected" | contact_email=user@example.com | action=skipped
```
