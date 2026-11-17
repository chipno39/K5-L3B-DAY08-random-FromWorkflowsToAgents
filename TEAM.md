# Team

## Thông tin nhóm

- Tên nhóm: random
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-random-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: `https://github.com/chipno39/K5-L3B-DAY08-random-FromWorkflowsToAgents` (branch: `main`)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Xuân Đạt | 2A202603958 | xuandat31804@gmail.com | chipno39 | Hoàn thiện shape A + Intergration |
|   2 |        |      |       |        |         |
|   3 | Hoàng Xuân Lĩnh | 2A202604017 | ai26.linhhx@vinuni.edu.vn | hxlinh | Shape C (orchestrator) + Handoff & Trace security |
|   4 | Nguyễn Bảo Trung | 2A202604013 | 26ai.trungnb@vinuni.edu.vn | trungnb | Người 4: Ledger/Crash + Evaluation, Benchmark & Research Card (B4) |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Xuân Đạt (2A202603958) | Workflow + Integration | `4f18f2a` (`src/lab8/shapes/workflow.py`) | `src/tests/test_compensation.py::test_workflow_ignores_ticket_delay_override_across_seeds` |
| Hoàng Xuân Lĩnh (2A202604017) | Orchestrator + Handoff & Trace security | `HASH` (`src/lab8/shapes/orchestrator.py`) | `src/tests/test_handoff_secret.py::test_make_handoff_drops_unlisted_facts_and_refuses_smuggled_secret`, `src/tests/test_handoff_secret.py::test_orchestrator_trace_has_only_sanitized_handoff` |
| Nguyễn Bảo Trung (2A202604013) | Ledger/Crash + Evaluation, Benchmark & Research Card (B4) | `PENDING_HASH` (`src/tests/test_refund_idempotency.py`, `src/bonus/`) | `src/tests/test_refund_idempotency.py::test_crash_before_checkpoint_then_conflict_and_valid_retry`, `src/tests/test_bonus_b4.py` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603958_NguyenXuanDat

### Phần việc phụ trách

- Cài đặt Shape A (`workflow`) trong `src/lab8/shapes/workflow.py` và phụ trách tích hợp (Integration), quản lý bảng đóng góp và kiểm tra điều kiện nộp bài.

### Đóng góp cụ thể

- Hoàn thiện hàm `run_claim` trong `src/lab8/shapes/workflow.py` xử lý đầy đủ các nhánh nghiệp vụ (`resolved`, `denied`, `needs_info`, `escalated`, `failed`), truyền quote đầy đủ và `crash_at` vào `durability.issue_refund`, ghi trace đạt chuẩn Trace contract v1.
- Thêm unit test `test_workflow_ignores_ticket_delay_override_across_seeds` vào `src/tests/test_compensation.py`.

### Bằng chứng

- Commit chính: `4f18f2a`
- Code: `src/lab8/shapes/workflow.py` (pass 62/62 test student `-k workflow`).
- Test: `src/tests/test_compensation.py::test_workflow_ignores_ticket_delay_override_across_seeds`.
- Báo cáo cá nhân: `report/team/2A202603958_NguyenXuanDat.md`.

## 2A202604017_HoangXuanLinh

### Phần việc phụ trách

- Cài đặt Shape C (`orchestrator`) trong `src/lab8/shapes/orchestrator.py` và phần bảo mật Handoff & Trace.

### Đóng góp cụ thể

- Hoàn thiện `run_claim` của orchestrator: handoff dựng bằng `make_handoff` (allowlist, chỉ tool đọc), trace chỉ ghi handoff đã sanitize, orchestrator tự tính tiền và gọi `durability.issue_refund` với quote đầy đủ và `crash_at`.
- Thêm 2 test vào `src/tests/test_handoff_secret.py`.
- Viết mục "Trace và bằng chứng" trong `decision_adr.md` và tiểu mục "Shape C" trong `group_report.md`.

### Bằng chứng

- Commit chính: e315a51
- Code: `src/lab8/shapes/orchestrator.py` (pass 62/62 test student `-k orchestrator`).
- Test: `src/tests/test_handoff_secret.py::test_make_handoff_drops_unlisted_facts_and_refuses_smuggled_secret`, `src/tests/test_handoff_secret.py::test_orchestrator_trace_has_only_sanitized_handoff`.
- Báo cáo cá nhân: `report/team/2A202604017_HoangXuanLinh.md`.

## 2A202604013_NguyenBaoTrung

### Phần việc phụ trách

- Người 4: Kiểm chứng Durability/Ledger (`buggy_workflow.py` vs `durability.py`), chạy đánh giá `benchmark` & `crash_test`, thực hiện Research Card **R4** và **Bonus B4**.

### Đóng góp cụ thể

- Thêm `test_crash_before_checkpoint_then_conflict_and_valid_retry` vào `src/tests/test_refund_idempotency.py`.
- Xây dựng gói Bonus B4 trong `src/bonus/` (`rules_only.py`, `swarm_pruned.py`, `ablations.py`, `stats.py`) và 6 test trong `src/tests/test_bonus_b4.py`.
- Sinh báo cáo đánh giá 4 shape (`src/reports/benchmark.json`) và phân tích ablation/thống kê (`src/reports/bonus_benchmark.json`, `src/reports/bonus_ablation.json`).
- Soạn mục R4 trong `report/team/research_card.md`, `report/team/bonus_report.md` và phần đánh giá/durability trong `report/team/group_report.md`.

### Bằng chứng

- Commit chính: `PENDING_HASH`
- Test: `src/tests/test_refund_idempotency.py::test_crash_before_checkpoint_then_conflict_and_valid_retry`, `src/tests/test_bonus_b4.py` (6 passed).
- Report: `src/reports/benchmark.json`, `src/reports/bonus_benchmark.json`, `src/reports/bonus_ablation.json`.
- Báo cáo cá nhân: `report/team/2A202604013_NguyenBaoTrung.md`.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
