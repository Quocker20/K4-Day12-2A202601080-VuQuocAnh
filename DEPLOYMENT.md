# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị token vào đây.**
> Repo này công khai — dán token vào là mất token.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Vũ Quốc Anh |
| Mã học viên | 2A202601080 |
| Repo | https://github.com/Quocker20/K4-Day12-2A202601080-VuQuocAnh|

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-day12-2a202601080-vuquocanh.onrender.com |
| Platform | Render |
| Ngày deploy | 10/8/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `API_TOKEN` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis Service trên Render |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `https://k4-day12-2a202601080-vuquocanh.onrender.com` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-day12-2a202601080-vuquocanh.onrender.com/healthz

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-day12-2a202601080-vuquocanh.onrender.com/readyz

# 3. Không có token — mong đợi 401 kèm header WWW-Authenticate
curl -i -X POST https://k4-day12-2a202601080-vuquocanh.onrender.com/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'

# 4. Có token — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-day12-2a202601080-vuquocanh.onrender.com/chat \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "X-Client-Id: sv-test" \
  -d '{"message":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-day12-2a202601080-vuquocanh.onrender.com/chat \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $API_TOKEN" \
    -H "X-Client-Id: sv-test" \
    -d '{"message":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```

1. (venv) PS C:\workspace\LAB\AITHUCCHIEN\LABS\K4-Day12-2A202601080-VuQuocAnh> curl.exe -i curl -i https://k4-day12-2a202601080-vuquocanh.onrender.com/healthz
curl: (6) Could not resolve host: curl
HTTP/1.1 200 OK
Date: Mon, 10 Aug 2026 09:50:09 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 6451914b-b109-4fc7
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a28e183a2fa59ca5-SIN
alt-svc: h3=":443"; ma=86400

2. (venv) PS C:\workspace\LAB\AITHUCCHIEN\LABS\K4-Day12-2A202601080-VuQuocAnh> curl.exe -i https://k4-day12-2a202601080-vuquocanh.onrender.com/readyz
HTTP/1.1 200 OK
Date: Mon, 10 Aug 2026 09:51:33 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 20f07d87-cdd8-44ae
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a28e1a482f73ce43-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}

3.(venv) PS C:\workspace\LAB\AITHUCCHIEN\LABS\K4-Day12-2A202601080-VuQuocAnh> curl.exe -i -X POST https://k4-day12-2a202601080-vuquocanh.onrender.com/chat \
>>   -H "Content-Type: application/json" \
>>   -d '{"message":"Hello"}'
HTTP/1.1 401 Unauthorized
Date: Mon, 10 Aug 2026 09:52:18 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: d3d00339-a0ed-4a44
Server: cloudflare
vary: Accept-Encoding
www-authenticate: Bearer
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a28e1b63ab29cf92-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing bearer token"}curl: (3) URL rejected: Bad hostname

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/healthz.png` — kết quả gọi `/healthz` từ trình duyệt hoặc curl

---


