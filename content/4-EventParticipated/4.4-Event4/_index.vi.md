---
title: "AWS Cloud Mastery Series #1"
date: 2025-11-15
weight: 4
chapter: false
pre: " <b> 4.4. </b> "
---

# Bào thu hoạch: “BUILDING AGENTIC AI - Context Optimization with Amazon Bedrock”

### Mục tiêu Sự kiện

* Cung cấp phần giới thiệu rõ ràng về Agentic AI và sự chuyển dịch sang các hệ thống AI tự động
* Trình bày Amazon Bedrock AgentCore cùng hệ sinh thái agentic mở rộng của AWS
* Minh họa các ví dụ thực tế về thiết kế Agentic Workflow trên AWS
* Làm nổi bật cách tiếp cận orchestration của CloudThinker và các phương pháp tối ưu hóa ngữ cảnh
* Cung cấp trải nghiệm thực hành xây dựng ứng dụng dựa trên agent bằng AWS Bedrock
* Tạo cơ hội kết nối với các chuyên gia trong cộng đồng AI và điện toán đám mây

### Diễn giả

* Nguyễn Gia Hưng – Head of Solutions Architect, AWS
* Kiên Nguyễn – Solutions Architect, AWS
* Việt Phạm – Founder & CEO, Diaflow
* Thắng Tôn – Co-founder & COO, CloudThinker
* Henry Bùi – Head of Engineering, CloudThinker
* Kha Văn – Community Leader, AWS

### Những Điểm Nổi Bật

#### Sự Phát Triển của Agentic AI

**ML Truyền thống / AI Cổ điển**

* Nhiệm vụ được định nghĩa hẹp, khả năng giới hạn
* Phụ thuộc mạnh vào dữ liệu có cấu trúc và xử lý tiền đề
* Khó mở rộng hoặc thích nghi với các trường hợp sử dụng mới

**Agentic AI Hiện Đại**

* Được vận hành bởi Foundation Models với khả năng suy luận đa bước
* Tự động phân rã nhiệm vụ và sử dụng công cụ
* Tích hợp API, thực thi workflow và truy cập tri thức
* Trở nên linh hoạt và sẵn sàng cho môi trường sản xuất khi kết hợp với AWS

#### Thách Thức Khi Triển Khai Agentic AI

* Các vấn đề hiệu năng như độ trễ, suy luận song song, thông lượng
* Mở rộng cho multi-agent và xử lý ngữ cảnh phức tạp
* Yêu cầu bảo mật như kiểm soát dữ liệu, luồng định danh, phân quyền truy cập
* Nhu cầu governance như logging, khả năng truy vết, giới hạn workflow

#### Danh Mục Agentic AI của AWS

* Amazon Bedrock AgentCore cho identity, memory, runtime, truy cập tools và workflows
* Agent Gateway cho tích hợp công cụ và API thống nhất
* Hỗ trợ nhiều mô hình như Anthropic, Meta Llama, Amazon Titan và nhiều hơn nữa
* Thiết kế hướng doanh nghiệp với guardrails, observability và khả năng mở rộng

#### Amazon Bedrock AgentCore

* Runtime để điều phối các nhiệm vụ đa bước
* Memory cho ngữ cảnh ngắn hạn và dài hạn
* Identity Flow để quản lý quyền và bảo mật
* Agent Gateway để kết nối tới công cụ, API và hệ thống doanh nghiệp
* Code Interpreter cho môi trường thực thi mã an toàn
* Browser Tool để thu thập thông tin từ nguồn bên ngoài
* Các tính năng quan sát gồm logs, traces và metrics

#### Xây Dựng Agentic Workflow trên AWS (Use Case từ Diaflow)

* Phối hợp giữa nhiều agent
* Truy xuất ngữ cảnh và function calling
* Tích hợp agent với hệ thống dữ liệu doanh nghiệp
* Thực thi logic kinh doanh bằng Bedrock và công cụ Diaflow
* Chiến lược thiết kế thực tế phù hợp với startup và SME

#### Orchestration & Tối Ưu Ngữ Cảnh của CloudThinker

* Các mô hình điều phối cấp cao cho hệ thống agent
* Lọc ngữ cảnh để cải thiện hiệu suất mô hình
* Workflow thích ứng thay đổi dựa trên thông tin từ agent
* Kỹ thuật đánh giá và tăng độ tin cậy trong suy luận
* Tích hợp workflow CloudThinker với Amazon Bedrock AgentCore

