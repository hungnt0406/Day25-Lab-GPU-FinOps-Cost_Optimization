# BÁO CÁO PHÂN TÍCH - GPU FinOps Lab

**Thông tin sinh viên**
- **Họ và tên:** Tran Ngoc Hung     
- **MSSV:** 2A202600429

---

## 1. Giới thiệu

**Mục tiêu của bài lab:**
Bài lab nhằm mục đích cung cấp kiến thức thực tiễn và kỹ năng triển khai các nguyên tắc FinOps (Cloud Financial Operations) vào việc quản lý tài nguyên GPU. Cụ thể, bài lab hướng dẫn cách theo dõi hiệu suất, phân bổ chi phí, sử dụng Spot Instances, thiết lập Autoscaling, và áp dụng các kỹ thuật tối ưu hóa mô hình (như Mixed Precision) để giảm thiểu chi phí huấn luyện AI/ML mà vẫn đảm bảo hiệu năng.

**Tổng quan về GPU FinOps:**
Trong bối cảnh chi phí phần cứng GPU (như NVIDIA A100, T4) trên môi trường Cloud (AWS, GCP, Azure) rất đắt đỏ, GPU FinOps là sự giao thoa giữa kỹ thuật (DevOps/MLOps) và quản lý tài chính. GPU FinOps tập trung vào việc tạo ra sự minh bạch về chi phí (visibility), tối ưu hóa việc sử dụng tài nguyên (optimization), và tự động hóa quy trình (automation) nhằm tối đa hóa ROI (Return on Investment) cho các dự án AI.

---

## 2. Phân tích từng phần

### Part 1-7: Phân tích kết quả từ Mock Cluster
- **Cluster monitoring insights:** Thông qua Gateway API, hệ thống cho phép theo dõi thời gian thực (real-time) các chỉ số sinh tồn của GPU như mức độ sử dụng (utilization), bộ nhớ (VRAM), công suất tiêu thụ (power draw) và nhiệt độ. Điều này giúp nhanh chóng xác định các tài nguyên đang bị "bỏ không" (idle).
- **Cost tracking observations:** Hệ thống theo dõi chi phí giống mô hình OpenCost, gán trực tiếp chi phí (USD) cho từng workload (train-resnet, inference-api). Việc quy trách nhiệm (chargeback) trở nên rõ ràng.
- **Spot instance savings analysis:** Việc giả lập đấu giá và sử dụng Spot Instances cho thấy tiềm năng tiết kiệm lên tới 60-70% chi phí so với On-Demand. Tuy nhiên, rủi ro bị thu hồi (preemption) đòi hỏi các workload phải có khả năng chịu lỗi (fault-tolerant) hoặc lưu checkpoint thường xuyên.
- **Autoscaling behavior:** Cấu hình Autoscaling (giống KEDA) giúp hệ thống tự động thêm/bớt Node dựa trên ngưỡng sử dụng (ví dụ: >70% scale up, <25% scale down). Điều này loại bỏ hoàn toàn chi phí lãng phí do over-provisioning (cấp phát thừa) khi không có job chạy.
- **Waste analysis và recommendations:** Các snapshot chi phí chỉ ra % lãng phí (Waste %) từ việc bật máy nhưng không chạy job. Hệ thống tự động phân loại mức độ nghiêm trọng và đề xuất các giải pháp khả thi như tắt node nhàn rỗi hoặc chuyển sang Spot.

### Part 8: Phân tích Real GPU Training trên Kaggle/Colab
- **FP32 vs Mixed Precision (AMP) comparison:** Quá trình huấn luyện mô hình ResNet-18 trên tập CIFAR-10 bằng GPU thực tế (T4) cho thấy sự khác biệt rõ rệt. Việc áp dụng Automatic Mixed Precision (AMP) thay cho FP32 truyền thống không chỉ giữ nguyên độ chính xác (accuracy) mà còn giúp tăng tốc độ huấn luyện.
- **Cost savings achieved:** Nhờ thời gian huấn luyện giảm xuống và lượng VRAM tiêu thụ ít hơn (cho phép tăng batch size), chi phí tổng thể của vòng huấn luyện giảm thiểu rõ rệt.
- **GPU utilization patterns:** Các biểu đồ Telemetry (thông qua pynvml/torch.cuda) cho thấy AMP đẩy mức GPU Utilization lên cao hơn trong thời gian ngắn hơn, tối ưu hóa công suất phần cứng (Power Draw/Watt) so với FP32.

### Part 8.5: Advanced Analysis
- **Multi-GPU scaling efficiency:** Việc mở rộng số lượng GPU (từ 1 lên 2, 4, 8) không mang lại hiệu suất tăng tuyến tính (sub-linear scaling) do chi phí giao tiếp mạng (communication overhead) giữa các GPU. Qua bảng phân tích, ta có thể tìm ra "điểm ngọt" (sweet spot) nơi chi phí trên mỗi đơn vị hiệu năng là thấp nhất.
- **Project cost forecasting:** Quản lý ngân sách dự án ML đòi hỏi tính toán chi phí qua nhiều giai đoạn (Data Prep, Training, Tuning). Việc tích hợp thêm biến số rủi ro (uncertainty) và đệm dự phòng (contingency buffer 15-20%) giúp dự báo chi phí thực tế sát với thực tiễn, tránh tình trạng đội vốn dự án (over-budget).
- **Optimization strategy prioritization:** Không phải mọi kỹ thuật tối ưu đều dễ áp dụng. Ma trận ROI so sánh giữa Tỉ lệ tiết kiệm (Savings %) và Công sức triển khai (Effort) giúp vạch ra lộ trình tối ưu hóa (Roadmap). Những giải pháp có rủi ro thấp/nỗ lực thấp (Quick wins) như dùng AMP sẽ được ưu tiên trước, sau đó mới đến kiến trúc Spot Instances phức tạp.

---

## 3. Kết luận và học hỏi

**Những kỹ năng FinOps đã học:**
- Thiết lập và giám sát các chỉ số Telemetry của GPU bằng Python.
- Cách đánh giá hiệu năng tài chính (Financial metrics) của một cluster song song với hiệu năng kỹ thuật.
- Xây dựng bảng dự toán chi phí đa giai đoạn và quản lý rủi ro ngân sách.

**Các chiến lược cost optimization hiệu quả nhất:**
1. **Quick Win:** Luôn mặc định sử dụng AMP (FP16/BF16) trong mọi dự án Deep Learning. Đòi hỏi thay đổi code rất nhỏ nhưng mang lại từ 20-50% hiệu suất thời gian và chi phí.
2. **High Impact:** Sử dụng Spot Instances cho các task batch-processing hoặc training có cơ chế checkpointing tốt.
3. **Automation:** Tích hợp bộ Autoscaler giám sát chặt chẽ mức độ sử dụng để tắt GPU ngay khi job hoàn thành.

**Ứng dụng thực tế trong projects:**
Kiến thức từ lab này hoàn toàn có thể áp dụng vào việc thiết lập các cụm máy chủ huấn luyện nội bộ (Kubernetes + KEDA) hoặc sử dụng trên nền tảng đám mây (AWS SageMaker, GCP Vertex AI). Trước khi khởi chạy một mô hình LLM tốn kém, ta có thể lập bảng phân tích Multi-GPU và dự toán ngân sách (Project Forecasting) để báo cáo cấp quản lý, đảm bảo dự án minh bạch về tài chính.

---
*Báo cáo được tự động tạo dựa trên sườn bài của SUBMISSION.md*