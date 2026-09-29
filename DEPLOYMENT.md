# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Huy Hoàng |
| Mã học viên | 2A202602738 |
| Repo | https://github.com/Hoang-H-Nguyen/K4-L3B-Day12-NguyenHuyHoang-2A202602738-Cloud-Service-And-Deployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-gewm.onrender.com |
| Platform | Render |
| Ngày deploy | 29/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value `day12-redis`; Blueprint gán connectionString vào REDIS_URL |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Chạy các lệnh sau với API key của service được nạp vào biến môi trường:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-gewm.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-gewm.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-gewm.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-gewm.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-gewm.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Kết quả thực tế của `/health`, `/ready` và `/ask` không có API key ngày 29/09/2026:

```
HTTP/2 200
date: Tue, 29 Sep 2026 05:30:52 GMT
content-type: application/json
rndr-id: 0446e6b5-95c0-418e
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a428992dab8f1073-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}

HTTP/2 200
date: Tue, 29 Sep 2026 05:30:53 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: 0b9e03d6-a03e-4c3b
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a42899341f2a1057-HKG
alt-svc: h3=":443"; ma=86400


{"status":"ready","redis":true}

HTTP/2 401
date: Tue, 29 Sep 2026 05:30:54 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: 49964a74-5620-4e2d
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a42899373d588a13-SIN
alt-svc: h3=":443"; ma=86400
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl
