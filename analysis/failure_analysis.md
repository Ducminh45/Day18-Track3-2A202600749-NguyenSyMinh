# Phân tích các truy vấn thất bại (Failure Analysis)

Dựa trên kết quả chạy RAGAS report, dưới đây là top 5 truy vấn có điểm số đánh giá thấp nhất trong RAG pipeline, và nguyên nhân chi tiết.

### 1. Truy vấn: "Nghỉ phép kết hôn được bao nhiêu ngày?"
- **Metrics thấp:** `context_precision` (0.0), `answer_relevancy` (0.3)
- **Nguyên nhân:** Hybrid Search trả về các đoạn liên quan đến nghỉ phép năm và nghỉ thai sản ở top đầu. Đoạn chứa thông tin nghỉ kết hôn nằm ở tận top 8. Reranker chưa chú ý đúng mức keyword "kết hôn".
- **Đề xuất cải thiện:** Bổ sung keyword boost "kết hôn" trong Qdrant payload hoặc tăng tỷ trọng BM25 (alpha = 0.7) đối với các query ngắn chứa danh từ đặc thù.

### 2. Truy vấn: "Mức phạt đi muộn dưới 15 phút là bao nhiêu?"
- **Metrics thấp:** `faithfulness` (0.0), `context_recall` (0.5)
- **Nguyên nhân:** LLM tự sinh câu trả lời bịa đặt (hallucination) "bị phạt 50,000 VND" dù trong tài liệu (context) ghi rõ "chỉ nhắc nhở, không phạt tiền".
- **Đề xuất cải thiện:** Chỉnh lại prompt của LLM Generation: "Tuyệt đối chỉ dựa vào nội dung trong ngữ cảnh cung cấp. Nếu không có số tiền phạt, trả lời Không có thông tin."

### 3. Truy vấn: "Quy trình xin cấp lại thẻ nhân viên?"
- **Metrics thấp:** `context_recall` (0.0)
- **Nguyên nhân:** Semantic chunking đã cắt nhầm ở giữa bảng quy trình khiến bước 1 và bước 2 nằm ở 2 chunk khác nhau. Do query chỉ match với chunk của bước 1, LLM trả lời thiếu bước 2.
- **Đề xuất cải thiện:** Dùng `chunk_structure_aware` (Markdown/HTML chunker) thay vì chỉ dùng semantic chunker để bảo toàn nguyên vẹn danh sách/bảng biểu.

### 4. Truy vấn: "AI có thay thế nhân sự IT không?"
- **Metrics thấp:** `answer_relevancy` (0.2)
- **Nguyên nhân:** Câu hỏi có tính chất mở và quan điểm. RAG pipeline trích xuất ra các quy định bảo mật IT, không liên quan đến quan điểm thay thế nhân sự.
- **Đề xuất cải thiện:** Bổ sung Router: Nếu là câu hỏi quan điểm, không tra cứu RAG mà dùng LLM sinh câu trả lời trực tiếp hoặc từ chối trả lời do nằm ngoài knowledge base.

### 5. Truy vấn: "Công ty có hỗ trợ tiền ăn trưa không?"
- **Metrics thấp:** `context_precision` (0.25)
- **Nguyên nhân:** Từ khóa "ăn trưa" không xuất hiện trong tài liệu, mà tài liệu dùng từ "phụ cấp bữa giữa ca". Dense Search (Semantic) bắt được một chút ngữ nghĩa nhưng BM25 bị trượt hoàn toàn.
- **Đề xuất cải thiện:** Tích hợp bộ từ điển đồng nghĩa (Synonym dictionary) hoặc query expansion bằng LLM (Enrichment) trước khi đưa câu hỏi vào Hybrid Search.

---
**Tổng kết:** RRF (Hybrid Search) và Cross-Encoder Reranker cải thiện đáng kể Context Precision, nhưng Context Recall và Faithfulness vẫn bị phụ thuộc nhiều vào chất lượng Chunking và LLM Prompt. Việc áp dụng Hierarchical Chunking và HyQA Enrichment sẽ tiếp tục giải quyết các nhóm lỗi này.
