# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Vũ Quốc Anh  Mã học viên: 2A202601080

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Khi deploy ứng dụng lên server/cloud mà quên cấu hình biến môi trường `API_TOKEN`. Nếu có giá trị mặc định `"changeme"`, ứng dụng vẫn khởi động thành công, dẫn đến nguy cơ kẻ xấu lợi dụng token mặc định này để truy cập API trái phép hoặc tiêu tốn tài nguyên. Nếu không có mặc định (fail fast), ứng dụng sẽ crash ngay khi khởi động và báo lỗi thiếu biến môi trường, giúp người quản trị phát hiện và khắc phục sự cố lập tức trước khi mở dịch vụ cho người dùng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Dòng log JSON thu được:
```json
{"event": "chat_completed", "severity": "INFO", "ts": "2026-08-10T14:48:00.123456+00:00", "client_id": "sv01", "prompt_tokens": 12, "completion_tokens": 34, "usd_cost": 0.00015}
```

Hai việc làm được với log JSON cấu trúc mà `print("đã trả lời xong")` không làm được:
1. **Lọc và truy vấn dữ liệu tự động (Filtering & Aggregation):** Các hệ thống thu thập log (Cloud Run, Datadog, ELK) tự động parse JSON thành các trường key-value, giúp lọc nhanh log theo `client_id`, hoặc tính tổng lượng token (`prompt_tokens`, `completion_tokens`) và chi phí (`usd_cost`) của từng client.
2. **Cảnh báo và giám sát hệ thống (Alerting & Monitoring):** Dễ dàng thiết lập dashboard giám sát realtime và cảnh báo tự động khi `severity` chuyển thành `ERROR` hoặc `usd_cost` vượt ngưỡng, thay vì phải viết regex bóc tách thủ công từ chuỗi text thuần không có cấu trúc.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~1.01 GB (~1010 MB) |
| Multi-stage | ~378 MB |

Giải thích: Phần dung lượng chênh lệch (~630 MB) bao gồm:
1. **Khác biệt về Base Image:** Image `python:3.11` đầy đủ chứa hệ điều hành Debian hoàn chỉnh kèm toàn bộ công cụ biên dịch (GCC, g++, make), thư viện C/C++ header (`-dev`), git, curl và nhiều tiện ích hệ thống không cần thiết ở runtime. Ngược lại, `python:3.11-slim` đã được giản lược tối đa chỉ giữ lại môi trường chạy Python tối thiểu.
2. **Kỹ thuật Multi-stage Build:** Ở stage `builder`, các tệp tạm, pip cache và công cụ build phát sinh trong quá trình cài đặt thư viện đều bị bỏ lại ở stage đầu. Stage runtime cuối cùng chỉ sao chép kết quả `site-packages` đã cài xong, giúp hình ảnh không bị tích tụ rác và các layer dư thừa.


---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

- **Khi sửa 1 ký tự trong `app/main.py`**: Tất cả các layer trước `COPY . .` (bao gồm `FROM`, `WORKDIR`, `COPY requirements.txt .`, `RUN pip install`) đều được dùng lại 100% từ cache. Chỉ riêng từ layer `COPY . .` trở đi mới phải thực hiện lại (chỉ mất chưa tới 1 giây).
- **Nếu đặt `COPY . .` lên trước `RUN pip install`**: Mỗi lần sửa dù chỉ 1 dòng code Python, layer `COPY . .` sẽ làm phá vỡ (invalidate) cache của tất cả các lệnh đứng sau nó. Kết quả là Docker buộc phải chạy lại toàn bộ bước `RUN pip install` tốn rất nhiều thời gian và băng thông để tải và cài đặt lại các thư viện.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

- **Chuỗi sự kiện tấn công**:
  1. Kẻ tấn công khai thác một lỗ hổng trong ứng dụng Python (như RCE, Arbitrary File Write).
  2. Vì container chạy bằng user `root`, tiến trình Python sở hữu đầy đủ quyền root bên trong container (UID 0).
  3. Kẻ tấn công lợi dụng lỗ hổng thoát khỏi container (Container Escape) để thâm nhập vào máy host. Do UID trong container là 0, khi sang máy host họ vẫn mang UID 0 và trở thành `root` trên OS máy host.
- **Lệnh `USER` cắt đứt ở đâu**: Lệnh `USER appuser` hạ quyền tiến trình xuống một user thường không có đặc quyền. Khi ứng dụng bị thâm nhập, kẻ tấn công chỉ có quyền hạn hạn chế của `appuser`. Dù có tìm cách thoát khỏi container, họ cũng không có quyền root trên máy host.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

- **Header `WWW-Authenticate: Bearer`**: Theo chuẩn RFC 6750 / HTTP spec, response 401 Unauthorized bắt buộc phải kèm header này để chỉ dẫn cho client/HTTP agent biết phương thức xác thực được chấp nhận ở endpoint này là gì (Bearer scheme).
- **Trả cùng một thông báo lỗi**: Nhằm tránh rò rỉ thông tin (Information Leakage). Phản hồi chi tiết (ví dụ: "sai token") vô tình xác nhận cho kẻ tấn công biết rằng header và scheme đã đúng, giúp họ thu hẹp phạm vi thử sai khi dò quét / brute-force token. Trả cùng một thông báo giúp bảo mật theo nguyên tắc blind error.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

- **Số request gửi được**: Tối đa **10 request** trước khi nhận lỗi 429. Dù im lặng 10 phút, sức chứa tối đa (`capacity`) của xô chỉ là 10 token.
- **Nếu bỏ `min(capacity, ...)`**: Sau 10 phút im lặng, lượng token nạp thêm là $10 \times 10 = 100$ token. Không có `min()`, xô sẽ tích lũy thành **100 token** (hoặc 110 token). Khi đó client có thể xả liên tục **100 request** trong 1 giây mà không bị chặn, làm mất đi khả năng kiểm soát burst traffic của Rate Limiter.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

- **Hạn mức $30/tháng**:
  - Thiệt hại tối đa: Có thể tiêu sạch toàn bộ **$30** chỉ trong vài giờ đầu tiên của sự cố.
  - Tự hồi phục: Phải chờ đến **đầu tháng tiếp theo** service mới mở lại tự động.
- **Hạn mức $1/ngày**:
  - Thiệt hại tối đa: Giới hạn tổn thất tối đa chỉ **$1** cho sự cố trong ngày.
  - Tự hồi phục: Ngay **00:00 UTC ngày tiếp theo**, nhãn ngày mới được tạo và service tự động hồi phục phục vụ trở lại.


---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *Câu trả lời của bạn*

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
