# Báo cáo cá nhân: Nguyễn Tài Minh Huy

## 1. Thông tin thành viên

| Thông tin | Nội dung |
| :--- | :--- |
| Họ tên | Nguyễn Tài Minh Huy |
| MSSV | 2A202603828 |
| Email | nguyentaiminhhuy831975@gmail.com |
| GitHub | [Nhánh NguyenTaiMinhHuy của repository nhóm](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/tree/NguyenTaiMinhHuy) |
| Nhóm | NhomChokevy, lớp L3B |

## 2. Phần việc phụ trách

Mình phụ trách **Shape C: Orchestrator và Handoff**, gồm triển khai luồng điều phối công việc, sử dụng cơ chế bàn giao dữ liệu an toàn cho các investigator và kiểm tra giới hạn quyền của những công cụ được cấp cho investigator.

Trong bài toán hoàn tiền của LogiX, Orchestrator lấy thông tin yêu cầu, tạo handoff và gọi các investigator chỉ có quyền đọc để kiểm tra đơn hàng, lịch sử hoàn tiền và mức bồi hoàn. Sau đó, Orchestrator sử dụng kết quả kiểm tra để đưa ra trạng thái xử lý. Thao tác hoàn tiền được thực hiện ở phía host, không giao cho investigator.

## 3. Đóng góp cụ thể

### 3.1. Triển khai Shape C

Mình cập nhật `src/lab8/shapes/orchestrator.py` để triển khai `run_claim()` theo giao diện của bài lab. Luồng xử lý sử dụng `make_handoff()` và `run_readonly_investigators()` có sẵn trong starter code. Khi bàn giao, chỉ những dữ liệu và công cụ thuộc danh sách cho phép mới được chuyển sang investigator.

Mình không viết lại `src/lab8/handoff.py`, vì đây là module do bài lab cung cấp. Phần đóng góp của mình là **tích hợp và kiểm tra cơ chế handoff** trong Shape C.

Các quyết định kỹ thuật chính là chỉ sử dụng công cụ đọc ở investigator, không đưa `parent_secret`, `raw_headers` hoặc dữ liệu đầy đủ của claim vào handoff và giữ quyền gọi `issue_refund` ở host. Số tiền đề xuất được lấy từ kết quả tính toán dựa trên đơn hàng, không tin vào số tiền hoặc số ngày trễ do khách tự ghi trong ticket. Mình cũng giữ nguyên các comment và TODO gốc trong file starter.

### 3.2. Bổ sung kiểm thử bảo mật Handoff

Mình bổ sung `src/tests/test_orchestrator_handoff.py` với ba test:

| Test | Nội dung kiểm tra |
| :--- | :--- |
| `test_handoff_only_forwards_approved_facts` | Handoff chỉ chuyển những dữ liệu thuộc allowlist, loại bỏ secret, thông tin liên hệ riêng tư và raw headers không được phép. |
| `test_handoff_rejects_every_write_tool_and_unknown_tool` | Từ chối `issue_refund`, `append_audit` và công cụ không nằm trong danh sách được phép. |
| `test_sanitize_rejects_secret_inside_allowed_fact` | Từ chối secret bị chèn vào một trường dữ liệu có tên hợp lệ. |

Ba test này kiểm tra trực tiếp hợp đồng của module `handoff.py`. Chúng không thay thế cho bộ test tích hợp của toàn bộ hệ thống.

### 3.3. Commit kỹ thuật

Commit chính của mình là **`a6dc301`** với nội dung `Implement orchestrator and handoff security tests`. Commit cập nhật `orchestrator.py` và tạo mới `test_orchestrator_handoff.py`.

## 4. Bằng chứng

| Bằng chứng | Kết quả hoặc đường dẫn |
| :--- | :--- |
| Commit của mình | [a6dc301](https://github.com/nguyenhung1901/K5-L3B-DAY08-NhomChokevy-FromWorkflowsToAgents/commit/a6dc301) |
| Nhánh cá nhân | `NguyenTaiMinhHuy` |
| Test Handoff | `python -m pytest -q tests/test_orchestrator_handoff.py` cho kết quả **3 passed** |
| Toàn bộ test trong repository nhóm | `python -m pytest -q -m "student or not student" -o addopts=` cho kết quả **309 passed** trong lần chạy mình đã kiểm tra |
| File code | `src/lab8/shapes/orchestrator.py` |
| File test do mình thêm | `src/tests/test_orchestrator_handoff.py` |

Kết quả 309 tests là kết quả tại thời điểm kiểm tra trên máy mình. Đây là các test có trong repository nhóm, chưa phải xác nhận đã vượt bộ hidden tests của giảng viên.

## 5. Công cụ AI đã dùng

Mình sử dụng **ChatGPT** để giải thích yêu cầu của Shape C, làm rõ sự khác nhau giữa Orchestrator và investigator, hỗ trợ triển khai code, gợi ý và tạo bộ test Handoff, cũng như hướng dẫn khắc phục lỗi môi trường Python và Git.

Mình tự kiểm tra bằng cách đọc lại `handoff.py`, đối chiếu các trường dữ liệu và danh sách công cụ được phép, chạy riêng ba custom tests và chạy lại toàn bộ bộ test trong repository nhóm. Mình cũng xem kết quả trên PowerShell và kiểm tra commit đã được push lên GitHub. Những kết quả được ghi trong báo cáo lấy từ lệnh và ảnh terminal thực tế, không phải số liệu do AI tự tạo.

Bài lab sử dụng `MockLLM` để kiểm thử, không dùng API key hay gọi model trả phí trong quá trình chạy lab.

## 6. Tự đánh giá

Qua phần việc này, mình hiểu rõ hơn cách Orchestrator giao việc cho investigator mà không chuyển toàn bộ dữ liệu nhạy cảm và không trao quyền ghi dữ liệu. Mình cũng hiểu vì sao việc kiểm tra quyền truy cập và giới hạn công cụ phải được thực hiện bằng code thay vì chỉ dựa vào lời nhắc của AI.

Mình đã hoàn thành phần Orchestrator và Handoff được phân công, bổ sung ba test riêng, chạy kiểm thử thành công và push commit lên nhánh cá nhân. Điểm mình cần cải thiện là khả năng tự triển khai và giải thích chi tiết toàn bộ luồng xử lý mà không cần trợ lý AI hỗ trợ, đặc biệt là các trường hợp retry, crash và kiểm tra quyền ở host.

Mình chưa xem kết quả hidden tests và chưa coi 309 test công khai đạt yêu cầu là bằng chứng hệ thống đã an toàn trong mọi trường hợp. Nếu có thêm thời gian, mình muốn bổ sung test tích hợp xác nhận investigator thực sự không thể gọi công cụ ghi khi Orchestrator đang chạy.
