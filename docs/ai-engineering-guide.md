# Chuẩn Hóa Quy Trình Vòng Đời Sản Phẩm Có Sử Dụng AI (The AI-Native Product Lifecycle Standard)

> **Tài liệu chuẩn hóa:** Định nghĩa danh mục các **hành động bắt buộc (Mandatory Deliverables & Gates)** trong toàn bộ vòng đời của một sản phẩm phần mềm có ứng dụng AI chuyên sâu, dựa trên nghiên cứu của Google (*The New SDLC With Vibe Coding*, 2026), tiêu chuẩn **SE4AI (Software Engineering for AI)**, và thực tiễn phát triển **Spec-Driven Development**.

---

## BẢN CHẤT: HAI HỆ THỐNG TRONG MỘT SẢN PHẨM AI

Một sản phẩm phần mềm có ứng dụng AI luôn chứa hai thế giới song song:
1. **Hệ thống Tất định (Deterministic System):** Core backend, database quan hệ, authentication, phân quyền, logic thanh toán, giao diện người dùng. Yêu cầu đúng/sai tuyệt đối (100% deterministic).
2. **Hệ thống Phi tất định (Probabilistic AI System):** LLMs, Vector embeddings, RAG pipeline, Semantic router, Agents. Kết quả mang tính xác suất, có thể trôi ngữ nghĩa (*drift*) và phát sinh ảo giác (*hallucination*).

Do đó, quy trình vòng đời của sản phẩm có AI **bắt buộc phải chia làm 3 phân tầng kiểm soát khép kín**:

```
┌────────────────────────────────────────────────────────────────────────┐
│ PHÂN TẦNG 1: KHỞI TẠO NỀN TẢNG (PROJECT FOUNDATION - BẮT BUỘC 1 LẦN)   │
│ 1. Thẩm định bài toán: AI vs Heuristic (Có thực sự cần AI không?)       │
│ 2. Hiến pháp dự án (constitution.md) & Dây cương Agent (AGENTS.md)     │
│ 3. Khóa từ điển nghiệp vụ bất biến (GLOSSARY.md)                       │
│ 4. Kiến trúc C4 & Tokens Design System (system-context.md, tokens.css) │
│ 5. Setup hạ tầng kiểm soát: Postman Workspace, Neon DB, Pre-commit     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ PHÂN TẦNG 2: VÒNG LẶP SPRINT LIÊN TỤC (INNER SPRINT LOOP - TỪNG FEATURE)│
│                                                                        │
│   [Bước 1: Specify] ➔ [Bước 2: Plan & ADR] ➔ [Bước 3: Prototype/Evals] │
│          ▲                                                   │         │
│          │                                                   ▼         │
│   [Bước 6: Converge] ◀── [Bước 5: Review]   ◀── [Bước 4: TDD Build]   │
│                                                                        │
│   *Lặp lại 6 bước này liên tục cho từng User Story đến khi xong Sprint*│
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
┌───────────────────────────────────▼────────────────────────────────────┐
│ PHÂN TẦNG 3: PHÁT HÀNH & VẬN HÀNH (OUTER RELEASE & FEEDBACK LOOP)      │
│ 1. Database Branching per PR (Neon Ephemeral Test)                     │
│ 2. Automated Contract Verification (Postman CLI CI)                    │
│ 3. Triển khai Production & Edge WAF (Cloudflare)                       │
│ 4. LLMOps Tracing hai tầng (Langfuse + Sentry + PostHog)               │
│ 5. Flywheel phản hồi: Lỗi người dùng đẩy ngược về bộ evals.json        │
└────────────────────────────────────────────────────────────────────────┘
```

---

## PHÂN TẦNG 1: GIAI ĐOẠN KHỞI TẠO NỀN TẢNG (PROJECT FOUNDATION)
*(Bắt buộc thực hiện 1 lần duy nhất ở đầu dự án hoặc khi có đợt đại tu kiến trúc)*

Đây là bước tạo dựng **"Dây Cương (The Harness)"**. Mô hình AI chỉ quyết định 10% chất lượng sinh mã, 90% thành bại của dự án nằm ở hệ thống quy tắc và ranh giới bạn thiết lập ở giai đoạn này.

