# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

> Đổi tên file này thành `MSSV_HoTen.md` của bạn, ví dụ `2A20260000_NguyenVanAn.md`, rồi điền từng mục. Xoá các dòng hướng dẫn in nghiêng khi xong.

- **MSSV:2A202603633**
- **Họ và tên:Nguyen Duc Minh**
- **SEED** 4814:

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | ☐ |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | ☐ |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | ☐ |

## 2. Kết quả chính

Model | F1 SUB 40 | p95 (ms) | USD/1k
LLM nhỏ zero-shot | 0.895 | 2896.43 | 0.00029
LLM lớn zero-shot | 0.950 | 3180.22 | 0.00291
LLM nhỏ few-shot | 0.919 | 3799.73 | 0.00073

LLM lớn có F1 cao nhất nhưng chi phí và độ trễ cũng cao hơn.

## 3. Dự đoán và thực tế

Dự đoán ban đầu:
- Mô hình lớn có F1 cao hơn mô hình nhỏ khoảng 0.02–0.05.
- Cả hai mô hình có F1 trong khoảng 0.85–0.95.
- Mô hình lớn có chi phí cao hơn khoảng 10 lần.
- Few-shot có thể cải thiện F1 nhẹ nhưng làm tăng chi phí vì prompt dài hơn.
  
Kết quả thực tế:
- LLM nhỏ zero-shot đạt F1 = 0.895.
- LLM lớn zero-shot đạt F1 = 0.950.
- Chênh lệch F1 là 0.055, cao hơn một chút so với dự đoán ban đầu.
- Chi phí LLM lớn khoảng 10 lần LLM nhỏ, đúng với dự đoán.
- Few-shot đạt F1 = 0.919, tăng 0.024 so với zero-shot.

## 4. Phân tích 50 case khó

Bộ dữ liệu hard evaluation được sử dụng để kiểm tra khả năng xử lý các trường hợp khó như:
- Spam giả dạng tin nhắn giao dịch hợp lệ.
- Ham chứa từ khóa giống quảng cáo hoặc khuyến mãi.
- Tin nhắn sử dụng ký tự thay thế để né bộ lọc.
- Prompt injection nhằm đánh lừa LLM.
- Tin nhắn bình thường thảo luận về spam.

Các mô hình có cách xử lý khác nhau đối với dữ liệu khó. BoW + Logistic Regression chủ yếu dựa vào đặc trưng từ vựng nên có thể nhầm những tin nhắn ham chứa nhiều từ khóa quảng cáo thành spam. Embedding và LLM có lợi thế trong việc xem xét ngữ cảnh, nhưng vẫn có thể bị đánh lừa bởi nội dung mơ hồ hoặc chỉ dẫn độc hại.

Hai trường hợp có nguy cơ gây nhầm lẫn:
- Tin nhắn thông báo ngân hàng hợp lệ nhưng chứa các từ như "account", "charged", "call" có thể bị nhận nhầm thành spam.
- Tin nhắn giả dạng thông báo giao hàng, yêu cầu thanh toán phí nhỏ qua liên kết giả có thể khiến mô hình phân loại thành ham.

=> Không thể chỉ dựa vào accuracy hoặc F1 trên tập dữ liệu thông thường để đánh giá toàn diện. Cần kiểm tra thêm từng nhóm hard cases để xác định điểm yếu của mô hình.

## 5. Thử thách cá nhân

Thử thách so sánh MLP nhỏ và MLP lớn bằng cách thay đổi số neuron trong hidden layer:
MLP nhỏ: hidden = 4.
MLP lớn: hidden = 512.
Cả hai mô hình sử dụng cùng tập dữ liệu train, validation và test để đảm bảo khả năng so sánh.

Các tiêu chí đánh giá gồm:
Số lượng tham số (parameters).
F1-score trên tập test.
Thời gian huấn luyện.

Giải thích:
- MLP nhỏ có ít tham số, tiết kiệm bộ nhớ và thời gian tính toán nhưng có thể không học đủ các đặc trưng phức tạp.
- MLP lớn có nhiều tham số hơn, khả năng học các quan hệ phức tạp tốt hơn nhưng có nguy cơ overfitting và tốn tài nguyên.
- Kết quả cuối cùng cần được đối chiếu từ cell thử thách sau khi chạy. Không thể kết luận MLP 512 tốt hơn chỉ dựa trên kích thước mạng.

## 6. Phương án cho sản phẩm thật

Mình chọn BoW + Logistic Regression làm mô hình chính vì tốc độ nhanh và chi phí thấp. Những tin nhắn có độ tin cậy thấp sẽ được chuyển sang Embedding hoặc LLM để phân tích thêm. Với 10 triệu SMS/ngày, hệ thống cần xử lý trung bình khoảng 116 tin/giây. LLM có p95 từ 2.9–3.8 giây nên không phù hợp với yêu cầu dưới 50 ms/tin. Phương án kết hợp giúp cân bằng giữa tốc độ, chi phí và độ chính xác.

## 7. AI log và điều đã học
|MLP|Sửa lỗi train_mlp()|Sửa cách truyền validation|Chạy lại code| 
|LLM|Sửa lỗi RateLimit 429|Đề xuất retry và cache |Chạy lại API |

Qua bài thực hành, mình học được cách xây dựng và đánh giá các mô hình Machine Learning và LLM. Mình hiểu sự khác biệt giữa zero-shot và few-shot, cách đánh giá bằng F1-score và tính toán chi phí API. Quan trọng nhất, mình biết cần cân bằng giữa độ chính xác, tốc độ và chi phí khi lựa chọn mô hình cho sản phẩm thực tế.

*Mỗi lần dùng AI: hỏi gì, nhận gì, bạn sửa gì. Sau đó 2–3 câu: điều bạn hiểu ra hôm nay mà trước đó chưa hiểu.*
