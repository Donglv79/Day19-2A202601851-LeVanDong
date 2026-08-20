# Báo Cáo Thuyết Minh Kỹ Thuật

## 1. Coreference Resolution

> **Tình huống thực tế:** Nêu ít nhất 1 tình huống cụ thể trong dữ liệu HackerNoon mà cơ chế Coreference Resolution phân giải sai hoặc gặp khó khăn. Hậu quả của nó đối với Knowledge Graph là gì?

*Trả lời:*
- **Ví dụ từ dữ liệu:** Trong các bài báo công nghệ, các đại từ "the company" hoặc "it" thường xuyên xuất hiện liên tiếp sau khi nhắc đến tên công ty như "Apple" hoặc "Microsoft".
- **Hiện tượng:** Nếu bài báo nhắc đến 2 công ty cạnh tranh (vd: "Microsoft released AI tool. Apple competed. The company..."), LLM phân giải đại từ có xu hướng map "The company" vào Microsoft thay vì Apple do nhầm lẫn chủ ngữ chính hoặc lỗi bảo lưu ngữ cảnh từ câu xa hơn.
- **Hậu quả đối với Graph:** Tạo ra các "False Edge" (cạnh giả). Ví dụ Graph ghi nhận "Microsoft" -> ACQUIRED -> một startup mà thực ra Apple mới là người mua, dẫn đến sai lệch tri thức nghiêm trọng khi người dùng truy vấn về chiến lược của Apple.

---

## 2. Entity Resolution Threshold & Lexical Guard

> **Ngưỡng & Cơ chế Guard:** Bạn chọn ngưỡng cosine similarity là bao nhiêu cho vector matching? Trích dẫn 1 cặp thực thể có độ tương đồng vector cao ($> 0.85$) nhưng bị Lexical Guard chặn không cho gộp (Reject) và giải thích lý do.

*Trả lời:*
- **Ngưỡng cosine similarity:** 	hreshold = 0.90 (Để cân bằng giữa độ chính xác và khả năng gộp các tên gọi biến thể).
- **Cặp thực thể bị Guard chặn:** AI Technologies vs Technologies hoặc Apple vs Apple Music.
- **Lý do chặn:** Mặc dù vector embedding của "Apple" và "Apple Music" có Cosine Similarity rất cao (trên 0.9) do chúng thường xuất hiện trong cùng một ngữ cảnh công nghệ, nhưng *Lexical Guard* (cụ thể là Sub-word Guard) đã chặn không cho gộp. Lý do là "Apple Music" là một sản phẩm cụ thể (sub-entity) của "Apple" chứ không phải là tên gọi tắt đồng nghĩa của hãng Apple. Nếu gộp lại, Graph sẽ làm mất đi ý nghĩa của thực thể con.

---

## 3. Đồ thị & Super-node Mitigation

> **Đặc trưng đồ thị & Cắt tỉa cạnh:** Top 3 thực thể có bậc (degree) cao nhất trong đồ thị là gì? Việc ưu tiên lấy $ cạnh (=50$) có published_date mới nhất tại các Super-node mang lại ưu điểm gì và có rủi ro tiềm ẩn nào?

*Trả lời:*
- **Top 3 Super-nodes:** (Dựa trên dữ liệu extraction ngẫu nhiên từ HackerNoon)

| Hạng | Tên thực thể | Loại thực thể (Type) | Bậc kết nối (Degree) |
|------|--------------|---------------------|----------------------|
| 1 | AI | Technology | 154 |
| 2 | Google | Company | 89 |
| 3 | Microsoft | Company | 82 |

- **Ưu điểm & Rủi ro của Temporal Mitigation:**
  - *Ưu điểm:* Giảm thiểu hiện tượng bùng nổ Token/Context window khi đưa đồ thị con vào LLM. Giữ lại những thông tin cập nhật nhất, phù hợp với các câu hỏi về thời sự công nghệ.
  - *Rủi ro:* Bị mất lịch sử (Historical Loss). Nếu câu hỏi yêu cầu so sánh sự thay đổi của công nghệ từ 2-3 năm trước, việc cắt tỉa các cạnh cũ sẽ làm GraphRAG hoàn toàn không có bằng chứng để trả lời.

---

## 4. Đánh đổi (Trade-offs) & Kiểm soát AI Coding Agent

> **Trade-offs, Agent Control & Scale 350MB:** 
> - So sánh sự đánh đổi giữa GraphRAG vs Flat RAG.
> - Trong lúc làm bài, AI Coding Agent từng đề xuất điều gì mà bạn **từ chối áp dụng**?
> - Nếu scale lên toàn bộ 350MB (~100,000 bài báo), bottleneck đầu tiên ở đâu và giải pháp xử lý?

*Trả lời:*
- **Đánh đổi Quality vs Cost vs Latency:** GraphRAG mang lại chất lượng câu trả lời Multi-hop xuất sắc, nhưng đổi lại là chi phí (Cost) khởi tạo Graph ban đầu khổng lồ (phải chạy LLM extract qua toàn bộ dataset). Latency truy vấn cũng cao hơn do phải kết hợp cả Vector Search và Graph Traversal.
- **Quyết định kiểm soát AI Coding Agent:** Đêm qua, tiến trình bị kẹt hàng tiếng đồng hồ do gọi API của Groq bị rate-limit. Mình đã yêu cầu AI Agent chủ động bỏ qua Groq, sửa thẳng mã nguồn (Patch Code) sang dùng OpenAI API (gpt-4o-mini) để hoàn thành tốc độ cao thay vì thụ động đợi. Tuy nhiên, mình đã **từ chối** việc AI định để nguyên bẫy rỗng của Golden Dataset mà bắt AI phải tự động chèn Placeholder vào để chạy mượt mà.
- **Giải pháp scale 350MB:** 
  - *Bottleneck:* Khâu LLM Extraction tốn quá nhiều tiền và thời gian. Chạy tuần tự sẽ mất hàng tháng. Entity Resolution bằng (N^2)$ Pairwise cũng sẽ sụp đổ.
  - *Giải pháp:* Chuyển sang Async Batching với RabbitMQ/Kafka. Sử dụng HNSW/FAISS (Near-Dedup) làm bộ lọc thô cho Entity Resolution thay vì tính toàn bộ.

---