---
title: "Worklog Tuần 12"
date: "2025-11-25"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---



### Mục tiêu Tuần 12: 🎯

* Hiểu rõ và ứng dụng dịch vụ **Amazon CloudFront** để phân phối nội dung (CDN).
* Nắm vững quy trình **Đóng gói ứng dụng (Containerization)** với **Docker** và triển khai **Docker Image**.
* **Triển khai thành công trang web chính thức** lên môi trường Production sử dụng CloudFront.

---

### Các công việc thực hiện trong tuần:
| Ngày | Nhiệm vụ | Ngày Bắt đầu | Ngày Hoàn thành | Tài liệu Tham khảo |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | -------------------------------------------------------------------------------------- |
| 1-2 | - **Nghiên cứu & Học tập về Amazon CloudFront** <br>&emsp; + Khái niệm, cơ chế hoạt động, và lợi ích của CDN. <br>&emsp; + **CloudFront Distribution** (Web/RTMP), **Origin**, **Cache Behavior**. <br>&emsp; + Cấu hình **Tên miền** và **Chứng chỉ SSL** cho CloudFront. | 25/11/2025 | 26/11/2025 | <https://000094.awsstudygroup.com/> |
| 3-4 | - **Chuẩn bị và Triển khai Docker Image** <br>&emsp; + Học về **Dockerfile** và quy trình xây dựng **Docker Image**. <br>&emsp; + Thực hành tạo Image cho ứng dụng web. <br>&emsp; + **Đẩy Image** lên **Amazon ECR** hoặc **Docker Hub** (chuẩn bị cho triển khai). | 27/11/2025 | 28/11/2025 | <https://000015.awsstudygroup.com/6-docker-image/> |
| 5-7 | - **Triển khai Trang web Chính thức qua CloudFront** <br>&emsp; + Cấu hình **Origin** (ví dụ: S3 Bucket, EC2/ALB) cho CloudFront Distribution. <br>&emsp; + Thực hiện **Triển khai hoàn chỉnh (Full Deployment)** của trang web chính thức. <br>&emsp; + Kiểm tra và xác minh hoạt động của trang web tại **d3lj47ilp0fgxy.cloudfront.net**. | 29/11/2025 | 01/12/2025 | <https://d3lj47ilp0fgxy.cloudfront.net> (Mục tiêu triển khai) |

---

### Thành tựu Tuần 12: ✅

* Đã **nghiên cứu chuyên sâu** và **hiểu rõ** về **Amazon CloudFront** và vai trò của nó như một **Mạng lưới Phân phối Nội dung (CDN)**.
* Nắm được các thành phần cốt lõi của CloudFront Distribution như **Origin** và **Cache Behavior**.
* Thành công trong việc học và thực hành quy trình **Đóng gói ứng dụng** bằng **Docker**.
* Đã **xây dựng thành công Docker Image** cho ứng dụng web theo tài liệu tham khảo.
* **Hoàn thành việc triển khai (Deployment)** trang web chính thức lên môi trường Production, có thể truy cập qua: **d3lj47ilp0fgxy.cloudfront.net**.
* Tích lũy kinh nghiệm trong việc phối hợp giữa các dịch vụ **AWS** (ví dụ: S3/EC2/Load Balancer) và **CloudFront** để tối ưu hóa hiệu suất và bảo mật.

---