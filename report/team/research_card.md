# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R4 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Zhang et al., "Rethinking the Evaluation of Efficiency Methods for Multi-Agent Systems", EMNLP 2026, https://arxiv.org/abs/2609.05933 — controlled diagnostic benchmark / empirical evaluation paper. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Đánh giá hiện nay hay phóng đại khả năng tiết kiệm chi phí của các phương pháp tối ưu MAS vì lợi ích đo được phụ thuộc vào prompt/topology riêng hoặc chạy trên task non-MAS-demanding nơi single-agent hay cắt tỉa ngẫu nhiên vẫn giữ nguyên độ chính xác; nhiều "efficiency gain" thực chất đến từ sụp đổ cấu trúc hoặc vô hiệu hóa tool pathway. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Benchmark chẩn đoán có kiểm soát của paper (dùng chung backbone model, agent registry và runtime khi thay đổi topology, quy mô, độ sâu và tool use). Áp sang LogiX-VN dùng MockLLM (latency=unavailable), corpus nhỏ (14 public claim) và host policy/ledger giữ quyền quyết định tiền. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Khi giữ cố định MockLLM, bộ tool đọc, seed (11,22,33,44,55), corpus và pricing.yaml, việc bỏ 1 thành phần phụ không tham gia tính tiền trên shape nhóm chọn (như draft model call hoặc read tool dư như lookup_policy thừa) sẽ giữ nguyên pass_rate và pass^5 (McNemar exact p = 1.0, seed_variance không đổi), trong khi giảm cost_per_success_vnd (~1 VND/trial nếu bỏ draft, ~10 VND/trial nếu bỏ 1 tool call dư) và giảm mean_steps nếu bước đó có ghi trace. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, claim corpus (data/public_claims.jsonl), seeds [11,22,33,44,55], giá (config/pricing.yaml), evaluator (lab8.metrics) và host gate (policy.py, durability.py). Biến duy nhất thay đổi: tắt/bỏ 1 thành phần phụ không tham gia compute_compensation trên biến thể của shape nhóm chọn (chạy qua --extra-shape của lab8.benchmark ở CP5, kèm đối chứng rules-only và swarm_pruned trong B4). |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: ngộ nhận mức tiết kiệm nhỏ từ bỏ thành phần phụ là ưu thế topology, hoặc cắt quá tay (over-pruning) làm rơi bước kiểm điều kiện hay bỏ gọi durability.issue_refund (thiếu audit policy_denied). Cách đo: pass_rate kèm bootstrap 95% CI resample theo claim, pass^5, seed_variance, cost_per_success_vnd, mean_steps, exact McNemar trên cặp (claim_id, seed), approval_bypass (= 0), secret_leak (= 0). |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm,rules_only,swarm_pruned,workflow_no_draft,workflow_trust_ticket,orchestrator_min_tools --extra-shape rules_only=bonus.rules_only:run_claim --extra-shape swarm_pruned=bonus.swarm_pruned:run_claim --extra-shape workflow_no_draft=bonus.ablations:run_workflow_no_draft --extra-shape workflow_trust_ticket=bonus.ablations:run_workflow_trust_ticket --extra-shape orchestrator_min_tools=bonus.ablations:run_orchestrator_min_tools --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json` và `python -m bonus.stats --in reports/bonus_benchmark.json --baseline workflow --pair rules_only:swarm --pair swarm_pruned:swarm --pair workflow_no_draft:workflow --pair workflow_trust_ticket:workflow --pair orchestrator_min_tools:orchestrator --pair single:workflow --pair orchestrator:workflow --pair swarm:workflow --out reports/bonus_ablation.json`
- Run id / file report: `src/reports/bonus_benchmark.json`, `src/reports/bonus_ablation.json` (`R-workflow-C-001-s11` .. `R-orchestrator_min_tools-C-014-s55`; tham chiếu chính `src/reports/benchmark.json`).
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng với hypothesis về chi phí và bảo toàn độ chính xác khi bỏ thành phần phụ:** trên 70 trial (14 claim × 5 seed), `workflow_no_draft` bỏ `draft` model call giữ nguyên `pass_rate = 1.000` (`95% CI [1.000, 1.000]`), `pass^5 = 1.000`, `seed_variance = 0.0` (McNemar exact `p = 1.0000`) và giảm `cost_per_success_vnd` từ `28.84` xuống `27.86 VND` (`-0.98 VND/trial`); `orchestrator_min_tools` bỏ `lookup_refund_history` và `get_clause` dư giữ nguyên `pass_rate = 1.000` (`p = 1.0000`) và giảm `cost_per_success_vnd` từ `49.19` xuống `39.64 VND` (`-9.55 VND/trial`). Ngược lại, bỏ bước kiểm dữ liệu (`workflow_trust_ticket`) làm `pass_rate` tụt xuống `0.929` (`95% CI [0.786, 1.000]`, `b=0, c=5`, `p = 0.0625`) và tăng `cost/success` lên `31.06 VND`. Riêng `rules_only` (`1.000`, `27.86 VND`) vs `swarm` (`0.986`, `49.57 VND`) chỉ lệch 1 trial (`b=1, c=0`, `p = 1.0000`) nên chưa kết luận khác biệt `pass_rate` có ý nghĩa thống kê trên 14 claim dù chênh chi phí rất rõ.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
