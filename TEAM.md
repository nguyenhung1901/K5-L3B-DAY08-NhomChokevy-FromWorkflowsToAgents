# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-NhomChokeVy-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Lê Chuẩn | 2A202603784 | lechuanpham@gmail.com | 22520160 | workflow + integration |
|   2 | Nguyễn Tài Minh Huy | 2A202603828 | nguyentaiminhhuy831975@gmail.com | ArthurTheMana | orchestrator + handoff + student_drop |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phạm Lê Chuẩn | workflow + integration | Chưa commit (working tree) | `src/tests/test_workflow_integration.py` (3 passed); student workflow matrix (62 passed) |
| Nguyễn Tài Minh Huy | Orchestrator + Handoff | `648fb4e` + `5457779` + `a6dc301` + `ed40d2c` | `src/tests/test_orchestrator_handoff.py` (3 passed); toàn bộ repository (309 passed); Student Drop (200 lượt Orchestrator, 0 lỗi thực thi, 0 nghi ngờ rò rỉ dữ liệu) |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603784_PhamLeChuan

### Phần việc phụ trách

- Phụ trách shape `workflow` và tích hợp với host: đọc claim/order qua tool, tính quote theo order, gọi `durability.issue_refund` bằng quote đầy đủ, và trả kết quả/trace theo contract chung.

### Đóng góp cụ thể

- Triển khai workflow xác định: đọc claim và order qua tool, lấy quote từ dữ liệu order, chuyển nguyên quote cho host ghi refund, và hoàn tất các nhánh status bằng helper dùng chung.

### Bằng chứng

- Mã: `src/lab8/shapes/workflow.py`; test riêng `src/tests/test_workflow_integration.py`: 3 passed; `python -m pytest -q -m student -o addopts= -k workflow`: 62 passed; `python -m lab8.smoke`: pass.

## 2A202603828_NguyenTaiMinhHuy

### Phần việc phụ trách

Phụ trách Shape C `orchestrator` và tích hợp cơ chế Handoff để điều phối các Investigator chỉ có quyền đọc dữ liệu. Đảm bảo việc bàn giao dữ liệu được kiểm soát, không chuyển thông tin nhạy cảm và không cho phép Investigator trực tiếp thực hiện hoàn tiền.

### Đóng góp cụ thể

Triển khai `src/lab8/shapes/orchestrator.py`, sử dụng `lab8.handoff.make_handoff` để tạo gói bàn giao an toàn cho Investigator.

Giới hạn công cụ được phép sử dụng, kiểm tra thông tin đơn hàng, lịch sử hoàn tiền và số tiền bồi thường. Quyền thực hiện hoàn tiền vẫn thuộc Host sau khi kiểm tra các điều kiện nghiệp vụ.

Bổ sung `src/tests/test_orchestrator_handoff.py` gồm 3 bài kiểm thử bảo mật, kiểm tra lọc dữ liệu nhạy cảm, từ chối công cụ ghi dữ liệu và phát hiện secret trong Handoff.

Sử dụng module `handoff.py` có sẵn của giảng viên, không thay đổi mã nguồn gốc của module này.

### Bằng chứng

**1. Mã nguồn và Commit**

Mã nguồn: `src/lab8/shapes/orchestrator.py`

Custom Test: `src/tests/test_orchestrator_handoff.py`

Commit: `a6dc301`

GitHub: https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/commit/a6dc301

**2. Public Tests và Custom Tests**

Custom Handoff Tests: 3/3 passed.

Toàn bộ repository nhóm: 309 passed.

**3. Crash Safety**

Crash Suite của hệ thống: 20/20 scenarios passed.

**4. Student Drop (Hidden Self-check)**

Sử dụng dữ liệu từ `student_drop/hidden_claims.jsonl` và `student_drop/hidden_world_state.json`.

Chạy tự kiểm tra trên 40 claims với 5 seeds, gồm 11, 22, 33, 44 và 55.

Kết quả riêng Shape C Orchestrator:

| Chỉ số | Kết quả |
|---|---:|
| Số lượt chạy (Trials) | 200 |
| Lỗi thực thi (Error Trials) | 0 |
| Nghi ngờ rò rỉ dữ liệu (Leak Suspect) | 0 |
| Số bước trung bình (Mean Steps) | 2,900 |
| P95 Steps | 3 |
| Tool Calls trung bình | 5,475 |
| Tổng chi phí mô phỏng | 10.950 VND |

Cả ba Shape Workflow, Single Agent và Orchestrator đã thực hiện tổng cộng 600 lượt chạy, không ghi nhận lỗi thực thi hoặc trường hợp nghi ngờ rò rỉ dữ liệu.

Do Student Drop không cung cấp nhãn đáp án (Unlabeled), kết quả này chỉ dùng để tự kiểm tra hoạt động và tính nhất quán, chưa thể xác nhận Pass Rate hoặc kết quả Hidden Grader chính thức.

File kết quả: `src/reports/hidden_selfcheck.json` trên máy kiểm thử.

**5. Báo cáo cá nhân**

`report/team/2A202603828_NguyenTaiMinhHuy.md`
