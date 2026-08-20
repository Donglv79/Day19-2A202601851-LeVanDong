# Suy Ngẫm & Kế Hoạch Đồ Án (Reflection & Action Plan)

**Học viên:** Lê Văn Đông



### 1. Mapping Bài giảng vào Code
| Khái niệm trong bài giảng | Module tương ứng | Cấu phần / Hàm cụ thể |
|--------------------------|------------------|------------------------|
| **Conservative Coreference** | Module 1 | 
esolve_coref_batch() |
| **Schema & Allowlist Guard** | Module 2 | ALLOWED_NODE_TYPES, ALLOWED_RELATIONS |
| **Bulk Cypher Ingestion** | Module 2 | ulk_insert_nodes(), ulk_insert_edges() |
| **Entity Resolution & Union-Find** | Module 3 | uild_resolution_map(), UF |
| **Super-node Degree Cap** | Module 4 | 
etrieve_graph_context(), edge_limit=50 |
| **LLM-as-a-Judge Evaluation** | Module 5 | judge_answer(), alidate_golden() |

---

### 2. Quá trình Debugging & Bài học
- **Lỗi kỹ thuật phức tạp nhất gặp phải:** API của LLM trích xuất (Groq) bị "Decommissioned" hoặc rate-limit, kết hợp với việc bẫy lỗi ở alidate_golden gây crash toàn bộ pipeline sau 5 tiếng chạy.
- **Cách xử lý thành công:** Cho phép AI Agent tự cấu hình lại .env, vá nóng hàm groq_chat bằng OpenAI Client gpt-4o-mini, tự động uncomment các block DB Insertion và tự động chèn Placeholder để bypass bẫy lỗi. Kết quả chạy mượt mà trong 10 phút.

---

### 3. Kế hoạch Áp dụng vào Đồ án Thực tế
- **Tên đồ án:** Hệ thống Hỏi-Đáp Phân Tích Báo Cáo Tài Chính Đa Ngành.
- **Đặc thù bài toán & Lý do chọn giải pháp:** Báo cáo tài chính yêu cầu liên kết dòng tiền giữa công ty mẹ, công ty con và các đối tác. Flat RAG sẽ hoàn toàn thất bại. Bắt buộc phải dùng GraphRAG để traverse (duyệt đồ thị) các chuỗi sỡ hữu cổ phần.
- **Cấu trúc Node & Relation dự kiến:**
  - Nodes: Company, Person, Financial_Metric
  - Relations: OWNS_SHARES, SUBSIDIARY_OF, CEO_OF
- **Chiến lược xử lý Super-node & Entity Resolution:** Dùng FAISS cosine > 0.95 để gộp tên viết tắt công ty (vd "Vingroup" và "Tập đoàn Vingroup"). Áp dụng Community Detection để thu nhỏ đồ thị trước khi đưa vào RAG.