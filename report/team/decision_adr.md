# Biên bản quyết định kiến trúc (Architecture Decision Record)



- Shape chọn: workflow
- Commit hash: 8de53cb
- Run id dẫn chứng: R-workflow-C-001-s11 (một lần chạy trong benchmark công khai, xem `src/reports/benchmark.json`)
- Pass rate / pass^5 / seed min-max / variance: 100,00% (70/70) / 100,00% / 100,00%–100,00% / 0,000000 (các seed: 11, 22, 33, 44, 55)
- Cost per success (mô phỏng, VND): 34,29 VND cho mỗi lần xử lý đúng; tổng chi phí mô phỏng 2.400,00 VND cho 70 lần chạy Workflow (`src/reports/benchmark.json`)
- Crash safety: 20/20 (`src/reports/crash.json`, tạo bởi `lab8.crash_test --count 20`)
- Secret leak / approval bypass: Chưa ghi nhận vi phạm trong các phép kiểm tra công khai đã thực hiện (302 public tests đã đạt trước khi bổ sung custom test; bài test thiếu phê duyệt đã được tạo riêng). Đây chỉ là bằng chứng từ các kiểm tra công khai, không phải kết quả kiểm thử ẩn. Trước khi nộp cần xác nhận lần chạy test cuối cùng và kiểm tra trace/stdout.

## Vì sao shape này

Nhóm chọn **Shape A – Deterministic Workflow (quy trình xử lý cố định)** cho bài toán xử lý yêu cầu hoàn phí vận chuyển của LogiX-VN. Chúng mình dựa trên kết quả so sánh cùng **14 yêu cầu công khai và 5 seed** cho cả bốn kiến trúc, tương ứng 70 lần chạy mỗi kiến trúc.

Workflow, Single Agent và Orchestrator đều đạt 70/70 lần chạy đúng (100%). Swarm đạt 69/70 (98,57%). Trong khi đó, Workflow có chi phí mô phỏng thấp nhất, **34,29 VND cho mỗi lần xử lý đúng**, trung bình 1,86 bước, mức p95 là 2 bước và không gọi mô hình AI. Single Agent tốn 35,95 VND, Orchestrator tốn 52,86 VND và Swarm tốn 49,57 VND cho mỗi lần xử lý đúng.

Vì việc hoàn phí dựa trên các quy tắc nghiệp vụ rõ ràng, chúng mình thấy Workflow đủ để xử lý dữ liệu, kiểm tra điều kiện và để chương trình kiểm soát số tiền hoàn. Trên bộ dữ liệu công khai này, việc bổ sung Agent chưa đem lại tỷ lệ đúng cao hơn, nhưng làm tăng chi phí hoặc số bước thực hiện. Dữ liệu đầy đủ nằm trong `src/reports/benchmark.json`.

| Kiến trúc (Shape) | Tỷ lệ đúng (Pass rate) | Pass^5 | Seed nhỏ nhất–lớn nhất | Phương sai (Variance) | VND mô phỏng / lần đúng | Trung bình / p95 bước |
|---|---:|---:|---:|---:|---:|---:|
| Workflow | 100,00% | 100,00% | 100,00%–100,00% | 0,000000 | 34,29 | 1,86 / 2 |
| Single Agent | 100,00% | 100,00% | 100,00%–100,00% | 0,000000 | 35,95 | 1,86 / 2 |
| Orchestrator | 100,00% | 100,00% | 100,00%–100,00% | 0,000000 | 52,86 | 2,79 / 3 |
| Swarm | 98,57% | 92,86% | 92,86%–100,00% | 0,000816 | 49,57 | 2,80 / 3 |

## Có đáng đổi lấy multi-agent không

**Với bằng chứng hiện tại thì chưa đáng.** Orchestrator không có tỷ lệ đúng cao hơn Workflow, nhưng phải thực hiện nhiều bước và nhiều lần gọi công cụ hơn. Swarm vừa có chi phí cao hơn, vừa mắc một lỗi trạng thái trong benchmark công khai: yêu cầu `C-002`, seed `55`, trả về `resolved` trong khi kết quả đúng phải là `denied` (`R-swarm-C-002-s55`). Đây là lỗi trạng thái trong benchmark; **không đủ bằng chứng để kết luận rằng tiền đã bị chuyển sai**.

Chúng mình sẽ cân nhắc lại nếu thử nghiệm trên bộ dữ liệu lớn hơn, độc lập hơn, cho thấy việc nhiều Agent phối hợp giúp xử lý tốt hơn các trường hợp thực sự cần phân tích riêng hoặc thu thập bằng chứng song song. Lợi ích đó phải đủ lớn để bù chi phí và độ phức tạp tăng thêm, đồng thời vẫn không được bỏ qua phê duyệt, làm lộ thông tin bí mật hay hoàn tiền trùng. Trước khi triển khai thực tế, chúng mình cũng cần đo thời gian phản hồi (latency) và chi phí vận hành thật.

## Trace và bằng chứng

Workflow ghi lại những sự kiện có giới hạn, chẳng hạn `quote` (số tiền tính được và mã điều khoản được áp dụng) và `decision` (trạng thái xử lý và số tiền đề xuất). Đường chạy có mô phỏng sự cố có thể ghi thêm dấu mốc `crash`. Mỗi lần chạy tạo một tệp trace JSON có `run_id` khớp với lần chạy đó, qua `lab8.trace.Trace` và `lab8.shapes.common.finish`.

Trace và đầu ra terminal **không được chứa** nguyên dòng dữ liệu yêu cầu hoặc đơn hàng, `parent_secret`, thông tin xác thực (authorization headers), số điện thoại, địa chỉ, chi tiết sản phẩm hay ghi chú nội bộ. Chỉ host được gọi `durability.issue_refund`; host có trách nhiệm kiểm tra phê duyệt gắn với yêu cầu và giới hạn số tiền hoàn của từng đơn hàng.

Các public tests và custom test về thiếu phê duyệt cung cấp bằng chứng cho những kiểm soát này, nhưng không thay thế việc rà soát đầy đủ mọi tệp đầu ra. Khi kiểm tra bảo mật, cần xem cả trace, các tệp có tiền tố `run_id` trong thư mục trace và `stdout/stderr`. Nếu log được ghi ở nơi khác, cũng phải kiểm tra riêng thay vì mặc định công cụ quét hiện tại đã bao phủ chúng. Tham khảo `docs/contracts.md`, `src/reports/crash.json` và các test liên quan.

## Giới hạn

Benchmark này dùng **MockLLM**, bảng giá tính bằng VND mô phỏng và bộ dữ liệu công khai chỉ gồm 14 yêu cầu, mỗi yêu cầu chạy với 5 seed. Kết quả **không phản ánh giá API thực tế** hoặc hành vi của mô hình AI trong môi trường sản xuất. Dữ liệu không cung cấp thời gian phản hồi (`latency: unavailable`), vì vậy chúng mình không đưa ra kết luận về tốc độ chạy thực tế.

Hiện chưa có kết quả đánh giá ẩn trên các yêu cầu bổ sung. Do đó, tỷ lệ đúng của bộ công khai không thể coi là tỷ lệ đúng trên hidden set, cũng không bảo đảm đạt ngưỡng 0,75 của hidden evaluation. Không được sửa bằng tay các tệp JSON benchmark và crash đã tạo. Trước khi nộp, cần điền commit hash thật sau khi đã commit code và bằng chứng kiểm thử.
