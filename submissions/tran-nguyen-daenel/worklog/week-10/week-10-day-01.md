Worklog - Ngày 15/09/2025
📅 Thông tin cơ bản
Ngày: 15/09/2025

Thứ: Thứ Hai

Tuần thực tập: Tuần thứ 10/10

Thời gian làm việc: 8:00 - 17:30

Mood: 🚀 The big day! Sẵn sàng cho ngày ra mắt.

🎯 Mục tiêu ngày hôm nay
[x] Thực hiện các bước kiểm tra cuối cùng trong Go-Live Checklist.

[x] Thực hiện chuyển đổi DNS (DNS Cutover) để trỏ traffic người dùng thật vào hệ thống EKS mới.

[x] Thực hiện các bài kiểm tra "smoke test" trên môi trường production.

[x] Bắt đầu giai đoạn "Hypercare" - giám sát chặt chẽ.

💼 Công việc đã thực hiện
1. Pre-flight Checks ⏱️ 2.5 giờ
Mô tả:

Họp nhanh với team (giả định) để rà soát lại kế hoạch.

Đi qua từng hạng mục trong Go-Live Checklist đã tạo ở Tuần 9: kiểm tra lại cấu hình HPA, trạng thái của cluster, health check của ALB, và kế hoạch rollback.

Kết quả:

Toàn đội ngũ đã thống nhất và sẵn sàng. 100% các mục trong checklist tiền triển khai được xác nhận là OK.

Tools/Tech: Go-Live Checklist (Notion/Google Sheets).

2. Go-Live: DNS Cutover ⏱️ 1.5 giờ
Mô tả:

Thông báo bắt đầu "cửa sổ bảo trì" (simulated).

Truy cập Amazon Route 53.

Thay đổi bản ghi CNAME của tên miền www.my-ecommerce.com từ Load Balancer cũ sang DNS name của Application Load Balancer mới do EKS quản lý.

Giảm giá trị TTL của bản ghi DNS xuống mức thấp (ví dụ: 60 giây) trước đó vài giờ để đảm bảo thay đổi được cập nhật nhanh chóng.

Kết quả:

DNS cutover được thực hiện thành công.

Tools/Tech: Amazon Route 53.

Links: [Screenshot: Route 53 record change]

3. Post-launch Validation & Hypercare ⏱️ 4 giờ
Mô tả:

Sau khi chuyển đổi DNS, liên tục sử dụng các công cụ như dnschecker.org để kiểm tra quá trình cập nhật DNS trên toàn cầu.

Chạy một bộ smoke test đã được chuẩn bị sẵn để kiểm tra các luồng nghiệp vụ quan trọng (đăng nhập, xem sản phẩm, thêm vào giỏ hàng).

Bắt đầu theo dõi chặt chẽ các dashboard đã tạo.

Kết quả:

Hệ thống mới đang phục vụ traffic của người dùng thật.

Các chức năng chính hoạt động ổn định.

Tools/Tech: dig, nslookup, Postman/Newman.

📚 Kiến thức học được
🔧 Technical Skills
AWS Services: Amazon Route 53.

DevOps: Go-Live procedures, DNS Cutover, Smoke Testing.

💡 Concepts & Theory
New Concepts: DNS Propagation, Time-To-Live (TTL), Hypercare.

🚧 Khó khăn và giải pháp
Vấn đề 1: DNS Propagation Delay

Mô tả: Một số người dùng vẫn có thể truy cập vào hệ thống cũ do các DNS server trên thế giới chưa cập nhật bản ghi mới.

Solution: Chấp nhận rằng đây là đặc tính của DNS. Việc giảm TTL trước khi chuyển đổi đã giúp giảm thiểu thời gian này. Sử dụng các công cụ kiểm tra DNS để theo dõi tiến độ.

Lesson: Lên kế hoạch cho việc chuyển đổi DNS là cực kỳ quan trọng.

💭 Reflection & Insights
Key Insights: Ngày Go-Live là đỉnh cao của nhiều tuần chuẩn bị. Sự thành công của nó không đến từ may mắn, mà đến từ một kế hoạch chi tiết, các bài kiểm thử toàn diện, và một kế hoạch rollback rõ ràng.

📋 Kế hoạch ngày mai
High: Hypercare - Giám sát chặt chẽ hệ thống trong ngày làm việc đầu tiên có đầy đủ traffic.

Medium: Phản ứng với các cảnh báo (nếu có).

📊 Self Assessment
Productivity: 10/10

Learning: 9/10

Overall Satisfaction: 10/10