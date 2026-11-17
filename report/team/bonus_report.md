# Bonus report: B4

- Topic + nguồn (tên, năm, slide): B4 · Baseline công bằng, ablation và thống kê — Zhang et al., *Rethinking the Evaluation of Efficiency Methods for Multi-Agent Systems*, EMNLP 2026 (`https://arxiv.org/abs/2609.05933`, slide 44, 59); AgentArch, AACL 2026 (slide 43).
- CLAIM của nguồn (1 câu): Các kết luận "efficiency gain" trong MAS có thể đến từ cách thiết kế benchmark, prompt/topology khởi đầu hoặc tác vụ non-MAS-demanding nơi baseline đơn giản hoặc random pruning vẫn giữ nguyên độ chính xác; cần so với baseline đơn giản trong cùng runtime, model và tool.
- BOUNDARY (model / dataset / setup): Benchmark chẩn đoán có kiểm soát của paper (chung backbone model, agent registry, runtime). Trên LogiX-VN bị giới hạn thêm bởi `MockLLM` (`latency = unavailable`), 14 public claim và host code (`policy.py`, `durability.py`) chịu trách nhiệm kiểm duyệt và ghi tiền.
- HYPOTHESIS trên LogiX: (1) `rules_only` (`model_calls = 0`) đạt `pass_rate` tương đương hoặc cao hơn `swarm` (McNemar exact `p = 1.0` trên 14 claim vì chỉ khác ở `C-002` seed `55`) nhưng giảm mạnh `cost_per_success_vnd` nhờ bỏ chi phí tool/investigator trung gian; (2) `swarm_pruned` giữ gần nguyên `pass_rate` của `swarm` với chi phí giảm nhẹ ở phần model call; (3) trên shape nhóm chọn, việc bỏ thành phần phụ không tham gia `compute_compensation` (như `draft` call hoặc tool dư) giữ nguyên `pass_rate` và chỉ giảm `cost_per_success_vnd`.
- Biến cố định / biến thay đổi: Cố định `MockLLM`, `data/public_claims.jsonl`, `seeds=[11,22,33,44,55]`, `config/pricing.yaml`, `lab8.metrics`, `durability.py`, `policy.py`; chỉ thay đổi topology/thành phần bị cắt (`rules_only`, `swarm_pruned`, và các biến thể ablation của shape nhóm chọn qua `--extra-shape`).
- Lệnh tái lập (copy-paste được):
  `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm,rules_only,swarm_pruned,workflow_no_draft,workflow_trust_ticket,orchestrator_min_tools --extra-shape rules_only=bonus.rules_only:run_claim --extra-shape swarm_pruned=bonus.swarm_pruned:run_claim --extra-shape workflow_no_draft=bonus.ablations:run_workflow_no_draft --extra-shape workflow_trust_ticket=bonus.ablations:run_workflow_trust_ticket --extra-shape orchestrator_min_tools=bonus.ablations:run_orchestrator_min_tools --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json && python -m bonus.stats --in reports/bonus_benchmark.json --baseline workflow --pair rules_only:swarm --pair swarm_pruned:swarm --pair workflow_no_draft:workflow --pair workflow_trust_ticket:workflow --pair orchestrator_min_tools:orchestrator --pair single:workflow --pair orchestrator:workflow --pair swarm:workflow --out reports/bonus_ablation.json`
- File report + run id: `src/reports/bonus_benchmark.json`, `src/reports/bonus_ablation.json` (`R-workflow-C-001-s11` .. `R-orchestrator_min_tools-C-014-s55`; đối chiếu 4 shape chính tại `src/reports/benchmark.json`).

## Kết quả

Số liệu trích trực tiếp từ `src/reports/bonus_ablation.json` (14 claim × 5 seed = 70 trial/biến thể, bootstrap `n=2000` theo `claim_id`, `rng_seed=42`). Các biến thể đều đạt `0` leak/bypass trên các trial pass kiểm tra của `trial_pass` (chưa ghi nhận lỗi bảo mật nào trong `reasons`):

| Biến thể | Pass rate | 95% CI (claim) | Pass^5 | Variance | Cost/success (VND) | Mean steps |
|---|---:|---:|---:|---:|---:|---:|
| `workflow` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 28.84 | 3.79 |
| `single` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 37.76 | 8.43 |
| `orchestrator` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 49.19 | 4.71 |
| `swarm` | 0.986 | [0.957, 1.000] | 0.929 | 0.000816 | 49.57 | 2.80 |
| `rules_only` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 27.86 | 3.79 |
| `swarm_pruned` | 0.986 | [0.957, 1.000] | 0.929 | 0.000816 | 48.90 | 3.80 |
| `workflow_no_draft` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 27.86 | 3.79 |
| `workflow_trust_ticket` | 0.929 | [0.786, 1.000] | 0.929 | 0.000000 | 31.06 | 3.86 |
| `orchestrator_min_tools` | 1.000 | [1.000, 1.000] | 1.000 | 0.000000 | 39.64 | 4.71 |

Kiểm định cặp Exact McNemar (70 cặp): `rules_only vs swarm` (`b=1, c=0, p=1.0000`), `swarm_pruned vs swarm` (`b=0, c=0, p=1.0000`), `workflow_no_draft vs workflow` (`b=0, c=0, p=1.0000`), `orchestrator_min_tools vs orchestrator` (`b=0, c=0, p=1.0000`), `workflow_trust_ticket vs workflow` (`b=0, c=5, p=0.0625`).

## Kết luận

- **Ủng hộ mạnh phần chi phí của hypothesis R4/B4; phần `pass_rate` giữa các cấu hình sạch chưa khác biệt có ý nghĩa thống kê trên 14 claim:** `workflow` (`28.84 VND`), `single` (`37.76 VND`), `orchestrator` (`49.19 VND`) và `rules_only` (`27.86 VND`) đều đạt `1.000`. Việc bỏ `draft` model call (`workflow_no_draft`) và bỏ tool điều tra dư (`orchestrator_min_tools`) không làm rơi 1 trial nào (`p = 1.0000`) mà giảm chi phí lần lượt `-0.98 VND` và `-9.55 VND/trial`. Trong khi đó, cắt bước kiểm tra ticket (`workflow_trust_ticket`) làm giảm `pass_rate` xuống `0.929` (`b=0, c=5`, `p = 0.0625`) và đội chi phí/ca thành công lên `31.06 VND`.
- **Điều kiện đảo ngược kết luận:** Nếu chuyển sang LLM thật trên dữ liệu ngôn ngữ tự nhiên phức tạp (không cấu trúc sẵn như `public_claims.jsonl`) hoặc tính phí token output đắt hơn nhiều so với `tool_call_vnd`, lợi thế chi phí và độ chính xác giữa `workflow` và `single`/`orchestrator` có thể đảo ngược; tương tự, trên tập lớn hơn (`>= 40` claim hidden) khi có `>= 6` cặp bất đồng nghiêng về một phía, phép McNemar sẽ đạt ngưỡng `p < 0.05`.

## Giới hạn

- `MockLLM` không phải model sản xuất; `latency` là `unavailable`.
- Tập public chỉ có 14 claim (70 trial ở 5 seed): kiểm định exact McNemar có lực thấp khi số cặp bất đồng nhỏ (`b + c` nhỏ), dễ dẫn tới `p = 1.0` ngay cả khi có chênh lệch 1 trial.
- Đơn giá mô phỏng trong `config/pricing.yaml` đặt `tool_call_vnd = 10` (cao hơn ~10 lần so với 1 lượt `MockLLM` ngắn), nên chi phí nhạy mạnh với số tool call dư hơn là token văn bản.