#### CloudThinker Hack: Phiên Thực Hành

* Thiết lập ban đầu Bedrock agents
* Xây dựng workflow agent đơn giản
* Thêm công cụ bên ngoài vào pipeline
* Khắc phục lỗi và tối ưu hành vi agent
* Triển khai bản proof-of-concept hoàn chỉnh

### Những Điểm Rút Ra Quan Trọng

#### Tư Duy Agentic AI

* AI đang chuyển từ phản hồi thụ động sang hành động tự chủ
* Agent hiệu quả cần kết hợp memory, tools, identity và orchestration
* Foundation Models + điều phối workflow = thế hệ ứng dụng AI tiếp theo
* AWS hiện là một trong những môi trường sẵn sàng sản xuất mạnh mẽ nhất cho agentic AI

#### Hiểu Biết Kỹ Thuật

* Vì sao tối ưu hóa ngữ cảnh quan trọng
* Cách công cụ và quản lý định danh ảnh hưởng đến độ tin cậy
* Cách AgentCore đơn giản hóa reasoning đa bước
* Vai trò của các mô hình orchestrator thực tế
* Phương pháp kết nối mô hình với API, tầng logic và nguồn dữ liệu

##### Kỹ Năng Phát Triển Thực Tế

* Thiết kế workflow AI hoàn chỉnh từ đầu đến cuối
* Tích hợp công cụ bên ngoài vào pipeline agent
* Điều phối đa agent hiệu quả
* Quản lý độ trễ, khả năng mở rộng và bảo mật
* Triển khai agent với đầy đủ guardrails và hệ thống giám sát

##### Ứng Dụng Vào Công Việc

Người tham dự có thể áp dụng trực tiếp:

* Xây dựng assistant, quy trình tự động hóa hoặc copilots nội bộ
* Kết nối mô hình Bedrock với nền tảng doanh nghiệp
* Phát triển workflow đa bước có cấu trúc với AgentCore
* Tạo prototype nhanh mà không cần hạ tầng DevOps phức tạp
* Ứng dụng phương pháp orchestration của CloudThinker để tăng hiệu suất
* Áp dụng mô hình thiết kế agentic cho cả startup và doanh nghiệp lớn

### Trải Nghiệm Sự Kiện

Workshop “Agentic Build AI – Optimization with Amazon Bedrock” đã mang đến góc nhìn toàn diện về phát triển Agentic AI hiện đại.

#### Kiến Thức Từ Chuyên Gia

* Các diễn giả từ AWS, CloudThinker và Diaflow chia sẻ trải nghiệm triển khai thực tế
* Hướng dẫn rõ ràng về cách mở rộng và vận hành giải pháp GenAI
* Ví dụ cụ thể về tự động hóa quy trình doanh nghiệp bằng Agentic AI

#### Trải Nghiệm Thực Hành

* Xây dựng workflow agent hoàn chỉnh trong workshop
* Hiểu rõ cách memory, identity và tool tương tác
* Trải nghiệm thực tế với API và các thành phần orchestration của Bedrock

#### Cơ Hội Kết Nối

* Gặp gỡ kiến trúc sư AWS, kỹ sư, founder và các thành viên cộng đồng
* Tìm hiểu các hướng phát triển sự nghiệp và thảo luận về sản phẩm AI
* Trao đổi ý tưởng về hệ thống AI cloud-native

#### Bài Học Rút Ra

* Agentic AI vượt xa chatbot và ứng dụng LLM truyền thống
* Triển khai thực tế yêu cầu identity, observability và khả năng mở rộng
* Bedrock và CloudThinker cung cấp lộ trình mạnh mẽ từ prototype đến sản xuất
* Dự án thực tế mang lại giá trị rõ ràng hơn so với các bài tập học thuật rời rạc

#### Một số hình ảnh sự kiện

<p align="center">
  <img src="/images/4-EventParticipated/event_4/photo1.jpg" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_4/photo2.jpg" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_4/photo3.jpg" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

> Tóm lại, workshop đã mang đến sự kết hợp hài hòa giữa kiến thức chiến lược và kỹ năng thực hành. Người tham gia rời sự kiện với khả năng thiết kế workflow, tinh chỉnh ngữ cảnh, tích hợp công cụ và xây dựng hệ thống agentic mở rộng, an toàn trên AWS.
