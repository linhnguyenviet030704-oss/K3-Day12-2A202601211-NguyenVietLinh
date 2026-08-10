# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Việt Linh  Mã học viên: 2A202601211

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu không chết thì user có thể sử dụng biến mặc định free, rút cạn quota -> bay ví. Việc chết sớm giúp mình nhận ra mình quên key set trên cloud -> dễ quản lý hơn

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> 1 dòng log json mang nhiều key:value, có thể pretty print để đọc dễ dàng. print muốn được như thế thì phải code cấu hình trong câu print.

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
| 1 stage (bản đầu) | 900 MB |
| Multi-stage | 63.7 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Build 1 stage thì image bao gồm compiler, pip cache, source code build tools. Build 2 stage thì stage 2 chỉ copy kết quả stage 1.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện tại đang chia 2 stage, cache stage 1 gần như không đổi. Khi đổi main.py thì layer COPY app/app sẽ đổi vì main.py trong đó, các layer khác cache. Copy .. lên trước RUN pip install thì đổi 1 dấu phẩy sẽ chạy lại toàn bộ, vì .. gồm toàn bộ project.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Vì Docker container khởi chạy với quyền root trên host theo mặc định, nếu container bị xâm nhập thì kẻ tấn công có thể leo thang quyền và tác động tới host hoặc các container khác. Lệnh USER cắt bằng cách cho container chạy với user không có quyền root. Khi code bị tấn công, tiến trình chỉ chạy với quyền hạn chế, nên kẻ tấn công khó/không thể thao tác các file hệ thống quan trọng hoặc tấn công host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Một người dùng có thể gửi tối đa 20 request trong 2 giây liên tiếp nếu tận dụng ranh giới phút. Cụ thể, người đó gửi 10 request vào giây 59 của phút trước, rồi ngay lập tức gửi 10 request nữa vào giây 00 của phút tiếp theo, vì bộ đếm phút cũ đã hết và bộ đếm phút mới được reset lúc giây 00.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit chặn số lượt hỏi, cost guard chặn lượng hỏi. User hỏi 2 câu nhưng token bằng 20 câu thì cost cực lớn. User hỏi nhiều câu cost ít thì cost cũng dồn -> cần cả 2.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối, endpoint `/health` bị gộp với `/ready` nên bắt đầu kiểm tra Redis. Khi Redis không trả lời, endpoint trả lỗi 503 hoặc không healthy. Orchestrator/liveness probe thấy cả 3 container không khỏe, nên đánh dấu chúng unhealthy và cố gắng khởi động lại. Vì Redis vẫn mất trong 30 giây, các container vừa khởi động lên lại lại bị probe fail tiếp và có thể bị restart liên tục. Kết quả là cụm 3 container bị ngắt ra khỏi dịch vụ, traffic không vào được, và khi Redis về lại thì mới phục hồi bình thường.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với --scale agent=3, nginx sẽ phân phối request qua 3 container agent khác nhau, nhưng vì state/history được lưu Redis, history_length vẫn ổn định với cùng X-User-Id. Nếu lịch sử lưu trong dict python thì các lần /ask cùng thời điểm sẽ cập nhật chồng lấn lên nhau

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> health check fail, check log và nhét log vào Codex :D. Lỗi tìm được là chỉ có 30s retry windows, /health chưa trả kết quả được -> fail
