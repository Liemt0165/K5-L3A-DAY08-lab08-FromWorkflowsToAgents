# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: Lab08 - From Workflows To Agents
- Thành viên: Nguyễn Thanh Liêm, Ngô Gia Bảo, Nguyễn Bá Thành, Chử Văn Hiệp (chi tiết xem `TEAM.md`)
- Commit chốt: `88ff12d` (đồng bộ với `report/team/decision_adr.md`)

## 2. Mục tiêu và cách tiếp cận

Bài toán yêu cầu xử lý khiếu nại (claim) hoàn cước và đền bù hư hỏng hàng hóa cho công ty logistics giả lập LogiX-VN. Nhóm triển khai cùng một quy trình nghiệp vụ trên bốn kiến trúc với mức độ tự chủ tăng dần:
1. **Shape A — Deterministic Workflow:** Chuỗi các hàm Python tuyến tính, cố định, không ReAct loop.
2. **Shape B — Single ReAct Agent:** Một agent tự chủ với vòng lặp ReAct tối đa 8 bước, chỉ dùng công cụ đọc.
3. **Shape C — Orchestrator + Investigators:** Mô hình phân cấp với orchestrator điều phối và các investigator nhận gói handoff an toàn (allowlist).
4. **Shape D — Swarm Reference:** Bầy agent giao tiếp ngang hàng (đối chứng).

Nhóm tiếp cận theo phương châm chia module độc lập: 3 thành viên phụ trách 3 shape, 1 thành viên phụ trách kiểm soát ledger, crash recovery và đánh giá benchmark. Mục tiêu cuối cùng là chọn ra kiến trúc "ít tự chủ nhất mà vẫn làm được việc".

## 3. Giải pháp

### Ranh giới Code vs. Model
- **Code (Host):** Độc quyền tính toán số tiền bồi thường (`compute_compensation`), kiểm tra quyền sở hữu đơn hàng (`principal_id`), xác thực approval và thực thi giao dịch ghi vào SQLite ledger có khóa chống ghi trùng (idempotency key `refund:{claim_id}`).
- **Model (MockLLM):** Chỉ được dùng để đọc hiểu ngữ nghĩa và soạn văn bản trả lời khách hàng (`draft`). Model tuyệt đối không được quyết định số tiền hay tự ghi cơ sở dữ liệu.
- **Dữ liệu ticket:** Coi là dữ liệu người dùng không tin cậy (untrusted user input). Mọi số ngày trễ ghi trong ticket (kể cả "trễ 9 ngày" hay "ignore policy") đều bị bỏ qua; số ngày trễ duy nhất được dùng là `delay_days` trong bảng `orders`.

### Cơ chế Handoff & Trace an toàn
- **Handoff (Shape C):** Sử dụng `lab8.handoff.make_handoff`, chỉ chuyển giao các trường trong allowlist (`FACT_KEYS`) và 3 công cụ đọc. Loại bỏ hoàn toàn `parent_secret`, headers kỹ thuật và ngữ cảnh thô của cha.
- **Trace Contract v1:** Trace chỉ ghi lại các sự kiện đã qua chuẩn hóa (`_order_trace`, `quote`, `decision`). Không bao giờ ghi `CANARY_SECRET_*`, SĐT, địa chỉ, tên hàng hay ghi chú nội bộ ra log hoặc stdout.

### Xử lý Sự cố & Độ bền (Durability)
- Cả 3 shape đều truyền tham số `crash_at` vào `durability.issue_refund`. Khi gặp `InjectedCrash`, shape kịp thời flush trace xuống đĩa rồi re-raise ngoại lệ ra ngoài, đảm bảo bài kiểm tra crash xác nhận được khả năng phục hồi sau sự cố mà không thanh toán đúp.

## 4. Kết quả

### Bảng đối chuẩn Benchmark 4 Shape (70 trials = 14 claims x 5 seeds)

