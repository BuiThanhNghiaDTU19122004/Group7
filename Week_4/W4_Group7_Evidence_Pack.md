# W4 Evidence Pack — Group 7
---

## Section 1 — Cover

| Field | Value |
|-------|-------|
| **Group Number** | Group 7 |
| **Member Names** | Bùi Thành Nghĩa, Lê Thị Thùy Trang, Trần Minh Quang, Hoàng Kim Hùng, Nguyễn Công Thịnh, Phạm Công Huy, Nguyễn Tất Văn, Lê Nguyễn Nhật Thành, Đỗ Phúc|
| **LLM Used** | Claude Sonnet 4.6 via Amazon Bedrock |
| **Framework / Approach** | Raw Bedrock API + custom Python orchestrator |
| **Repository Link** | `<!-- https://github.com/hunghk43/GeekBrain_AI -->` |

---

## Section 2 — Architecture Overview

### 2.1 System Architecture Diagram

> **Hướng dẫn:** Dán sơ đồ Mermaid, ảnh chụp whiteboard, hoặc export từ Draw.io/Lucidchart vào đây.  
> Sơ đồ phải thể hiện rõ: S3 → Bedrock KB → LLM, Database/Monitoring API → Tool Functions → LLM, và luồng Memory (nếu có L4).

![Architecture Diagram](./L1-L2%20png/archetec.jpg)


---

### 2.2 Component List

> **Hướng dẫn:** Liệt kê từng thành phần trong hệ thống và vai trò của nó.

| Component | AWS Service / Tool | Role |
|-----------|-------------------|------|
| Document Store | Amazon S3 | Lưu 36 file markdown làm nguồn dữ liệu cho RAG |
| Vector Search & RAG | Amazon Bedrock Knowledge Bases + OpenSearch Serverless | Chunking, embedding, similarity search |
| LLM / Reasoning | Amazon Bedrock (Claude Sonnet) | VD: Sinh câu trả lời từ context và tool results |
| Structured DB | SQLite | Lưu costs, incidents, SLA targets, daily metrics |
| Live Monitoring | Monitoring API (local Python) | Cung cấp current status, latency, error rate |
| Tool Orchestration | Raw API | Điều phối khi nào gọi tool, khi nào retrieve |
| Memory / State | In-memory dict | Lưu conversation history |

---

### 2.3 Data Flow — End-to-End

> **Hướng dẫn:** Mô tả hành trình của một câu hỏi qua hệ thống, từ input đến output.

```
[User Question]
      │
      ▼
[Orchestrator / App]
      │
      ├──── L1/L2: Retrieve from Bedrock KB (vector search, top-K chunks)
      │              └──► Inject chunks into prompt → Call LLM → Return answer + source
      │
      ├──── L3: Detect tool-required question
      │         ├── Tool: query_database(sql)  →  SQLite/RDS  →  historical numbers
      │         └── Tool: get_service_metrics(service)  →  Monitoring API  →  live data
      │              └──► Inject tool results into prompt → Call LLM → Return answer
      │
      └──── L4: Prepend conversation history (last N turns) → Rewrite query if needed
                 └──► Same pipeline as above, but LLM can resolve pronouns
```

---

### 2.4 System Running — Startup Screenshot
 Ảnh terminal hoặc browser cho thấy hệ thống đang chạy.

 ![System Running](./L1-L2%20png/tmal%20Log.png)
---

## Section 3 — Decision Log

> **Hướng dẫn:** Ghi lại **3 quyết định kỹ thuật quan trọng** trong tuần.  
> Ít nhất **1 thứ đã không hoạt động** và cách nhóm xử lý. Đây là phần trainers đánh giá tư duy kỹ thuật của nhóm.

---

### Decision 1: Raw Bedrock Converse API + custom tool loop

**What we chose:**
> Dùng `bedrock-runtime.converse` với tool definitions và vòng lặp `tool_use` trong `app/agent.py`.

**Why:**
> Cần kiểm soát routing và dễ debug (log rõ ràng), đúng yêu cầu raw API.

**What we learned:**
> Phải viết thêm glue code, nhưng hành vi tool call rõ ràng và dễ trace.

---

