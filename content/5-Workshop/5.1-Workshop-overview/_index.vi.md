---
title : "Giới thiệu"
date: 2025-09-09
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---


# WEBSITE TRỰC TUYẾN THEO DÕI VÀ DỰ BÁO QUỸ ĐẠO BÃO

Trong workshop này, nhóm chúng em trình bày cách xây dựng một nền tảng trực tuyến cho phép người dùng truy cập Internet có thể tự do kiểm tra, theo dõi và thậm chí dự đoán đường đi của các cơn bão đang hoạt động tại khu vực Tây Thái Bình Dương. Nền tảng này giúp người dùng chủ động chuẩn bị cho các thảm họa tự nhiên sắp xảy ra và giảm thiểu thiệt hại tiềm tàng.

Nền tảng cung cấp **hai chức năng chính**:

1. **Hiển thị các cơn bão gần nhất** – Cho phép người dùng xem đường đi, cường độ, tốc độ gió và các đặc điểm khác của bão gần đây trong khu vực Tây Thái Bình Dương.

2. **Dự đoán quỹ đạo bão** – Cho phép người dùng nhập dữ liệu vị trí bão trong quá khứ (vĩ độ và kinh độ; tối thiểu 9 điểm dữ liệu) để hệ thống dự đoán hướng di chuyển trong tương lai.

Phân chia theo tiến độ, chúng em sẽ lần lượt trình bày về **bộ dữ liệu**, **quá trình tiền xử lý**, **pipeline huấn luyện mô hình**, và quá trình **xây dựng nền tảng trực tuyến bằng các dịch vụ AWS**. Chúng em cũng sẽ perform kỹ thuật tăng cường dữ liệu được đề xuất — **Stepwise Temporal Fading Augmentation (STFA)** cùng với việc ứng dụng **machine learning dựa trên các quy tắc vật lý (physics-informed ML)**. Những phương pháp này giúp dữ liệu huấn luyện trở nên chân thực hơn và cải thiện đáng kể độ chính xác trong dự đoán đường đi bão, tuổi thọ bão và tổng quãng đường di chuyển.

<p align="center">
  <img src="/images/5-Workshop/5.1-Workshop-overview/trainning_process.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1 : Pipeline mô hình</strong>
</p>

Sau khi hoàn thành quá trình huấn luyện mô hình, chúng em triển khai xây dựng nền tảng trực tuyến bằng **kiến trúc serverless**. Đây là kiến trúc tiết kiệm chi phí, có khả năng mở rộng tốt, và dễ dàng bảo trì/triển khai — rất phù hợp cho mục tiêu dự án. Dưới đây là các dịch vụ AWS chính được sử dụng:

* **AWS Lambda** – Chạy các mô hình ML và xử lý logic phía backend
* **Amazon S3** – Lưu trữ file tĩnh, mô hình ML và dữ liệu bão
* **Amazon API Gateway** – Định tuyến và phần luồng các yêu cầu của người dùng đến Lambda phù hợp, tùy theo việc họ xem dữ liệu bão gần đây hay chạy dự đoán
* **Amazon CloudFront** – Tăng tốc phân phối nội dung thông qua các edge location
* **AWS Secrets Manager** – Lưu trữ khóa API và các thông tin nhạy cảm
* **…** – Các dịch vụ hỗ trợ bổ sung khác khi cần

<p align="center">
  <img src="/images/5-Workshop/5.1-Workshop-overview/skynet_diagram.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2 : Kiến trúc nền tảng</strong>
</p>
