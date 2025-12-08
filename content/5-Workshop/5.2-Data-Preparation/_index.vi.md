---
title : "Chuẩn Bị Dữ Liệu"
date: 2025-09-09
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---


## Thu Thập và Xử Lý Dữ Liệu

Dữ liệu là một thành phần quan trọng trong dự án. Nó không chỉ cung cấp nguồn kiến thức cho **mô hình machine learning** mà còn được hiển thị trực tiếp cho người dùng cuối để theo dõi **các cơn bão mới nhất** tại khu vực Tây Thái Bình Dương. Vì dữ liệu có mục đích kép — **huấn luyện mô hình** và **trực quan hóa theo thời gian thực** — chúng em đã xem xét kỹ lưỡng nhiều nguồn data source đáng tin cậy và có thẩm quyền trước khi chọn một bộ dữ liệu đáp ứng đầy đủ các yêu cầu, đó là: **Dữ liệu bão của NOAA**.

**NOAA (National Oceanic and Atmospheric Administration)** là cơ quan khoa học thuộc Bộ Thương mại Hoa Kỳ. NOAA cung cấp dữ liệu môi trường có độ chính xác cao phục vụ nghiên cứu, bao gồm quan sát thời tiết toàn cầu, ảnh vệ tinh và thông tin về các xoáy thuận nhiệt đới. Với nhiều thập kỷ đầu tư vào công nghệ tiên tiến như vệ tinh địa tĩnh, hệ thống radar và mạng lưới giám sát khí hậu, NOAA được xem là một trong những nguồn cung cấp dữ liệu bão đáng tin cậy nhất trên thế giới.

Trong dự án này, chúng em sử dụng dữ liệu từ **International Best Track Archive for Climate Stewardship (IBTrACS)** — một dự án do NOAA khởi xướng và là bộ dữ liệu xoáy thuận nhiệt đới toàn diện nhất thế giới. IBTrACS tổng hợp dữ liệu đường đi của bão trong lịch sử từ nhiều cơ quan khí tượng (ví dụ: JTWC, JMA, CMA, NHC). Bằng cách hợp nhất các nguồn vào một định dạng thống nhất, IBTrACS cải thiện khả năng so sánh giữa các cơ quan và đảm bảo các nhà nghiên cứu trên toàn thế giới có quyền truy cập dữ liệu chất lượng cao nhất.

Phiên bản mới nhất của bộ dữ liệu này chứa **226.153 dòng** ghi nhận quan sát bão. Mỗi dòng bao gồm nhiều thuộc tính giá trị như:

* `sid` – mã cơn bão
* `number` – số thứ tự bão
* `basin` / `subbasin` – phân loại khu vực
* `nature` – loại bão (ví dụ: áp thấp, bão nhiệt đới, siêu bão)
* `iso_time` – thời gian
* `lat` / `lon` – tọa độ tâm bão
* … và nhiều thông số khí tượng học khác

Tuy nhiên, đối với mô hình machine learning, chúng em chỉ tập trung vào **bốn cột chính**:
`sid`, `iso_time`, `lat`, và `lon`.
Đây là chuỗi thời gian cơ bản dùng để dự đoán quỹ đạo di chuyển của bão.

Bộ dữ liệu bao gồm các cơn bão từ **1870 đến 2025**, được lọc chỉ giữ lại các cơn bão trong **khu vực Tây Thái Bình Dương** — phạm vi địa lý mà dự án hướng đến. Bộ dữ liệu gốc được công khai tại:
[https://data.humdata.org/dataset/vnm-ibtracs-tropical-storm-tracks#](https://data.humdata.org/dataset/vnm-ibtracs-tropical-storm-tracks#)


### **Làm sạch dữ liệu và Trích xuất đặc trưng dựa trên quy luật vật lý**

Một ưu điểm của IBTrACS là dữ liệu đã được bảo trì tốt và có tính nhất quán cao. Việc tiền xử lý chỉ yêu cầu các bước tối thiểu, chủ yếu là loại bỏ giá trị thiếu.

Sau khi làm sạch, chúng em áp dụng bước đầu tiên của **machine learning dựa trên vật lý (physics-informed ML)** — kỹ thuật đưa kiến thức vật lý trực tiếp vào pipeline dữ liệu. Từ tọa độ vĩ độ – kinh độ, nhóm tính thêm hai đặc trưng bằng **công thức Haversine**:

1. **Khoảng cách** giữa hai điểm bão liên tiếp
2. **Góc phương vị (bearing)** — hướng di chuyển

Các đặc trưng này mang ý nghĩa vật lý: chúng phản ánh quy luật chuyển động thực tế, thay vì những biến đổi tùy ý. Nhờ đó, chúng tăng cường bối cảnh về quán tính và hướng di chuyển của bão, giúp mô hình học hiệu quả hơn và dự đoán chính xác hơn.

<p align="center">
  <img src="/images/5-Workshop/5.2-Data Preparation/dataset.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1 : Mô tả dữ liệu</strong>
</p>

## **Dữ Liệu Hiển Thị Trên Nền Tảng**

Dữ liệu dùng để **hiển thị** trên nền tảng khác với dữ liệu dùng để **huấn luyện**, dù cả hai đều xuất phát từ NOAA. Dữ liệu huấn luyện là tĩnh, còn dữ liệu hiển thị phải **cập nhật theo tình hình bão hiện tại**.

Để xử lý yêu cầu này, nhóm đã triển khai một **AWS Lambda** chạy theo lịch, tự động lấy dữ liệu đường đi bão mới nhất vào cuối mỗi ngày. Điều này đảm bảo nền tảng luôn hiển thị thông tin mới, chính xác và kịp thời cho người dùng.

Dữ liệu hiển thị sau xử lý được lưu dưới dạng **file JSON** trong S3. Khi người dùng truy cập website:

1. Frontend gửi yêu cầu tới **API Gateway**
2. API Gateway kích hoạt **Lambda** tương ứng
3. Lambda lấy file JSON từ S3
4. Dữ liệu được trả về cho người dùng để hiển thị trực quan

Pipeline này đảm bảo khả năng cập nhật thời gian thực, serverless và tiết kiệm chi phí.

Bộ dữ liệu bão có thể truy cập tại:
[https://ncics.org/ibtracs/](https://ncics.org/ibtracs/)

<p align="center">
  <img src="/images/5-Workshop/5.2-Data Preparation/web_crawl.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2 : Web để crawl dữ liệu</strong>
</p>