### Decision 2: Tăng retrieval K + xử lý conflict bằng prompt

**What we chose:**
> `RETRIEVAL_K=10` và system prompt yêu cầu ưu tiên document mới hơn (không archived) khi conflict.

**Why:**
> L2 cần lấy đủ chunk từ nhiều tài liệu và resolve xung đột v1/v2.

**What we learned:**
> K lớn giúp coverage tốt hơn, nhưng vẫn cần rule trong prompt để tránh chọn sai doc.

---

### Decision 3: Memory bằng query rewriting + window

**What we chose:**
> Lưu history per-session trong memory; rewrite câu hỏi bằng LLM từ 4 turn gần nhất; giữ window `MEMORY_WINDOW=10`.

**Why:**
> Resolve pronoun mà không phải đẩy toàn bộ lịch sử vào context.

**What we learned:**
> Query rewriting nhẹ, hiệu quả cho L4 demo; nhược điểm là mất state khi restart.

---
## Section 4 — Per-Level Evidence

### L1 — Retrieval (Simple RAG)

**Test question used:**
```
<!-- VD: "Who is the Team Platform lead?" -->
```

**Expected answer:** `<!-- VD: Alex Chen, source: team_platform.md -->`

**Actual answer from system:**
> `<!-- Paste text output của hệ thống tại đây -->`

---

#### L1 — Screenshot: Correct Answer with Source Citation

<!-- 📸 SCREENSHOT [REQUIRED]:
     Chụp màn hình output của hệ thống, cho thấy:
     - Câu trả lời đúng (VD: "Alex Chen")
     - Tên document được cite (VD: "Source: team_platform.md")
     
     ![L1 Correct Answer](./screenshots/L1_correct_answer.png)
-->

> ![L1 Correct Answer](./L1-L2%20png/L1%20who%20is%20Team%20Lead.png) — Output của hệ thống + source citation

---

#### L1 — Proof: Retrieval Actually Happened

<!-- 📸 SCREENSHOT / LOG [REQUIRED]:
     Chụp terminal log hoặc CloudWatch log cho thấy:
     - Bedrock Retrieve API được gọi
     - Chunks được trả về (VD: chunk text chứa "Alex Chen" từ team_platform.md)
     Nếu có Observability Dashboard (Bonus A): chụp dashboard thay cho log.
     
     ![L1 Retrieval Proof](./screenshots/L1_retrieval_log.png)
-->

> ![L1 Retrieval Proof](./L1-L2%20png/L1%20Log.png) — Terminal log chứng minh retrieval pipeline hoạt động

**Notes:**
> `<!-- VD: "Bedrock KB sync thành công với 36 docs. Retrieval trả về top-5 chunks, chunk từ team_platform.md có relevance score cao nhất." -->`

---

### L2 — Multi-Source Retrieval (Advanced RAG)
#### Conflict Resolution

**Test question used:**
```
<!-- VD: "What is GeekBrain's API rate limit for PaymentGW?" -->
```

**Expected answer:** `1000 req/min (from current v2 doc, NOT archived v1 doc with 500 req/min)`

**Actual answer from system:**
Output cho thấy hệ thống resolve conflict đúng: chọn 1000 (v2) thay vì 500 (v1 archived).

>  ![L2 Conflict Resolution](./L1-L2%20png/L2respone.png)

**How our system handles conflicting documents:**
> `<!-- Mô tả kỹ thuật: VD: "Tăng K=10 để retrieve cả 2 docs. System prompt yêu cầu LLM ưu tiên document có date mới hơn và không có tag 'archived'." -->`

---

### L3 — Retrieval + Tools (Tool-Augmented RAG)

####  Database Query Tool

**Test question used:**
```
<!-- VD: "What was PaymentGW's total infrastructure cost in Q1 2026?" -->
```

**Expected answer:** `$16,500 (from monthly_costs table: Jan + Feb + Mar 2026)`

**Actual answer from system:**
> `<!-- Paste text output — phải chính xác là $16,500 -->`

<!-- 📸 SCREENSHOT [REQUIRED]:
     Chụp output hiển thị câu trả lời đúng ($16,500).
     
     ![L3 DB Answer](./screenshots/L3_database_answer.png)
-->

