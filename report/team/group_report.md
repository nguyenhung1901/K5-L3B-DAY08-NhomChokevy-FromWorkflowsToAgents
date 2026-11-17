# Báo cáo nhóm: Lab 8 LogiX VN

## 1. Thông tin nhóm

**Tên nhóm:** NhomChokevy  
**Repository:** [K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents)  
**Thành viên và phân công:** Xem [`TEAM.md`](../../TEAM.md).  
**Commit chốt:** **[CẦN BỔ SUNG SAU KHI NHÓM MERGE VÀ CHỐT MAIN]**. Mã này phải trùng với `report/team/decision_adr.md`.

## 2. Mục tiêu và cách tiếp cận

Nhóm thực hiện bài toán xử lý yêu cầu hoàn phí vận chuyển của LogiX VN. Hệ thống phải đọc yêu cầu của khách, xác minh đơn hàng, áp dụng chính sách đang có hiệu lực, kiểm tra phê duyệt và quyết định trạng thái xử lý. Mục tiêu không chỉ là trả lời đúng mà còn phải ngăn hoàn tiền trái phép, hoàn trùng khi chạy lại và để lộ dữ liệu nhạy cảm trong trace.

Nhóm so sánh bốn cách tổ chức xử lý cùng một nghiệp vụ: **Workflow (A)**, **Single Agent (B)**, **Orchestrator và các Investigator chỉ đọc (C)**, cùng **Swarm tham chiếu (D)** có sẵn. Nhóm triển khai ba shape đầu, sử dụng shape D làm đối chứng. Việc so sánh sử dụng cùng bộ claim, cùng năm seed, công cụ, chính sách và cách chấm; mỗi trial có cơ sở dữ liệu SQLite riêng để tránh ảnh hưởng chéo. Phân công chính thức và commit của từng người được ghi trong `TEAM.md`.

## 3. Giải pháp

### 3.1. Shape A: Workflow

`src/lab8/shapes/workflow.py` xử lý theo các bước Python xác định trước. Chương trình đọc claim qua `get_claim_view`, kiểm tra `order_id` và quyền xem đơn qua `lookup_order`, rồi tính số tiền bằng `compute_compensation` dựa trên dữ liệu đơn và chính sách. Khi số tiền đề xuất bằng 0, hệ thống từ chối mà không gọi công cụ hoàn tiền. Khi số tiền lớn hơn 0, code chuyển **toàn bộ quote** cho `durability.issue_refund`; host tự kiểm approval, số tiền đã hoàn cho đơn đó và tính phần còn được hoàn. Workflow không cho model quyết định tiền hoặc trực tiếp ghi sổ.

### 3.2. Shape B: Single Agent

`src/lab8/shapes/single.py` sử dụng vòng ReAct qua `lab8.runtime.run_react`, giới hạn tối đa tám bước. Agent chọn các công cụ đọc để lấy thông tin và hình thành đề xuất; host vẫn chịu trách nhiệm xác thực điều kiện và ghi hoàn tiền. Văn bản khách hàng cung cấp không được dùng để thay thế dữ liệu đơn hàng hoặc chính sách có hiệu lực. Các lần gọi model và công cụ được ghi nhận vào budget để so sánh chi phí mô phỏng.

### 3.3. Shape C: Orchestrator và Handoff

`src/lab8/shapes/orchestrator.py` điều phối investigator nhằm thu thập dữ liệu cần thiết. Orchestrator tạo gói bàn giao bằng `lab8.handoff.make_handoff`. Handoff chỉ giữ các trường thuộc allowlist và chỉ cấp công cụ đọc, chẳng hạn kiểm tra đơn hàng hoặc lịch sử hoàn tiền. `lab8.runtime.run_readonly_investigators` thực hiện phần điều tra. Investigator không có quyền gọi `issue_refund` hoặc `append_audit`; host là nơi duy nhất thực hiện thao tác ghi sau khi kiểm tra điều kiện.

Trong phần test riêng cho handoff, nhóm kiểm tra việc loại trường không được phép, từ chối công cụ ghi và phát hiện secret nằm trong dữ liệu được chuyển. Module `handoff.py` do starter cung cấp; phần nhóm triển khai là tích hợp trong Shape C và kiểm thử cơ chế này.

