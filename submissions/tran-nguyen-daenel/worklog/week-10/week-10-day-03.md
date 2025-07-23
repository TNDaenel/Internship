Worklog - Ngày 17/09/2025
📅 Thông tin cơ bản
Ngày: 17/09/2025

Thứ: Thứ Tư

Tuần thực tập: Tuần thứ 10/10

Thời gian làm việc: 8:30 - 17:30

Mood: 📈 Data analysis and reflection.

🎯 Mục tiêu ngày hôm nay
[x] Thực hiện bài đánh giá chi phí đầu tiên sau 48h go-live bằng Cost Explorer.

[x] Thực hiện bài đánh giá hiệu năng dựa trên dữ liệu thực tế.

[x] So sánh chi phí/hiệu năng thực tế với dự kiến.

[x] Lên danh sách các điểm cần tối ưu hóa trong chu kỳ tiếp theo.

💼 Công việc đã thực hiện
1. Post-Launch Cost Review ⏱️ 4 giờ
Mô tả:

Vào AWS Cost Explorer, lọc theo khoảng thời gian 2 ngày vừa qua.

Phân tích chi phí của các dịch vụ chính: EKS, EC2 (worker nodes), ALB, NAT Gateway, Data Transfer.

So sánh chi phí ước tính của EKS với chi phí của hệ thống ECS cũ.

Kết quả: Một báo cáo chi phí ban đầu, cho thấy chi phí vận hành EKS có thể cao hơn một chút do Control Plane fee nhưng chi phí worker node có thể được tối ưu tốt hơn với Karpenter/Spot.

Tools/Tech: AWS Cost Explorer.

2. Post-Launch Performance Review ⏱️ 4 giờ
Mô tả:

Xem lại các dashboard hiệu năng.

Phân tích HPA đã hoạt động hiệu quả như thế nào.

Tìm ra các endpoint có độ trễ cao nhất dưới tải trọng thực.

Kết quả:

Xác nhận hệ thống đáp ứng được các mục tiêu về hiệu năng (SLOs).

Lập ra một danh sách các "món nợ kỹ thuật" (technical debt) và các điểm cần tối ưu cho phiên bản V2.

Tools/Tech: CloudWatch, Grafana, X-Ray.

📚 Kiến thức học được
🔧 Technical Skills
DevOps: FinOps, Performance Management in production.

💡 Concepts & Theory
New Concepts: Day-2 Operations, Continuous Optimization.

💭 Reflection & Insights
Key Insights: Dữ liệu thực tế từ người dùng luôn mang lại những insight quý giá nhất mà không một bài kiểm thử nào có thể mô phỏng hoàn hảo. Vòng lặp Build -> Measure -> Learn không kết thúc sau khi ra mắt, mà nó chỉ mới thực sự bắt đầu.

📋 Kế hoạch ngày mai
High: Hoàn thiện toàn bộ tài liệu kiến trúc và vận hành.

Medium: Chuẩn bị cho buổi Project Retrospective.