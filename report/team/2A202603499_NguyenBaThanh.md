# Báo cáo cá nhân — 2A202603499_NguyenBaThanh

## 1. Thông tin thành viên

- Họ tên: Nguyễn Bá Thành
- MSSV: 2A202603499
- Email: 26ai.thanhnb3@vinuni.edu.vn
- GitHub: Nguyen-Ba-Thanh
- Lớp: L3A
- Vai trò: Single Agent (shape B).

## 2. Phần việc phụ trách

Phụ trách triển khai `src/lab8/shapes/single.py`, bổ sung test an toàn trong `src/tests/test_single_safety.py` và kiểm tra kết quả riêng của single. Cung cấp số liệu benchmark cho nhóm để đối chiếu với workflow, orchestrator và swarm.

## 3. Đóng góp cụ thể

Commit triển khai: `2dabc6f` — `Complete single agent, safety tests and contribution reports`.

- Thay stub bằng `run_react`, giới hạn tối đa 8 lượt. Controller đọc observation của tool trước để chọn hành động tiếp theo hoặc dừng sớm.
- Luồng đọc: `get_claim_view` → `lookup_order` theo principal → `compute_compensation` từ order có cấu trúc. Claim không tồn tại trả `failed`; thiếu mã đơn trả `needs_info`; không thấy đơn hoặc sai chủ sở hữu trả `escalated`.
- Dùng `worker=True` để giới hạn tool trong vòng lặp ở quyền chỉ đọc. Sau vòng lặp, code gọi `durability.issue_refund` nếu quote dương; agent không tự ghi ledger.
- Truyền quote đầy đủ cho host kiểm approval, trần theo đơn và refund lịch sử. Không tự trừ phần đã hoàn để tránh trừ hai lần. Host từ chối trả `denied`, conflict trả `failed`, thành công trả `resolved`.
- Truyền `crash_at` vào host; khi gặp `InjectedCrash`, ghi trace rồi ném lại để test quan sát crash thật. Đóng connection trong `finally`.
- Dùng `Budget` ghi nhận model/tool calls và cost; `finish()` lấy refund và audit từ ledger. Trace chỉ ghi field chọn lọc, không dump ticket, tool arguments hoặc row database thô.

Ba test bổ sung:

| Test | Mục đích |
| --- | --- |
| `test_single_unknown_claim_writes_valid_trace` | Claim không tồn tại không tạo refund và vẫn có trace hợp lệ |
| `test_single_step_cap_never_calls_refund` | Khi hết lượt, trả `failed/step_cap`, không gọi host ghi refund |
| `test_single_ticket_cannot_override_order_or_leak` | Ticket nói quá số ngày, chứa chỉ dẫn bỏ policy và canary secret không thay đổi quote hoặc làm rò secret vào trace/kết quả |

## 4. Bằng chứng

- Code: [single.py](../../src/lab8/shapes/single.py).
- Test: [test_single_safety.py](../../src/tests/test_single_safety.py).
- Report: [single_benchmark.json](../../src/reports/single_benchmark.json).

Các lệnh chạy trong `src/`, với môi trường lab. Kết quả kiểm tra lại ngày 16/11/2026:

```powershell
python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"
python -m pytest -q tests/test_single_safety.py
python -m lab8.benchmark --shapes single --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/single_benchmark.json
```

- Ma trận student riêng single: **62 passed**, gồm T11 retry và T12 crash/resume.
- Test bổ sung: **3 passed**.
- Benchmark: 14 claim × 5 seed = **70 trial**, mỗi trial dùng SQLite mới.

| Chỉ số | Kết quả |
| --- | ---: |
| Trial thành công | 70/70 |
| Pass rate | 1,000 (100%) |
| Pass^5 | 1,000 (100%) |
| Seed min–max | 1,000–1,000 |
| Seed variance | 0 |
| Cost per success, mô phỏng | 30,922857 VND |
| Tổng cost mô phỏng | 2.164,60 VND |
| Mean steps | 8,571429 |
| P95 steps | 9 |
| Mean model calls | 4,785714 |
| Mean tool calls | 2,785714 |

Số liệu lấy từ `by_shape.single` của report do công cụ tạo, không sửa JSON bằng tay. Steps là số event trace, khác số lượt ReAct; P95 steps bằng 9 không vi phạm giới hạn 8 lượt. Report không có run id tổng; ví dụ run id của trial C-001 seed 11 là `R-single-C-001-s11` theo `make_run_id`.

Đây là kết quả public riêng của single, không thay thế benchmark đủ bốn shape. Chưa có kết quả hidden trong bằng chứng cá nhân này; không suy ra đã đạt ngưỡng hidden hoặc đã tốt hơn các kiến trúc khác.

## 5. Công cụ AI đã dùng

Sử dụng ChatGPT/Codex để giải thích yêu cầu và module, hỗ trợ trực tiếp triển khai code single, viết ba test bổ sung, chạy kiểm tra và soạn báo cáo từ dữ liệu thực tế. Code và test được tạo với sự hỗ trợ của AI.

Cách kiểm chứng kết quả: đối chiếu luồng với business rules và output/trace contract; chạy ma trận student riêng single, test bổ sung và benchmark với năm seed cố định. Phát hiện `-k single` chọn thêm hàm `test_public_single_run_cases` của các shape khác, nên sửa biểu thức lọc để chỉ kiểm phần phụ trách. Không chỉnh host, config, data hay JSON để thay kết quả chấm.

## 6. Tự đánh giá

Phần single đã có bằng chứng public cho xử lý nghiệp vụ, retry, crash/resume, giới hạn vòng lặp và chống ảnh hưởng từ ticket. Việc tính tiền và ghi refund được giữ trong code/host; trace hạn chế dữ liệu đưa ra ngoài.

Giới hạn: MockLLM chỉ sinh văn bản mô phỏng, không trả tool action có cấu trúc. Controller Python ánh xạ observation thành action; lời gọi MockLLM mô phỏng lượt model và tính cost. Đây chưa phải model sản xuất tự chọn tool. Chi phí là mô phỏng và latency là `unavailable`; pass rate 100% trên public không chứng minh kết quả ngoài bộ dữ liệu này.

Cần phối hợp nhóm chạy benchmark chung trên các shape đã tích hợp để chốt ADR. Research card/ablation, crash suite chung và hidden evaluation cần hoàn tất theo phân công; báo cáo cá nhân này không nhận các phần đó là đóng góp đã thực hiện.
