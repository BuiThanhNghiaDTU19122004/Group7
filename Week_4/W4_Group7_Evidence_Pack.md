# W4 Evidence Pack — Build an AI That Actually Answers
> **Lưu ý cho nhóm:** Tất cả các ô `<!-- 📸 SCREENSHOT: ... -->` là nơi bạn chèn ảnh vào.  
> Cú pháp chèn ảnh trong Markdown: `![Mô tả ảnh](./screenshots/ten_anh.png)`  
> Hãy tạo thư mục `docs/screenshots/` để lưu toàn bộ ảnh trước khi commit.

---

## Section 1 — Cover

| Field | Value |
|-------|-------|
| **Group Number** | Group 7 |
| **Member Names** | Bùi Thành Nghĩa
Lê Thị Thùy Trang
Trần Minh Quang
Hoàng Kim Hùng
Nguyễn Công Thịnh
Phạm Công Huy
Nguyễn Tất Văn
Lê Nguyễn Nhật Thành
Đỗ Phúc|
| **LLM Used** | Claude Sonnet 4.6 via Amazon Bedrock |
| **Framework / Approach** | Raw Bedrock API + custom Python orchestrator |
| **Repository Link** | `<!-- URL GitHub/GitLab repo của nhóm -->` |

---

## Section 2 — Architecture Overview

### 2.1 System Architecture Diagram

> **Hướng dẫn:** Dán sơ đồ Mermaid, ảnh chụp whiteboard, hoặc export từ Draw.io/Lucidchart vào đây.  
> Sơ đồ phải thể hiện rõ: S3 → Bedrock KB → LLM, Database/Monitoring API → Tool Functions → LLM, và luồng Memory (nếu có L4).

<!-- 📸 SCREENSHOT / DIAGRAM [REQUIRED]:
     Chèn architecture diagram của nhóm tại đây.
     - Nếu dùng Mermaid: dán code vào block ```mermaid ... ``` bên dưới
     - Nếu dùng ảnh: ![Architecture Diagram](./screenshots/architecture_diagram.png)
-->


---

### 2.2 Component List

> **Hướng dẫn:** Liệt kê từng thành phần trong hệ thống và vai trò của nó.

| Component | AWS Service / Tool | Role |
|-----------|-------------------|------|
| Document Store | Amazon S3 | Lưu 36 file markdown làm nguồn dữ liệu cho RAG |
| Vector Search & RAG | Amazon Bedrock Knowledge Bases + OpenSearch Serverless | Chunking, embedding, similarity search |
| LLM / Reasoning | Amazon Bedrock (Claude Sonnet) |  VD: Sinh câu trả lời từ context và tool results |
| Structured DB | SQLite |  Lưu costs, incidents, SLA targets, daily metrics  |
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

