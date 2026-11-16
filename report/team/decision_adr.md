# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 3316bef
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.0 / 1.0 / 1.0-1.0 / 0.0
- Cost per success (mô phỏng, VND): 28.87
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

## Vì sao shape này

Nhóm chọn Shape A (workflow) vì đây là kiến trúc đạt hiệu quả tuyệt đối theo nguyên lý "ít tự chủ nhất mà vẫn làm được việc". Khi đối chuẩn trên cùng bộ dữ liệu 14 public claims x 5 seeds (70 trials), workflow đạt tỷ lệ thành công pass rate 1.0, pass^5 đạt 1.0, độ dao động giữa các seed (seed_variance) bằng 0.0 tuyệt đối và chi phí mô phỏng rẻ nhất toàn hệ thống (28.87 VND/success so với 30.92 VND của single, 47.90 VND của orchestrator và 49.57 VND của swarm). Toàn bộ nghiệp vụ bồi thường của LogiX-VN có luật lệ rõ ràng, việc tính tiền thuộc thẩm quyền của code (`compute_compensation`), nên quy trình tuần tự tất định loại bỏ hoàn toàn rủi ro ảo giác hoặc bị thao túng bởi câu chữ trong ticket của khách hàng.

## Có đáng đổi lấy multi-agent không

Hoàn toàn không đáng. Trong bài toán này, việc tăng mức độ tự chủ từ workflow lên single agent, orchestrator hay swarm không hề làm tăng pass rate (workflow đã đạt 1.0), trong khi chi phí tăng từ 1.07 lần (single) đến 1.72 lần (swarm). Đặc biệt ở swarm, sự tự do của các peer agent dẫn đến phương sai seed_variance = 0.000816 và thất bại ở seed 55 tại test T02 do tin vào số ngày trễ ghi trong ticket thay vì dữ liệu sự thật trong cơ sở dữ liệu. Quyết định chỉ đảo lại khi bài toán xuất hiện các yếu tố mở: ticket chứa khiếu nại bằng văn bản tự do phi cấu trúc cần nhiều vòng đàm phán, chính sách thay đổi liên tục chưa kịp mã hóa thành code, hoặc bài toán đòi hỏi tổng hợp đa góc nhìn mà mã nguồn tuyến tính không thể bao quát hết các nhánh ngoại lệ.

## Trace và bằng chứng

Trace của nhóm ghi nhận các sự kiện: bắt đầu xử lý, kiểm tra mã đơn (`has_order_id`), tra cứu đơn hàng qua allowlist (`lookup_order`), tính toán báo giá (`quote`), gọi cổng duyệt (`issue_refund`), sự kiện crash giả lập và quyết định cuối cùng (`decision`). Nhóm cố ý KHÔNG ghi nhận các trường nhạy cảm: `parent_secret`, `ticket` thô, `raw_headers_json`, `phone`, `internal_note`, `item_name` và `principal_id` của khách hàng. Nếu trace nằm ngoài vùng scanner quét được, đội security có thể chứng minh tính an toàn bằng cách đối chiếu log với danh mục trường allowlist trong `lab8.tools`, kiểm tra bộ lọc `safe_dump` và xác nhận không có bất kỳ chuỗi `CANARY_SECRET_*` hay thông tin PII nào xuất hiện trong các bản ghi SQLite và stdout.

## Giới hạn

MockLLM là mô hình giả lập cố định theo seed, không phản ánh đầy đủ độ trễ mạng thực tế, độ trôi ngữ cảnh (context drift) hay chi phí API thương mại thực của các mô hình sản xuất (production LLMs). Chỉ số latency được ghi nhận là `unavailable`. Nhóm cam kết số liệu hoàn toàn trích xuất tự động từ `reports/benchmark.json` và `reports/crash.json`, không chỉnh sửa thủ công để đạt ngưỡng.