### 3.4. Ranh giới giữa code, model và trace

**Code/host** quyết định quyền sở hữu đơn, chính sách, số tiền hoàn, approval, hạn mức và thao tác ghi bền vững vào ledger. **Model/Agent** chỉ hỗ trợ đọc, lựa chọn công cụ trong giới hạn cho phép và soạn nội dung trả lời, không có quyền tự duyệt hoặc chuyển tiền. `proposal_amount` luôn là quote đầy đủ; host tự tính phần còn được hoàn khi có các claim trước đó.

Mỗi lượt chạy tạo `RunResult` và tệp trace JSON với `run_id` tương ứng. Trace ghi các sự kiện cần cho kiểm toán như quote, quyết định và diễn biến xử lý, không sao chép nguyên claim hoặc order. Các giá trị như `parent_secret`, header, số điện thoại, địa chỉ, tên mặt hàng và ghi chú nội bộ không được xuất hiện trong trace, log hoặc stdout. Xem [`docs/contracts.md`](../../docs/contracts.md) và [`docs/architecture.md`](../../docs/architecture.md).

## 4. Kết quả

### 4.1. Kiểm thử chức năng và khả năng phục hồi

Trong lần chạy đầy đủ gần nhất tại repository nhóm, lệnh `python -m pytest -q -m "student or not student" -o addopts=` cho kết quả **309 passed, 0 failed** (31,53 giây). Bộ test riêng `tests/test_orchestrator_handoff.py` đạt **3/3 passed**. Đây là kết quả test local trên mã của nhóm tại thời điểm chạy, không phải kết quả hidden test của giảng viên.

Tệp `src/reports/crash.json` đã ghi nhận **20/20** kịch bản đạt yêu cầu. Các kịch bản bao gồm crash trước tác động, sau tác động nhưng trước checkpoint, sau checkpoint, xung đột số tiền khi retry và retry khi thiếu approval. Kết quả này hỗ trợ đánh giá về khả năng tránh hoàn tiền trùng trong các trường hợp đã kiểm thử; không chứng minh rằng mọi sự cố thực tế đều đã được bao phủ.

### 4.2. Benchmark công khai

Dữ liệu dưới đây trích từ `src/reports/benchmark.json` được cung cấp trong quá trình thực hành: **14 claims, 5 seeds (11, 22, 33, 44, 55), 70 trials cho mỗi shape và tổng cộng 280 trials**. Backend là `MockLLM`; chi phí được **mô phỏng bằng VND**, không phải hóa đơn API; latency không khả dụng.

| Shape | Pass rate | Pass⁵ | Seed min đến max | Variance | Cost/success (VND mô phỏng) | Mean / p95 steps |
| :--- | ---: | ---: | :---: | ---: | ---: | :---: |
| Workflow | 100,00% | 100,00% | 100,00% đến 100,00% | 0,000000 | 34,29 | 1,86 / 2 |
| Single Agent | 100,00% | 100,00% | 100,00% đến 100,00% | 0,000000 | 35,95 | 1,86 / 2 |
| Orchestrator | 100,00% | 100,00% | 100,00% đến 100,00% | 0,000000 | 52,86 | 2,79 / 3 |
| Swarm | 98,57% | 92,86% | 92,86% đến 100,00% | 0,000816 | 49,57 | 2,80 / 3 |

Workflow, Single Agent và Orchestrator đều đúng **70/70 trials**. Swarm đúng **69/70 trials**, sai ở `C-002` với seed `55`: kết quả thực tế `resolved`, trong khi nhãn mong đợi là `denied` (`R-swarm-C-002-s55`). Đây là lỗi trạng thái theo evaluator; chưa đủ cơ sở khẳng định đã phát sinh một giao dịch hoàn tiền sai.

**Quyết định:** Nhóm ưu tiên **Workflow** cho bài toán hiện tại, vì cùng đạt 100% như Single Agent và Orchestrator nhưng chi phí mô phỏng thấp nhất, trung bình ít bước hơn Orchestrator. Việc bổ sung nhiều Agent chưa cho thấy lợi ích về tỷ lệ đúng trong bộ public. Lập luận chi tiết nằm trong [`decision_adr.md`](decision_adr.md); phiếu nghiên cứu R3 và phần so sánh kiến trúc nằm trong [`research_card.md`](research_card.md). Chưa có bằng chứng về kết quả `student_drop/`, nên nhóm không đưa ra khẳng định về hidden evaluation.

