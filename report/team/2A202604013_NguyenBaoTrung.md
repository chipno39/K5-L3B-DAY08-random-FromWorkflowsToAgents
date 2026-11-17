# Báo cáo cá nhân — 2A202604013_NguyenBaoTrung

## 1. Thông tin thành viên

- Họ tên: Nguyễn Bảo Trung
- MSSV: 2A202604013
- Email: 26ai.trungnb@vinuni.edu.vn
- GitHub: trungnb

## 2. Phần việc phụ trách

Người 4 (nhóm `random`): Ledger/Crash + Evaluation, Benchmark & Research Card (kèm Bonus B4).

## 3. Đóng góp cụ thể

- Commit chính: `17c4e7b` (`feat(eval): add durability test, 4-shape benchmark, and B4 ablations`)
- Test đã thêm:
  - `src/tests/test_refund_idempotency.py::test_crash_before_checkpoint_then_conflict_and_valid_retry`: kiểm chứng crash tại `after_effect_before_checkpoint` trên `C-011`, retry lệch số (`10000`) bị chặn bởi `amount_conflict` (giữ nguyên `35000` và ghi 1 audit `amount_conflict`), sau đó retry đúng số (`35000`) tự bù bảng `checkpoints` và trả `idempotent_replay` với đúng `1` dòng ledger và `1` audit `refund_committed`.
  - `src/tests/test_bonus_b4.py`: 6 test kiểm tra baseline `bonus.rules_only`, `bonus.swarm_pruned`, cluster bootstrap 95% CI theo `claim_id`, exact McNemar, các biến thể ablation B4 và tích hợp với `lab8.benchmark`.
- Module & báo cáo đã thực hiện:
  - Bộ Bonus B4 trong `src/bonus/` (`__init__.py`, `rules_only.py`, `swarm_pruned.py`, `ablations.py`, `stats.py` thuần thư viện chuẩn).
  - Điền phần lý thuyết câu hỏi **R4** (*Evaluation confounders*, Zhang et al., EMNLP 2026, `arXiv:2609.05933`) và phương án ablation trong `report/team/research_card.md`; dựng khung `report/team/bonus_report.md`; điền phần phụ trách Vị trí 4 trong `report/team/group_report.md` và `TEAM.md`.
- Quyết định kỹ thuật:
  - Dùng `10000` VND (< `approved_vnd = 35000`) ở nhánh resume conflict cho `C-011` vì `policy.authorize()` cắt trần `min(proposed, approved)` trước bước so sánh ledger; truyền lớn hơn `35000` sẽ bị cap về `35000` và đi vào `idempotent_replay` thay vì `amount_conflict`.
  - Tính 95% CI bằng cluster bootstrap resample theo `claim_id` (giữ tương quan 5 seed cùng claim) và kiểm định cặp bằng exact two-sided McNemar thay cho $\chi^2$ vì cỡ mẫu bất đồng trên 14 public claim nhỏ (`b + c < 25`).

## 4. Bằng chứng

- Link commit: [`17c4e7b`](https://github.com/chipno39/K5-L3B-DAY08-random-FromWorkflowsToAgents/commit/17c4e7b)
- Tên test đã chạy:
  - `src/tests/test_refund_idempotency.py::test_crash_before_checkpoint_then_conflict_and_valid_retry`
  - `src/tests/test_bonus_b4.py` (`6 passed`)
  - Cổng host `python -m pytest -q`: `78 passed, 248 deselected` (kiểm lại sau khi merge code 3 shape và thêm test ablation B4).
  - Ma trận student `python -m pytest -q -m student -o addopts=`: `248 passed, 78 deselected`.
- Report & run id CP4–CP5:
  - `src/reports/benchmark.json`: `workflow` (`1.000`, `28.84 VND`), `single` (`1.000`, `37.76 VND`), `orchestrator` (`1.000`, `49.19 VND`), `swarm` (`0.986`, `49.57 VND`); run id `R-workflow-C-001-s11` .. `R-swarm-C-014-s55`.
  - `src/reports/bonus_benchmark.json`, `src/reports/bonus_ablation.json` (B4 baselines + ablations + exact McNemar + 95% CI).
  - `src/reports/crash.json`: `crash_safety = 20/20`.

## 5. Công cụ AI đã dùng

- Công cụ: Trợ lý AI Antigravity (dùng trong giai đoạn đọc hiểu repo, phân tích hợp đồng durability/metrics, gợi ý thiết kế test `test_refund_idempotency.py`, tra cứu nguồn bài báo R4 `arXiv:2609.05933`, dựng Bonus B4 `src/bonus/` + `src/tests/test_bonus_b4.py`, chạy benchmark/ablation và soạn báo cáo Vị trí 4).
- Cách tự kiểm tra kết quả:
  - Trực tiếp cài môi trường `src/.venv` và chạy kiểm chứng: `python -m lab8.smoke`, `python -m lab8.demo.buggy_workflow`, `python -m pytest -q tests/test_bonus_b4.py`, `python -m pytest -q`, `python -m pytest -q -m student -o addopts=`, `python -m lab8.benchmark`, `python -m bonus.stats`, `python -m lab8.check_submission`.
  - Đối chiếu toàn bộ số liệu trực tiếp từ các file JSON sinh ra trong `src/reports/` (`benchmark.json`, `bonus_benchmark.json`, `bonus_ablation.json`, `crash.json`), không sửa tay JSON hay bịa số.

## 6. Tự đánh giá

- Đã hoàn thành đầy đủ phần việc Vị trí 4 từ CP1 đến CP5 (test durability, benchmark 4 shape, Bonus B4 với 2 baseline + 3 ablation + thống kê cluster bootstrap & exact McNemar, hoàn thiện `research_card.md`, `bonus_report.md`, phần đánh giá trong `group_report.md`, `TEAM.md` và báo cáo cá nhân).
- Nhóm đã chốt `Shape chọn: workflow` trong `report/team/decision_adr.md` (`0e03bde`), bổ sung `src/reports/hidden_selfcheck.json`, và `python -m lab8.check_submission` đạt `0 FAIL, 0 WARN`.
