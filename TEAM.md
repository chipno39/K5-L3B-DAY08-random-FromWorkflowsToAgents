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

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Xuân Đạt (2A202603958) | Workflow + Integration | `4f18f2a` (`src/lab8/shapes/workflow.py`) | `src/tests/test_compensation.py::test_workflow_ignores_ticket_delay_override_across_seeds` |

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

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
