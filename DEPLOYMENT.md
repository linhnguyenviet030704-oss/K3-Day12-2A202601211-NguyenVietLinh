# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên |  Nguyễn Việt Linh |
| Mã học viên | 2A202601211 |
| Repo | https://github.com/linhnguyenviet030704-oss/K3-Day12-2A202601211-NguyenVietLinh |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ |
| Platform | Railway  |
| Ngày deploy | 10/8/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | (điền: Redis add-on của platform / Upstash / ...) |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
Invoke-RestMethod -Uri https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
Invoke-RestMethod -Uri https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
Invoke-RestMethod -Uri "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -Method Post -ContentType "application/json" -Body '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
Invoke-RestMethod -Uri "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -Method Post -Headers @{ "Content-Type" = "application/json"; "X-API-Key" = $AGENT_API_KEY; "X-User-Id" = "sv-test" } -Body '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
1..15 | % { curl.exe -s -o NUL -w "%{http_code} " -X POST "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv-test" -d '{"question":"test"}' }; Write-Host ""
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```

PS C:\Users\Admin> Invoke-RestMethod -Uri https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/health

status service     version
------ -------     -------
ok     day12-agent 1.0.0


PS C:\Users\Admin> Invoke-RestMethod -Uri https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ready

status redis
------ -----
ready   True

PS C:\Users\Admin> Invoke-RestMethod -Uri "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -Method Post -ContentType "application/json" -Body '{"question":"Hello"}'
Invoke-RestMethod : {"detail":"invalid or missing API key"}
At line:1 char:1
+ Invoke-RestMethod -Uri "https://k3-day12-2a202601211-nguyenvietlinh-p ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:HttpWebRequest) [Invoke-RestMethod], WebExc
   eption
    + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShell.Commands.InvokeRestMethodCommand


PS C:\Users\Admin> Invoke-RestMethod -Uri "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -Method Post -Headers @{"X-API-Key"=$AGENT_API_KEY; "X-User-Id"="sv-test"} -ContentType "application/json; charset=utf-8" -Body $bodyBytes -DisableKeepAlive


answer         : CÃ¢u há»i hay. Deploy lÃ gÃ¬ thÆ°á»ng ÄÆ°á»£c giáº£i quyáº¿t báº±ng cÃ¡ch chuáº©n hÃ³a mÃ´i
                 trÆ°á»ng cháº¡y: cÃ¹ng má»t image cháº¡y giá»ng nhau á» laptop vÃ trÃªn cloud.
user_id        : sv-test
history_length : 0
cost_usd       : 2.145E-05
tokens         : @{in=3; out=35}

PS C:\Users\Admin> 1..15 | % { curl.exe -s -o NUL -w "%{http_code} " -X POST "https://k3-day12-2a202601211-nguyenvietlinh-production.up.railway.app/ask" -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv-test" -d '{"question":"test"}' }; Write-Host ""
422 422 422 422 422 422 422 422 422 422 422 422 422 422 422

    

```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
(điền lý do nếu dùng phương án dự phòng, ngược lại xóa mục này)
```
