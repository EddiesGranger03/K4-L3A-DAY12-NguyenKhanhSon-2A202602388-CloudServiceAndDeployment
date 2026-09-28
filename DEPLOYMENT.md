# Thông tin deploy — Checkpoint 5

> Đã deploy trên Render và kiểm tra các endpoint công khai. Chỉ ghi **tên**
> biến môi trường; không ghi API key hay
> Redis URL chứa mật khẩu vào repository.

## Thông tin học viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Khánh Sơn |
| Mã học viên | 2A202602388 |
| Repo hiện tại | [GitHub](https://github.com/EddiesGranger03/K4-L3A-DAY12-NguyenKhanhSon-2A202602388-CloudServicesAndDeployment) |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-mpaz.onrender.com |
| Platform | Render |
| Ngày kiểm tra | 28/09/2026 |
| Trạng thái | Dashboard `/` 200; `/health` 200; `/ready` 200; `/ask` không có key 401 |

## Cấu hình trên Render

Blueprint [`render.yaml`](render.yaml) tạo web service và Render Key Value.
Khi tạo Blueprint, Render yêu cầu nhập `AGENT_API_KEY` vì biến này có
`sync: false`. Hãy tạo **khóa mới**, lưu trong dashboard và không gửi qua chat.
`REDIS_URL` được Blueprint nối tự động từ Render Key Value qua
`connectionString`; không cần dùng Upstash cho phương án này.

| Biến | Nguồn giá trị | Trạng thái |
|------|---------------|------------|
| `PORT` | Render cấp cho web service | Service đã nhận request |
| `AGENT_API_KEY` | Nhập trong dashboard lúc tạo Blueprint | `/ask` từ chối request thiếu key |
| `REDIS_URL` | Kết nối nội bộ từ Render Key Value | `/ready` báo `redis: true` |
| `RATE_LIMIT_PER_MINUTE` | Blueprint: 10 | Đã khai báo trong Blueprint |
| `MONTHLY_BUDGET_USD` | Blueprint: 10.0 | Đã khai báo trong Blueprint |
| `LOG_LEVEL` | Blueprint: INFO | Đã khai báo trong Blueprint |

## Kiểm tra sau khi deploy (PowerShell)

Chạy từ PowerShell. Chỉ đặt khóa ở biến
PowerShell trên máy cá nhân; không thêm khóa vào tài liệu này.

```powershell
$serviceUrl = "https://day12-agent-mpaz.onrender.com"
curl.exe -i "$serviceUrl/health"
curl.exe -i "$serviceUrl/ready"
curl.exe -i -X POST "$serviceUrl/ask" -H "Content-Type: application/json" -d '{"question":"Hello"}'
```

Kết quả mong đợi lần lượt là HTTP 200 với `status=ok`, HTTP 200 với
`status=ready`, và HTTP 401 khi `/ask` không có API key. Để kiểm tra request
có xác thực, điền khóa trong biến môi trường cục bộ `DEPLOY_API_KEY` rồi chạy
`pytest tests/test_cp5.py -v`; test này là phần bổ sung và tự bỏ qua nếu
`DEPLOY_API_KEY` để trống.

## Kết quả chạy thật

Đã gọi trực tiếp Public URL ngày 28/09/2026:

```text
GET  /       → HTTP 200 (dashboard HTML)
GET  /health → HTTP 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready  → HTTP 200 {"status":"ready","redis":true}
POST /ask    → HTTP 401 {"detail":"invalid or missing API key"}
```

Lệnh POST gửi JSON `{"question":"Hello"}` và không gửi `X-API-Key`.
Trước khi thêm dashboard, route `/` trả HTTP 404; bản hiện tại đã có giao diện.
Request có xác thực chưa kiểm tra vì `DEPLOY_API_KEY` trên máy đang để trống;
đây là phần kiểm tra bổ sung của CP5.

## Ảnh chụp màn hình

- `screenshots/health.png`: ảnh phản hồi thật của `/health` trên Render.
- `screenshots/app-dashboard.png`: ảnh giao diện `/` chạy trên Render.
- `screenshots/dashboard.png`: trang Render Deploys của `day12-agent`, trạng thái Live.

Che mọi giá trị API key và mật khẩu Redis trước khi chụp dashboard.
