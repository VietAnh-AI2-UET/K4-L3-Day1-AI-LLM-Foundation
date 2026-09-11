# K4 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 4 tiếng
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay dòng `*Câu trả lời của bạn*` bằng câu
trả lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> *Temperature nhỏ hơn 1.0 làm câu từ càng ngắn gọn xúc tích, từ 1.0 trở đi, câu từ càng sáng tạo, bay bổng. temperature==0.0 sẽ gây hiện tượng lặp từ.*

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> *Để temperature=1.0, giữ nguyên phân phối xác suất, hỗ trợ khách hàng không lên để temperature cao gây lan man, cũng không nên để quá thấp gây lỗi sinh văn bản*

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> *3x350x10000 =  10500000 (token/ngày). 4o (10$/1M) đắt hơn 4o-mini (0.6$/1M) 16.67 lần (khá lớn). Chúng ta cần ưu tiên chất lượng của câu trả lời hơn số lượng. 4o có khả năng tư duy tốt hơn 4o-mini, từ đó các tác vụ cần tính suy luận cao thì nên dùng 4o, các tác vụ có tính lặp lại thì nên dùng 4o-mini*

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> *Khi đóng vai giáo viên tiểu học, model sử dụng từ vựng rất đơn giản, ví von blockchain như "cuốn sổ tay của nhóm bạn" hay "trò chơi xếp hình" với cấu trúc câu ngắn gọn, gần gũi. Ngược lại, khi đóng vai chuyên gia tài chính, model đưa ra định nghĩa học thuật, sử dụng hàng loạt thuật ngữ phức tạp (như sổ cái phân tán, mã hóa, cơ chế đồng thuận, phi tập trung). Điều này cho thấy system prompt định hình mạnh mẽ "bối cảnh" và "nhân cách" của model, chi phối trực tiếp đến độ dài, giọng văn, mức độ chuyên sâu của từ vựng và cả cách chọn ví dụ minh hoạ.*

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> *Đo đếm thực tế thường cao hơn ước lượng từ 35% đến 100%. Tiếng Việt tốn nhiều token hơn vì bộ mã hoá của OpenAI được tối ưu cho tiếng Anh (1 từ ≈ 1 token). Trong khi đó, do có dấu thanh điệu, một từ tiếng Việt thường bị băm nhỏ thành 2-3 token (sub-words).*

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> *Streaming tốt khi muốn tăng tương tác với người dùng, tránh họ phải chờ cảm giác lâu. Non-Streaming tốt hơn cho các tác vụ chạy ngầm mà người dùng không thấy được*

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> *Lợi thế của exponential backoff là giãn khoảng cách thời gian giữa các lần gửi request ngày càng xa hơn, giúp server có thêm "khoảng thở" và thời gian phục hồi khi đang bị quá tải. Nếu hàng nghìn client cùng retry với delay cố định giống nhau, nó sẽ gây ra hiện tượng "hiệu ứng bầy đàn" (thundering herd): toàn bộ nghìn request đó sẽ cùng ập tới server vào cùng một thời điểm ngay khi hết delay, tạo thành một đợt tấn công dồn dập khiến server vừa ngóc đầu lên lại tiếp tục quá tải và sập đổ.*

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> *System prompt: "Bạn là trợ giảng thân thiện của khóa AI, trả lời ngắn gọn bằng tiếng Việt."*
> *Giải thích: Từ "trợ giảng" định hướng model đóng vai trò người hướng dẫn thay vì tự động giải bài thay học viên. Từ "ngắn gọn" giúp tiết kiệm lượng token đầu ra (tối ưu chi phí) và hiển thị gọn gàng trên màn hình dòng lệnh. Cụm "bằng tiếng Việt" giúp ép model luôn giữ nhất quán ngôn ngữ, không tự động chuyển sang tiếng Anh khi người dùng hỏi các thuật ngữ kỹ thuật.*

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> *Hạn chế lớn nhất: Bộ nhớ ngắn hạn (chỉ nhớ 3 lượt cuối), nếu chat cuộc hội thoại dài bot sẽ quên mất bối cảnh ban đầu.*
> *Đề xuất cải thiện: Cơ chế "Tóm tắt cửa sổ trượt" (Rolling Summary).*
> *Cách triển khai: Khi history vượt quá giới hạn (ví dụ 3 lượt), thay vì xóa hẳn phần cũ nhất đi, ta gọi một luồng API phụ (bằng model giá rẻ như gpt-4o-mini) để tóm tắt những nội dung cũ đó thành vài câu ngắn gọn. Sau đó nhúng đoạn tóm tắt này vào cuối system_prompt (VD: "Bối cảnh trước đó: ..."). Vậy là bot có bộ nhớ dài hạn với chi phí token cực thấp.*

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/`, push lên fork và dán link trên trang bài Lab ở VLearn trước 23:59 ngày 11/09/2026