### Những việc BẮT BUỘC PHẢI LÀM:

#### 1. Thẩm định tính khả thi: AI vs Heuristic (Feasibility Gate)
* **Bắt buộc:** Trả lời câu hỏi: *Bài toán này có thực sự cần dùng AI không?*
* **Quy tắc chặn:** Nếu tính năng có thể giải quyết dứt điểm bằng regex, thuật toán toán học, cây quyết định `if/else`, hoặc truy vấn SQL thuần túy $\rightarrow$ **CẤM DÙNG LLM**. Chỉ đưa AI vào các bài toán xử lý ngôn ngữ tự nhiên không cấu trúc, trích xuất ngữ nghĩa, hoặc suy luận đa bước. Việc này giúp tiết kiệm 80% chi phí vận hành và loại bỏ độ trễ không đáng có.

#### 2. Thiết lập Hiến pháp dự án (`spec/.specify/memory/constitution.md`)
* **Bắt buộc:** Khóa cứng 4 nguyên tắc phát triển tối cao của dự án:
  - **Test-First (Non-Negotiable):** Mọi logic phải có test trước khi có code implementation.
  - **Adapter-First for AI:** Mọi dịch vụ AI phải bị cô lập sau Interface trừu tượng.
  - **Library-First:** Module hóa mã nguồn thành các thư viện độc lập, dễ test cô lập.
  - **Zero-Trust AI Output:** Mọi kết quả do AI sinh ra phải qua validation (Zod/Pydantic) trước khi đưa vào Database hoặc render lên màn hình.

#### 3. Khóa Dây Cương Hoạt Động Của Agent (`AGENTS.md` / `CLAUDE.md`)
* **Bắt buộc:** Khai báo rõ ràng:
  - Lệnh chạy test, lệnh typecheck, lệnh build.
  - Vị trí bắt buộc lưu kế hoạch (`.agents/plans/plan-*.md`), vị trí lưu nháp (`.agents/note.md`).
  - **Danh mục file cấm sửa:** Cấm Agent tự ý sửa file môi trường (`.env`), file container (`docker-compose.yml`), và migrations DB nếu không có lệnh rõ ràng.

#### 4. Khóa Từ Điển Nghiệp Vụ Bất Biến (`GLOSSARY.md`)
* **Bắt buộc:** Định nghĩa danh từ và động từ duy nhất trong toàn bộ codebase (ví dụ: bắt buộc dùng `Patient`, cấm dùng lẫn lộn `Customer`, `User`, `Client`). Mọi tên biến, tên bảng DB, tên hàm sau này phải ánh xạ 100% theo file này.

#### 5. Định hình Kiến Trúc C4 & Tokens Design System
* **Bắt buộc:** 
  - `docs/architecture/system-context.md`: Vẽ sơ đồ C4 Level 1 & 2, phân tách rõ vùng Deterministic (Web, Core API, RDBMS) và vùng Probabilistic (Vector DB, LLMs Gateway).
  - `tokens.css`: Khóa toàn bộ biến màu sắc, typography, spacing. AI sau này chỉ được gọi `var(--color-...)`, không được tự ý bịa mã màu CSS.

#### 6. Thiết lập Chốt Chặn Tự Động (Automation Tooling)
* **Bắt buộc:** Cài đặt Git Pre-commit hook (`setup-pre-commit`: Husky + lint-staged) để tự động chặn đứng các commit có lỗi linter hoặc typecheck. Tạo Neon Postgres project và Postman Workspace.

> **CỔNG KIỂM SOÁT (EXIT GATE 1):** Không được viết bất kỳ dòng mã logic nào nếu chưa có: `constitution.md`, `AGENTS.md`, `GLOSSARY.md`, `system-context.md`, và Pre-commit hooks đã hoạt động.

---

## PHÂN TẦNG 2: VÒNG LẶP SPRINT LIÊN TỤC (THE INNER SPRINT LOOP)
*(Trái tim của việc phát triển: Bắt buộc lặp đi lặp lại 6 bước này cho TỪNG tính năng / User Story)*

