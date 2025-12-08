---
title: "Cloud Mastery 1"
date: "2025-11-15"
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “AWS Cloud Mastery Series #1”

### Mục Đích Của Sự Kiện

* Cung cấp kiến thức thực tiễn về AWS Bedrock và quy trình phát triển AI hiện đại
* Giới thiệu các khái niệm cốt lõi về foundation models và kỹ thuật prompt engineering hiệu quả
* Trình diễn quy trình RAG và các kỹ thuật tìm kiếm dựa trên embedding
* Giới thiệu các dịch vụ AI phổ biến của AWS dành cho các giải pháp thực tế
* Giải thích AgentCore và cách xây dựng kiến trúc ứng dụng GenAI sẵn sàng cho sản xuất

### Diễn Giả

* **Lâm Tuấn Kiệt** – Senior DevOps Engineer, FPT Software
* **Đặng Hoàng Hiếu Nghi** – AI Engineer, Reonova Cloud
* **Đinh Lê Hoàng Anh** – Cloud Engineer Trainee, FCJ
* **Kha** – Chia sẻ định hướng xây dựng các dự án thiên về sản phẩm để tăng sức mạnh cho CV
* **Đại diện FPT** – Chia sẻ ví dụ về các doanh nghiệp ứng dụng AI dựa trên cloud để tối ưu chi phí và hiệu suất

### Những Điểm Nổi Bật

#### Foundation Models & Xu Hướng Ngành

* Các mô hình ML truyền thống chỉ phù hợp với từng tác vụ cụ thể và phụ thuộc nhiều vào dữ liệu gán nhãn.
* Foundation models trong GenAI được huấn luyện trên lượng lớn dữ liệu không gán nhãn, cho phép xử lý linh hoạt nhiều tác vụ.
* AWS Bedrock hiện hỗ trợ các mô hình như **OpenAI** và **DeepSeek**.
* Ngày càng nhiều doanh nghiệp chuyển workload AI lên cloud để giảm chi phí vận hành.

#### Góc Nhìn từ Lâm Tuấn Kiệt

* Giới thiệu tổng quan về sự chuyển dịch từ ML truyền thống → foundation models.
* Nhấn mạnh vai trò của prompt engineering trong việc tạo ra kết quả chất lượng cao.
* Trình bày cách Bedrock loại bỏ phức tạp DevOps khi truy cập các mô hình tiên tiến.
* Nhấn mạnh việc xây dựng sản phẩm AI đòi hỏi lặp lại liên tục, kỹ năng, và thực hành có trách nhiệm.

#### Prompt Engineering

* **Zero-shot prompting** – Ít ngữ cảnh; kết quả có thể chung chung.
* **Few-shot prompting** – Thêm ví dụ giúp mô hình trả lời rõ hơn và chính xác hơn.
* **Chain-of-thought prompting** – Khuyến khích lý luận từng bước để tăng độ chi tiết và đúng đắn.

#### Retrieval-Augmented Generation (RAG)

* Truy xuất thông tin liên quan từ nguồn dữ liệu bên ngoài.
* Kết hợp ngữ cảnh truy xuất với prompt ban đầu một cách tự động.
* Tạo ra câu trả lời chính xác và phù hợp ngữ cảnh hơn.

#### Embeddings

* Chuyển đổi văn bản thành vector biểu diễn ý nghĩa ngữ nghĩa.
* Các khái niệm tương tự nằm gần nhau trong không gian vector.
* **AWS Titan Text Embeddings** hỗ trợ đa ngôn ngữ hơn 100+ ngôn ngữ.

#### Dịch Vụ AI của AWS

* **Rekognition** – Phân tích hình ảnh/video
* **Translate** – Dịch đa ngôn ngữ
* **Textract** – Trích xuất văn bản và bố cục
* **Transcribe** – Chuyển giọng nói thành văn bản, hỗ trợ phân biệt người nói
* **Polly** – Chuyển văn bản thành giọng nói tự nhiên
* **Comprehend** – NLP, trích xuất thực thể và phát hiện quan hệ
* **Kendra** – Tìm kiếm thông minh cho doanh nghiệp
* **Lookout Family** – Phát hiện bất thường cho dữ liệu chỉ số, thiết bị và hình ảnh
* **Personalize** – Gợi ý theo thời gian thực
* **Pipecat** – Framework xây dựng pipeline AI agent

Tất cả dịch vụ đều có thể tích hợp thông qua API.

#### Amazon Bedrock AgentCore

* Cho phép xây dựng ứng dụng AI mà không cần hạ tầng hoặc DevOps phức tạp.
* Tương thích với các framework agent hiện đại như **LangGraph** và **LangChain**.
* Giúp đội ngũ chuyển từ prototype → production nhanh chóng và an toàn.

**Các thành phần cốt lõi**

* Runtime
* Memory
* Identity
* Gateway
* Code Interpreter
* Browser Tool
* Observability

### Những Điểm Rút Ra

#### Tư Duy Phát Triển AI

* Xây dựng sản phẩm thực tế mang lại giá trị lớn hơn nhiều so với chỉ hoàn thành bài tập lý thuyết.
* Foundation models hỗ trợ thử nghiệm nhanh, lặp lại và triển khai dễ dàng.
* Prompt engineering vẫn là chìa khóa để đạt chất lượng đầu ra cao.

#### Tác Động Kỹ Thuật

* RAG tăng độ chính xác bằng cách dựa trên tài liệu thật.
* Embeddings nâng cao tìm kiếm ngữ nghĩa và truy xuất thông minh.
* Dịch vụ AI của AWS hỗ trợ toàn bộ quy trình AI – từ giọng nói, hình ảnh đến văn bản và phân tích.

#### Tính Thực Tiễn

* Các tổ chức đang áp dụng AI dựa trên cloud để giảm chi phí và mở rộng nhanh hơn.
* Kỹ năng về prompt engineering, embeddings và Bedrock đang trở nên rất cần thiết.

### Trải Nghiệm Sự Kiện

Tham dự **AWS Cloud Mastery Series #1** giúp tôi có thêm trải nghiệm thực hành trong xây dựng hệ thống AI trên AWS.

#### 1. Học trực tiếp từ các kỹ sư

* Diễn giả chia sẻ kinh nghiệm thực tế từ DevOps, AI và cloud engineering.

#### 2. Kiến thức kỹ thuật thực hành

* Trình diễn kỹ thuật prompting
* Ví dụ về quy trình RAG
* Trường hợp thực tế của từng dịch vụ AI trong AWS

#### 3. Xây dựng cho tương lai

* Khuyến khích tập trung xây dựng sản phẩm thực tế có thể triển khai
* Kỹ năng Cloud + AI được nhấn mạnh là quan trọng cho nghề nghiệp hiện đại
* Tập trung mạnh vào ứng dụng thực tế hơn lý thuyết

#### Một số hình ảnh sự kiện

<p align="center">
  <img src="/images/4-EventParticipated/event_2/photo1.jpg" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_2/photo2.jpg" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

<p align="center">
  <img src="/images/4-EventParticipated/event_2/photo3.jpg" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

> Tóm lại, tôi đã học được rất nhiều về AI và Amazon Bedrock, và tôi sẽ áp dụng vào dự án của mình để cải thiện tốt hơn.
