---
title: "Worklog Tuần 4"
date: "2024-09-29"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Học về dịch vụ VM Import/Export trong AWS
* Hiểu về Amazon RDS (Relational Database Service)
* Thực hành quản lý database trong AWS

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 1   | - Học về AWS VM Import/Export: <br>&emsp; + Các định dạng được hỗ trợ (OVA, VMDK, VHD) <br>&emsp; + Yêu cầu và IAM roles <br>&emsp; + Use cases cho migration | 29/09/2024 | 29/09/2024      | <https://000014.awsstudygroup.com/> |
| 2   | - Tiếp tục VM Import/Export: <br>&emsp; + Chuẩn bị VM để import <br>&emsp; + Upload lên S3 <br> - **Thực hành:** Import VM image lên AWS | 30/09/2024 | 30/09/2024      | <https://000014.awsstudygroup.com/> |
| 3   | - Học Amazon RDS cơ bản: <br>&emsp; + Database engines (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server) <br>&emsp; + DB instances và instance classes <br>&emsp; + Storage types | 01/10/2024 | 01/10/2024      | <https://000005.awsstudygroup.com/> |
| 4   | - Tiếp tục học RDS: <br>&emsp; + Multi-AZ deployments <br>&emsp; + Read replicas <br>&emsp; + Backup và restore <br>&emsp; + Security groups cho RDS | 02/10/2024 | 02/10/2024      | <https://000005.awsstudygroup.com/> |
| 5   | - **Thực hành RDS:** <br>&emsp; + Tạo RDS instance <br>&emsp; + Kết nối đến database <br>&emsp; + Cấu hình backups <br>&emsp; + Test Multi-AZ failover | 03/10/2024 | 04/10/2024      | <https://000005.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

* **Kiến thức VM Import/Export:**
  * Hiểu quy trình di chuyển máy ảo lên AWS
  * Học về các định dạng VM được hỗ trợ và yêu cầu chuyển đổi
  * Nắm vững cấu hình IAM role cho VM import
  * Import thành công VM images từ on-premises lên AWS
  * Hiểu use cases cho các kịch bản hybrid cloud

* **Chuyên môn Amazon RDS:**
  * Hiểu sâu về dịch vụ RDS và các database engines được hỗ trợ
  * Học về các DB instance classes khác nhau và trường hợp sử dụng
  * Hiểu các storage types (General Purpose SSD, Provisioned IOPS, Magnetic)
  * Nắm vững Multi-AZ deployments cho high availability
  * Học về Read Replicas để scale read operations

* **Quản lý Database:**
  * Tạo và cấu hình thành công RDS instances
  * Kết nối đến databases sử dụng nhiều clients khác nhau
  * Cấu hình automated backups và manual snapshots
  * Hiểu point-in-time recovery
  * Triển khai security best practices với security groups và encryption

* **High Availability và Disaster Recovery:**
  * Hiểu kiến trúc Multi-AZ và automatic failover
  * Học về backup retention và restoration procedures
  * Test các failover scenarios
  * Hiểu các khái niệm RTO và RPO