Trong một Sprint, bạn không làm việc theo kiểu dồn hết việc vào cuối sprint. Mỗi tính năng nhỏ (Vertical Slice) phải đi trọn vẹn qua 6 bước vi mô dưới đây:

```
[ Step 1: Specify ] ➔ [ Step 2: Plan & ADR ] ➔ [ Step 3: Prototype & Evals ]
         ▲                                                     │
         │                                                     ▼
  [ Step 6: Converge ] ◀── [ Bước 5: Review 2 Trục ] ◀── [ Step 4: TDD Build ]
```

### Bước 1: Làm Rõ & Khóa Đặc Tả (Elicit & Specify)
* **Bắt buộc phải làm:**
  1. **Khóa Non-Goals:** Ghi rõ những gì tính năng này KHÔNG làm (chiếc phanh ngăn AI sinh code thừa).
  2. **AI Phỏng vấn ngược:** Dùng `speckit-clarify` hoặc `/sharpen` để AI chất vấn ngược lại bạn về các trường hợp biên (*edge cases*), phân quyền Actor, và ứng xử khi lỗi mạng.
  3. **Viết kịch bản hành vi:** Tạo `specs/[feature]/spec.md` với các kịch bản kiểm thử dạng `Given - When - Then`.
  4. **Khóa hợp đồng kiểu dữ liệu tĩnh:** Tạo `specs/[feature]/contracts.ts` (khai báo 100% Interfaces, DTOs, Zod schemas).

### Bước 2: Phân Rã Kế Hoạch & Ghi Nhận Kiến Trúc (Plan, ADR & Grilling)
* **Bắt buộc phải làm:**
  1. **Phân rã thành lát cắt dọc (Vertical Slices):** Dùng `speckit-tasks` chia tính năng thành 4–8 task nhỏ độc lập trong `tasks.md`. Mỗi task phải đi xuyên suốt từ UI $\rightarrow$ API $\rightarrow$ Core Logic.
  2. **Viết ADR (Architecture Decision Record):** Nếu có quyết định kỹ thuật mới, bắt buộc tạo `docs/adr/ADR-xxx.md` ghi nhận Bối cảnh, Giải pháp, Lý do, và **Đánh đổi phần cứng/chi phí (Trade-offs)**.
  3. **Chạy Grilling:** Kích hoạt skill `grilling` để AI đóng vai Kỹ sư trưởng, chất vấn bóc trần điểm mù: *Xử lý race condition thế nào? Timeout 60s thì UI hiện gì? Token rate-limit tính sao?*
  4. **Tạo Instant API Mocking:** Dùng Postman MCP tạo Mock Server từ contract để Frontend có thể chạy demo ngay lập tức mà không cần chờ Backend.

### Bước 3: Thử Nghiệm AI & Chuẩn Bị Bộ Đo Lường (Prototype & Evals)
*(Bắt buộc đối với các tính năng có nhúng LLM, RAG hoặc Vector Search)*
* **Bắt buộc phải làm:**
  1. **Chạy Prototype độc lập:** Tuyệt đối không code LLM thẳng vào core app. Phải viết script test độc lập trong `prototype/` để đo:
     - Thời gian xuất hiện chữ đầu tiên (*TTFT - Time to First Token*).
     - Chi phí token trung bình trên mỗi lượt gọi.
     - Tỷ lệ trả về đúng cấu trúc JSON mong đợi.
  2. **Tạo bộ dữ liệu vàng (`specs/[feature]/evals.json`):** Chuẩn bị 10–20 test case mẫu (câu hỏi, ngữ cảnh, câu trả lời chuẩn, tiêu chí trích dẫn, tiêu chí an toàn).

