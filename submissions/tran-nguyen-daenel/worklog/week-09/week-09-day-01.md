Worklog - Ngày 08/09/2025
📅 Thông tin cơ bản
Ngày: 08/09/2025

Thứ: Thứ Hai

Tuần thực tập: Tuần thứ 9/10

Thời gian làm việc: 8:00 - 17:30

Mood: 🚦 Bắt đầu tuần kiểm thử, sẵn sàng tìm ra giới hạn của hệ thống.

🎯 Mục tiêu ngày hôm nay
[x] Tìm hiểu các khái niệm về Performance Testing (Load, Stress, Soak testing).

[x] Cài đặt và làm quen với công cụ Load Testing hiện đại: k6.

[x] Viết một kịch bản kiểm thử (test script) đơn giản bằng JavaScript cho k6.

[x] Chạy bài kiểm thử tải đầu tiên để xác định hiệu năng cơ bản (baseline).

💼 Công việc đã thực hiện
1. Performance Testing Fundamentals ⏱️ 3 giờ
Mô tả:

Nghiên cứu sự khác biệt giữa các loại kiểm thử hiệu năng.

Tìm hiểu các chỉ số quan trọng cần đo lường: requests per second (RPS), response time (p95, p99), error rate.

Cài đặt k6 và thực hành các ví dụ cơ bản.

Kết quả:

Nắm vững lý thuyết và chọn k6 làm công cụ chính do khả năng viết script bằng JavaScript và hiệu năng cao.

Tools/Tech: k6, Performance Testing concepts.

2. Writing and Running the First Load Test ⏱️ 5 giờ
Mô tả:

Viết một file load-test.js.

Kịch bản mô phỏng 50 người dùng ảo (Virtual Users) truy cập đồng thời trong 5 phút.

Các VUs thực hiện các hành động: truy cập trang chủ, xem danh sách sản phẩm, xem chi tiết một sản phẩm.

Chạy test và phân tích kết quả output trên terminal.

Kết quả:

Một bài kiểm thử tải có thể lặp lại đã được tạo.

Có được các chỉ số hiệu năng cơ bản của hệ thống dưới tải trọng vừa phải.

Phát hiện độ trễ tăng nhẹ khi số lượng VUs tăng lên.

Tools/Tech: k6, JavaScript.

Links: [load-test.js script on GitHub]

📚 Kiến thức học được
🔧 Technical Skills
DevOps Tools: k6.

Programming: Viết kịch bản kiểm thử bằng JavaScript.

DevOps: Load Testing, Performance Benchmarking.

💡 Concepts & Theory
New Concepts: Virtual Users (VUs), Requests Per Second (RPS), Response Time Percentiles (p95, p99).

🚧 Khó khăn và giải pháp
Vấn đề 1: Kịch bản test quá đơn giản, không phản ánh đúng hành vi người dùng.

Mô tả: Ban đầu chỉ test một API endpoint duy nhất.

Solution: Cải tiến script để mô phỏng một "user journey" hoàn chỉnh. Thêm các khoảng sleep ngẫu nhiên giữa các request để làm cho kịch bản thực tế hơn.

Result: Kết quả kiểm thử trở nên đáng tin cậy hơn.

Lesson: Một kịch bản kiểm thử tốt cũng quan trọng như chính công cụ kiểm thử.

💭 Reflection & Insights
Key Insights: Load testing là cách duy nhất để trả lời câu hỏi "Hệ thống của chúng ta chịu được bao nhiêu tải?". Nó giúp chuyển các cuộc tranh luận về hiệu năng từ "tôi nghĩ là" thành "dữ liệu cho thấy là".

📋 Kế hoạch ngày mai
High: Phân tích kết quả load test và tinh chỉnh hệ thống.

Medium: Tìm hiểu và cấu hình Horizontal Pod Autoscaler (HPA) trong Kubernetes.

📊 Self Assessment
Productivity: 8/10

Learning: 9/10

Overall Satisfaction: 9/10