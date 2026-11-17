# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn:
- Commit hash:
- Run id dẫn chứng:
- Pass rate / pass^5 / seed min-max / variance:
- Cost per success (mô phỏng, VND):
- Crash safety:
- Secret leak / approval bypass:

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Viết 1 đoạn. So cùng corpus và cùng seed, không so với một lần chạy lẻ.

## Có đáng đổi lấy multi-agent không

Nếu pass nhỉnh hơn mà cost và variance cao hơn, nhóm giữ hay đổi? Điều kiện nào làm quyết định đảo lại?

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
