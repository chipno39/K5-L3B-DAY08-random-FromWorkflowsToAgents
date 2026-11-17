# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm:
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):

## 2. Mục tiêu và cách tiếp cận

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

...

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

### Phần đóng góp Người 2: Single Agent (ReAct) + Policy

Shape `single` chạy vòng ReAct qua `lab8.runtime.run_react` với giới hạn 8 bước. Agent chỉ điều phối các tool đọc `get_claim_view`, `lookup_order` và `lookup_refund_history`; không có tool ghi refund. Ticket không được đưa vào prompt hay trace. Sau khi đọc order, code gọi `compute_compensation` trên order view, nên số ngày do khách tự khai và policy cũ trong ticket không quyết định số tiền. Quote đầy đủ được chuyển cho `durability.issue_refund`; host kiểm approval gắn với đúng claim, giới hạn hoàn theo order, idempotency và ghi ledger/audit. Trace chỉ ghi field đã lọc, quote, quyết định và tên tool.

`MockLLM` được gọi trong từng lượt ReAct để ghi nhận model calls và chi phí mô phỏng. Vì MockLLM của lab chỉ trả text tổng quát, trình điều khiển trong `decide` quyết định tuần tự các tool đọc dựa trên trạng thái quan sát; kết quả của model không được dùng để chọn tiền hay quyền ghi.

### Phần đóng góp Người 3 (Hoàng Xuân Lĩnh): Orchestrator + Handoff & Trace Security (`src/lab8/shapes/orchestrator.py`)

**Luồng xử lý.**

1. Orchestrator đọc claim qua `get_claim_view` (không `SELECT *`). Không có claim → `failed`; thiếu `order_id` → `needs_info`, dừng trước khi giao việc, không đoán mã đơn từ ticket.
2. Dựng handoff bằng `make_handoff(request_id, claim_id, task, facts={}, allowed_tools=["lookup_order", "get_clause", "lookup_refund_history"])`.
3. Ghi handoff **đã sanitize** vào trace (event `handoff`).
4. `run_readonly_investigators` chạy các tool đọc có tên trong handoff. Investigator chỉ nhận handoff và `principal_id` (tham số hàm để kiểm quyền sở hữu đơn), không nhận claim row.
5. Đơn không thấy hoặc không thuộc principal → `escalated`.
6. Orchestrator (code) tự gọi `compute_compensation` trên order view mà investigator trả về. Quote = 0 → `denied`, không gọi host.
7. Quote > 0 → `durability.issue_refund(conn, claim_id, <quote đầy đủ>, crash_at=crash_at)`. `committed`/`idempotent_replay` → `resolved`; `conflict` → `failed`; host từ chối → `denied` (host ghi audit `policy_denied`). `InjectedCrash` không bị nuốt: ghi event `crash`, flush trace rồi ném lại.

**Ranh giới code / model.** MockLLM chỉ được gọi 2 lần: `investigate` (prompt là handoff đã lọc) và `draft` (soạn câu trả lời). Model không chọn tool ghi, không quyết số tiền, không quyết status. Investigator không có write registry: `make_handoff` từ chối `issue_refund`/`append_audit`, và `runtime.dispatch(worker=True)` cũng từ chối lần nữa.

**Handoff được sanitize thế nào.** Có ba lớp, đều là code xác định:

| Lớp | Cơ chế | Chặn được gì |
| --- | --- | --- |
| Nguồn | `get_claim_view` không SELECT `parent_secret`, `raw_headers_json` | Secret không bao giờ vào bộ nhớ của shape |
| Allowlist | `make_handoff` chỉ giữ `ALLOWED_FIELDS`, `facts` chỉ giữ `FACT_KEYS`, tool chỉ thuộc `READONLY_TOOLS` | Key lạ (`history`, `ticket`, `phone`…) bị bỏ; write tool bị từ chối |
| Kiểm cuối | `sanitize` serialize cả gói, raise nếu còn `CANARY_SECRET_`, `parent_secret`, `raw_headers` | Secret bị nhét vào một key hợp lệ (ví dụ `facts.carrier`) |

