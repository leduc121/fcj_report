---
title: "Worklog Tuần 6"
date: "2024-10-13"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

* Học quản lý truy cập EC2 sử dụng resource tags thông qua IAM
* Hiểu tối ưu chi phí với Lambda automation
* Triển khai automated resource management

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 1   | - Học về resource tagging: <br>&emsp; + Tag best practices <br>&emsp; + Tag policies <br>&emsp; + Cost allocation tags <br>&emsp; + Tag-based access control | 13/10/2024 | 13/10/2024      | <https://000028.awsstudygroup.com/> |
| 2   | - Học IAM policies với tags: <br>&emsp; + Condition keys cho tags <br>&emsp; + Tag-based permissions <br>&emsp; + Resource-level permissions <br> - **Thực hành:** Tạo tag-based IAM policies | 14/10/2024 | 14/10/2024      | <https://000028.awsstudygroup.com/> |
| 3   | - **Thực hành:** Quản lý EC2 access với tags: <br>&emsp; + Tag EC2 instances <br>&emsp; + Tạo IAM policies dựa trên tags <br>&emsp; + Test access control <br>&emsp; + Verify permissions | 15/10/2024 | 15/10/2024      | <https://000028.awsstudygroup.com/> |
| 4   | - Học Lambda cho cost optimization: <br>&emsp; + Lambda cơ bản <br>&emsp; + Event-driven automation <br>&emsp; + CloudWatch Events/EventBridge <br>&emsp; + Lambda với EC2 API | 16/10/2024 | 16/10/2024      | Tài liệu AWS Lambda |
| 5   | - **Thực hành:** Tối ưu EC2 costs với Lambda: <br>&emsp; + Tạo Lambda function để stop idle instances <br>&emsp; + Schedule Lambda với EventBridge <br>&emsp; + Monitor và test automation <br>&emsp; + Tính toán cost savings | 17/10/2024 | 18/10/2024      | Tài liệu AWS Lambda |

### Kết quả đạt được tuần 6:

* **Resource Tagging:**
  * Nắm vững AWS tagging best practices
  * Hiểu tag naming conventions và strategies
  * Học về mandatory tags và tag policies
  * Triển khai cost allocation tags cho billing
  * Tạo comprehensive tagging strategy cho tổ chức

* **IAM Tag-Based Access Control:**
  * Học sử dụng condition keys trong IAM policies (aws:RequestTag, aws:ResourceTag)
  * Tạo policies cấp quyền truy cập dựa trên resource tags
  * Triển khai attribute-based access control (ABAC)
  * Quản lý thành công EC2 access sử dụng resource tags
  * Test và verify tag-based permissions

* **Lambda Automation:**
  * Hiểu về AWS Lambda serverless computing
  * Học event-driven architecture patterns
  * Nắm vững Lambda function creation và deployment
  * Hiểu Lambda execution roles và permissions

* **Tối ưu Chi phí với Lambda:**
  * Tạo Lambda functions để identify và stop idle EC2 instances
  * Triển khai automated scheduling với EventBridge
  * Thiết lập notifications cho automated actions
  * Monitor Lambda execution và costs
  * Tính toán actual cost savings từ automation

* **Automation Best Practices:**
  * Học triển khai safe automation với proper error handling
  * Hiểu tầm quan trọng của testing automation trong non-production
  * Triển khai logging và monitoring cho automated tasks
  * Tạo documentation cho automated processes
