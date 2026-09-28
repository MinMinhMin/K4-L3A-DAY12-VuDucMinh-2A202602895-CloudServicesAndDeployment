# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> Bài làm cá nhân, viết dựa trên những gì tôi đã quan sát khi chạy lab.
>
> Họ và tên: VŨ ĐỨC MINH  
> Mã học viên: 2A202602895

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu tôi quên đặt `AGENT_API_KEY` thì ứng dụng sẽ fail fast thay vì chạy với một khóa mà ai cũng có thể đoán được. Nếu để mặc định là `changeme`, service vẫn báo healthy và tôi có thể vô tình public một API không được bảo vệ. Việc khởi động thất bại buộc tôi kiểm tra lại biến môi trường trước khi có người gọi API và phát sinh chi phí.

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu hai việc bạn làm được với dòng log đó mà `print("đã trả lời xong")` không làm được.

> Một dòng log tôi thu được khi gọi `/ask` là:
>
> ```json
> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T14:05:08.300790+00:00", "user_id": "exercise-log", "tokens_in": 5, "tokens_out": 39, "cost_usd": 2.415e-05}
> ```
>
> Vì log là JSON, tôi có thể lọc riêng các event `ask_completed` theo `user_id` để biết user nào đang gọi service. Tôi cũng có thể cộng `cost_usd` hoặc tổng số token theo thời gian để theo dõi chi phí và phát hiện mức sử dụng bất thường. Một dòng `print` thông thường không có cấu trúc thống nhất nên khó lọc và thống kê tự động.

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật.

| Bản | Dung lượng |
|-----|-----------:|
| 1 stage (`python:3.11`) | 446.6 MB |
| Multi-stage (`python:3.11-slim`) | 63.9 MB |

> Bản one-stage mang theo base image đầy đủ và toàn bộ phần cài đặt trong cùng một image. Bản multi-stage chỉ copy dependency cần chạy và source code sang runtime image slim, nên không mang theo các thành phần dư thừa của stage build. Chênh lệch đo được là khoảng 382.7 MB, chủ yếu đến từ base image lớn hơn và các công cụ/file không cần thiết khi chạy production.

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt `COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tôi chỉ sửa `app/main.py`, layer `COPY requirements.txt` và layer `pip install` ở builder vẫn được dùng lại vì `requirements.txt` không đổi. Layer `COPY app ./app` bị chạy lại; các layer runtime nằm sau nó như copy source còn lại, tạo user và healthcheck cũng phải được xây lại theo layer cha mới. Nếu đặt `COPY . .` trước `RUN pip install`, chỉ cần sửa một dòng source là Docker mất cache của layer copy và phải chạy lại `pip install`, làm build chậm hơn rất nhiều.

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ một lỗ hổng
trong code Python tới kẻ tấn công có quyền cao trên máy host, và lệnh `USER`
cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép chạy lệnh từ request, attacker có thể thực thi lệnh với quyền của process trong container. Nếu process chạy bằng root thì attacker có quyền đọc, sửa hoặc xóa nhiều thứ hơn trong container và có mức độ nguy hiểm cao hơn nếu khai thác tiếp được container runtime hoặc mount nhầm file của host. Lệnh `USER appuser` làm process chạy bằng user thường, vì vậy kể cả khi có lỗi RCE thì quyền ban đầu của attacker cũng bị giới hạn. Nó không thay thế các lớp bảo mật khác, nhưng giảm đáng kể hậu quả của lỗi.

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút?

> Có thể gửi tối đa 20 request trong khoảng 2 giây. Cụ thể, người dùng gửi 10 request ở giây 59 của phút hiện tại, sau đó chờ sang giây 00 và gửi tiếp 10 request. Bộ đếm theo phút đồng hồ đã reset nên cho qua cả hai nhóm, dù thực tế có 20 request trong khoảng thời gian rất ngắn. Sliding window 60 giây tránh được khe hở này.

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một khoảng thời gian, còn cost guard giới hạn tổng chi phí theo từng user trong tháng. Ví dụ user mới chỉ gửi request thứ hai trong phút nên rate limit cho qua, nhưng trước đó đã tiêu gần hết ngân sách và request mới làm vượt monthly budget, nên cost guard phải trả 402. Ngược lại, user có ngân sách tháng còn nhiều và mỗi request rất rẻ thì cost guard cho qua, nhưng nếu gửi request thứ 11 trong cùng một phút thì rate limiter phải trả 429.

### Câu 8 — `/health` khác `/ready` (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây?

> Nếu endpoint duy nhất kiểm tra Redis trả lỗi, load balancer sẽ coi cả ba container đều không healthy và có thể loại chúng khỏi traffic hoặc khởi động lại chúng. Khi Redis chỉ mất kết nối tạm thời, việc restart cả cụm tạo ra một vòng restart không cần thiết và làm gián đoạn service. Thiết kế hiện tại tách hai mục đích: `/health` chỉ kiểm tra process nên vẫn trả 200, còn `/ready` kiểm tra Redis và trả 503 để ngừng nhận request mới nhưng không restart container.

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, các request đi qua những container khác nhau vẫn nhìn thấy cùng một lịch sử; sau mỗi request, `history_length` tăng theo hai message user và assistant, ví dụ 0, 2, 4. Nếu dùng một dict trong RAM, mỗi container có một bản sao riêng. Request đầu vào container A có thể trả 0 rồi request tiếp theo vào container B cũng trả 0, hoặc lịch sử bị mất hoàn toàn khi container restart. Vì vậy state hội thoại phải nằm ngoài process.

### Câu 10 — Deploy thật (CP5)

Ghi lại một lỗi bạn gặp khi deploy lên cloud: thông báo lỗi là gì, bạn tìm ra
nguyên nhân bằng cách nào, và sửa ra sao.

> Lỗi thực tế tôi gặp là `/ready` trả `503` với body `{"status":"not ready","redis":false}`, trong khi `/health` vẫn trả `200`. Điều đó cho thấy container và domain vẫn chạy nhưng app không ping được Redis. Tôi đối chiếu hai endpoint, kiểm tra lại biến `REDIS_URL` trong Railway và nhận ra service Agent cần tham chiếu tới `REDIS_URL` của Railway Redis cùng project/environment, không dùng hostname `redis` của Docker Compose local. Sau khi sửa reference variable, redeploy service và cập nhật key deploy ở local, `/ready` trả `200` với `redis:true` và test CP5 pass.
