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
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | MAST, NeurIPS 2025, Why Do Multi-Agent LLM Systems Fail? (https://arxiv.org/abs/2503.13657), benchmark & empirical taxonomy |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Hệ thống Multi-Agent dễ gặp lỗi kiểm chứng thông tin (verification failure) và lệch mục tiêu khi tiếp nhận dữ liệu không đáng tin cậy |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Task xử lý bồi thường LogiX-VN với dữ liệu ticket người dùng có thể chứa thông tin mâu thuẫn cơ sở dữ liệu, chạy trên MockLLM qua 5 seed |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu chuyển từ Workflow (Shape A) sang Swarm (Shape D), pass rate sẽ giảm từ 1.0 xuống dưới 1.0 và phương sai seed_variance tăng từ 0.0 lên > 0 do lỗi kiểm chứng tại seed 55 |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, bộ công cụ allowlist, 14 public claims, seed [11, 22, 33, 44, 55], bảng giá mô phỏng, evaluator. Biến duy nhất thay đổi: Topology kiến trúc (Deterministic Workflow vs Decentralized Swarm) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi kiểm chứng thông tin ticket (verification failure). Đo bằng: pass rate (Workflow 1.0 vs Swarm 0.9857), pass^5 (Workflow 1.0 vs Swarm 0.9286), seed_variance (0.0 vs 0.000816), cost_per_success (28.87 vs 49.57 VND) |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `R-workflow-C-001-s11` / `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Giả thuyết hoàn toàn ĐÚNG. Swarm thất bại đúng tại seed 55 trên claim C-002 (T02) với lỗi `status resolved != denied: 1` do tin vào số ngày trễ ghi trong ticket (5 ngày) thay vì cột delay_days trong database (2 ngày). Sự cố này làm pass rate của Swarm tụt xuống 0.9857, pass^5 tụt xuống 0.9286, phương sai tăng lên 0.000816 và chi phí tăng lên 49.57 VND/success, trong khi Workflow đạt 1.0 pass rate, 0.0 phương sai và chi phí tối ưu 28.87 VND/success.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.