Orchestrator chủ động truyền `facts={}`: lúc giao việc nó chưa biết gì đáng tin về đơn hàng (ticket là dữ liệu người dùng), investigator tự đọc order row. Không dùng LLM để "tóm tắt" parent context làm handoff, vì bản tóm tắt có thể chép lại secret hoặc bị câu chữ trong ticket điều khiển.

**Trace ghi gì.** `claim_view` (claim_id, order_id) → `handoff` → `investigator` (order view đã allowlist + `refund_count`/`refunded_vnd`) → `quote` → `decision`, thêm `crash` nếu có. Không ghi ticket, `principal_id`, prompt của model, hay bất kỳ row thô nào. Chi tiết và bằng chứng: mục "Trace và bằng chứng" trong `decision_adr.md`.

**Kiểm tra.** `python -m pytest -q -m student -o addopts= -k orchestrator` → 62 passed (T01–T14 × 5 seed, T11 retry, T12 crash). Tự kiểm thêm 3 điểm crash (`before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`) trên C-001, C-003, C-012, C-014: sau resume luôn đúng 1 dòng ledger và 1 audit `refund_committed`. T14 (C-014): `proposal_amount` 150.000, ledger 120.000, vì host tự trừ 30.000 đã hoàn.

**Test thêm** (trong `src/tests/test_handoff_secret.py`, chạy cùng cổng host):

- `test_make_handoff_drops_unlisted_facts_and_refuses_smuggled_secret`: key lạ trong `facts` bị bỏ; secret giấu trong key hợp lệ thì bị chặn.
- `test_orchestrator_trace_has_only_sanitized_handoff` (14 claim): trace không chứa secret/PII; handoff chỉ có field allowlist và tool chỉ đọc.

**Ghi chú.** `run_readonly_investigators` hiện bỏ qua `get_clause` (`continue`). Nhóm vẫn cấp tool này theo phân công nhưng nó không gây tool call nào. Khi cần đọc clause, orchestrator vẫn phải tự kiểm `is_current`, vì `get_clause` trả cả policy cũ.

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

#### So sánh workflow và single (đóng góp Người 2)

Benchmark đối chiếu riêng hai shape trên cùng 14 public claims × 5 seed (70 trial/shape), dùng MockLLM và cùng cấu hình giá. File được sinh bằng `lab8.benchmark` tại `src/reports/benchmark_workflow_single.json`; đây là phép so sánh hai shape, chưa thay thế benchmark nộp bài đủ bốn shape ở `src/reports/benchmark.json`.

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean / p95 steps | Mean model / tool calls |
|---|---:|---:|---:|---:|---:|---:|---:|
| workflow | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 28.84 | 3.79 / 4 | 1.00 / 2.79 |
| single | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 37.76 | 8.43 / 9 | 2.79 / 3.64 |

Trên bộ public này, cả hai shape đều pass toàn bộ trial và không dao động giữa seed. `single` tốn chi phí mô phỏng cao hơn khoảng 31% mỗi lần thành công, đồng thời có nhiều bước trace và nhiều lần gọi model/tool hơn. Kết quả này cho thấy vòng ReAct chưa cải thiện độ đúng trên corpus public so với workflow cố định; nó chỉ cho phép cấu trúc điều phối đọc linh hoạt hơn. Đây là kết quả của MockLLM và 14 claims public, không chứng minh hiệu năng tương tự với model hoặc dữ liệu production. Latency không được đo.

Lệnh tạo file so sánh:

```bash
python -m lab8.benchmark --shapes workflow,single --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark_workflow_single.json
```

Crash suite của nhánh hiện tại: `crash_safety` = `20/20`, được tạo tại `src/reports/crash.json`.

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

Ban đầu lệnh yêu cầu `python -m pytest -q -m student -o addopts= -k single` chọn thêm shape khác do tên test ma trận chung `test_public_single_run_cases` cũng chứa từ `single`. Đổi tên hàm ma trận thành `test_public_matrix_cases` để `-k single` lọc đúng shape theo parameter ID; không đổi assertion hay coverage. Sau sửa, lệnh nguyên văn đạt 62 passed. Test mới về approval mượn của claim khác: 1 passed. Cổng host: 56 passed, 248 deselected.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
