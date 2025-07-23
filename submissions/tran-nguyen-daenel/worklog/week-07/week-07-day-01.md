Worklog - Ngày 23/06/2025
📅 Thông tin cơ bản
Ngày: 23/06/2025

Thứ: Thứ Hai

Tuần thực tập: Tuần thứ 7/10

Thời gian làm việc: 8:00 - 17:30

Mood: ☸️ Bắt đầu một hành trình mới đầy thử thách với Kubernetes.

🎯 Mục tiêu ngày hôm nay
[x] Hiểu các khái niệm cốt lõi của Kubernetes: Cluster, Node, Pod, Deployment, Service.

[x] Cài đặt các công cụ dòng lệnh cần thiết: kubectl, eksctl.

[x] Sử dụng eksctl để tạo một Amazon EKS (Elastic Kubernetes Service) cluster.

[x] Kết nối thành công tới cluster bằng kubectl để xác nhận.

💼 Công việc đã thực hiện
1. Kubernetes Core Concepts Deep Dive ⏱️ 3.5 giờ
Mô tả:

Hoàn thành module "Introduction to Kubernetes" trên trang chủ Kubernetes.io.

Vẽ sơ đồ kiến trúc thể hiện mối quan hệ giữa các thành phần Control Plane (API Server, etcd, scheduler, controller manager) và Worker Nodes (kubelet, kube-proxy).

Phân biệt rõ sự khác nhau giữa Pod (đơn vị nhỏ nhất để chạy container) và Deployment (công cụ quản lý vòng đời của Pods).

Kết quả:

Nắm vững các thuật ngữ và kiến trúc cơ bản của Kubernetes, tạo nền tảng vững chắc cho các ngày tiếp theo.

Tools/Tech: Kubernetes Documentation, Draw.io.

Links: [K8s Architecture Diagram]

2. EKS Cluster Creation with eksctl ⏱️ 4.5 giờ
Mô tả:

Cài đặt phiên bản mới nhất của kubectl và eksctl.

Viết một file config cluster.yaml cho eksctl, định nghĩa tên cluster (e-commerce-prod), region, phiên bản Kubernetes, và cấu hình cho managed node group (ví dụ: 2 instance t3.medium).

Chạy lệnh eksctl create cluster -f cluster.yaml và theo dõi quá trình CloudFormation tạo tài nguyên.

Sau khi cluster được tạo, eksctl tự động cập nhật file ~/.kube/config.

Chạy lệnh kubectl get nodes để xác nhận kết nối và thấy 2 worker node ở trạng thái Ready.

Kết quả:

Một EKS cluster hoàn chỉnh đang chạy trên AWS.

Môi trường đã sẵn sàng để triển khai ứng dụng.

Tools/Tech: Amazon EKS, eksctl CLI, kubectl CLI, YAML.

Links: [cluster.yaml configuration file on GitHub]

📚 Kiến thức học được
🔧 Technical Skills
AWS Services: Amazon EKS.

DevOps Tools: kubectl, eksctl.

Architecture: Kubernetes Control Plane vs. Data Plane.

IaC: Sử dụng eksctl như một công cụ IaC cấp cao.

💡 Concepts & Theory
New Concepts: Container Orchestration, Pods, Deployments, Services, Nodes.

Best Practices: Sử dụng Managed Node Groups để đơn giản hóa việc quản lý.

🚧 Khó khăn và giải pháp
Vấn đề 1: Lệnh eksctl create cluster thất bại với lỗi liên quan đến IAM.

Mô tả: eksctl không thể tạo các CloudFormation stack cần thiết do user IAM không đủ quyền.

Solution: Đọc kỹ log lỗi. Gắn các policy cần thiết (ví dụ: AdministratorAccess cho môi trường lab, hoặc các policy chi tiết hơn cho môi trường production) cho user/role đang sử dụng để chạy eksctl.

Lesson: eksctl tạo ra rất nhiều tài nguyên AWS (VPC, IAM Roles, EC2, ASG...), do đó nó đòi hỏi một bộ quyền hạn rất rộng.

💭 Reflection & Insights
What went well today?: Việc cài đặt công cụ và tạo cluster diễn ra khá suôn sẻ nhờ eksctl.

Key Insights: Kubernetes là một "hệ điều hành cho cloud". eksctl là một công cụ tuyệt vời giúp trừu tượng hóa sự phức tạp của việc thiết lập một cluster K8s trên AWS.

📋 Kế hoạch ngày mai
High: Triển khai ứng dụng container đầu tiên lên EKS bằng Kubernetes Deployments.

Medium: Viết file manifest YAML đầu tiên cho ứng dụng NodeJS.

📊 Self Assessment
Productivity: 8/10 (Thời gian chờ tạo cluster khá lâu).

Learning: 9/10.