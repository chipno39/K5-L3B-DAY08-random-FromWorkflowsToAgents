# Báo cáo cá nhân — 2A202604017_HoangXuanLinh

## 1. Thông tin thành viên

- Họ tên: Hoàng Xuân Lĩnh
- MSSV: 2A202604017
- Email: ai26.linhhx@vinuni.edu.vn
- GitHub: hxlinh

## 2. Phần việc phụ trách

- **Shape C (`orchestrator`)** trong `src/lab8/shapes/orchestrator.py`: orchestrator đọc claim, giao việc cho investigator chỉ có tool đọc, rồi tự tính tiền và gọi host ghi refund.
- **Bảo mật Handoff & Trace**: handoff chỉ chứa field trong allowlist; trace không lộ secret hay thông tin cá nhân.
- **Test**: thêm test vào `src/tests/test_handoff_secret.py`.
- **Báo cáo**: mục "Trace và bằng chứng" trong `decision_adr.md` và tiểu mục "Shape C" trong `group_report.md`.

## 3. Đóng góp cụ thể

**Code `orchestrator.py`**

- Đọc claim qua `get_claim_view` (không `SELECT *`). Claim không tồn tại → `failed`; thiếu `order_id` → `needs_info`, dừng trước khi giao việc.
- Dựng handoff bằng `make_handoff` với `facts={}` và chỉ cấp tool đọc (`lookup_order`, `get_clause`, `lookup_refund_history`). Không đưa claim row, `parent_secret`, `raw_headers` hay ticket vào handoff.
- Chỉ ghi handoff **đã sanitize** vào trace, không ghi parent context.
- Gọi `run_readonly_investigators`. Đơn không thấy hoặc không thuộc khách → `escalated`.
- Orchestrator tự tính tiền bằng `compute_compensation` từ order row (không tin số ngày trễ trong ticket). Quote = 0 → `denied`; quote > 0 → `issue_refund(..., quote đầy đủ, crash_at=crash_at)`. Không nuốt `InjectedCrash`.

**Quyết định kỹ thuật**

- `facts={}`: lúc giao việc, orchestrator chưa có dữ kiện đáng tin (ticket là chữ của khách), nên để investigator tự đọc từ DB. Handoff càng ít dữ liệu thì càng ít thứ có thể lộ.
- Trace cố ý không ghi `ticket`, `principal_id`, prompt của model và mọi row thô. Chỉ ghi các view đã qua allowlist.
- Gom mọi nhánh kết thúc vào một hàm `done()`, để nhánh nào cũng có event `decision` và trace được ghi qua `finish()`.

**Test thêm** (chạy trong cổng host `pytest -q`)

- `test_make_handoff_drops_unlisted_facts_and_refuses_smuggled_secret`: field lạ trong `facts` (ticket, phone, parent_secret) bị bỏ; secret nhét vào field hợp lệ thì bị chặn (`HandoffError`).
- `test_orchestrator_trace_has_only_sanitized_handoff`: chạy orchestrator trên 14 claim public; trace không chứa `CANARY_`, `Bearer`, SĐT, địa chỉ, internal note, tên hàng; handoff chỉ có field allowlist và tool chỉ đọc.

## 4. Bằng chứng

- Commit chính: e315a51
- File code: `src/lab8/shapes/orchestrator.py`
- Test: `src/tests/test_handoff_secret.py` (2 test mới, 1 test chạy trên 14 claim)
- Kết quả chạy trên máy:
  - `python -m pytest -q -m student -o addopts= -k orchestrator` → `62 passed`
  - `python -m pytest -q` → `70 passed`
  - Tự kiểm crash ở 3 điểm (`before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`) trên C-001, C-003, C-012, C-014: sau khi chạy lại luôn có 1 dòng ledger và 1 audit `refund_committed`.
- Run id tiêu biểu: `R-orchestrator-C-014-s11`. Kết quả `resolved`, `proposal_amount` 150.000, ledger ghi 120.000 (host trừ 30.000 đã hoàn trước).

## 5. Công cụ AI đã dùng

- **Công cụ:** Claude Code.
- **Dùng vào việc:** đọc repo và tóm tắt yêu cầu, hỗ trợ viết code
- **Cách tự kiểm tra:** chạy pytest (orchestrator 62 passed, cổng host 71 passed), mở trace `R-orchestrator-C-014-s11.json` kiểm từng event.


## 6. Tự đánh giá

- **Mức hoàn thành:** Đã hoàn thành phần việc Shape C, test handoff/trace, phần báo cáo bảo mật
- **Điều học được:** Handoff phải được lọc bằng allowlist trong code chứ không nhờ LLM tóm tắt, vì câu trả lời của LLM không kiểm chứng được, còn allowlist thì có thể kiểm bằng test.