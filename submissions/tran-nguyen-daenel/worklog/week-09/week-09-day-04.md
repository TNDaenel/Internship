Worklog - Ngày 11/09/2025
📅 Thông tin cơ bản
Ngày: 11/09/2025

Thứ: Thứ Năm

Tuần thực tập: Tuần thứ 9/10

Thời gian làm việc: 8:30 - 17:30

Mood: 💥 Breaking things systematically.

🎯 Mục tiêu ngày hôm nay
[x] Hiểu cách sử dụng AWS Fault Injection Simulator (FIS).

[x] Tạo một "Experiment Template" trong FIS.

[x] Thiết kế một thí nghiệm chaos tự động để kiểm tra khả năng tự phục hồi của EKS.

[x] Chạy thí nghiệm và phân tích kết quả.

💼 Công việc đã thực hiện
1. Introduction to AWS FIS ⏱️ 3 giờ
Mô tả: Tìm hiểu cách FIS cho phép thực hiện các thí nghiệm chaos một cách an toàn và có kiểm soát trên các tài nguyên AWS. Khám phá các "actions" có sẵn (ví dụ: aws:eks:pod-delete).

Kết quả: Nắm được cách tạo một thí nghiệm với Target, Action, và Stop Condition.

2. Automated Chaos Experiment with FIS ⏱️ 5 giờ
Mô tả:

Tạo một IAM Role cho FIS để có quyền thực hiện các hành động trên EKS.

Tạo một Experiment Template:

Target: Chọn 50% số Pods trong NodeJS deployment.

Action: aws:eks:pod-delete.

Stop Condition: Một CloudWatch Alarm đã tạo trước đó, sẽ kích hoạt nếu tỷ lệ lỗi 5xx trên ALB vượt quá 5%.

Chạy thí nghiệm.

Kết quả:

FIS tự động xóa 50% số Pods.

Quan sát thấy HPA và Deployment controller của Kubernetes ngay lập tức hành động để tạo lại các Pods đã mất.

Alarm "Stop Condition" không bị kích hoạt, cho thấy hệ thống phục hồi tốt.

Tools/Tech: AWS Fault Injection Simulator (FIS), Amazon EKS, CloudWatch Alarms.

Links: [FIS Experiment Template JSON]

📚 Kiến thức học được
🔧 Technical Skills
AWS Services: AWS Fault Injection Simulator (FIS).

DevOps: Resilience Engineering, Automated chaos testing.

💡 Concepts & Theory
New Concepts: Fault Injection, Blast Radius, Stop Conditions.

🚧 Khó khăn và giải pháp
Vấn đề 1: Khó thiết lập Stop Condition một cách hiệu quả.

Mô tả: Nếu alarm quá nhạy, thí nghiệm sẽ dừng lại trước khi có tác động. Nếu quá lỏng lẻo, có thể gây ảnh hưởng đến hệ thống.

Solution: Bắt đầu với một ngưỡng an toàn và quan sát. Dựa trên kết quả, tinh chỉnh lại ngưỡng của CloudWatch Alarm cho các lần thí nghiệm sau để nó phản ánh đúng "trạng thái không thể chấp nhận được" của hệ thống.

Lesson: Stop Condition chính là "cầu chì an toàn" của Chaos Engineering, việc thiết lập nó đúng đắn là cực kỳ quan trọng.

💭 Reflection & Insights
Key Insights: AWS FIS đưa Chaos Engineering từ một hoạt động thủ công, rủi ro thành một quy trình tự động, có thể lặp lại và an toàn. Nó biến khả năng phục hồi từ một "hy vọng" thành một "đặc tính có thể kiểm chứng".

📋 Kế hoạch ngày mai
High: Hoàn thiện tài liệu và tạo Go-Live Checklist.

Medium: Viết báo cáo tổng kết tuần.

