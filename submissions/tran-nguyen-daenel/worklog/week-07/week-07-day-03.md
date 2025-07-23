Worklog - Ngày 25/06/2025
📅 Thông tin cơ bản
Ngày: 25/06/2025

Thứ: Thứ Tư

Tuần thực tập: Tuần thứ 7/10

Thời gian làm việc: 8:00 - 18:00

Mood: 🌉 Building bridges to the outside world.

🎯 Mục tiêu ngày hôm nay
[x] Hiểu vai trò của Kubernetes Service (ClusterIP) trong việc giao tiếp nội bộ.

[x] Triển khai AWS Load Balancer Controller vào EKS cluster.

[x] Sử dụng một Ingress resource để tự động tạo một Application Load Balancer.

[x] Truy cập ứng dụng NodeJS thành công từ internet.

💼 Công việc đã thực hiện
1. Internal Communication with Services ⏱️ 2 giờ
Mô tả: Viết một file nodejs-service.yaml với kind: Service và type: ClusterIP. Sử dụng selector để trỏ Service đến các Pods của NodeJS.

Kết quả: Service được tạo ra với một tên DNS nội bộ ổn định (nodejs-service).

2. AWS Load Balancer Controller Setup ⏱️ 4.5 giờ
Mô tả: Thực hiện quy trình cài đặt phức tạp theo hướng dẫn của AWS, bao gồm: tạo IAM OIDC provider, tạo IAM Role cho Service Account (IRSA), và cài đặt controller bằng Helm.

Kết quả: AWS Load Balancer Controller pod đang chạy thành công trong kube-system namespace.

Tools/Tech: AWS Load Balancer Controller, IAM OIDC Provider, IRSA, Helm.

3. Exposing Traffic with Ingress ⏱️ 2 giờ
Mô tả: Viết file api-ingress.yaml. Định nghĩa rule để trỏ traffic từ /api/* đến nodejs-service. Thêm các annotation cần thiết (kubernetes.io/ingress.class: alb). kubectl apply file manifest.

Kết quả: Một Application Load Balancer được tự động tạo ra trên AWS. Truy cập vào DNS name của ALB và API hoạt động.

Tools/Tech: Kubernetes Ingress.

Links: [api-ingress.yaml on GitHub]

📚 Kiến thức học được
🔧 Technical Skills
Kubernetes: Services (ClusterIP), Ingress.

AWS Integration: AWS Load Balancer Controller, IRSA.

DevOps Tools: Helm.

💡 Concepts & Theory
New Concepts: Service Discovery in K8s, Ingress Controller, North-South traffic.

🚧 Khó khăn và giải pháp
Vấn đề 1: Sau khi tạo Ingress, ALB được tạo ra nhưng Target Group không có target nào.

Solution: Gỡ lỗi bằng kubectl describe ingress và kubectl logs <controller-pod>. Phát hiện ra Security Group của Worker Node chưa cho phép traffic từ Security Group của ALB. Cần phải tag các Subnet một cách chính xác (kubernetes.io/role/elb) để controller có thể tự động cấu hình Security Group.

💭 Reflection & Insights
Key Insights: AWS Load Balancer Controller là "cầu nối" mạnh mẽ giữa thế giới Kubernetes và các dịch vụ native của AWS, cho phép tận dụng sức mạnh của ALB một cách tự động.

📋 Kế hoạch ngày mai
High: Di chuyển nốt ứng dụng ReactJS.

Medium: Tìm hiểu về ConfigMaps và Secrets trong Kubernetes.