### Bước 4: Vòng Lặp Lập Trình Vi Mô (The TDD Build Loop)
* **Bắt buộc phải làm:**
  1. **Adapter Pattern:** Mọi lời gọi AI (OpenAI, Gemini, Grok) bắt buộc phải nằm sau một Interface trừu tượng (ví dụ: `ILlmService`).
  2. **Mock Provider trong Unit Test:** Khi chạy unit test, bắt buộc sử dụng Mock Provider để test chạy trong 5 mili-giây với kết quả tất định 100%, không tốn tiền gọi API ngoài.
  3. **Quy tắc Red $\rightarrow$ Green:**
     - Viết 1 test thất bại tại Public Seam (**Red**).
     - AI sinh lượng mã tối thiểu vừa đủ để test pass (**Green**).
     - **CẤM REFACTOR trong bước này** để tránh làm loãng ngữ cảnh.
     - Lặp lại cho task tiếp theo trong `tasks.md`.

### Bước 5: Thẩm Định Hai Trục Độc Lập & Bảo Mật (Two-Axis Review & Security)
* **Bắt buộc phải làm:**
  1. **Khởi chạy 2 Sub-Agents độc lập:**
     - **Trục 1 (Standards Sub-agent):** Quét sạch danh mục mã bốc mùi Fowler (Mysterious Name, Feature Envy, Primitive Obsession, Duplicated Code).
     - **Trục 2 (Spec Match Sub-agent):** Đối chiếu từng dòng `git diff` với `spec.md` để xóa bỏ tính năng thừa hoặc bổ sung kịch bản lỗi còn thiếu.
  2. **Kiểm tra bảo mật AI (OWASP Top 10 for LLMs):** Kiểm tra chống Prompt Injection, Insecure Output Handling (ngăn LLM sinh mã HTML độc hại gây XSS trên Frontend), và rò rỉ API key.
  3. Tiến hành refactor dọn dẹp mã nguồn theo khuyến nghị của 2 sub-agent.

### Bước 6: Kiểm Thử Hội Tụ & Xoay Vòng Sprint (Converge & Loop)
* **Bắt buộc phải làm:**
  - Chạy `speckit-converge` kiểm tra xem toàn bộ các task trong `tasks.md` đã hoàn thành chưa.
  - **Nếu còn task / User Story trong sprint:** Quay lại Bước 1 để thực hiện tính năng tiếp theo.
  - **Nếu đã hoàn thành mục tiêu sprint:** Tạo Pull Request và chuyển sang Phân tầng 3.

> **CỔNG KIỂM SOÁT (EXIT GATE 2):** 100% test tại Public Seams đạt Green, 100% test case trong `evals.json` đạt điểm chuẩn, và 2 sub-agent review không còn cảnh báo nghiêm trọng.

---

## PHÂN TẦNG 3: PHÁT HÀNH, VẬN HÀNH & PHẢN HỒI (OUTER RELEASE & FEEDBACK LOOP)
*(Vòng lặp bên ngoài: Triển khai kiểm thử trên hạ tầng tạm thời, giám sát AI và nạp phản hồi nuôi sản phẩm)*

### Những việc BẮT BUỘC PHẢI LÀM:

#### 1. Kiểm thử trên Nhánh Cơ Sở Dữ Liệu Tạm (Database Branching per PR)
* **Bắt buộc:** Mỗi khi tạo Pull Request, hệ thống CI (GitHub Actions) tự động dùng **Neon Postgres Branching** để nhân bản một nhánh database riêng biệt từ production data (với dữ liệu đã ẩn danh hóa) trong 1 giây.
* Toàn bộ test E2E (Playwright) và Integration test chạy trên dữ liệu thật này. Khi merge PR, nhánh DB tự động bị hủy, đảm bảo không bao giờ làm hỏng database chính.

#### 2. Tự Động Hóa Kiểm Thử Hợp Đồng API (Postman CLI CI)
* **Bắt buộc:** Pipeline CI tự động chạy bộ Postman Collection để đảm bảo không có endpoint nào bị phá vỡ hợp đồng (*Breaking Changes*) trước khi mã được đưa vào nhánh `main`.