> ![L3 Metrics Answer](./L1-L2%20png/l3%20answer.png)
---
LOG/PROOF [REQUIRED — CRITICAL]:
     Đây là proof quan trọng nhất. Chụp terminal log cho thấy:
     - LLM ra quyết định gọi tool "query_database"
     - SQL query được tạo ra (VD: SELECT SUM(total_cost) FROM monthly_costs WHERE ...)
     - Kết quả thực từ DB (VD: [{"sum": 16500}])
     - LLM nhận kết quả và sinh ra final answer   

> ![L3 DB Tool Call Proof](./L1-L2%20png/l3log2.png)

---


### L4 — Memory (Multi-Turn Conversation) `[1.0 điểm]`

---

**Memory strategy used:**
> `<!-- In memory session store -->`
> `<!-- Mô tả implementation cụ thể: VD: "Lưu last 10 messages trong in-memory list. Append vào mỗi LLM call. -->`

**Test conversation used:**
```
Turn 1: "Which service had the highest cost in March 2026?"
Turn 2: "Why did its costs spike?"       ← "its" phải resolve = PaymentGW
Turn 3: "Which team is responsible?"     ← vẫn về PaymentGW
Turn 4: "The postmortem mentioned a review deadline. Is it overdue?"
```
     
![L4 Conversation 1](./L1-L2%20png/l4%20response-1.png)

![L4 Conversation 2](./L1-L2%20png/l4%20response%20-2.png)

![L4 Conversation 3](./L1-L2%20png/l4%20response%20-3.png)

![L4 Conversation 4](./L1-L2%20png/l4%20response%20-4.png)

**Notes:**
> `<!-- VD: "Sử dụng window memory với last-10-turns. Turn 2 'its' được resolve sang PaymentGW nhờ Turn 1 vẫn còn trong context. Production limitation: in-memory state sẽ mất khi restart." -->`

---

### Bonus A — Observability Dashboard `[+0.5 điểm]`


> ![L5 Conversation 1](./L1-L2%20png/observation_board.jpg) — Observability dashboard đang process một câu hỏi

---

### Bonus B — Agent Reasoning (Structured Investigation) `[+0.5 điểm]`

**Status:** `<!-- Attempted ✅ / Not attempted ❌ -->`

**Test question used:**
```
<!-- VD: "GeekBrain wants to cut infrastructure costs by 15% in Q2. Analyze current spending and recommend which services to optimize first." -->
```


> ![L5 Conversation 1](./L1-L2%20png/trace.jpg)
> — Structured investigation output với visible reasoning

## Section 5 — Reflection

### Hardest Level and Why

> `<!-- Thành thật về level nào khó nhất và tại sao.
>      VD: "L3 khó nhất vì tool routing không ổn định — LLM thỉnh thoảng gọi sai tool hoặc không gọi tool mà hallucinate số liệu.
>           Lý do: tool description trong prompt tụi em chưa mô tả chi tiết rõ ràng về khi nào dùng tool để query DB vs khi nào dùng API monitoring." -->`

---

## Scoring Summary (Self-Assessment)

> **Nhóm tự đánh giá** trước khi present — trainers sẽ verify lại.

| Level | Max Points | Self-Score (1-5) | Trainers Points | Status |
|-------|-----------|-----------------|-----------------|--------|
| L1 — Retrieval | 2.0 | ` 5/5` | ` -/2.0` | `<!-- ✅ Working -->` |
| L2 — Multi-Source | 3.0 | ` 5/5` | ` -/3.0` | `<!-- ✅ Working -->` |
| L3 — Tools | 4.0 | ` 4/5` | ` -/4.0` | `<!-- ✅ Working -->` |
| L4 — Memory | 1.0 | ` 5/5` | ` -/1.0` | `<!-- ✅ Working -->` |
| **Base Total** | **10.0** | | ` -/10.0` | |
| Bonus A — Dashboard | +0.5 | 0.5 | ` -/0.5` | `<!-- ✅ -->` |
| Bonus B — Agent Reasoning | +0.5 | 0.5 | ` -/0.5` | `<!-- ✅ -->` |
| **Grand Total** | **11.0 max** | 10.0/10.0| ` -/10.0` | |
