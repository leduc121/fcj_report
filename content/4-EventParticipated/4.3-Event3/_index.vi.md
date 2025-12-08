---
title: "AWS Cloud Mastery Series #2"
date: 2025-11-17
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Bài thu hoạch “Devops on aws”

### Mục đích của sự kiện

* Giới thiệu các nguyên tắc cốt lõi của DevOps và tư duy văn hóa thúc đẩy quy trình phát triển phần mềm hiện đại
* Trình diễn cách xây dựng pipeline CI/CD tự động bằng các dịch vụ AWS
* Hướng dẫn triển khai Infrastructure as Code (IaC)
* So sánh các nền tảng container của AWS cho ứng dụng cloud-native
* Chia sẻ best practices về monitoring, observability và khả năng quan sát vận hành

### Diễn Giả

* **Bảo Huỳnh** – AWS Community Builder
* **Thịnh Nguyễn** – AWS Community Builder
* **Vi Trần** – AWS Community Builder

### Những Điểm Nổi Bật

#### Hiểu về Tư Duy DevOps

* Sự hợp tác chặt chẽ giữa team phát triển và vận hành giúp tăng tốc độ release
* Tự động hóa giảm công việc lặp lại và tăng tính nhất quán
* Vòng phản hồi liên tục tạo ra hệ thống ổn định và bền vững hơn

#### Pipeline CI/CD trên AWS

Một pipeline tự động hoàn chỉnh được mô phỏng qua bốn giai đoạn:

* **Source Control**: CodeCommit để lưu trữ và quản lý phiên bản mã
* **Build & Test**: CodeBuild để biên dịch, kiểm thử và đóng gói
* **Deployment**: CodeDeploy hỗ trợ rolling, canary, và blue/green
* **Orchestration**: CodePipeline kết nối và tự động hóa toàn bộ các giai đoạn

Demo trực tiếp cho thấy mỗi lần commit code sẽ tự động kích hoạt build → test → deploy → rollback khi cần.

#### Infrastructure as Code (IaC)

Chuyển từ cấu hình thủ công sang hạ tầng nhất quán và có kiểm soát phiên bản.

* **AWS CloudFormation**

  * Template YAML/JSON dạng khai báo
  * Hỗ trợ parameters, conditions, outputs và định nghĩa tài nguyên
  * Drift detection đảm bảo hạ tầng thực tế trùng khớp với template

* **AWS CDK (Cloud Development Kit)**

  * Tạo hạ tầng bằng TypeScript, Python, Java, và nhiều ngôn ngữ khác
  * Các construct L1/L2/L3 hỗ trợ mẫu tái sử dụng
  * CLI hỗ trợ synth, diff, deploy

Ví dụ cho thấy kiến trúc giống nhau có thể tái tạo hoàn toàn bằng IaC thay vì “ClickOps.”

#### Containers trên AWS

Giới thiệu kiến thức cơ bản về Docker và các tùy chọn container trên AWS:

* **Amazon ECR**: Registry bảo mật, có quét lỗ hổng
* **Amazon ECS**: Orchestration thuần AWS, lựa chọn EC2 hoặc Fargate
* **Amazon EKS**: Kubernetes được quản lý
* **AWS App Runner**: Hosting container đơn giản, ít vận hành

Phần so sánh giúp chọn đúng dịch vụ tùy theo kỹ năng, nhu cầu mở rộng và mô hình ứng dụng.

#### Observability và Monitoring

Các thực hành quan trọng để đảm bảo sức khỏe ứng dụng:

* **Amazon CloudWatch**

  * Metrics, logs, dashboards và alarms

* **AWS X-Ray**

  * Tracing phân tán trong microservices để phát hiện độ trễ và bottleneck

Nhấn mạnh xây dựng dashboard hữu ích, cảnh báo có hành động, và chiến lược giám sát chủ động.

### Những Điểm Rút Ra

#### DevOps Practices

* Tự động hóa tăng tốc độ và độ tin cậy
* Sự đồng bộ giữa dev và ops là yếu tố then chốt
* Sử dụng DORA metrics để cải tiến liên tục
* Tích hợp vòng phản hồi xuyên suốt phát triển

#### Infrastructure as Code

* Giảm cấu hình thủ công trong môi trường production
* CloudFormation mạnh về mô hình khai báo
* CDK linh hoạt nhờ định nghĩa hạ tầng bằng code
* Xem hạ tầng như phần mềm: test, version, automate

#### Application Delivery

* CI/CD giảm lỗi con người và tăng tốc release
* Chọn chiến lược triển khai dựa trên mức độ rủi ro
* Tự động kiểm thử cần có trong mọi giai đoạn pipeline

#### Chiến Lược Container

* Container tăng tính portable, nhất quán và module hóa
* ECS → mô hình vận hành đơn giản
* EKS → hệ sinh thái Kubernetes linh hoạt
* App Runner → mô hình ít vận hành nhất
* ECR là kho trung tâm cho container images

#### Observability

* Kết hợp logs, metrics và traces để có cái nhìn toàn diện
* CloudWatch dashboards + X-Ray service maps giúp troubleshooting dễ dàng hơn
* Xây dựng cảnh báo chủ động thay vì chờ lỗi xảy ra

### Ứng Dụng Vào Công Việc

* **Tự động hóa CI/CD** bằng CodePipeline, CodeBuild, CodeDeploy
* **Triển khai IaC** với CloudFormation hoặc CDK cho môi trường nhất quán
* **Containerize ứng dụng** và chọn ECS, EKS hoặc App Runner tùy theo nhu cầu
* **Tăng cường observability** với CloudWatch metrics, logs, dashboards, alarms
* **Dùng AWS X-Ray** để trace và debug hệ thống phân tán
* **Áp dụng DORA metrics** để đo hiệu suất và cải tiến DevOps

### Trải Nghiệm Sự Kiện

Tham dự **“Cloud Mastery Series #2 – DevOps on AWS”** mang lại cả góc nhìn chiến lược lẫn kiến thức thực tiễn để áp dụng DevOps hiệu quả trên môi trường cloud.

#### Học từ các diễn giả có chuyên môn cao

* Giải thích rõ ràng và chi tiết về CI/CD, container, IaC và monitoring
* Các ví dụ thực tế cho thấy DevOps vận hành trong hệ thống production như thế nào

#### Thực hành kỹ thuật

* Quan sát CI/CD end-to-end từ commit → build → deploy
* Hiểu cách CloudFormation và CDK đảm bảo hạ tầng nhất quán
* Hiểu trade-offs giữa ECS, EKS và App Runner

#### Tận dụng công cụ hiện đại

* IaC đảm bảo tính nhất quán và loại bỏ sai lệch cấu hình
* CloudWatch và X-Ray là nền tảng cho vận hành xuất sắc

#### Networking và thảo luận

* Gặp gỡ các chuyên gia và cộng đồng
* Trao đổi sâu về văn hóa DevOps, tự động hóa và cải tiến liên tục

#### Bài học rút ra

* Tự động hóa là yếu tố sống còn khi mở rộng hệ thống
* Observability là nền tảng cho độ tin cậy
* Chọn đúng nền tảng container giúp giảm gánh nặng vận hành

#### Một số hình ảnh sự kiện

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo1.jpg" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo2.jpg" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_3/photo3.jpg" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

> Nhìn chung, workshop mang lại sự hiểu biết rõ ràng và thực tiễn về văn hóa DevOps, quy trình CI/CD, IaC, triển khai container và observability — tất cả đều là những thành phần quan trọng của phát triển ứng dụng cloud-native hiện đại.
