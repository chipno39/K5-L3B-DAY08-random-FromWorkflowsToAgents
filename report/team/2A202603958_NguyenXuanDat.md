# Báo cáo cá nhân — 2A202603958_NguyenXuanDat

## 1. Thông tin thành viên

- Họ tên: Nguyễn Xuân Đạt
- MSSV: 2A202603958
- Email: xuandat31804@gmail.com
- GitHub: chipno39

## 2. Phần việc phụ trách

- **Shape A (`workflow`):** Thiết kế và cài đặt hoàn chỉnh luồng xử lý claim cố định bằng Python trong `src/lab8/shapes/workflow.py` (từ bước đọc claim qua view an toàn, kiểm tra đơn hàng, tính toán mức đền bù bằng code, gọi hoàn tiền qua cổng idempotent của host, đến xuất `RunResult` và ghi `Trace contract v1`).
- **Kiểm thử (Unit Testing):** Bổ sung unit test kiểm chứng tính bất biến của `workflow` trước thông tin nhiễu trong ticket của khách hàng qua nhiều seed trong `src/tests/test_compensation.py`.
- **Tích hợp & Quản lý nộp bài (Integration):** Thiết lập cấu trúc phân công nhóm trong `TEAM.md`, kiểm tra cổng host (`pytest -q`), bộ test student (`-m student -k workflow`) và `lab8.check_submission`.

## 3. Đóng góp cụ thể

- **Cài đặt `src/lab8/shapes/workflow.py` (CP2 & CP3):**
  - Sử dụng `get_claim_view` và `lookup_order` thay vì truy vấn raw SQL `SELECT *` để không chạm vào `parent_secret`, header xác thực, số điện thoại, địa chỉ hay ghi chú nội bộ.
  - Xử lý đầy đủ các nhánh biên: claim không tồn tại (`failed`), thiếu mã đơn `order_id` (`needs_info`), đơn không tồn tại hoặc không thuộc quyền sở hữu của `principal_id` (`escalated`).
  - Tách biệt hoàn toàn ranh giới **Code vs. Model**: Tiền bồi thường (`proposal`) được tính hoàn toàn bằng `compute_compensation(conn, order_view["order"])` dựa trên dữ liệu đơn hàng thực tế và chính sách hiện hành (`is_current = 1`); `MockLLM.complete(..., purpose="draft")` chỉ dùng để soạn thảo thông báo phản hồi.
  - Tuân thủ quy tắc **trần hoàn tiền theo đơn (T13, T14)** và **audit `policy_denied` (T07)**: Khi `proposal == 0` trả về `denied` ngay và không gọi `issue_refund`; khi `proposal > 0` truyền **quote đầy đủ** vào `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)` để host tự kiểm tra approval và trừ phần đã hoàn của các claim trước trên cùng đơn hàng.
  - Đảm bảo **Crash Safety & Trace Contract v1**: Truyền `crash_at` xuống `issue_refund`, ghi nhận sự kiện crash vào trace trước khi `raise` lại `InjectedCrash` (không nuốt ngoại lệ), và luôn đóng gói kết quả qua `lab8.shapes.common.finish()`.
- **Bổ sung test `test_workflow_ignores_ticket_delay_override_across_seeds` trong `src/tests/test_compensation.py`:**
  - Kiểm tra trường hợp ticket ghi trễ 5 ngày nhưng dữ liệu đơn hàng thực tế chỉ trễ 2 ngày (`C-002`) trên cả `seed=11` và `seed=55`, đảm bảo `workflow` luôn từ chối (`status="denied"`, `proposal_amount=0`, không tạo dòng ledger), khắc phục triệt để điểm yếu `REFERENCE_WEAKNESS` của `swarm`.

## 4. Bằng chứng

- **Commit chính:** `4f18f2a` (`feat(workflow): implement shape A workflow and add delay override test`)
- **File code:** `src/lab8/shapes/workflow.py`
- **Test đã thêm:** `src/tests/test_compensation.py::test_workflow_ignores_ticket_delay_override_across_seeds`
- **Kết quả kiểm thử trên máy cá nhân:**
  - `python -m lab8.smoke` → `lab8.smoke pass`
  - `python -m pytest -q` → `55 passed in 3.24s`
  - `python -m pytest -q -m student -o addopts= -k workflow` → `62 passed in 3.26s` (vượt qua toàn bộ T01–T14 trên 5 seed `11, 22, 33, 44, 55`, bao gồm T11 idempotency và T12 crash recovery).
- **Run ID tiêu biểu:** `R-workflow-C-001-s11`, `R-workflow-C-002-s55`, `R-workflow-C-014-s11`.

## 5. Công cụ AI đã dùng

- **Công cụ:** Trợ lý AI (Antigravity / Gemini).
- **Mục đích sử dụng:** Phân tích yêu cầu đề bài, rà soát các điều kiện must-pass trong `docs/business_rules.md` và `docs/contracts.md`, hỗ trợ rà soát luồng xử lý các trạng thái trả về của `workflow.py` và viết unit test kiểm tra tính nhất quán theo seed.
- **Cách tự kiểm tra kết quả:**
  - Đọc đối chiếu trực tiếp mã nguồn `src/lab8/durability.py`, `src/lab8/tools.py`, `src/lab8/shapes/common.py` để xác minh chữ ký hàm và cấu trúc dữ liệu trả về.
  - Chạy trực tiếp bộ kiểm thử `python -m pytest -q` và `python -m pytest -q -m student -o addopts= -k workflow` để xác nhận toàn bộ 62/62 test cases của shape `workflow` và test mới thêm đều pass.
  - Kiểm tra nội dung file trace sinh ra trong `src/traces/R-workflow-*.json` để chắc chắn không chứa `CANARY_SECRET_*`, `parent_secret` hay thông tin cá nhân (PII).

## 6. Tự đánh giá

- **Mức độ hoàn thành:** 100% nhiệm vụ của Người 1 (Shape A `workflow`, unit test bổ sung, và khung tích hợp nhóm).
- **Bài học rút ra:** Hiểu rõ vì sao các thao tác tính tiền, kiểm tra quyền sở hữu, kiểm tra approval và chống ghi trùng (idempotency qua ledger SQLite) bắt buộc phải nằm trong code xác định (deterministic host/workflow), cũng như lý do kiến trúc `workflow` đạt độ ổn định tuyệt đối (`seed_variance = 0`) với chi phí thấp nhất khi bài toán có cấu trúc rõ ràng.