<!-- 📸 SCREENSHOT [REQUIRED]:
     Chèn ảnh terminal hoặc browser cho thấy hệ thống đang chạy.
     VD: terminal với `uvicorn app:app --port 8080` đang run, hoặc URL cloud-deployed.
     
     ![System Running](./screenshots/system_startup.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Terminal hoặc URL cho thấy app đang chạy.

---

## Section 3 — Decision Log

> **Hướng dẫn:** Ghi lại **3 quyết định kỹ thuật quan trọng** trong tuần.  
> Ít nhất **1 thứ đã không hoạt động** và cách nhóm xử lý. Đây là phần trainers đánh giá tư duy kỹ thuật của nhóm.

---

### Decision 1: `<!-- Tiêu đề quyết định, VD: Chọn Bedrock KB thay vì custom ChromaDB -->`

**What we chose:**
> `<!-- Mô tả quyết định cụ thể —  dùng service/approach nào, config ra sao -->`

**Why:**
> `<!-- Lý do kỹ thuật: tốc độ setup, trade-off giữa control vs convenience, hạn chế về thời gian... -->`

**What we learned:**
> `<!-- Điều gì hoạt động tốt? Điều gì bị hạn chế? Nếu làm lại sẽ thay đổi gì? -->`

---

### Decision 2: `<!-- VD: Tăng retrieval K từ 3 lên 10 để giải quyết conflict -->`

**What we chose:**
> `<!-- ... -->`

**Why:**
> `<!-- ... -->`

**What we learned:**
> `<!-- ... -->`

---

### Decision 3: `<!-- VD: Dùng raw API tool calling thay vì Bedrock Agents -->`

**What we chose:**
> `<!-- ... -->`

**Why:**
> `<!-- ... -->`

**What we learned:**
> `<!-- ... -->`

---

### ❌ What Did NOT Work (Required)

**We tried:** `<!-- Mô tả approach/service/config đã thử -->`

**It failed because:** `<!-- Nguyên nhân cụ thể: timeout, empty results, wrong routing, auth error... -->`

**So we switched to:** `<!-- Giải pháp thay thế và kết quả -->`

---

## Section 4 — Per-Level Evidence

> **Quy tắc:** Mỗi level cần **2 thứ**:
> 1. **Screenshot output đúng** — câu trả lời của hệ thống
> 2. **Proof** — log/trace cho thấy hệ thống thực sự xử lý (không phải hardcode hoặc hallucinate)

---

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

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Output của hệ thống + source citation

---

#### L1 — Proof: Retrieval Actually Happened

<!-- 📸 SCREENSHOT / LOG [REQUIRED]:
     Chụp terminal log hoặc CloudWatch log cho thấy:
     - Bedrock Retrieve API được gọi
     - Chunks được trả về (VD: chunk text chứa "Alex Chen" từ team_platform.md)
     Nếu có Observability Dashboard (Bonus A): chụp dashboard thay cho log.
     
     ![L1 Retrieval Proof](./screenshots/L1_retrieval_log.png)
-->

> ⬆️ **[CHÈN ẢNH / LOG TẠI ĐÂY]** — Terminal log chứng minh retrieval pipeline hoạt động

**Notes (1-2 dòng):**
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
> `<!-- Paste text output — phải là 1000, không phải 500 -->`

<!-- 📸 SCREENSHOT [REQUIRED]:
     Chụp output cho thấy hệ thống resolve conflict đúng: chọn 1000 (v2) thay vì 500 (v1 archived).
     Lý tưởng nhất là output giải thích lý do chọn document nào.
     
     ![L2 Conflict Resolution](./screenshots/L2_conflict_resolution.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Output + conflict resolution explanation

**How our system handles conflicting documents:**
> `<!-- Mô tả kỹ thuật: VD: "Tăng K=10 để retrieve cả 2 docs. System prompt yêu cầu LLM ưu tiên document có date mới hơn và không có tag 'archived'." -->`

---

### L3 — Retrieval + Tools (Tool-Augmented RAG)

#### L3a — Database Query Tool

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

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Output với con số chính xác từ DB

---

<!-- 📸 LOG/PROOF [REQUIRED — CRITICAL]:
     Đây là proof quan trọng nhất. Chụp terminal log cho thấy:
     - LLM ra quyết định gọi tool "query_database"
     - SQL query được tạo ra (VD: SELECT SUM(total_cost) FROM monthly_costs WHERE ...)
     - Kết quả thực từ DB (VD: [{"sum": 16500}])
     - LLM nhận kết quả và sinh ra final answer
     
     Nếu có Observability Dashboard: chụp dashboard thay thế.
     
     ![L3 DB Tool Call Proof](./screenshots/L3_tool_call_database.png)
-->

> ⬆️ **[CHÈN LOG/TRACE TẠI ĐÂY]** — Proof tool `query_database` được gọi với real data

---

#### L3b — Service Metrics Tool (Live API)

**Test question used:**
```
<!-- VD: "What is PaymentGW's current p99 latency?" -->
```

**Expected answer:** `~185ms (from Monitoring API, NOT from a document)`

**Actual answer from system:**
> `<!-- Paste text output -->`

<!-- 📸 SCREENSHOT [REQUIRED]:
     Chụp output với latency thực từ Monitoring API.
     
     ![L3 Metrics Answer](./screenshots/L3_metrics_answer.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Output với live metrics

<!-- 📸 LOG/PROOF [REQUIRED]:
     Log cho thấy HTTP call tới http://localhost:8000/metrics/PaymentGW được thực hiện
     và kết quả JSON trả về được inject vào LLM context.
     
     ![L3 Metrics Tool Proof](./screenshots/L3_tool_call_metrics.png)
-->

> ⬆️ **[CHÈN LOG TẠI ĐÂY]** — HTTP call tới Monitoring API + JSON response

---
#### L3c — Combined Tool Call (Bonus complexity)

**Test question used:**
```
<!-- VD: "Is NotificationSvc meeting its SLA targets?" -->
```

**Expected answer:** `No — current latency ~3200ms vs SLA target 2000ms; error rate 2.1% vs target 1.0%`

**Actual answer from system:**
> `<!-- Paste text output -->`

<!-- 📸 SCREENSHOT [OPTIONAL nhưng rất có giá trị]:
     Câu hỏi này yêu cầu BOTH tools: get_service_metrics (live) + query_database (SLA targets).
     Chụp output cho thấy hệ thống so sánh đúng hai nguồn data.
     
     ![L3 Combined Tools](./screenshots/L3_combined_tools.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY — OPTIONAL]** — Output so sánh live metrics vs SLA targets từ DB

**Notes:**
> `<!-- Mô tả tool routing: VD: "LLM gọi get_service_metrics trước để lấy current latency/error, sau đó gọi query_database để lấy SLA target, cuối cùng compare và kết luận." -->`

---

### L4 — Memory (Multi-Turn Conversation) `[1.0 điểm]`

> **Lưu ý:** L4 chỉ chiếm 10% điểm. Nếu nhóm còn thời gian mới implement. Nếu không, ghi rõ "Not attempted" và giải thích ngắn.

**Status:** `<!-- Attempted ✅ / Not attempted ❌ -->`

---

**Memory strategy used:**
> `<!-- Chọn một: Buffer (all turns) / Window (last N turns) / Query rewriting / DynamoDB session store -->`
> `<!-- Mô tả implementation cụ thể: VD: "Lưu last 10 messages trong in-memory list. Append vào mỗi LLM call. Production sẽ dùng DynamoDB với PK=session_id, SK=turn_number." -->`

**Test conversation used:**
```
Turn 1: "Which service had the highest cost in March 2026?"
Turn 2: "Why did its costs spike?"       ← "its" phải resolve = PaymentGW
Turn 3: "Which team is responsible?"     ← vẫn về PaymentGW
Turn 4: "The postmortem mentioned a review deadline. Is it overdue?"
```

<!-- 📸 SCREENSHOT [REQUIRED nếu L4 attempted]:
     Chụp TOÀN BỘ conversation (4 turns) trong một màn hình hoặc nhiều ảnh liên tiếp.
     Phải thấy rõ hệ thống resolve "its" / "their" / "it" đúng sang service/team cụ thể.
     
     ![L4 Multi-Turn Conversation](./screenshots/L4_conversation_full.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Full 4-turn conversation với pronoun resolution đúng

**Notes (1-2 dòng):**
> `<!-- VD: "Sử dụng window memory với last-10-turns. Turn 2 'its' được resolve sang PaymentGW nhờ Turn 1 vẫn còn trong context. Production limitation: in-memory state sẽ mất khi restart." -->`

---

### Bonus A — Observability Dashboard `[+0.5 điểm]`

**Status:** `<!-- Attempted ✅ / Not attempted ❌ -->`

<!-- 📸 SCREENSHOT [REQUIRED nếu claim Bonus A]:
     Chụp dashboard đang hoạt động khi xử lý một câu hỏi:
     - Panel "Retrieved Chunks": hiển thị top-K chunks với source
     - Panel "Tool Calls": tool nào được gọi, params gì, result gì
     - Panel "LLM Input/Output": full prompt gửi vào và answer trả về
     
     ![Bonus A Dashboard](./screenshots/bonus_A_dashboard.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Observability dashboard đang process một câu hỏi

---

### Bonus B — Agent Reasoning (Structured Investigation) `[+0.5 điểm]`

**Status:** `<!-- Attempted ✅ / Not attempted ❌ -->`

**Test question used:**
```
<!-- VD: "Is NotificationSvc in a healthy state? Assess its reliability and flag anything that needs attention." -->
```

<!-- 📸 SCREENSHOT [REQUIRED nếu claim Bonus B]:
     Chụp structured output với visible reasoning steps:
     - Step 1: Agent plan ("I need to check: current status, latency, error rate, SLA targets, recent incidents")
     - Step 2: Tool calls executed (với results)
     - Step 3: Structured report/conclusion
     
     ![Bonus B Agent Reasoning](./screenshots/bonus_B_agent_reasoning.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — Structured investigation output với visible reasoning

---

### Bonus C — Knowledge Base Auto-Sync `[+0.5 điểm]`

**Status:** `<!-- Attempted ✅ / Not attempted ❌ -->`

**Approach:**
> `<!-- VD: "S3 Event Notification → Lambda trigger → StartIngestionJob API call" hoặc "Jupyter notebook on-demand sync" -->`

<!-- 📸 SCREENSHOT [REQUIRED nếu claim Bonus C]:
     Chụp Lambda console hoặc script output cho thấy sync được trigger khi S3 thay đổi.
     
     ![Bonus C KB Sync](./screenshots/bonus_C_kb_sync.png)
-->

> ⬆️ **[CHÈN ẢNH TẠI ĐÂY]** — KB sync mechanism in action

---

## Section 5 — Reflection

### Hardest Level and Why

> `<!-- Thành thật về level nào khó nhất và tại sao.
>      VD: "L3 khó nhất vì tool routing không ổn định — LLM thỉnh thoảng gọi sai tool hoặc không gọi tool mà hallucinate số liệu.
>           Root cause: tool description không đủ rõ ràng về khi nào dùng DB vs khi nào dùng API." -->`

### What We Would Do Differently With One More Day

> `<!-- VD:
>      1. Implement hybrid search (BM25 + vector) để improve L2 retrieval accuracy
>      2. Thêm query rewriting cho L4 thay vì chỉ dùng buffer memory
>      3. Viết test cases tự động để verify L3 numerical accuracy trước Friday -->`

---

## Scoring Summary (Self-Assessment)

> **Nhóm tự đánh giá** trước khi present — trainers sẽ verify lại.

| Level | Max Points | Self-Score (1-5) | Estimated Points | Status |
|-------|-----------|-----------------|-----------------|--------|
| L1 — Retrieval | 2.0 | ` /5` | ` /2.0` | `<!-- ✅ Working / ⚠️ Partial / ❌ Failed -->` |
| L2 — Multi-Source | 3.0 | ` /5` | ` /3.0` | `<!-- ✅ Working / ⚠️ Partial / ❌ Failed -->` |
| L3 — Tools | 4.0 | ` /5` | ` /4.0` | `<!-- ✅ Working / ⚠️ Partial / ❌ Failed -->` |
| L4 — Memory | 1.0 | ` /5` | ` /1.0` | `<!-- ✅ Working / ⚠️ Partial / ❌ Not attempted -->` |
| **Base Total** | **10.0** | | ` /10.0` | |
| Bonus A — Dashboard | +0.5 | — | ` /0.5` | `<!-- ✅ / ❌ -->` |
| Bonus B — Agent Reasoning | +0.5 | — | ` /0.5` | `<!-- ✅ / ❌ -->` |
| Bonus C — KB Sync | +0.5 | — | ` /0.5` | `<!-- ✅ / ❌ -->` |
| **Grand Total** | **11.0 max** | | ` /11.0` | |

---

*Evidence Pack committed: `<!-- VD: 2026-05-08 09:45 +07:00 -->` | Commit: `<!-- hash -->`*