## 5. Khó khăn và cách xử lý

**Môi trường Python trên Windows:** Máy ban đầu dùng Python 3.14, trong khi lab yêu cầu bản tương thích 3.11 đến 3.13. Nhóm thực hành sử dụng Python 3.13 với `.venv` và cài dependencies từ `src/requirements.txt`. PowerShell từng chặn `Activate.ps1`; cách xử lý là giới hạn execution policy trong phiên hiện tại hoặc gọi trực tiếp `.venv\Scripts\python.exe`.

**Chạy lệnh từ sai thư mục:** Khi chạy module từ thư mục gốc, Python báo `No module named lab8`. Nhóm chuyển vào `src` để chạy các lệnh test, benchmark và crash. Module crash thực tế là `lab8.crash_test`, sử dụng `--count 20 --out reports/crash.json` thay vì tham số `--shapes`.

**Tích hợp repository nhóm:** Khi các shape chưa đồng bộ, một lần chạy trên repo nhóm ghi nhận `124 failed, 181 passed`. Sau khi các thay đổi được tích hợp, lần chạy được chụp lại đạt **309 passed**. Điều này cho thấy cần chạy regression test sau mỗi lần cập nhật các nhánh.

**Kiểm soát handoff:** Nhóm không sửa module `handoff.py` gốc mà bổ sung `test_orchestrator_handoff.py` để kiểm tra allowlist, quyền công cụ và phát hiện secret. Kết quả test riêng đạt **3/3 passed**. Các lệnh và ảnh chụp terminal là bằng chứng bổ sung trong lịch sử làm bài của nhóm.

## 6. Tổng hợp đóng góp

Chi tiết thành viên, vai trò, commit và test được quản lý trong [`TEAM.md`](../../TEAM.md) và các báo cáo cá nhân ở thư mục `report/team/`. Đóng góp đã xác nhận của **Nguyễn Tài Minh Huy** là tích hợp Shape C Orchestrator với Handoff và bổ sung ba custom security tests trong `src/tests/test_orchestrator_handoff.py`, commit [`a6dc301`](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/commit/a6dc301). Các thành viên khác cần đối chiếu và điền commit, test của chính mình trong `TEAM.md` trước khi nộp.

## 7. Hạn chế và hướng cải thiện

Benchmark dùng MockLLM, giá VND mô phỏng và 14 claim công khai nên chưa phản ánh điều kiện sản xuất. Trường latency là `unavailable`; không thể kết luận về tốc độ phản hồi thực tế. Kết quả 309 tests chỉ chứng minh các kiểm tra đã được chạy tại thời điểm đó, không thay thế đánh giá ẩn hay kiểm toán bảo mật độc lập. Ngoài ra, khi thay đổi kiến trúc, số lần gọi công cụ và luồng xử lý cũng thay đổi nên chưa thể quy mọi chênh lệch chỉ cho số lượng Agent.

Để cải thiện, nhóm cần thử dữ liệu đa dạng hơn, đo latency và chi phí thật, kiểm tra các trường hợp ticket chứa thông tin đánh lừa hoặc chính sách thay đổi, rà soát trace/stdout và đối chiếu toàn bộ kết quả với hidden selfcheck nếu được phát. Nhóm sẽ cân nhắc kiến trúc phức tạp hơn khi nó chứng minh lợi ích rõ ràng về chất lượng, khả năng xử lý hoặc vận hành mà vẫn giữ các ranh giới an toàn.

## 8. Tài liệu tham khảo

1. [Kiến trúc Lab 8](../../docs/architecture.md), [hợp đồng kết quả và trace](../../docs/contracts.md), [quy tắc thực hành](../../RULES.md) và [hướng dẫn nộp bài](../../SUBMISSION.md).
2. Kết quả tự sinh: `src/reports/benchmark.json` và `src/reports/crash.json`.
3. [AgentArch: A Benchmark for Evaluating Agent Architectures in Enterprise Workflows, AACL 2026](https://aclanthology.org/2026.aacl-long.44/), nguồn tham khảo cho phiếu nghiên cứu R3.