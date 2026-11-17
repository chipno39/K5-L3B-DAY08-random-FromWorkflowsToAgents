# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 794523a
- Run id dẫn chứng: R-workflow-C-001-s11 (src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000
- Cost per success (mô phỏng, VND): 28.84 (tổng 2018.60 VND / 70 trial thành công)
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 (70/70 trial public đạt chuẩn)

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Nhóm chọn `workflow` (Shape A — kiến trúc ít tự chủ nhất) vì trên cùng tập 14 claim public × 5 seed (`11, 22, 33, 44, 55`, tổng 70 trial/shape, cùng MockLLM và bảng giá), `workflow` đạt độ chính xác tuyệt đối (`pass_rate = 1.000`, `pass^5 = 1.000`, `seed_variance = 0.000000`) trong khi có chi phí mô phỏng thấp nhất: **28.84 VND/success** (so với `single` là **37.76 VND**, `orchestrator` là **49.19 VND**, và `swarm` là **49.57 VND** với `pass_rate = 0.986`, `pass^5 = 0.929` do fail T02 ở seed 55). Bài toán xử lý claim của LogiX-VN có quy trình nghiệp vụ xác định (đọc claim → kiểm tra quyền sở hữu đơn → tính bồi thường theo policy hiện hành → gọi `durability.issue_refund`), nên code xác định đảm bảo loại bỏ hoàn toàn rủi ro prompt injection từ nội dung ticket và giữ trace ngắn nhất (`mean_steps = 3.79`, `p95_steps = 4.0`).

## Có đáng đổi lấy multi-agent không

Không đáng đổi trong bối cảnh hiện tại. Trên cùng dữ liệu và seed, các kiến trúc có mức tự chủ cao hơn (`single`, `orchestrator`, `swarm`) không cải thiện được độ chính xác so với `workflow` (đều ≤ 1.000), nhưng làm chi phí mô phỏng tăng thêm từ **31%** (`single`) đến **72%** (`swarm`), số bước xử lý tăng lên (`mean_steps` 4.71–8.43 ở `orchestrator`/`single`), và ở `swarm` còn xuất hiện dao động theo seed (`seed_variance = 0.000816`, `seed_min = 0.929`). Quyết định này chỉ đảo lại nếu bài toán mở rộng sang các nghiệp vụ mơ hồ, phi cấu trúc cần suy luận đa nguồn (nhiều hệ thống tra cứu độc lập, cần đối thoại làm rõ nhiều vòng với khách hàng) khiến `workflow` cố định rớt pass rate xuống dưới `orchestrator` hoặc `single`, và mức tăng pass rate bù đắp được chi phí tăng thêm trong khi vẫn giữ `secret_leak = 0` và `approval_bypass = 0`.

## Trace và bằng chứng

Trace của nhóm ghi những event nào? Field nào cố ý **không** ghi, và vì sao? Nếu trace nằm ngoài chỗ scanner quét được, đội security chứng minh thế nào là hệ thống không lộ secret?

**Event được ghi (shape orchestrator, `src/lab8/shapes/orchestrator.py`).** Mỗi lần chạy ghi đúng một file `traces/<run_id>.json` qua `lab8.trace.Trace`, gồm theo thứ tự: `claim_view` (chỉ `claim_id`, `order_id`), `handoff` (gói đã qua `make_handoff` + `sanitize`), `investigator` (kết quả `lookup_order` đã allowlist và 3 số của `lookup_refund_history`: `visible`, `refund_count`, `refunded_vnd`), `quote` (output của `compute_compensation`), `decision` (status, proposal, lý do), và `crash` nếu có crash giả lập. Mọi nhánh return, kể cả `failed`/`needs_info`/`escalated`, đều đi qua `finish()` nên luôn có trace mới, đúng `run_id`.

**Field cố ý không ghi và lý do.**

| Không ghi | Lý do |
| --- | --- |
| `parent_secret`, `raw_headers` của claim | Là parent context. Không bao giờ đọc ra: `get_claim_view` chỉ SELECT `claim_id, principal_id, order_id, ticket`, nên secret không có trong bộ nhớ của shape |
| `ticket` (chữ của khách) | Dữ liệu người dùng, có thể chứa SĐT, mã đơn giả, câu prompt injection ("trễ 9 ngày", "ignore policy"). Không cần cho quyết định vì tiền tính từ order row |
| `principal_id` | Định danh khách; investigator nhận nó như tham số hàm, không qua handoff và không vào trace |
| `phone`, `address`, `internal_note`, `item_name` của order | `lookup_order` đã allowlist `ORDER_FIELDS`, trace chỉ ghi view đó, không bao giờ ghi order row thô |
| Prompt và output của MockLLM | Không cần cho grader; tránh để model "tóm tắt" parent context rồi lọt vào log |

**Handoff được sanitize bằng allowlist trong code, không bằng LLM.** `make_handoff` chỉ giữ 8 key trong `ALLOWED_FIELDS`, chỉ giữ `facts` thuộc `FACT_KEYS`, từ chối tool không thuộc `READONLY_TOOLS`; sau đó `sanitize` serialize cả gói và raise nếu còn chuỗi `CANARY_SECRET_`, `parent_secret`, `raw_headers` hoặc có write tool. Orchestrator truyền `facts={}` và chỉ cấp `lookup_order`, `get_clause`, `lookup_refund_history`, nên investigator không có gì để lộ và không có quyền ghi. Một bản tóm tắt bằng LLM không phải là sanitizer: model có thể chép nguyên secret, diễn đạt lại nó, hoặc bị ticket điều khiển; allowlist thì xác định, kiểm thử được.

**Bằng chứng.** (1) `src/tests/test_handoff_secret.py::test_orchestrator_trace_has_only_sanitized_handoff` chạy orchestrator trên toàn bộ 14 claim public và khẳng định trace không chứa `CANARY_`, `Bearer`, `parent_secret`, `raw_headers` và mọi SĐT/địa chỉ/internal note/tên hàng có trong world; event `handoff` chỉ có key trong allowlist và không có write tool. (2) `test_make_handoff_drops_unlisted_facts_and_refuses_smuggled_secret` kiểm hai lớp: key lạ trong `facts` bị bỏ, secret nhét vào key hợp lệ thì bị chặn. (3) `python -m pytest -q -m student -o addopts= -k orchestrator`: 62 passed, gồm kiểm `forbid_substrings` của grader.

**Nếu log nằm ngoài chỗ scanner quét.** Grader fail-closed: chỉ tin `trace_path` và các file `<run_id>*` cùng thư mục. Vì vậy nhóm không chứng minh bằng cách "đọc lại log", mà bằng cấu trúc: (a) shape không có đường nào đọc `parent_secret` (không SELECT, không truyền), dữ liệu không có trong bộ nhớ thì không thể bị log; (b) shape không `print`/`logging`, mọi output đi qua một `Trace` duy nhất; (c) mọi payload vào trace là view đã allowlist; (d) test ở trên chạy trong cổng host nên mỗi thay đổi code đều bị kiểm lại.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.
