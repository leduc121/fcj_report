---
title: "Blog 3"
date: "2025-10-09"
weight: 3 # Sửa từ 1 sang 3
chapter: false
pre: " <b> 3.3. </b> " # Giữ nguyên
---

# **Xử lý tài liệu thông minh quy mô lớn với AI sinh tạo và Amazon Bedrock Data Automation**

**Tác giả: Nikita Kozodoi, Aiham Taleb, Francesco Cerizzi, Liza (Elizaveta) Zinovyeva, Nuno Castro, Ozioma Uzoegwu, Eren Tuncer và Zainab Afolabi | vào ngày: 11 tháng 07 năm 2025 | trong: [Amazon Bedrock](#), [Amazon Bedrock Data Automation](#), [Generative AI](#), [Intermediate (200)](#), [Technical How-to](#) | [Permalink](#) | [Comments](#) | [Share](#)**

---

Trích xuất thông tin từ các tài liệu phi cấu trúc quy mô lớn là một nhiệm vụ kinh doanh thường xuyên. Các trường hợp sử dụng phổ biến bao gồm tạo bảng tính năng sản phẩm từ mô tả, trích xuất metadata từ tài liệu, và phân tích hợp đồng pháp lý, đánh giá của khách hàng, bài báo tin tức và nhiều hơn nữa. Một phương pháp cổ điển để trích xuất thông tin từ văn bản là nhận dạng thực thể có tên (NER). NER xác định các thực thể từ các danh mục được xác định trước, chẳng hạn như con người và tổ chức. Mặc dù nhiều dịch vụ và giải pháp AI hỗ trợ NER, phương pháp này bị giới hạn ở các tài liệu văn bản và chỉ hỗ trợ một tập hợp thực thể cố định. Hơn nữa, các mô hình NER cổ điển không thể xử lý các loại dữ liệu khác như điểm số (chẳng hạn như sentiment) hoặc văn bản tự do (chẳng hạn như tóm tắt). **AI sinh tạo** mở khóa những khả năng này mà không cần chú thích dữ liệu tốn kém hoặc huấn luyện mô hình, cho phép xử lý tài liệu thông minh (IDP) toàn diện hơn.

**AWS** gần đây đã công bố tính khả dụng chung của **Amazon Bedrock Data Automation**, một tính năng của **Amazon Bedrock** tự động hóa việc tạo ra những thông tin có giá trị từ nội dung đa phương thức phi cấu trúc như tài liệu, hình ảnh, video và âm thanh. Dịch vụ này cung cấp các khả năng được xây dựng sẵn cho IDP và trích xuất thông tin thông qua một API thống nhất, loại bỏ nhu cầu về kỹ thuật prompt phức tạp hoặc fine-tuning, và làm cho nó trở thành một lựa chọn tuyệt vời cho quy trình xử lý tài liệu quy mô lớn. Để tìm hiểu thêm về Amazon Bedrock Data Automation, tham khảo [Đơn giản hóa AI sinh tạo đa phương thức với Amazon Bedrock Data Automation](#).

Amazon Bedrock Data Automation là phương pháp được khuyến nghị cho trường hợp sử dụng IDP do tính đơn giản, độ chính xác hàng đầu trong ngành và khả năng dịch vụ được quản lý. Nó xử lý độ phức tạp của việc phân tích tài liệu, quản lý ngữ cảnh và lựa chọn mô hình tự động, vì vậy các nhà phát triển có thể tập trung vào logic kinh doanh của họ thay vì các chi tiết triển khai IDP.

Mặc dù Amazon Bedrock Data Automation đáp ứng hầu hết nhu cầu IDP, một số tổ chức yêu cầu tùy chỉnh bổ sung trong các pipeline IDP của họ. Ví dụ, các công ty có thể cần sử dụng các foundation models (FMs) tự lưu trữ cho IDP do yêu cầu quy định. Một số khách hàng có các nhóm builder có thể thích duy trì toàn quyền kiểm soát đối với pipeline IDP thay vì sử dụng dịch vụ được quản lý. Cuối cùng, các tổ chức có thể hoạt động tại các AWS Regions nơi Amazon Bedrock Data Automation không khả dụng (khả dụng tại us-west-2 và us-east-1 tính đến tháng 6 năm 2025). Trong những trường hợp như vậy, các builder có thể sử dụng trực tiếp Amazon Bedrock FMs hoặc thực hiện nhận dạng ký tự quang học (OCR) với **Amazon Textract**.

Bài viết này trình bày một ứng dụng IDP end-to-end được hỗ trợ bởi Amazon Bedrock Data Automation và các dịch vụ AWS khác. Nó cung cấp một infrastructure as code (IaC) AWS có thể tái sử dụng để triển khai một pipeline IDP và cung cấp một giao diện người dùng trực quan để chuyển đổi tài liệu thành các bảng có cấu trúc quy mô lớn. Ứng dụng chỉ yêu cầu người dùng cung cấp các tài liệu đầu vào (như hợp đồng hoặc email) và một danh sách các thuộc tính cần được trích xuất. Sau đó nó thực hiện IDP với AI sinh tạo.

Mã ứng dụng và hướng dẫn triển khai [có sẵn trên GitHub](#) theo giấy phép MIT.

## Tổng quan giải pháp

Giải pháp IDP được trình bày trong bài viết này được triển khai dưới dạng IaC sử dụng **AWS Cloud Development Kit (AWS CDK)**. Amazon Bedrock Data Automation phục vụ như công cụ chính cho việc trích xuất thông tin. Đối với các trường hợp yêu cầu tùy chỉnh thêm, giải pháp cũng cung cấp các đường xử lý thay thế sử dụng Amazon Bedrock FMs và tích hợp Amazon Textract.

Chúng tôi sử dụng **AWS Step Functions** để orchestrate quy trình IDP và song song hóa xử lý cho nhiều tài liệu. Như một phần của quy trình, chúng tôi sử dụng các hàm **AWS Lambda** để gọi Amazon Bedrock Data Automation hoặc Amazon Textract và Amazon Bedrock (tùy thuộc vào chế độ phân tích đã chọn). Các tài liệu đã xử lý và các thuộc tính được trích xuất được lưu trữ trong **Amazon Simple Storage Service (Amazon S3)**.

Một quy trình Step Functions với logic kinh doanh được gọi thông qua một cuộc gọi API được thực hiện bằng cách sử dụng AWS SDK. Chúng tôi cũng xây dựng một ứng dụng web được đóng gói trong container chạy trên **Amazon Elastic Container Service (Amazon ECS)** có sẵn cho người dùng cuối thông qua **Amazon CloudFront** để đơn giản hóa tương tác của họ với giải pháp. Chúng tôi sử dụng **Amazon Cognito** cho xác thực và truy cập an toàn vào các API.

Sơ đồ sau minh họa kiến trúc và quy trình của giải pháp IDP.

![Architecture Diagram](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/24/architecture-3.png)

Quy trình IDP bao gồm các bước sau:

1.  Người dùng đăng nhập vào ứng dụng web bằng thông tin xác thực được quản lý bởi Amazon Cognito, chọn các tài liệu đầu vào và xác định các trường cần được trích xuất từ chúng trong UI. Tùy chọn, người dùng có thể chỉ định chế độ phân tích, LLM để sử dụng và các cài đặt khác.
2.  Người dùng khởi động pipeline IDP.
3.  Ứng dụng tạo một pre-signed S3 URL cho các tài liệu và tải chúng lên Amazon S3.
4.  Ứng dụng kích hoạt Step Functions để khởi động state machine với các S3 URIs và cài đặt IDP làm đầu vào. Map state bắt đầu xử lý các tài liệu đồng thời.
5.  Tùy thuộc vào loại tài liệu và chế độ phân tích, nó phân nhánh đến các hàm Lambda khác nhau thực hiện IDP, lưu kết quả vào Amazon S3 và gửi chúng trở lại UI:
    * **Amazon Bedrock Data Automation** – Các tài liệu được chuyển đến hàm Lambda "Run Data Automation". Hàm Lambda tạo một blueprint với lược đồ các trường do người dùng xác định và khởi chạy một tác vụ Amazon Bedrock Data Automation không đồng bộ. Amazon Bedrock Data Automation xử lý độ phức tạp của việc xử lý tài liệu và trích xuất thuộc tính sử dụng các prompts và mô hình được tối ưu hóa. Khi kết quả tác vụ đã sẵn sàng, chúng được lưu vào Amazon S3 và gửi trở lại UI. Phương pháp này cung cấp sự cân bằng tốt nhất về độ chính xác, dễ sử dụng và khả năng mở rộng cho hầu hết các trường hợp sử dụng IDP.
    * **Amazon Textract** – Nếu người dùng chỉ định Amazon Textract làm chế độ phân tích, pipeline IDP chia thành hai bước. Đầu tiên, hàm Lambda "Perform OCR" được gọi để chạy một tác vụ phân tích tài liệu không đồng bộ. Các đầu ra OCR được xử lý bằng cách sử dụng thư viện `amazon-textract-textractor` và được định dạng dưới dạng Markdown. Thứ hai, văn bản được chuyển đến hàm Lambda "Extract attributes" (Bước 6), hàm này gọi một Amazon Bedrock FM với văn bản và lược đồ thuộc tính. Các đầu ra được lưu vào Amazon S3 và gửi đến UI.
    * **Xử lý tài liệu văn phòng** – Các tài liệu có hậu tố như .doc, .ppt và .xls được xử lý bởi hàm Lambda "Parse office", hàm này sử dụng các document loaders của LangChain để trích xuất nội dung văn bản. Các đầu ra được chuyển đến hàm Lambda "Extract attributes" (Bước 6) để tiếp tục với pipeline IDP.
6.  Nếu người dùng chọn một Amazon Bedrock FM cho IDP, tài liệu được gửi đến hàm Lambda "Extract attributes". Nó chuyển đổi một tài liệu thành một tập hợp các hình ảnh, được gửi đến một multimodal FM với lược đồ thuộc tính như một phần của prompt tùy chỉnh. Nó phân tích phản hồi LLM để trích xuất các đầu ra JSON, lưu chúng vào Amazon S3 và gửi nó trở lại UI. Luồng này hỗ trợ các tài liệu .pdf, .png và .jpg.
7.  Ứng dụng web kiểm tra kết quả thực thi state machine định kỳ và trả về các thuộc tính được trích xuất cho người dùng khi chúng có sẵn.

## Điều kiện tiên quyết

Bạn có thể triển khai giải pháp IDP từ máy tính local của mình hoặc từ một notebook instance của **Amazon SageMaker**. Các bước triển khai được mô tả chi tiết trong tệp README của giải pháp.

Nếu bạn chọn triển khai bằng SageMaker notebook, điều này được khuyến nghị, bạn sẽ cần truy cập vào tài khoản AWS với quyền tạo và khởi chạy một notebook instance SageMaker.

## Triển khai giải pháp

Để triển khai giải pháp vào tài khoản AWS của bạn, hoàn thành các bước sau:

1.  Mở **AWS Management Console** và chọn Region mà bạn muốn triển khai giải pháp IDP.
2.  Khởi chạy một notebook instance SageMaker. Cung cấp tên notebook instance và kiểu notebook instance, bạn có thể đặt thành `ml.m5.large`. Để các tùy chọn khác là mặc định.
3.  Điều hướng đến Notebook instance và mở IAM role được đính kèm với notebook. Mở role trên bảng điều khiển **AWS Identity and Access Management (IAM)**.
4.  Đính kèm một inline policy vào role và chèn policy JSON sau:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "cloudformation:*",
            "s3:*",
            "iam:*",
            "sts:AssumeRole"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "ssm:GetParameter",
            "ssm:GetParameters"
          ],
          "Resource": "arn:aws:ssm:*:*:parameter/cdk-bootstrap/*"
        }
      ]
    }
    ```

5.  Khi trạng thái notebook instance được đánh dấu là *InService*, chọn **Open JupyterLab**.
6.  Trong môi trường JupyterLab, chọn **File**, **New**, và **Terminal**.
7.  Clone repository giải pháp bằng cách chạy các lệnh sau:

    ```bash
    cd SageMaker
    git clone [https://github.com/aws-samples/intelligent-document-processing-with-amazon-bedrock.git](https://github.com/aws-samples/intelligent-document-processing-with-amazon-bedrock.git)
    ```

8.  Điều hướng đến thư mục repository và chạy script để cài đặt requirements:

    ```bash
    cd intelligent-document-processing-with-amazon-bedrock
    sh install_deps.sh
    ```

9.  Chạy script để tạo một virtual environment và cài đặt dependencies:

    ```bash
    sh install_env.sh
    source .venv/bin/activate
    ```

10. Trong thư mục repository, sao chép `config-example.yml` thành `config.yml` để chỉ định tên stack của bạn. Tùy chọn, cấu hình các dịch vụ và chỉ định các modules bạn muốn triển khai (ví dụ, để vô hiệu hóa việc triển khai UI, thay đổi `deploy_streamlit` thành `False`). Đảm bảo bạn thêm email người dùng của mình vào danh sách người dùng Amazon Cognito.
11. Cấu hình truy cập mô hình Amazon Bedrock bằng cách mở bảng điều khiển Amazon Bedrock trong Region được chỉ định trong tệp `config.yml`. Trong ngăn điều hướng, chọn **Model Access** và đảm bảo kích hoạt truy cập cho các model IDs được chỉ định trong `config.yml`.
12. Bootstrap và triển khai AWS CDK trong tài khoản của bạn:

    ```bash
    cdk bootstrap
    cdk deploy
    ```

Lưu ý rằng bước này có thể mất một chút thời gian, đặc biệt là trong lần triển khai đầu tiên. Khi triển khai hoàn tất, bạn sẽ thấy thông báo như trong ảnh chụp màn hình sau. Bạn có thể truy cập Streamlit frontend bằng cách sử dụng URL phân phối CloudFront được cung cấp trong các đầu ra của **AWS CloudFormation**. Thông tin đăng nhập tạm thời sẽ được gửi đến email được chỉ định trong `config.yml` trong quá trình triển khai.

![Deployment Output](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-2-1.jpeg)

## Sử dụng giải pháp

Phần này hướng dẫn bạn qua hai ví dụ để giới thiệu các khả năng IDP.

### Ví dụ 1: Phân tích tài liệu tài chính

Trong kịch bản này, chúng tôi trích xuất các tính năng chính từ một báo cáo tài chính nhiều trang bằng cách sử dụng Amazon Bedrock Data Automation. Chúng tôi sử dụng một tài liệu mẫu ở định dạng PDF với sự kết hợp của bảng, hình ảnh và văn bản, và trích xuất một số chỉ số tài chính. Hoàn thành các bước sau:

1.  Tải lên một tài liệu bằng cách đính kèm một tệp thông qua UI của giải pháp.

    ![Upload Document](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-3-1.png)

2.  Trên tab **Describe Attributes**, liệt kê thủ công tên và mô tả của các thuộc tính hoặc tải lên các trường này ở định dạng JSON. Chúng tôi muốn tìm các chỉ số sau:
    * Current cash in assets in 2018
    * Current cash in assets in 2019
    * Operating profit in 2018
    * Operating profit in 2019

    ![Describe Attributes](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-4-1.png)

3.  Chọn **Extract attributes** để khởi động pipeline IDP.

    ![Extract Attributes Button](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-5-1.png)

Các thuộc tính được cung cấp được tích hợp vào một blueprint tùy chỉnh với danh sách thuộc tính được suy luận, sau đó được sử dụng để gọi một tác vụ data automation trên các tài liệu đã tải lên.
Sau khi pipeline IDP hoàn tất, bạn sẽ thấy một bảng kết quả trong UI. Nó bao gồm một chỉ số cho mỗi tài liệu trong cột `_doc`, một cột cho mỗi thuộc tính bạn xác định và một cột `file_name` chứa tên tài liệu.

![Results Table](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-6-1.png)

Từ các đoạn trích báo cáo sau, chúng ta có thể thấy rằng Amazon Bedrock Data Automation đã có thể trích xuất chính xác các giá trị cho tài sản hiện tại và lợi nhuận hoạt động.

![Statement Excerpts](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-7-1.png)

Giải pháp IDP cũng có thể thực hiện các phép tính phức tạp vượt ra ngoài các thực thể được xác định rõ ràng. Giả sử chúng ta muốn tính toán các chỉ số kế toán sau:
* Tỷ lệ thanh khoản (Tài sản hiện tại/Nợ phải trả hiện tại)
* Vốn lưu động (Tài sản hiện tại – Nợ phải trả hiện tại)
* Tăng doanh thu ((Doanh thu năm 2/Doanh thu năm 1) – 1)

Chúng tôi xác định các thuộc tính và công thức của chúng như các phần của lược đồ thuộc tính. Lần này, chúng tôi chọn một Amazon Bedrock LLM làm chế độ phân tích để minh họa cách ứng dụng có thể sử dụng một multimodal FM cho IDP. Khi sử dụng một Amazon Bedrock LLM, việc khởi động pipeline IDP bây giờ sẽ kết hợp các thuộc tính và mô tả của chúng thành một template prompt tùy chỉnh, được gửi đến LLM với các tài liệu được chuyển đổi thành hình ảnh. Là người dùng, bạn có thể chỉ định LLM hỗ trợ việc trích xuất và các tham số suy luận của nó, chẳng hạn như temperature.

Đầu ra, bao gồm các kết quả đầy đủ, được hiển thị trong ảnh chụp màn hình sau.

![Complex Calculations](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-8-1.png)

### Ví dụ 2: Xử lý email khách hàng

Trong kịch bản này, chúng tôi muốn trích xuất nhiều tính năng từ một danh sách email có khiếu nại của khách hàng do chậm trễ trong việc vận chuyển sản phẩm bằng cách sử dụng Amazon Bedrock Data Automation. Đối với mỗi email, chúng tôi muốn tìm những điều sau:
* Tên khách hàng
* ID vận chuyển
* Ngôn ngữ email
* Sentiment email
* Chậm trễ vận chuyển (tính bằng ngày)
* Tóm tắt vấn đề
* Phản hồi đề xuất

Hoàn thành các bước sau:
1.  Tải lên các email đầu vào dưới dạng các tệp .txt. Bạn có thể tải xuống các email mẫu từ GitHub.

    ![Upload Emails](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-9-1.png)

2.  Trên tab **Describe Attributes**, liệt kê tên và mô tả của các thuộc tính.

    ![Describe Attributes Emails](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-10-1.png)

3.  Bạn có thể thêm các ví dụ few-shot cho một số trường (chẳng hạn như delay) để giải thích cho LLM cách các giá trị của các trường này nên được trích xuất. Bạn có thể làm điều này bằng cách thêm một đầu vào ví dụ và đầu ra dự kiến cho thuộc tính vào mô tả.
4.  Chọn **Extract attributes** để khởi động pipeline IDP.

    ![Extract Attributes Button](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-11.png)

Các thuộc tính được cung cấp và mô tả của chúng sẽ được tích hợp vào một blueprint tùy chỉnh với danh sách thuộc tính được suy luận, sau đó được sử dụng để gọi một tác vụ data automation trên các tài liệu đã tải lên. Khi pipeline IDP hoàn tất, bạn sẽ thấy các kết quả.

![Email Results](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-12.png)

Ứng dụng cho phép tải xuống các kết quả trích xuất dưới dạng tệp CSV hoặc tệp JSON. Điều này làm cho nó trở nên đơn giản để sử dụng các kết quả cho các tác vụ downstream, chẳng hạn như tổng hợp các điểm sentiment của khách hàng.

![Download Results](https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/08/ML-17229-image-13-1.png)

## Chi phí

Trong phần này, chúng tôi tính toán ước tính chi phí để thực hiện IDP trên AWS với giải pháp của chúng tôi.

Amazon Bedrock Data Automation cung cấp một lược đồ định giá minh bạch tùy thuộc vào kích thước tài liệu đầu vào (số lượng trang, hình ảnh hoặc phút). Khi sử dụng Amazon Bedrock FMs, giá phụ thuộc vào số lượng input và output tokens được sử dụng như một phần của cuộc gọi trích xuất thông tin. Cuối cùng, khi sử dụng Amazon Textract, OCR được thực hiện và được định giá riêng dựa trên số lượng trang trong tài liệu.

Sử dụng các kịch bản trước đó làm ví dụ, chúng tôi có thể xấp xỉ chi phí tùy thuộc vào chế độ phân tích đã chọn. Trong bảng sau, chúng tôi hiển thị chi phí sử dụng hai bộ dữ liệu: 100 tài liệu tài chính 20 trang và 100 email khách hàng 1 trang. Chúng tôi bỏ qua chi phí của Amazon ECS và Lambda.

| AWS service | Use case 1 (100 20-page financial documents) | Use case 2 (100 1-page customer emails) |
| :--- | :--- | :--- |
| **IDP option 1: Amazon Bedrock Data Automation** | | |
| Amazon Bedrock Data Automation (custom output) | $20.00 | $1.00 |
| **IDP option 2: Amazon Bedrock FM** | | |
| Amazon Bedrock (FM invocation, Anthropic's Claude 4 Sonnet) | $1.79 | $0.09 |
| **IDP option 3: Amazon Textract and Amazon Bedrock FM** | | |
| Amazon Textract (document analysis job with layout) | $30.00 | $1.50 |
| Amazon Bedrock (FM invocation, Anthropic's Claude 3.7 Sonnet) | $1.25 | $0.06 |
| **Orchestration and storage (shared costs)** | | |
| Amazon S3 | $0.02 | $0.02 |
| AWS CloudFront | $0.09 | $0.09 |
| Amazon ECS | – | – |
| AWS Lambda | – | – |
| **Total cost: Amazon Bedrock Data Automation** | **$20.11** | **$1.11** |
| **Total cost: Amazon Bedrock FM** | **$1.90** | **$0.20** |
| **Total cost: Amazon Textract and Amazon Bedrock FM** | **$31.36** | **$1.67** |

Phân tích chi phí cho thấy rằng sử dụng Amazon Bedrock FMs với template prompt tùy chỉnh là một phương pháp hiệu quả về chi phí cho IDP. Tuy nhiên, phương pháp này yêu cầu chi phí vận hành lớn hơn, bởi vì pipeline cần được tối ưu hóa tùy thuộc vào LLM và yêu cầu quản lý bảo mật và quyền riêng tư thủ công. Amazon Bedrock Data Automation cung cấp một dịch vụ được quản lý sử dụng lựa chọn các FMs có hiệu suất cao thông qua một API duy nhất.

## Dọn dẹp

Để xóa các tài nguyên đã triển khai, hoàn thành các bước sau:

1.  Trên bảng điều khiển AWS CloudFormation, xóa stack đã tạo. Ngoài ra, chạy lệnh sau:
    ```bash
    cdk destroy --region <YOUR_DEPLOY_REGION>
    ```
2.  Trên bảng điều khiển Amazon Cognito, xóa user pool.

## Kết luận

Trích xuất thông tin từ các tài liệu phi cấu trúc quy mô lớn là một nhiệm vụ kinh doanh thường xuyên. Bài viết này thảo luận về một ứng dụng IDP end-to-end thực hiện trích xuất thông tin bằng cách sử dụng nhiều dịch vụ AWS. Giải pháp được hỗ trợ bởi Amazon Bedrock Data Automation, cung cấp một dịch vụ được quản lý đầy đủ để tạo ra thông tin chi tiết từ tài liệu, hình ảnh, âm thanh và video. Amazon Bedrock Data Automation xử lý độ phức tạp của việc xử lý tài liệu và trích xuất thông tin, tối ưu hóa cả hiệu suất và độ chính xác mà không yêu cầu chuyên môn về kỹ thuật prompt. Để có tính linh hoạt mở rộng và khả năng tùy chỉnh trong các kịch bản cụ thể, giải pháp của chúng tôi cũng hỗ trợ IDP bằng cách sử dụng các cuộc gọi LLM tùy chỉnh Amazon Bedrock và Amazon Textract cho OCR.

Giải pháp hỗ trợ nhiều loại tài liệu, bao gồm văn bản, hình ảnh, PDF và tài liệu Microsoft Office. Tại thời điểm viết bài, việc hiểu chính xác thông tin trong các tài liệu có nhiều hình ảnh, bảng và các yếu tố trực quan khác chỉ có sẵn cho PDF và hình ảnh. Chúng tôi khuyến nghị chuyển đổi các tài liệu Office phức tạp sang PDF hoặc hình ảnh để có hiệu suất tốt nhất. Một giới hạn giải pháp khác là kích thước tài liệu. Tính đến tháng 6 năm 2025, Amazon Bedrock Data Automation hỗ trợ các tài liệu lên đến 20 trang để trích xuất thuộc tính tùy chỉnh. Khi sử dụng các LLM tùy chỉnh Amazon Bedrock cho IDP, cửa sổ ngữ cảnh 300.000 token của Amazon Nova LLMs cho phép xử lý các tài liệu có tối đa khoảng 225.000 từ. Để trích xuất thông tin từ các tài liệu lớn hơn, hiện tại bạn sẽ cần phải chia tệp thành nhiều tài liệu.

Trong các phiên bản tiếp theo của giải pháp IDP, chúng tôi dự định tiếp tục thêm hỗ trợ cho các language models tiên tiến nhất có sẵn thông qua Amazon Bedrock và lặp lại về kỹ thuật prompt để cải thiện thêm độ chính xác trích xuất. Chúng tôi cũng dự định triển khai các kỹ thuật để mở rộng kích thước của các tài liệu được hỗ trợ và cung cấp cho người dùng một chỉ dẫn chính xác về vị trí chính xác trong tài liệu nơi thông tin được trích xuất đến từ đâu.

Để bắt đầu với IDP với giải pháp được mô tả, tham khảo kho lưu trữ GitHub. Để tìm hiểu thêm về Amazon Bedrock, tham khảo tài liệu.

## Về các tác giả

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/kozodoi.jpg" width="120" alt="Nikita Kozodoi">

**Nikita Kozodoi, Tiến sĩ**, là Nhà Khoa học Ứng dụng Cấp cao tại Trung tâm Đổi mới AI Tạo sinh của AWS, nơi ông làm việc ở tiên phong của nghiên cứu AI và kinh doanh. Với kinh nghiệm phong phú trong AI Tạo sinh và các lĩnh vực đa dạng của ML, Nikita đầy nhiệt huyết trong việc sử dụng AI để giải quyết các vấn đề kinh doanh thực tế đầy thách thức trên nhiều ngành công nghiệp.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/zafolabi.jpg" width="120" alt="Zainab Afolabi">

**Zainab Afolabi** là Nhà Khoa học Dữ liệu Cấp cao tại Trung tâm Đổi mới AI Tạo sinh ở London, nơi cô tận dụng chuyên môn sâu rộng của mình để phát triển các giải pháp AI mang tính chuyển đổi trên nhiều ngành công nghiệp khác nhau. Cô có hơn tám năm kinh nghiệm chuyên môn trong trí tuệ nhân tạo và học máy, cùng với niềm đam mê chuyển đổi các khái niệm kỹ thuật phức tạp thành các ứng dụng kinh doanh thực tế.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/aitaleb.png" width="120" alt="Aiham Taleb">

**Aiham Taleb, Tiến sĩ**, là Nhà Khoa học Ứng dụng Cấp cao tại Trung tâm Đổi mới AI Tạo sinh, làm việc trực tiếp với các khách hàng doanh nghiệp của AWS để tận dụng Gen AI trên nhiều trường hợp sử dụng có tác động cao. Aiham có bằng Tiến sĩ về học biểu diễn không giám sát, và có kinh nghiệm trong ngành trải rộng trên nhiều ứng dụng học máy khác nhau, bao gồm thị giác máy tính, xử lý ngôn ngữ tự nhiên và hình ảnh y tế.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/zinovyee.png" width="120" alt="Liza (Elizaveta) Zinovyeva">

**Liza (Elizaveta) Zinovyeva** là Nhà Khoa học Ứng dụng tại Trung tâm Đổi mới AI Tạo sinh của AWS và đang làm việc tại Berlin. Cô giúp đỡ khách hàng trên nhiều ngành công nghiệp khác nhau để tích hợp AI Tạo sinh vào các ứng dụng và quy trình làm việc hiện có của họ. Cô đam mê các chủ đề về AI/ML, tài chính và bảo mật phần mềm. Trong thời gian rảnh, cô thích dành thời gian với gia đình, thể thao, học các công nghệ mới và các cuộc thi đố vui.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/24/nunoca_100x125.jpg" width="120" alt="Nuno Castro">

**Nuno Castro** là Giám đốc Khoa học Ứng dụng Cấp cao tại Trung tâm Đổi mới AI Tạo sinh của AWS. Ông lãnh đạo các dự án hợp tác với khách hàng về AI Tạo sinh, giúp hàng trăm khách hàng AWS tìm ra trường hợp sử dụng có tác động lớn nhất từ ý tưởng, nguyên mẫu cho đến sản xuất. Ông có 19 năm kinh nghiệm trong AI tại các ngành như tài chính, sản xuất và du lịch, lãnh đạo các đội AI/ML trong 12 năm.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/ozioma_resized.jpg" width="120" alt="Ozioma Uzoegwu">

**Ozioma Uzoegwu** là Kiến trúc sư Giải pháp Chính tại Amazon Web Services. Trong vai trò của mình, ông giúp các khách hàng dịch vụ tài chính trên khắp EMEA chuyển đổi và hiện đại hóa trên AWS Cloud, cung cấp hướng dẫn kiến trúc và các thực tiễn tốt nhất trong ngành. Ozioma có nhiều năm kinh nghiệm với phát triển web, kiến trúc, đám mây và quản lý CNTT. Trước khi gia nhập AWS, Ozioma đã làm việc với một Đối tác Tư vấn Nâng cao của AWS với vai trò Kiến trúc sư Trưởng cho Bộ phận AWS. Ông đam mê sử dụng các công nghệ mới nhất để xây dựng một hệ thống CNTT dịch vụ tài chính hiện đại trên các lĩnh vực ngân hàng, thanh toán, bảo hiểm và thị trường vốn.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/eren_resized.png" width="120" alt="Eren Tuncer">

**Eren Tuncer** là Kiến trúc sư Giải pháp tại Amazon Web Services tập trung vào Serverless và xây dựng các ứng dụng AI Tạo sinh. Với hơn mười lăm năm kinh nghiệm trong phát triển và kiến trúc phần mềm, ông giúp khách hàng trên nhiều ngành công nghiệp khác nhau đạt được mục tiêu kinh doanh của họ bằng cách sử dụng các công nghệ đám mây với các thực tiễn tốt nhất. Là một người xây dựng, ông đam mê tạo ra các giải pháp với các công nghệ tiên tiến nhất, chia sẻ kiến thức và giúp các tổ chức điều hướng việc áp dụng đám mây.

<img src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/06/12/francesco_resized.jpg" width="120" alt="Francesco Cerizzi">

**Francesco Cerizzi** là Kiến trúc sư Giải pháp tại Amazon Web Services khám phá các biên giới công nghệ đồng thời lan tỏa kiến thức về AI tạo sinh và xây dựng các ứng dụng. Với nền tảng là một nhà phát triển full stack, ông giúp khách hàng trên nhiều ngành công nghiệp khác nhau trong hành trình lên đám mây của họ, chia sẻ những hiểu biết sâu sắc về tiềm năng chuyển đổi của AI trên đường đi. Ông đam mê Serverless, kiến trúc hướng sự kiện và microservices nói chung. Khi không đắm mình trong công nghệ, ông là một fan hâm mộ cuồng nhiệt của F1 và yêu thích quần vợt.