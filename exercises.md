# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: viết nội dung của mình dưới mỗi câu hỏi.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Khánh Sơn  Mã học viên: 2A202602388

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu tôi quên đặt `AGENT_API_KEY` lúc deploy, `Settings` báo lỗi ngay khi
> khởi động, nên tôi biết phải sửa cấu hình trước khi mở service. Nếu code có
> khóa mặc định `changeme`, service vẫn chạy và người biết khóa mặc định đó có
> thể gửi `/ask` như một người dùng hợp lệ. Đây là rủi ro vì endpoint này có
> lưu hội thoại và tiêu ngân sách; lỗi cấu hình sẽ khó phát hiện hơn.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log thu được khi gọi `/ask` bằng client cục bộ với Redis giả:
>
> `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:52:54.612978+00:00", "user_id": "exercise-local", "tokens_in": 4, "tokens_out": 36, "cost_usd": 2.22e-05}`
>
> Tôi có thể lọc theo `event` và `user_id` để tìm các lần gọi của một người
> dùng, và cộng `cost_usd` để theo dõi chi phí. Chuỗi `print("đã trả lời xong")`
> không có các trường dữ liệu đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f Dockerfile.single-stage -t agent:single .
docker build -t agent:multi .
docker images agent:single
docker images agent:multi
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build hai image trên máy bằng `Dockerfile.single-stage` và `Dockerfile`:
> Docker báo 1.73 GB và 271 MB, chênh khoảng 1.46 GB. Bản một stage dùng
> `python:3.11` đầy đủ, cài dependency ngay trong image chạy ứng dụng và còn
> cache của pip. Bản multi-stage dùng `python:3.11-slim`, cài thư viện với
> `--no-cache-dir` ở builder và chỉ copy phần đã cài sang runtime.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Tôi thêm một dấu chấm trong comment ở `app/main.py`, rồi build lại với
> `--progress=plain`. Output ghi `CACHED` cho `COPY requirements.txt`,
> `RUN pip install` và `COPY --from=builder`; `COPY app`, `COPY utils` và
> `RUN useradd` hiện `DONE` nên được thực hiện lại. Sau đó tôi đã bỏ dấu chấm
> để giữ nguyên mã nguồn. Nếu `COPY . .` đứng trước `RUN pip install`, thay
> đổi ở `app/main.py` cũng làm bước cài thư viện mất cache.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu một lỗ hổng cho phép chạy lệnh trong process Python, kẻ tấn công có
> quyền của user đang chạy process. Khi container chạy bằng root, quyền đó là
> root **bên trong container**; nếu còn có cấu hình nguy hiểm như mount thư
> mục host nhạy cảm hoặc lỗ hổng thoát container, phạm vi ảnh hưởng có thể
> lan sang host. `USER appuser` làm process chạy với UID 10001, nên bước đầu
> tiên không còn quyền root trong container. Nó giảm thiểu rủi ro, nhưng không
> tự bảo đảm chống mọi kiểu thoát container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request: gửi 10 request ở giây 59 ngay trước khi phút cũ kết
> thúc, rồi gửi thêm 10 request ở giây 00 của phút mới. Bộ đếm theo phút
> đồng hồ đã reset nên cả hai đợt đều hợp lệ dù cách nhau khoảng 2 giây.
> Sliding window nhìn lại đúng 60 giây gần nhất sẽ chặn đợt thứ hai.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm số request trong 60 giây gần nhất; cost guard so tổng tiền
> của từng user trong tháng với ngân sách. Một user mới chỉ gửi một request
> trong phút nhưng đã tiêu vượt ngân sách tháng sẽ qua rate limit và bị cost
> guard chặn bằng 402. Ngược lại, user còn nhiều ngân sách nhưng đã gửi đủ
> 10 request trong 60 giây sẽ bị rate limit chặn bằng 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm probe gộp trả lỗi. Cụm đánh dấu cả ba agent không
> khỏe, rồi có thể restart chúng dù process Python vẫn hoạt động. Các agent
> khởi động lại vẫn không kết nối được Redis, nên probe tiếp tục lỗi và
> traffic bị gián đoạn cho đến khi Redis phục hồi. Với hai endpoint riêng,
> `/health` vẫn cho biết process còn sống, còn `/ready` báo chưa thể nhận
> request cần Redis; không cần restart cả cụm chỉ vì dependency tạm mất.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi 3 agent đang chạy sau Nginx, tôi gọi `/ask` năm lần với cùng một
> `X-User-Id` mới. Cả năm lần đều trả 200 và `history_length` lần lượt là
> 0, 2, 4, 6, 8. Redis lưu chung nên mỗi lần đều thấy hai tin nhắn của lượt
> trước. Nếu mỗi container dùng một dict Python riêng, request chuyển sang
> container khác có thể thấy số nhỏ hơn hoặc quay về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi tôi mở URL gốc của bản deploy trên Render, trang trả HTTP 404 với
> `{"detail":"Not Found"}`. Tôi kiểm tra lại các route trong `app/main.py` và
> gọi `/health`: endpoint này trả HTTP 200, còn `/ready` trả HTTP 200 với
> `redis: true`. Nguyên nhân là app chưa định nghĩa route `/`; đây là lỗi chọn
> đường dẫn khi kiểm tra, không phải lỗi build hay kết nối Redis. Ban đầu
> tôi kiểm tra đúng URL `/health` và `/ready`; sau đó tôi thêm route `/` để
> hiện dashboard. URL gốc hiện trả HTTP 200.
