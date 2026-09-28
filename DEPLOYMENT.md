# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | VŨ ĐỨC MINH |
| Mã học viên | 2A202602895 |
| Repo | https://github.com/MinMinhMin/K4-L3A-DAY12-VuDucMinh-2A202602895-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-vuducminh-2a202602895-cloudservices-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 28/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn giá trị; không ghi giá trị secret.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Railway tự gán |
| `AGENT_API_KEY` | ✅ | Railway service variable, secret |
| `REDIS_URL` | ✅ | Reference tới `REDIS_URL` của Railway Redis service |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

```bash
export BASE_URL="https://k4-l3a-day12-vuducminh-2a202602895-cloudservices-production.up.railway.app"
set -a
source .env
set +a

curl -i "$BASE_URL/health"
curl -i "$BASE_URL/ready"
curl -i -X POST "$BASE_URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
curl -i -X POST "$BASE_URL/ask" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST "$BASE_URL/ask" \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $DEPLOY_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done
echo
```

## Kết Quả Chạy Thật

Đã kiểm tra public service sau khi nối Railway Redis:

```text
/health  -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
/ready   -> 200 {"status":"ready","redis":true}
/ask không key -> 401 {"detail":"invalid or missing API key"}
/ask có key -> 200
{"user_id":"cp5-doc-check","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}
```

Không ghi API key hoặc Redis password vào tài liệu.

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Railway service, Redis service và domain.
- `screenshots/health.png` — kết quả gọi `/health`, `/ready` và `/ask`.

Không đưa API key, Redis password hoặc Railway token vào tài liệu và ảnh chụp.
