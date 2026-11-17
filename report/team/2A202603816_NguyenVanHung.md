# Báo cáo cá nhân — MSSV_HoTen

## 1. Thông tin thành viên

- Họ tên: Nguyễn Văn Hùng
- MSSV: 2A202603816
- Email:
- GitHub: nguyenhung1901
...
## 2. Phần việc phụ trách

Phần được phân công là `ledger + evaluation`: kiểm tra tính an toàn của việc ghi hoàn tiền trên SQLite ledger, đặc biệt là idempotency khi xử lý lại yêu cầu và khi phục hồi sau sự cố; đồng thời chạy kiểm thử, benchmark và crash suite để đánh giá bốn kiến trúc của Lab 8. 
...
## 3. Đóng góp cụ thể

- Chạy benchmark với cùng 14 claim công khai và 5 seed `11,22,33,44,55` cho `workflow`, `single`, `orchestrator` và `swarm`. Tạo hai file kết quả `src/reports/benchmark.json` và `src/reports/crash.json` bằng lệnh của lab
- Kiểm tra kết quả đánh giá để đối chiếu tỷ lệ thành công, `pass^5`, phương sai giữa các seed, chi phí mô phỏng trên mỗi lượt đúng và số bước xử lý. Các số liệu là cơ sở để nhóm cân nhắc lựa chọn Workflow, không phải kết quả từ bộ hidden.
- Bổ sung ở máy cá nhân test tham số `test_cross_shape_retry_keeps_one_refund` trong `src/tests/test_refund_idempotency.py`, gồm ba lượt chuyển shape: `workflow` → `single`, `single` → `orchestrator`, `orchestrator` → `workflow`. Test chạy hai shape trên cùng SQLite ledger cho claim `C-001`, rồi kiểm tra chỉ có một dòng refund và một sự kiện audit `refund_committed`.
- Bổ sung test tham số hóa test_cross_shape_retry_keeps_one_refund trong src/tests/test_refund_idempotency.py, gồm ba trường hợp Workflow → Single Agent, Single Agent → Orchestrator và Orchestrator → Workflow. Các trường hợp sử dụng cùng một SQLite ledger để xác minh việc xử lý lại cùng một claim không tạo thêm refund hoặc audit refund_committed. Đây là phần kiểm thử bổ sung về idempotency giữa các kiến trúc, không phải phần triển khai cơ chế ledger của host.
...
## 4. Bằng chứng

Chạy từ thư mục `src`:

```powershell
python -m lab8.smoke
python -m pytest -q tests/test_refund_idempotency.py
python -m pytest -q -o addopts=
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
python -m lab8.crash_test --count 20 --out reports/crash.json
```

- Kết quả lần kiểm tra gần nhất trên máy: `lab8.smoke pass`; toàn bộ **312/312 tests passed**. 
- Benchmark công khai gồm 70 lượt mỗi kiến trúc: `workflow` 70/70 (100%, 34,29 VND/lượt đúng), `single` 70/70 (100%, 35,95 VND), `orchestrator` 70/70 (100%, 52,86 VND), `swarm` 69/70 (98,57%, 49,57 VND). Đây là chi phí mô phỏng; latency không có dữ liệu.
- Crash suite: `crash_safety 20/20`. File dẫn chứng: `src/reports/benchmark.json` và `src/reports/crash.json`.
- Kết quả kiểm thử test_refund_idempotency.py: 6/6 passed, phần test được lưu tại commit [ecfd50c], kiểm tra trên nhánh nộp của nhóm.
...
## 5. Công cụ AI đã dùng

Sử dụng ChatGPT để hỗ trợ đọc yêu cầu Lab 8, phân tích phạm vi `ledger + evaluation`, gợi ý test kiểm tra idempotency giữa các shape và rà soát nội dung báo cáo.
...
## 6. Tự đánh giá

Đã thực hiện được phần đánh giá chung của bốn kiến trúc, tạo và đẩy hai file benchmark/crash theo phân công, đồng thời bổ sung thử nghiệm kiểm tra không hoàn tiền trùng khi chuyển giữa các shape trên máy cá nhân. Kết quả public hiện tại không phát hiện hồi quy và crash suite đạt 20/20. Kết luận về độ đúng và chi phí mới giới hạn ở bộ public với MockLLM, chưa thay thế được đánh giá hidden hoặc vận hành thực tế.
...