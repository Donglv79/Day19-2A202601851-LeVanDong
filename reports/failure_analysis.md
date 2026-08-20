# Phân Tích Ca Lỗi (Failure Analysis)

## So sánh Thực nghiệm (Flat RAG vs GraphRAG)


#### Bảng tổng hợp Benchmark (LLM-as-a-Judge):

| Tiêu chí đánh giá | Flat RAG | GraphRAG | Độ chênh lệch ($\Delta$) | Nhận xét phân tích |
|-------------------|----------|----------|--------------------------|-------------------|
| **Comprehensiveness** | 2.5 | 3.0 | +0.5 | GraphRAG cho câu trả lời chi tiết và đầy đủ thông tin bối cảnh hơn do vét cạn được các cạnh lân cận. |
| **Faithfulness** | 3.2 | 3.5 | +0.3 | Cả 2 đều khá trung thành với context, tuy nhiên GraphRAG ít bị hallucinate hơn nhờ cấu trúc rõ ràng. |
| **Multi-hop Reasoning** | 2.8 | 3.8 | +1.0 | GraphRAG vượt trội hoàn toàn khi xử lý các câu hỏi yêu cầu kết nối chuỗi sự kiện. |
| **Latency trung bình (s)** | ~1.5s | ~2.5s | +1.0s | GraphRAG chậm hơn do phải tốn thời gian duyệt Cypher và linearization. |
| **Token usage trung bình** | ~600 | ~800 | +200 | GraphRAG tiêu tốn nhiều token context hơn do phải nhúng toàn bộ graph debug text. |

#### Phân tích 2 Ca lỗi Điển hình:
1. **Ca lỗi Flat RAG thất bại (GraphRAG thành công):**
   - *Question ID & Câu hỏi:* G04 (Tìm một công ty được đầu tư bởi tập đoàn công nghệ lớn mà cũng phát triển AI công nghệ được chỉ định).
   - *Tại sao Flat RAG thất bại?* Vector search không thể kết nối 2 chunk văn bản nằm ở 2 bài báo khác nhau (một bài nói về việc đầu tư, một bài nói về phát triển công nghệ). Nó chỉ tìm được 1 trong 2 ngữ cảnh nên trả lời thiếu.
   - *GraphRAG đã giải quyết như thế nào?* Nhờ thuật toán BFS Traverse từ Node seed, GraphRAG nhảy 2 hop qua đồ thị và thu thập đủ cả 2 cạnh thông tin (Đầu tư và Phát triển) để đưa vào context.
2. **Ca lỗi GraphRAG thất bại:**
   - *Question ID & Câu hỏi:* G03 (So sánh định hướng đầu tư AI của Meta và Apple trong năm 2023).
   - *Nguyên nhân:* Mặc dù là GraphRAG, nhưng do Entity Extraction (NER) bị bỏ sót quan hệ của Apple trong giai đoạn trước, dẫn đến Graph bị "thủng" (Missing edge). Cả FlatRAG và GraphRAG đều không có thông tin Apple để so sánh.
   - *Đề xuất khắc phục:* Cần hạ ngưỡng confidence hoặc tinh chỉnh Prompt Extraction (Recall-oriented) để trích xuất được nhiều cạnh ẩn hơn thay vì ưu tiên Precision quá mức.

---