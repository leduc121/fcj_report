---
title: "Blog 2"
date: "2025-10-08"
weight: 2 # Sửa từ 1 sang 2
chapter: false
pre: " <b> 3.2. </b> " # Sửa từ 3.3. sang 3.2.
---

# **CrazyGames nâng cấp nền tảng với hệ thống bạn bè thời gian thực sử dụng AWS AppSync**

**Tác giả: Emily McKinzie | vào ngày: 08 tháng 10 năm 2024 | trong: [Amazon EC2](#), [Amazon MemoryDB](#), [AWS AppSync](#), [Compute](#), [Customer Solutions](#), [Database](#), [Front-End Web & Mobile](#), [Game Development](#), [Industries](#), [Top Posts](#) | [Permalink](#) | [Comments](#) | [Share](#)**

---

![CrazyGames Gamer](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2024/10/01/CrazyGames_Gamer.jpg)

Nền tảng game nhiều người chơi **CrazyGames** thu hút hơn 35 triệu người chơi trên toàn thế giới với các tựa game trên trình duyệt như *Ludo King* và *Paper Delivery Boy* bằng 24 ngôn ngữ khác nhau. Dù chơi qua thiết bị máy tính để bàn hay di động, tất cả game thủ giờ đây đều có thể tận hưởng trải nghiệm xã hội được nâng cao với hệ thống bạn bè thời gian thực mới được xây dựng bằng **Amazon Web Services (AWS) AppSync**.

CrazyGames đã all-in với AWS ngay từ ngày đầu tiên, chạy trên một instance **Amazon Elastic Compute Cloud (Amazon EC2)** duy nhất khi được xây dựng ban đầu vào năm 2014. Kể từ đó, nền tảng đã phát triển một cách tự nhiên và hiện tại lưu trữ hơn 3.000 game thuộc nhiều thể loại khác nhau. Nhìn lại vai trò mà AWS đã đóng trong quá trình phát triển liên tục của công ty, Nhà sáng lập và CEO của CrazyGames Raf Mertens nhận xét:

> "AWS cung cấp một loạt các dịch vụ có tính khả dụng cao, có thể mở rộng mà chúng tôi có thể thử nghiệm, kiểm tra và sử dụng một cách dễ dàng trong một khoảng thời gian rất ngắn. Bằng cách sử dụng AWS AppSync, chúng tôi đã giảm thời gian phát triển từ tám tháng xuống còn tám tuần."

### Nâng cao trải nghiệm xã hội

Chơi game cùng bạn bè thú vị hơn vô cùng so với chơi đơn độc và có thể giúp người chơi nâng cao kỹ năng của họ. May mắn thay, các game xã hội cung cấp cơ hội để tạo dựng tình bạn mới với những người có chung sở thích đồng thời củng cố các mối quan hệ hiện có. Các game nhiều người chơi với các yếu tố xã hội mạnh mẽ cũng mang lại lợi ích. Chúng có xu hướng thu hút người chơi tốt hơn, tăng khả năng giữ chân lâu dài và thu hút người dùng mới khi người chơi mời bạn bè của họ.

Cho đến gần đây, khách truy cập CrazyGames không phải lúc nào cũng có thể dễ dàng chơi cùng bạn bè. Họ phải đối mặt với trải nghiệm rời rạc, điều này đã truyền cảm hứng cho công ty thiết kế và triển khai hệ thống bạn bè mới. Giờ đây, người dùng có thể kết bạn với nhau, xem khi nào bạn bè của họ trực tuyến, xem các game họ đang chơi và gửi lời mời chơi game theo thời gian thực.

Để phát triển tính năng bạn bè, CrazyGames đã sử dụng phương pháp 'shape up', bao gồm việc thành lập một nhóm nhỏ, chuyên trách tập trung hoàn toàn vào việc tạo ra hệ thống bạn bè sẵn sàng cho sản xuất. Các nhà phát triển đã xây dựng chức năng mới bằng **AWS AppSync**, công cụ điều phối trạng thái của nhiều người dùng gần như theo thời gian thực. Người chơi có thể thấy ngay lập tức khi bạn bè của họ đăng nhập hoặc đăng xuất.

Khi bắt đầu phát triển, nhóm cũng đã tìm hiểu một framework mã nguồn mở và một giải pháp tùy chỉnh được xây dựng bằng WebSocket API trước khi lựa chọn AWS AppSync.

> "Người dùng ngày nay mong đợi sự hài lòng ngay lập tức, vì vậy các hoạt động thời gian thực là điều cần thiết. AWS AppSync nổi lên như giải pháp lý tưởng cho hệ thống bạn bè của chúng tôi. Nó yêu cầu ít nỗ lực phát triển hơn do sự trừu tượng hóa trên WebSocket API, hiệu quả về chi phí hơn so với các lựa chọn khác và tích hợp liền mạch với GraphQL client hiện có của chúng tôi," Mertens giải thích. "AWS AppSync bao phủ tất cả những gì chúng tôi cần, vì vậy việc thiết lập rất dễ dàng và nó tiếp tục hoạt động tốt khi xử lý hàng chục nghìn người dùng đồng thời với các cập nhật trạng thái thời gian thực."

Ngoài AWS AppSync, CrazyGames sử dụng **Amazon Aurora** để quản lý dữ liệu tồn tại lâu dài, chẳng hạn như thông báo, trong hệ thống bạn bè mới. Đối với dữ liệu tạm thời hơn đòi hỏi cập nhật nhanh chóng, như lời mời và trạng thái người dùng, công ty sử dụng **Redis Cloud on AWS**, với **Amazon MemoryDB**.

![CrazyGames Devices](https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2024/10/01/CrazyGames_Devices.jpg)

### Xây dựng với AWS

Với một thập kỷ lịch sử làm việc với AWS, CrazyGames đã phát triển hệ sinh thái công nghệ của mình để bao gồm một loạt các giải pháp và dịch vụ AWS. Điều này giúp nhóm của họ tập trung vào phát triển phần mềm thay vì xây dựng, cấu hình và bảo trì máy chủ. Năm ngoái, công ty đã triển khai một hệ thống phân tích mới để có thêm hiểu biết về hành vi và nhu cầu của người dùng. Hệ thống phân tích theo dõi hiệu quả hàng triệu sự kiện hàng ngày trên hơn hai triệu người dùng hàng ngày của nền tảng khi họ tương tác với các tính năng phức tạp.

> "Hầu hết các dịch vụ AWS mà chúng tôi đã chọn đều xử lý các trách nhiệm vận hành như tính khả dụng và bảo trì. Điều này có nghĩa là nhóm của chúng tôi có thể dành nhiều thời gian hơn cho việc phát triển phần mềm," Mertens cho biết. "Hơn nữa, nền tảng của chúng tôi chịu các đợt tăng đột biến lưu lượng truy cập, và vì sức mạnh tính toán của chúng tôi có thể mở rộng, những đợt tăng đột biến này được backend tự động hấp thụ mà không có thời gian chết hoặc độ trễ."

Hiện tại, CrazyGames đã tích hợp hệ thống bạn bè mới vào 30 game trên nền tảng của họ, từ game bắn súng góc nhìn thứ nhất như *Kour.io* đến các game kinh điển như *8 Ball Pool Billiards*. Khi họ có kế hoạch triển khai hệ thống bạn bè rộng rãi hơn trong 12 tháng tới, công ty đang tìm cách đơn giản hóa trải nghiệm onboarding. Mertens chia sẻ: "Một khi người dùng biết tính năng bạn bè mới tồn tại, họ thực sự yêu thích nó, nhưng chúng tôi vẫn phải hướng dẫn họ qua các bước cụ thể để bắt đầu. Chúng tôi muốn làm cho quy trình đó dễ dàng hơn cho họ."

### Lên kế hoạch cho tính năng mới tiếp theo

Bằng cách tận dụng AWS, các nhà phát triển CrazyGames có thể nhanh chóng và hiệu quả về chi phí trải nghiệm các tính năng mới, giúp công ty duy trì danh tiếng là điểm đến hàng đầu cho các game xã hội hấp dẫn dựa trên web. Mertens kết luận:

> "Với các dịch vụ được quản lý của AWS, chúng tôi có thể xây dựng nhiều tính năng hơn với thời gian của mình, thay vì phải đối phó với việc quản lý cơ sở hạ tầng. Thêm vào đó, có sự giảm thiểu rủi ro rất lớn. Bạn có thể thử nghiệm và đưa các tính năng mới lên nhanh hơn để tìm hiểu xem chúng có hoạt động hay không, thay vì lãng phí tài nguyên kỹ thuật."

Tìm hiểu thêm về việc xây dựng các trải nghiệm giải trí, có thể mở rộng với tốc độ và sự linh hoạt.
Liên hệ với [Đại diện AWS](#) để biết cách chúng tôi có thể giúp đẩy nhanh doanh nghiệp của bạn.

---

<img src="https://d2908q01vomqb2.cloudfront.net/fb644351560d8296fe6da332236b1f8d61b2828a/2023/05/18/EmilyPhoto.jpg" width="150" alt="Emily McKinzie">

**Emily McKinzie**
*Emily McKinzie là Quản lý Marketing Ngành tại Amazon Web Services.*