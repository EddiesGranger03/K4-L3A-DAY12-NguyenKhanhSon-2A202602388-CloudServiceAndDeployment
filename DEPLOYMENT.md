# Thông tin deploy — Checkpoint 5

> Đang chuẩn bị deploy trên Render. Cập nhật URL, kết quả và ảnh chụp sau khi
> service chạy thật. Chỉ ghi **tên** biến môi trường; không ghi API key hay
> Redis URL chứa mật khẩu vào repository.

## Thông tin học viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Khánh Sơn |
| Mã học viên | 2A202602388 |
| Repo hiện tại | [GitHub](https://github.com/EddiesGranger03/K4-L3A-DAY12-NguyenKhanhSon-2A202602388-Cloud-Service-And-Deployment) |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | Chờ Render cấp sau khi deploy |
| Platform | Render |
| Ngày deploy | Chờ deploy thành công |
| Trạng thái | Chưa xác nhận deploy thành công |

## Cấu hình dự kiến trên Render

Blueprint [`render.yaml`](render.yaml) tạo web service và Render Key Value.
Khi tạo Blueprint, Render yêu cầu nhập `AGENT_API_KEY` vì biến này có
`sync: false`. Hãy tạo **khóa mới**, lưu trong dashboard và không gửi qua chat.
`REDIS_URL` được Blueprint nối tự động từ Render Key Value qua
`connectionString`; không cần dùng Upstash cho phương án này.

| Biến | Nguồn giá trị | Trạng thái |
|------|---------------|------------|
| `PORT` | Render cấp cho web service | Chờ deploy |
| `AGENT_API_KEY` | Nhập trong dashboard lúc tạo Blueprint | Chờ nhập |
| `REDIS_URL` | Kết nối nội bộ từ Render Key Value | Chờ deploy |
| `RATE_LIMIT_PER_MINUTE` | Blueprint: 10 | Chờ deploy |
| `MONTHLY_BUDGET_USD` | Blueprint: 10.0 | Chờ deploy |
| `LOG_LEVEL` | Blueprint: INFO | Chờ deploy |

## Kiểm tra sau khi deploy (PowerShell)

Điền URL thật vào `$serviceUrl` và chạy từ PowerShell. Chỉ đặt khóa ở biến
PowerShell trên máy cá nhân; không thêm khóa vào tài liệu này.

```powershell
$serviceUrl = "https://TEN-SERVICE.onrender.com"
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

Chưa có URL và output của bản deploy. Điền trạng thái HTTP và nội dung phản
hồi thật sau khi kiểm tra ba endpoint trên.

## Ảnh chụp màn hình

Lưu ảnh dashboard Render và kết quả gọi `/health` vào `screenshots/`, sau đó
ghi tên file tại đây. Che mọi giá trị API key và mật khẩu Redis trước khi chụp.
