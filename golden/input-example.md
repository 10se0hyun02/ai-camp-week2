# 실제 입력 예시 — 에러 로그

> 실제 서버 로그에서 가져온 데이터. 민감 정보(IP, 사용자 ID)는 익명화.

---

## 케이스 1 — 일반 에러 로그 묶음 (2026-05-15 오전 운영 환경)

```
2026-05-15T02:14:33Z [ERROR] service=api-gateway requestId=req-8a2f status=500
  NullPointerException at com.example.gateway.AuthFilter.doFilter(AuthFilter.java:87)
  caused by: token validation returned null for userId=user-***-001

2026-05-15T02:14:34Z [ERROR] service=api-gateway requestId=req-8a30 status=500
  NullPointerException at com.example.gateway.AuthFilter.doFilter(AuthFilter.java:87)
  caused by: token validation returned null for userId=user-***-002

2026-05-15T02:14:35Z [ERROR] service=api-gateway requestId=req-8a31 status=500
  NullPointerException at com.example.gateway.AuthFilter.doFilter(AuthFilter.java:87)
  caused by: token validation returned null for userId=user-***-003

2026-05-15T02:17:01Z [WARN]  service=payment-service requestId=req-9b10 status=200
  Slow query detected: 4823ms — SELECT * FROM orders WHERE user_id=? AND status='pending'
  index hint missing, table scan triggered

2026-05-15T02:18:45Z [ERROR] service=payment-service requestId=req-9b22 status=503
  ConnectionPoolExhaustedException: no available connections (pool=20/20)
  at com.example.db.PoolManager.acquire(PoolManager.java:134)

2026-05-15T02:18:46Z [ERROR] service=payment-service requestId=req-9b23 status=503
  ConnectionPoolExhaustedException: no available connections (pool=20/20)
  at com.example.db.PoolManager.acquire(PoolManager.java:134)

2026-05-15T02:20:10Z [ERROR] service=notification-service requestId=req-7c05 status=502
  ExternalApiException: POST https://api.smtp-provider.com/send → 502 Bad Gateway
  retry=3/3 failed, message queued for manual review

2026-05-15T02:22:55Z [ERROR] service=api-gateway requestId=req-8b01 status=500
  NullPointerException at com.example.gateway.AuthFilter.doFilter(AuthFilter.java:87)
  caused by: token validation returned null for userId=user-***-007
```

---

## 케이스 2 — 까다로운 케이스 (간헐적·원인 불명)

```
2026-05-15T08:05:12Z [ERROR] service=data-pipeline jobId=job-4421 status=FAILED
  DataCorruptionException: checksum mismatch on file=batch_20260515_0800.csv
  expected=a3f2c1d9 actual=00000000
  — 이전 실행(job-4420)은 정상 완료

2026-05-15T08:31:44Z [WARN]  service=auth-service requestId=req-cc01
  RefreshTokenExpiredException: token issued=2026-05-14T08:31:44Z ttl=86400s
  — 정상 만료이지만 재발 빈도 평소 대비 3배

2026-05-15T09:14:02Z [ERROR] service=api-gateway requestId=req-8c55 status=500
  ClassCastException at com.example.gateway.ResponseMapper.map(ResponseMapper.java:201)
  — 최근 배포(v2.3.1, 2026-05-14 23:00) 이후 처음 발생
```