| Tiêu chí | Shape A (`workflow`) | Shape B (`single`) | Shape C (`orchestrator`) | Shape D (`swarm`) |
| :--- | :---: | :---: | :---: | :---: |
| **Pass rate** | **1.0 (70/70)** | **1.0 (70/70)** | **1.0 (70/70)** | 0.9857 (69/70) |
| **Pass^5 (Pass cả 5 seed)** | **1.0** | **1.0** | **1.0** | 0.9286 |
| **Seed min – max** | 1.0 – 1.0 | 1.0 – 1.0 | 1.0 – 1.0 | 0.9286 – 1.0 |
| **Seed variance (Phương sai)** | **0.0** | **0.0** | **0.0** | 0.000816 |
| **Chi phí/thành công (VND)** | **28.87 VND** | 30.92 VND | 47.90 VND | 49.57 VND |
| **Mean / P95 steps** | 5.43 / 6.0 | 8.57 / 9.0 | 3.71 / 4.0 | 2.80 / 3.0 |
| **Model calls TB** | 1.0 | 4.79 | 2.0 | 3.0 |

### Kết quả Crash Safety
- Lệnh chạy: `python -m lab8.crash_test --count 20 --out reports/crash.json`
- Kết quả: **`crash_safety: 20/20`** (Đạt tuyệt đối 20/20 kịch bản crash tại các mốc `before_effect`, `after_effect_before_checkpoint`, và `after_checkpoint`).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

1. **Rào cản môi trường Sandbox & Network:** Khi cài đặt ban đầu, pip không thể tải thư viện trong sandbox không mạng. Nhóm đã giải quyết bằng cách cài đặt thư viện ngoài sandbox (`BypassSandbox: true`), sau đó chạy toàn bộ code, test và benchmark trong Standard Sandbox Mode.
2. **Xử lý trần theo đơn (Per-order cap - T13, T14):** Ban đầu có sự nhầm lẫn giữa việc agent tự trừ tiền đã nhận trước đó hay host trừ. Nhóm đã thống nhất quy tắc: Agent luôn truyền quote đầy đủ, Host chịu trách nhiệm trừ phần đã hoàn trước đó trong cùng transaction ghi.
3. **Giải quyết xung đột Git:** Do nhiều thành viên cùng cập nhật `TEAM.md`, nhóm đã thiết lập cấu trúc bảng 4 dòng độc lập và hướng dẫn thành viên giải quyết xung đột qua git merge/resolve conflicts an toàn.

## 6. Tổng hợp đóng góp

- **Nguyễn Thanh Liêm (`ledger + evaluation`):** Quản lý tính toàn vẹn dữ liệu SQLite ledger, cơ chế khóa idempotency, chạy crash test (20/20), chạy benchmark đối chuẩn 4 shape, viết test `test_client_retry_after_unacknowledged_checkpoint_crash_replays_safely` (commit `64d4366`), phụ trách `decision_adr.md` và `research_card.md`.
- **Ngô Gia Bảo (`workflow + integration`):** Hiện thực Shape A `workflow.py`, viết test `test_workflow_shape.py` (commit `34f980f`) và test tích hợp `test_integration_shapes.py` (commit `4c32df3`).
- **Nguyễn Bá Thành (`single`):** Hiện thực Shape B `single.py` với vòng lặp ReAct giới hạn 8 bước, viết test an toàn `test_single_safety.py` (commit `2dabc6f`).
- **Chử Văn Hiệp (`orchestrator + handoff`):** Hiện thực Shape C `orchestrator.py`, thiết lập cơ chế chuyển giao allowlist an toàn, viết test cách ly `test_handoff_secret.py` (commit `a204a46`).

## 7. Hạn chế và hướng cải thiện

- MockLLM là mô hình giả lập theo seed, không phản ánh trễ mạng và tính bất định tự nhiên của các mô hình thương mại lớn (như GPT-4 hay Gemini).
- Hệ thống hiện tại tối ưu cho dữ liệu cấu trúc rõ ràng. Nếu mở rộng ra khiếu nại bằng ngôn ngữ tự nhiên phức tạp cần đàm phán, kiến trúc Orchestrator hoặc Single ReAct có thể sẽ phát huy ưu thế linh hoạt hơn.

## 8. Tài liệu tham khảo

- MAST, NeurIPS 2025: *Why Do Multi-Agent LLM Systems Fail?*
- Google Research 2026: *Towards a Science of Scaling Agent Systems*.
- Tài liệu nội bộ Lab 8: `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`.
