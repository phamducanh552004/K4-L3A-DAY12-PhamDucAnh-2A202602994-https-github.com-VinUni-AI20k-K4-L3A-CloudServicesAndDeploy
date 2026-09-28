# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay phần trả lời trống dưới mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Đức Anh  Mã học viên: 2A202602994

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy Railway, nếu không có `AGENT_API_KEY` thì service sẽ không được
> phép nhận câu hỏi. Fail fast giúp deployment báo lỗi ngay thay vì API vẫn mở
> với khóa mặc định như `changeme`; nếu có người đoán được khóa đó, họ có thể
> gọi `/ask` và làm phát sinh chi phí AI. Tôi đã đặt khóa qua Railway Variables.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ log JSON có cấu trúc là `{"event":"request_completed","status":200,
> "path":"/ask"}`. Từ log này tôi có thể lọc theo `status` để đếm lỗi 401/429
> và truy vết endpoint nào xảy ra lỗi theo thời gian. `print` chỉ là văn bản
> tự do nên khó lọc, tổng hợp hoặc đưa vào công cụ quan sát hệ thống.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Chưa tạo image so sánh riêng |
| Multi-stage | Đã dùng để deploy Railway |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi không tạo Dockerfile một stage riêng để lấy số MB vì bài làm dùng trực
> tiếp Dockerfile multi-stage. Điểm quan trọng là stage cuối chỉ nhận dependency
> runtime và mã nguồn cần chạy; compiler, cache pip và file build tạm ở stage
> builder không đi sang image chạy thật. Vì vậy image nhỏ hơn, ít bề mặt tấn
> công hơn và tải lên cloud nhanh hơn.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, layer cài dependency được dùng lại từ cache vì
> file khai báo dependency không đổi; Docker chỉ copy lại source và tạo image
> cuối. Nếu `COPY . .` đứng trước lệnh cài dependency, mỗi lần sửa một file
> source sẽ làm invalid cache và phải cài lại thư viện, khiến build chậm hơn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗi thực thi mã tùy ý trong API có thể cho kẻ tấn công shell trong
> container. Nếu process là root, họ có thể sửa file hệ thống trong container,
> đọc secret có quyền truy cập và tận dụng cấu hình Docker sai để leo thang sang
> host. Lệnh `USER` chạy app bằng tài khoản không đặc quyền, nên shell có được
> cũng không có quyền root; đây là lớp giảm thiểu thiệt hại, không thay thế việc
> vá lỗi ứng dụng.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với bộ đếm reset theo phút, người dùng có thể gửi 10 request ở giây 59 và
> thêm 10 request ở giây 00, tức tối đa 20 request trong khoảng 2 giây. Sliding
> window luôn nhìn lại đúng 60 giây gần nhất nên các request ở cuối phút trước
> vẫn được tính và không có "khe hở" ở ranh giới phút.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit kiểm soát tốc độ request theo thời gian và người dùng; cost guard
> kiểm soát tổng ngân sách đã tiêu. Một user gửi request thứ 3 trong phút có thể
> chưa chạm limit nhưng bị chặn nếu ngân sách tháng đã hết. Ngược lại, tài khoản
> còn ngân sách nhưng gửi request thứ 11 trong một phút sẽ bị rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu một endpoint vừa dùng cho liveness vừa kiểm tra Redis, Redis mất 30 giây
> sẽ khiến cả ba container trả không khỏe. Orchestrator có thể lần lượt restart
> các container dù code Python vẫn đang chạy; khi Redis trở lại, các container
> khởi động lại gây gián đoạn không cần thiết. Tách `/health` chỉ kiểm tra process
> còn sống, còn `/ready` báo 503 để load balancer tạm ngừng gửi request mới.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Redis làm các instance cùng đọc một lịch sử theo `X-User-Id`, nên
> `history_length` tăng dần dù request vào các container khác nhau. Nếu dùng
> dict Python, mỗi container có một bản riêng: response có thể là 1, rồi 1 hoặc
> 2 tùy request được load balancer gửi tới instance nào. Người dùng sẽ thấy
> chatbot mất ngữ cảnh khi chuyển container.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy đầu tiên bị crash do chưa cấu hình biến môi trường cần thiết trên
> Railway. Tôi mở phần Variables của service, thêm `AGENT_API_KEY`, tham chiếu
> `REDIS_URL` từ dịch vụ Redis và các biến giới hạn. Sau khi Railway deploy lại,
> dashboard báo Active; tôi gọi `/health` được 200, `/ready` được 200 với
> `redis: true`, và `/ask` không có key trả 401.
