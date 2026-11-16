# Team

## Thông tin nhóm

- Tên nhóm: lab08
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-lab08-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/Liemt0165/K5-L3A-DAY08-lab08-FromWorkflowsToAgents, nhánh main

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Thanh Liêm | 2A202603358 | 26ai.liemnt@vinuni.edu.vn | Liemt0165 | ledger + evaluation |
|   2 | Nguyễn Bá Thành | 2A202603499 | 26ai.thanhnb3@vinuni.edu.vn | Nguyen-Ba-Thanh | single |
|   3 | Ngô Gia Bảo | 2A202603482 | 26ai.baong4@vinuni.edu.vn | b-baocode | workflow + integration |
|   4 | Chử Văn Hiệp | 2A202603382 | 26ai.hiepcv@vinuni.edu.vn | hiepchuvanpc | orchestrator + handoff |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Thanh Liêm | ledger + evaluation | 64d4366 | test_client_retry_after_unacknowledged_checkpoint_crash_replays_safely |
| Ngô Gia Bảo | workflow + integration | 34f980f, 4c32df3 | test_workflow_shape.py, test_integration_shapes.py |
| Nguyễn Bá Thành | single | 2dabc6f | test_single_safety.py |
| Chử Văn Hiệp | orchestrator + handoff | a204a46 | test_orchestrator_handoff_blocks_write_tools_and_unauthorized_facts |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-