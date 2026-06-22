# Reflection: Lecture → Project

### Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Dùng threshold 0.85 giúp tách các đoạn dựa trên độ tương đồng ngữ nghĩa, hạn chế việc bị cắt ngang giữa ý như tách theo kí tự hoặc paragraph thuần túy. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF giải quyết vấn đề chênh lệch thang điểm giữa BM25 (keyword matching) và Dense Search (semantic), giúp dung hòa và tận dụng tối đa ưu điểm của 2 phương pháp. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Reranking bằng Cross-encoder (như BAAI/bge-reranker-v2-m3) cải thiện độ chính xác thứ hạng đáng kể so với bi-encoder, nhờ đánh giá trực tiếp mối liên hệ query-document. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Nhờ chia nhỏ thành các metrics (faithfulness, answer relevancy, context precision, context recall), ta dễ dàng xác định pipeline lỗi ở retrieval hay ở generation. |
| Contextual embeddings | M5 | `contextual_prepend()` | Việc prepend thêm 1-2 câu giải thích bối cảnh giúp giảm đáng kể retrieval failure do các chunk đứng một mình thường bị mất ngữ cảnh (loss of context). |

### Phần 2: Khó khăn & giải quyết

- **Lỗi gặp phải:** Tokenizer `underthesea` dùng dấu gạch dưới `_` để nối các từ ghép (VD: `nghỉ_phép`), trong khi query gốc từ người dùng có thể là khoảng trắng `nghỉ phép`. Điều này làm BM25 không khớp được từ khóa.
- **Cách debug:** In thử kết quả tokenize và phân tích logic BM25 search.
- **Cách khắc phục:** Cần `.replace("_", " ")` sau khi chạy `word_tokenize` để tách các từ ghép ra, giúp `BM25Okapi` có thể split theo khoảng trắng và matching chính xác.

### Phần 3: Action Plan cho project

## Project: Hệ thống Tra cứu Nội bộ (Internal Knowledge Base RAG)

### Hiện tại
- RAG pipeline hiện tại: Naive RAG, chunking cố định 500 token, chỉ dùng Dense Search.
- Known issues: Thường xuyên lấy thiếu ngữ cảnh (Context Recall thấp), và lấy nhầm các đoạn có keyword tương tự nhưng khác context (VD: chính sách lương năm 2023 vs 2024).

### Plan áp dụng
1. [x] **Chunking strategy**: Dùng Hierarchical Chunking (Parent-Child) để vừa đảm bảo độ chính xác lúc search (child) vừa đủ ngữ cảnh lúc generate (parent).
2. [x] **Search**: Chuyển sang Hybrid Search (Qdrant + BM25) để giải quyết các truy vấn cần tra cứu chính xác mã nhân viên, mã dự án.
3. [x] **Reranking**: Bổ sung Cross-encoder model cho bước xếp hạng top-3 document để LLM không bị nhiễu thông tin.
4. [x] **Evaluation**: Tích hợp RAGAS vào CI/CD pipeline để tự động đo lường thay vì test tay bằng mắt.
5. [x] **Enrichment**: Thêm Contextual Prepend bằng GPT-4o-mini để mỗi chunk luôn chứa thông tin title/bối cảnh.

### Timeline
- Tuần 1: Thiết lập lại Qdrant collection, sửa lại hàm chunking sang dạng Hierarchical.
- Tuần 2: Xây dựng module Hybrid Search và Reranking.
- Tuần 3: Viết script chạy benchmark tự động bằng RAGAS.
- Tuần 4: Tối ưu Enrichment (nếu cần cải thiện thêm Context Recall).
