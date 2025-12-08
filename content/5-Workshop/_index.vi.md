---
title: "Workshop"
date: 2025-09-12
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# NỀN TẢNG THEO DÕI VÀ DỰ ĐOÁN BÃO

## Tổng Quan

Bão là một trong những thảm họa thiên nhiên nguy hiểm nhất, có thể gây thiệt hại nghiêm trọng đến cơ sở hạ tầng và đe dọa tính mạng con người. Việc phát hiện sớm và đưa ra cảnh báo kịp thời là vô cùng quan trọng để người dân trong khu vực bị ảnh hưởng có đủ thời gian chuẩn bị và sơ tán an toàn.

Để đáp ứng nhu cầu này, dự án của chúng tôi hướng đến việc xây dựng một nền tảng trực tuyến cho phép người dùng truy cập miễn phí vào thông tin về những cơn bão mới nhất ở khu vực Tây Thái Bình Dương, sử dụng dữ liệu từ NOAA (Cơ quan Quản lý Khí quyển và Đại dương Quốc gia Hoa Kỳ) — một nguồn dữ liệu đáng tin cậy. Bên cạnh đó, sinh viên, nhà khí tượng hoặc bất kỳ ai quan tâm đến động lực học của bão đều có thể tương tác với hệ thống bằng cách cung cấp quỹ đạo đầu vào của riêng họ và nhận về dự đoán được tạo bởi mô hình học máy của chúng tôi.

Workshop này trình bày toàn bộ quy trình xây dựng mô hình dự báo bão, bao gồm nhiều kỹ thuật chuỗi thời gian mới — **Stepwise Temporal Fading** và **Plausible Geodesic-Aware Augmentation** — cùng với phần giải thích chi tiết từng bước về cách chúng tôi xây dựng và triển khai nền tảng từ con số 0.

Với sự hỗ trợ của các dịch vụ AWS như **Amazon S3, AWS Lambda, API Gateway và CloudFront**, chúng tôi xây dựng một kiến trúc hoàn toàn serverless. Giải pháp này mang lại sự đơn giản, khả năng mở rộng linh hoạt và hiệu quả chi phí dài hạn, đồng thời đảm bảo hiệu suất ổn định và phản hồi nhanh.

<p align="center">
  <img src="/images/5-Workshop/5.1-Workshop-overview/skynet_diagram.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Kiến trúc Nền tảng</strong>
</p>

Nền tảng cuối cùng cung cấp hai chức năng cốt lõi:

1. **Xem Thông Tin Bão**
   Người dùng có thể khám phá thông tin mới nhất về các cơn bão ở Tây Thái Bình Dương, bao gồm quỹ đạo lịch sử, tốc độ gió, nhiệt độ và các thông số liên quan khác.

2. **Dự Đoán Quỹ Đạo Bão**
   Người dùng có thể nhập quỹ đạo một phần của cơn bão và nhận về dự đoán quãng đường tiếp theo được tạo bởi mô hình đã huấn luyện.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị dữ liệu](5.2-Data_Preparation/)
3. [Kiến tạo mô hình ML](5.3-ML-Model/)
4. [Kiến trúc Front&Back-end](5.4-Front&Back-end/)
5. [API](5.5-Platform-API/)
