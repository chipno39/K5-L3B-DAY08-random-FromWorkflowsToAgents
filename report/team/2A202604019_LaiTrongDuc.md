# Báo cáo cá nhân — Lai Trong Duc

## 1. Thông tin thành viên

- Họ tên: Lại Trọng Đức
- MSSV: 2A202604019
- Email: 26ai.duclt2@vinuni.edu.vn
- GitHub: laitrongduc

## 2. Phần việc phụ trách

Triển khai Shape B `single` (Single Agent/ReAct) và policy-safe refund. Phụ trách `src/lab8/shapes/single.py`, bổ sung test approval binding, so sánh workflow với single và ghi phần kết quả liên quan trong báo cáo nhóm.

## 3. Đóng góp cụ thể

- Triển khai vòng `run_react` có giới hạn 8 lượt, dùng các tool đọc để lấy claim, order và lịch sử hoàn tiền. Agent không có đường ghi refund.
- Tính quote từ order view và policy hiện hành bằng `compute_compensation`; không dùng thông tin delay hoặc điều khoản do khách ghi trong ticket để quyết định số tiền.
- Chuyển quote đầy đủ cho `durability.issue_refund` để host kiểm tra approval gắn đúng claim, order cap, idempotency và ghi ledger. Truyền tiếp `crash_at`, không bắt `InjectedCrash`.
- Ghi trace theo allowlist; loại ticket và dữ liệu nhạy cảm khỏi prompt/trace.
- Thêm test `test_single_shape_rejects_approval_bound_to_another_claim`, xác nhận approval của claim khác bị từ chối, không tạo refund và có audit denial.
- Đổi tên test parameterized chung trong `test_public_matrix.py` để lệnh `-k single` lọc đúng shape theo parameter ID; giữ nguyên assertions và coverage.
- Viết so sánh workflow/single vào `report/team/group_report.md` và sinh benchmark tương ứng.

Commit chính: [`00cd34c`](https://github.com/chipno39/K5-L3B-DAY08-random-FromWorkflowsToAgents/commit/00cd34c) — `feat: implement single agent ReAct shape`.

## 4. Bằng chứng

- `python -m pytest -q -m student -o addopts= -k single`: **62 passed, 242 deselected**.
- `python -m pytest -q -m student -o addopts= -k workflow`: **62 passed, 242 deselected**.
- `python -m pytest -q`: **56 passed, 248 deselected**.
- `python -m pytest -q tests/test_order_cap_and_approval_binding.py -k single_shape`: **1 passed, 8 deselected**.
- `python -m lab8.crash_test --count 20 --out reports/crash.json`: **crash_safety 20/20**.
- Benchmark workflow và single trên 14 public claims × 5 seed: mỗi shape **70/70 passed**, pass rate/pass^5 là **1.000**, seed variance **0**. Cost/success mô phỏng: workflow **28.84 VND**, single **37.76 VND**. Kết quả tại `src/reports/benchmark_workflow_single.json`.
- Báo cáo so sánh: phần “So sánh workflow và single” trong `report/team/group_report.md`.

Các kết quả chi phí là mô phỏng bằng MockLLM, không phải hóa đơn API; latency không được đo. Benchmark này chỉ đối chiếu workflow với single, chưa thay thế benchmark đủ bốn shape của nhóm.

## 5. Công cụ AI đã dùng

Đã dùng ChatGPT/Codex để đọc contract và API hiện có, hỗ trợ triển khai ReAct, rà soát trace và soạn regression test/báo cáo. Tự kiểm tra bằng test approval binding, ma trận student của single, cổng host, crash suite và benchmark do công cụ lab sinh ra. Không dùng số liệu do AI tạo; các con số trong báo cáo được lấy từ output và artifact benchmark.

## 6. Tự đánh giá

Phần triển khai giữ quyền quyết định và ghi tiền ở code/host, đồng thời để vòng ReAct điều phối các bước đọc dữ liệu. Test cho thấy single xử lý đúng các claim public và tình huống approval binding, order cap, retry/crash. Trong benchmark public này, single không tăng pass rate so với workflow nhưng chi phí mô phỏng mỗi lần thành công cao hơn khoảng 31% và có nhiều bước hơn. Kết luận chỉ áp dụng cho corpus public và MockLLM; cần benchmark đủ bốn shape và dữ liệu khác trước khi đưa ra quyết định kiến trúc cuối cho cả nhóm.
