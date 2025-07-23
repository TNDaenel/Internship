Worklog - Ngày 16/09/2025
📅 Thông tin cơ bản
Ngày: 16/09/2025

Thứ: Thứ Ba

Tuần thực tập: Tuần thứ 10/10

Thời gian làm việc: 8:00 - 17:30

Mood: 👀 Vigilant and watchful.

🎯 Mục tiêu ngày hôm nay
[x] Giám sát chặt chẽ các dashboard trong suốt ngày làm việc đầu tiên có traffic đầy đủ.

[x] Phân tích các chỉ số hiệu năng và lỗi ban đầu.

[x] Thực hành quy trình phản ứng sự cố (Incident Response) nếu có cảnh báo.

[x] Thu thập dữ liệu ban đầu về chi phí và hiệu năng.

💼 Công việc đã thực hiện
1. Hypercare Monitoring ⏱️ 6 giờ
Mô tả: Dành phần lớn thời gian trong ngày để theo dõi các dashboard chính:

Dashboard CloudWatch (ALB Latency, Error rates, ECS/EKS CPU/Mem).

Dashboard Grafana/Prometheus (nếu có).

ArgoCD UI để đảm bảo trạng thái luôn là Synced.

X-Ray Service Map để xem có điểm nghẽn bất thường không.

Kết quả: Hệ thống hoạt động ổn định dưới tải trọng thực tế. HPA đã tự động scale up/down một vài lần trong ngày.

2. Initial Data Analysis ⏱️ 2.5 giờ
Mô tả:

Bắt đầu xem xét các log truy cập của ALB và log ứng dụng để tìm các pattern sử dụng của người dùng thật.

Ghi nhận các chỉ số hiệu năng cơ bản (p95, p99 latency) dưới tải trọng thực.

Kết quả: Có được một bộ dữ liệu ban đầu để so sánh và tối ưu trong tương lai.

📚 Kiến thức học được
🔧 Technical Skills
DevOps: Hypercare, Incident Response (khái niệm).

Monitoring: Real-time monitoring, Anomaly detection.

🚧 Khó khăn và giải pháp
Vấn đề 1: Một CloudWatch Alarm về "CPU Utilization cao" được kích hoạt.

Mô tả: Một cảnh báo được gửi đi khi CPU của một service tăng đột biến.

Solution: Thực hiện theo runbook đã viết:

Kiểm tra X-Ray để xem có request nào bất thường không.

Kiểm tra log của service đó để tìm lỗi.

Kiểm tra HPA để xem nó có đang scale up không.
=> Kết luận: Đây là một lần scale-up hợp lệ do có lượng truy cập tăng, không phải là sự cố. Tinh chỉnh lại ngưỡng của alarm để tránh cảnh báo sai trong tương lai.

Lesson: Một hệ thống cảnh báo tốt cần được tinh chỉnh liên tục dựa trên hành vi thực tế.

💭 Reflection & Insights
Key Insights: Giai đoạn Hypercare sau khi ra mắt cũng quan trọng như chính ngày ra mắt. Đây là lúc để xác nhận các giả định thiết kế và thu thập dữ liệu thực tế để tối ưu.

📋 Kế hoạch ngày mai
High: Thực hiện bài đánh giá chi phí và hiệu năng đầu tiên sau khi có đủ dữ liệu.

Medium: Bắt đầu viết các phần đầu tiên của báo cáo tổng kết dự án.