#### 3. Giám Sát Hai Tầng: APM Web + LLMOps Tracing
* **Bắt buộc cài đặt:**
  - **Tầng Web APM:** Sentry (bắt lỗi crash client/server), PostHog (phân tích hành vi tương tác).
  - **Tầng LLMOps Tracing (Langfuse):** Truy vết chi phí token ($), độ trễ P95, số lượng truy vấn bị chặn bởi Guardrail, và tỷ lệ ảo giác (*Hallucination*).

#### 4. Vòng Lặp Phản Hồi Tự Động (Human-in-the-loop Flywheel)
* **Bắt buộc xây dựng:**
  - Trên giao diện Web App, bắt buộc có nút đánh giá (Like/Dislike) hoặc form Báo cáo câu trả lời sai.
  - Khi người dùng bấm Dislike, hệ thống tự động trích xuất cặp `(Prompt - Câu trả lời lỗi)` và đẩy ngược về tệp `evals.json`.
  - Tự động tạo một task mới trong Backlog của Sprint tiếp theo để AI và lập trình viên cải thiện chất lượng mô hình!

> **CỔNG KIỂM SOÁT (EXIT GATE 3):** CI pipeline pass 100%, bản deploy preview hoạt động ổn định trên Cloudflare Edge, và Langfuse telemetry đã bắt đầu nhận dữ liệu giám sát.

---

## BẢNG MA TRẬN TỔNG HỢP: VIỆC BẮT BUỘC & CÔNG CỤ THEO GIAI ĐOẠN

| Phân tầng | Tên Giai Đoạn | Việc Bắt Buộc Phải Làm | Tệp Tin Đầu Ra Bắt Buộc | Công Cụ & MCP Áp Dụng |
| :--- | :--- | :--- | :--- | :--- |
| **Foundation** | Khởi Tạo Nền Tảng (1 lần) | Khóa Hiến pháp, Harness, Từ điển nghiệp vụ, C4, Tokens, Pre-commit | `constitution.md`, `AGENTS.md`, `GLOSSARY.md`, `tokens.css` | `speckit-constitution`, `domain-modeling`, `setup-pre-commit` |
| **Sprint Loop** | 1. Elicit & Specify | Khóa Non-goals, phỏng vấn edge cases, khóa kiểu dữ liệu tĩnh | `spec.md`, `contracts.ts` | `speckit-specify`, `speckit-clarify`, `Firecrawl`, `/sharpen` |
| **Sprint Loop** | 2. Plan & ADR | Phân rã lát cắt dọc, viết ADR đánh đổi, grilling, tạo Mock API | `tasks.md`, `ADR-xxx.md`, Postman Mock | `speckit-plan`, `speckit-tasks`, `codebase-design`, `grilling`, Postman |
| **Sprint Loop** | 3. Prototype & Evals | Đo TTFT, token cost, tạo 10-20 test cases vàng kiểm chứng AI | `evals.json`, `prototype/` | `prototype`, pstack `arena runners`, Ragas, Promptfoo |
| **Sprint Loop** | 4. TDD Build Loop | Giấu LLM sau Adapter, TDD Red $\rightarrow$ Green từng lát cắt dọc | `tests/*.spec.ts`, `src/core/` | Matt Pocock `tdd`, pstack `feature` (`cursor-grok-4.6-high-fast`) |
| **Sprint Loop** | 5. Two-Axis Review | 2 sub-agent quét Fowler Smells và so khớp spec.md; quét OWASP LLM | Báo cáo review mã | Matt Pocock `code-review`, `GitNexus`, `claude-opus-5` |
| **Sprint Loop** | 6. Converge & Loop | Kiểm tra hoàn tất tasks; lặp lại cho feature kế tiếp trong sprint | `speckit-converge` log | `speckit-converge` |
| **Release & Ops**| 7. Ephemeral Deploy | Database Branching per PR, Postman Contract CI, Cloudflare Edge | `.github/workflows/deploy.yml` | `neon-postgres-branches`, `ci-integration`, Playwright |
| **Release & Ops**| 8. LLMOps & Feedback | Giám sát token cost, latency, nạp Dislike ngược về evals.json | Langfuse Traces, Issue Backlog | `Langfuse`, `Sentry`, `PostHog`, Yuiyui `format-doc` |
