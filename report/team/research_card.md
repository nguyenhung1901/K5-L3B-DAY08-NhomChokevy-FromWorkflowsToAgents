# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

## Chủ đề nghiên cứu

**R3: Lựa chọn kiến trúc (Architecture Choices)**

**Câu hỏi nghiên cứu:** Khi xử lý yêu cầu hoàn tiền vận chuyển của LogiX VN, việc chuyển từ quy trình cố định (Workflow) sang kiến trúc điều phối nhiều tác nhân (Orchestrator) có tăng tỷ lệ thành công đủ để bù lại chi phí hay không?

## Phiếu của nhóm

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R3: Lựa chọn kiến trúc (Architecture Choices). So sánh Workflow với Orchestrator trong bài toán xử lý yêu cầu hoàn tiền. |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng | Tara Bogavelli, Hari Subramani và Roshnee Sharma (2026), *AgentArch: A Benchmark for Evaluating Agent Architectures in Enterprise Workflows*, AACL 2026. Nguồn: https://aclanthology.org/2026.aacl-long.44/ . Loại bằng chứng: nghiên cứu benchmark thực nghiệm trên 18 cấu hình agent trong các tác vụ doanh nghiệp. |
| CLAIM | Một kết luận gốc của nguồn | Nghiên cứu AgentArch cho thấy hiệu quả kiến trúc phụ thuộc vào mô hình và loại nhiệm vụ. Không có một cách tổ chức Agent nào luôn tốt nhất cho mọi trường hợp. Đây là nhận định của bài nghiên cứu, không phải kết luận rằng Workflow luôn tốt hơn Multi Agent. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | AgentArch nghiên cứu 18 cấu hình kết hợp chiến lược điều phối, kiểu Agent, bộ nhớ và công cụ suy luận trên hai tình huống doanh nghiệp. Nghiên cứu không trực tiếp thử bài toán hoàn tiền LogiX VN. Kết quả lab này chỉ áp dụng cho 14 claim công khai, 5 seed, MockLLM và công cụ mô phỏng. |
| HYPOTHESIS | Dự đoán tác động khi đổi một kiến trúc | Với bài toán hoàn tiền có chính sách và bước phê duyệt rõ ràng, chuyển từ Workflow sang Orchestrator sẽ không tăng tỷ lệ thành công trên bộ public, trong khi chi phí trên mỗi lần xử lý thành công dự kiến tăng ít nhất 30%. Đây là giả thuyết phân tích được phát biểu sau khi đã có dữ liệu, không phải giả thuyết đăng ký trước khi chạy. |
| METHOD | Giữ cố định các biến và chỉ thay một biến chính | Dùng cùng tập `data/public_claims.jsonl`, gồm 14 claim; cùng seed `11,22,33,44,55`; cùng MockLLM, bộ công cụ, dữ liệu chính sách, bảng giá và evaluator. Mỗi lượt có SQLite riêng. Biến so sánh chính là kiến trúc triển khai: `workflow` và `orchestrator`. Lưu ý hai triển khai khác nhau cả về số bước và cách gọi công cụ, nên đây là so sánh kiến trúc ở mức hệ thống chứ chưa phải phép cô lập thuần túy topology. |
| FAILURE / METRIC | Lỗi dự kiến và cách đo | Chỉ số chính: `pass_rate`, `pass_5`, `seed_variance`, `cost_per_success_vnd`, `mean_steps`, `mean_tool_calls`. Rủi ro: thêm khâu điều phối làm tăng chi phí và độ phức tạp mà không tăng tỷ lệ thành công. Dùng lỗi của Swarm ở claim `C-002`, seed `55` làm ví dụ phụ về sai trạng thái, không quy lỗi này cho Orchestrator. |

## Kết quả ablation

**Lệnh đã chạy (copy nguyên văn):**

```powershell
python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json
```

**File kết quả:** `src/reports/benchmark.json`

**Run ID minh họa:** `R-workflow-C-001-s11` và `R-orchestrator-C-001-s11`. Kết luận được tổng hợp từ toàn bộ 70 lượt chạy của mỗi kiến trúc, không chỉ từ hai run này.

### Kết quả so sánh chính

| Chỉ số | Workflow | Orchestrator |
| --- | ---: | ---: |
| Số lượt thử | 70 | 70 |
| Số lượt đạt | 70 | 70 |
| Tỷ lệ thành công (Pass Rate) | 100% | 100% |
| Pass⁵ | 100% | 100% |
| Phương sai giữa các seed (Seed Variance) | 0 | 0 |
| Chi phí mô phỏng trên mỗi lượt thành công | 34,29 VND | 52,86 VND |
| Số bước trung bình | 1,86 | 2,79 |
| Số lần gọi công cụ trung bình | 3,43 | 5,29 |

**Đối chiếu giả thuyết:** Kết quả phù hợp với dự đoán trên bộ public. Tỷ lệ thành công không tăng, trong khi chi phí trên mỗi lượt thành công của Orchestrator cao hơn khoảng 54,2% so với Workflow. Đây là bằng chứng trong môi trường mô phỏng, không phải khẳng định về mọi hệ thống Multi Agent ngoài thực tế.

**Kết quả tham khảo từ các kiến trúc còn lại:** Single Agent đạt 70/70, chi phí 35,95 VND trên mỗi lượt thành công. Swarm đạt 69/70, chi phí 49,57 VND trên mỗi lượt thành công. Swarm có một lượt sai ở claim `C-002`, seed `55`: kết quả thực tế `resolved` nhưng kết quả mong đợi `denied`.

### Giới hạn nghiên cứu

Mô hình được sử dụng là MockLLM, không phải mô hình AI triển khai thực tế. Chi phí trong file benchmark là số liệu mô phỏng, không phải giá API thật. Trường `latency` có giá trị `unavailable`, vì vậy chưa thể kết luận kiến trúc nào nhanh hơn theo thời gian thực. Chỉ có 14 claim công khai được thử, chưa có kết quả bộ hidden. Các Shape không chỉ khác topology mà còn khác quy trình và số lần gọi công cụ, nên kết quả không thể tách riêng hoàn toàn tác động của topology.

### Kết luận

Đối với nghiệp vụ hoàn tiền có quy tắc rõ ràng trong Lab08, nhóm ưu tiên **Workflow** vì đạt cùng tỷ lệ thành công như Orchestrator với chi phí mô phỏng thấp hơn. Nếu sau này có những trường hợp yêu cầu điều tra độc lập, nhiều nguồn thông tin hoặc thay đổi chính sách phức tạp, cần đánh giá lại bằng dữ liệu và test mới trước khi chuyển sang Multi Agent.

**Nguồn đối chiếu:** https://aclanthology.org/2026.aacl-long.44/
