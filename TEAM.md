# Team

## Thông tin nhóm

- Tên nhóm: NhomChokevy
- Lớp: L3B
- Repo nhóm: `K5-L3B-DAY08-NhomChokeVy-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Lê Chuẩn | 2A202603784 | lechuanpham@gmail.com | 22520160 | workflow + integration |
|   2 | Nguyễn Tài Minh Huy | 2A202603828 | nguyentaiminhhuy831975@gmail.com | ArthurTheMana | orchestrator + handoff + student_drop |
|   3 | Nguyễn Văn Hùng | 2A202603816 |  | `nguyenhung1901` | ledger + evaluation |
|   4 | Nguyễn Trung Hải | 2A202603728 | trunghaicva@gmail.com | nguyentrunghai7 | single agent |
Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phạm Lê Chuẩn | workflow + integration | Chưa commit (working tree) | `src/tests/test_workflow_integration.py` (3 passed); student workflow matrix (62 passed) |
| Nguyễn Tài Minh Huy | Orchestrator + Handoff | `648fb4e` + `5457779` + `a6dc301` + `ed40d2c` | `src/tests/test_orchestrator_handoff.py` (3 passed); toàn bộ repository (309 passed); Student Drop (200 lượt Orchestrator, 0 lỗi thực thi, 0 nghi ngờ rò rỉ dữ liệu) |
| Nguyễn Văn Hùng | ledger + evaluation | `ecfd50c` | `src/tests/test_refund_idempotency.py::test_cross_shape_retry_keeps_one_refund` (3 tham số) |
| Nguyễn Trung Hải | single agent | [4232137](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/commit/4232137) | `src/tests/test_single_agent.py` (1 passed); ma trận public Single Agent (62 passed) |

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

## 2A202603816_NguyenVanHung

### Phần việc phụ trách

Phụ trách Ledger + Evaluation: kiểm thử tính idempotency của quá trình hoàn tiền, đánh giá khả năng phục hồi sau crash và thực hiện benchmark so sánh bốn kiến trúc Workflow, Single Agent, Orchestrator và Swarm.

### Đóng góp cụ thể

- Bổ sung kiểm thử test_cross_shape_retry_keeps_one_refund gồm 3 trường hợp xử lý lại yêu cầu hoàn tiền giữa các kiến trúc khác nhau, xác minh không tạo bản ghi refund và audit trùng trên cùng SQLite ledger.

- Thực hiện benchmark trên 14 claim công khai với 5 seed; chạy 20 kịch bản crash safety và tạo hai file kết quả benchmark.json, crash.json.

- Tổng hợp và phân tích các chỉ số pass rate, pass⁵, seed variance và cost/success để hỗ trợ nhóm lựa chọn kiến trúc Workflow và hoàn thiện Research Card R3.

### Bằng chứng

- Commit kiểm thử: ecfd50c.

- Kiểm thử: src/tests/test_refund_idempotency.py đạt 6/6, gồm 3 trường hợp mới; toàn bộ dự án đạt 312/312 tests tại lần kiểm tra gần nhất.

- Benchmark: src/reports/benchmark.json, Workflow đạt 100% với chi phí mô phỏng 34,29 VND/lượt xử lý đúng.

- Crash safety: src/reports/crash.json, đạt 20/20 kịch bản.

## 2A202603728_NguyenTrungHai

### Phần việc phụ trách

Phụ trách Shape B `single`. Vòng ReAct tối đa 8 bước đọc claim, tra cứu đơn hàng và tính khoản đề xuất. Host nhận toàn bộ khoản đề xuất và kiểm tra điều kiện trước khi ghi hoàn tiền.

### Đóng góp cụ thể

- Tích hợp phần triển khai `src/lab8/shapes/single.py` có sẵn từ repo nguồn vào nhánh `NguyenTrungHai`.
- Bổ sung `test_single_step_cap_stops_before_refund` trong `src/tests/test_single_agent.py` để xác minh khi hết số bước, agent trả về `failed` với lý do `step_cap` và dừng trước khi gọi ghi hoàn tiền.

### Bằng chứng

- Commit phần Single Agent: [4232137](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/commit/4232137).
- Ma trận public của `single` đạt 62/62 test, gồm ca chạy lại claim và phục hồi sau crash. Test giới hạn số bước đạt 1/1.
- Benchmark public riêng cho `single` đạt 70/70 lượt, pass⁵ 100% và chi phí mô phỏng 35,95 VND/lượt đúng. Run ID minh họa: `R-single-C-001-s11`.
- [Báo cáo cá nhân](report/team/2A202603728_NguyenTrungHai.md).

- Crash safety: src/reports/crash.json, đạt 20/20 kịch bản.
