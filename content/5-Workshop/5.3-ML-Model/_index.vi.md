---
title : "Mô Hình Học Máy"
date: 2025-09-09
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# HUẤN LUYỆN MÔ HÌNH

Mục sau đây trình bày về quá trình phát triển mô hình dự đoán được thiết kế để dự đoán vị trí địa lý tiếp theo của một cơn bão, dựa trên dữ liệu quan sát từ quỹ đạo di chuyển trước đó. Nói cách đơn giản, nhóm sử dụng chuỗi các giá trị vĩ độ và kinh độ trong quá khứ để dự đoán vĩ độ và kinh độ tại bước thời gian kế tiếp.

## Trích Xuất Đặc Trưng

Sau khi hoàn tất bước tiền xử lý dữ liệu, nhóm tiến hành chia bộ dữ liệu thành **70% để huấn luyện**, **10% để kiểm định**, và **20% để kiểm tra**. Việc chia này được thực hiện **theo mã định danh cơn bão (storm ID)**, đảm bảo không có cơn bão nào xuất hiện đồng thời trong cả tập huấn luyện và tập kiểm định/kiểm tra. Điều này giúp ngăn rò rỉ dữ liệu và đảm bảo độ tin cậy của quá trình đánh giá.

Trong quá trình huấn luyện mô hình, mỗi mẫu đầu vào bao gồm **chuỗi 4 bước thời gian liên tiếp**, trong đó mỗi bước cách nhau **3 giờ**. Như vậy, một chuỗi đầu vào tương ứng với quãng di chuyển **9 giờ** của cơn bão.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/dataset_split.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1 : Dataset Distribution</strong>
</p>

## Áp dụng Kỹ thuật Stepwise Temporal Fading Augmentation (STFA)

Để tăng độ đa dạng của dữ liệu huấn luyện, nhóm áp dụng phương pháp tự đề xuất — **Stepwise Temporal Fading Augmentation (STFA)** — lên **50% tập huấn luyện**, được lựa chọn dựa trên từng storm ID riêng biệt. Các chuỗi gốc của những cơn bão này sẽ được thay thế bằng chuỗi đã được tăng cường, đảm bảo kích thước tập huấn luyện cuối cùng vẫn giữ nguyên (xấp xỉ 100% kích thước ban đầu).

Như đã đề cập trong phần đề xuất mô hình, STFA thay đổi các điểm **cũ hơn** trong chuỗi, trong khi giữ **các quan sát mới nhất không đổi**. Với mỗi chuỗi 4 bước:

* **2 bước mới nhất** được giữ nguyên
* **2 bước cũ hơn** được nhân với hệ số giảm dần: **[0.98, 0.99]**

Mặc dù những giá trị này có vẻ nhỏ, nhưng **vĩ độ và kinh độ cực kỳ nhạy cảm**. Một thay đổi nhỏ — ví dụ từ 6.7 lên 6.8 — có thể tương ứng với **hàng chục kilomet** dịch chuyển ngoài thực tế. Do đó, mức điều chỉnh nhỏ như vậy là hợp lý và phù hợp với quy luật vật lý, giúp dữ liệu tăng cường vẫn mang tính chân thực.


### Ví dụ STFA trên một chuỗi 4 bước thời gian

| Row | Original (lat, lon) | Augmented (lat, lon) | Operation         |
| --- | ------------------- | -------------------- | ----------------- |
| 1   | [-6.8, 107.5]       | [-6.66, 105.35]      | nhân với **0.98** |
| 2   | [-7.0, 107.1]       | [-6.93, 106.03]      | nhân với **0.99** |
| 3   | [-7.3, 106.7]       | [-7.3, 106.7]        | giữ nguyên        |
| 4   | [-7.5, 106.4]       | [-7.5, 106.4]        | giữ nguyên        |

Quy trình trên làm giảm giá trị của các quan sát **cũ**, đồng thời giữ nguyên các bước **mới**. Việc tăng cường này giúp mô hình có thêm biến thiên có kiểm soát, cải thiện khả năng tổng quát hóa trong dự báo quỹ đạo.

Trước đó, nhóm đã sử dụng machine learning dựa trên quy luật vật lý để tính **khoảng cách** và **góc phương vị** bằng công thức Haversine. Sau khi STFA được áp dụng, các giá trị này sẽ được **tính lại** dựa trên tọa độ đã tăng cường để đảm bảo các đặc trưng vật lý vẫn chính xác và nhất quán.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/storm_augmentations_grid.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2 : Comparison of Augmentation Techniques on Storm Trajectories</strong>
</p>


## Thiết lập Mô hình

### 1. Hàm Loss dựa trên quy luật vật lý

Việc ứng dụng công thức Haversine không chỉ dừng lại ở bước trích xuất đặc trưng. Ngoài việc tạo ra các giá trị khoảng cách và hướng, nhóm còn tích hợp **Công thức Haversine như một hàm loss tùy chỉnh**, sử dụng cùng với các hàm loss truyền thống như MSE, RMSE và MAPE.

Công thức Haversine đo khoảng cách địa lý thực giữa hai điểm, nên đây là metric tự nhiên để đánh giá sai số dự đoán vị trí bão. Khoảng cách Haversine càng lớn nghĩa là dự đoán càng sai; ngược lại, giá trị gần 0 km cho thấy mô hình hoạt động tốt.

Ví dụ:

* Dự đoán: **[-6.72, 107.1]**
* Giá trị thật: **[-6.8, 107.5]**
* Haversine loss: **45.06 km**

Giá trị 45.06 km phản ánh chính xác sai số vị trí ngoài thực tế, giúp việc giải thích mô hình dễ dàng và ý nghĩa hơn.

### 2. Kiến trúc mô hình

Các bài toán mô hình hóa chuỗi thường được xử lý bằng RNN, LSTM hoặc GRU, nhưng các nghiên cứu gần đây cho thấy mô hình dựa trên tích chập (CNN) có thể vượt trội hơn trong nhiều tác vụ time-series.

Do đó, nhóm sử dụng kiến trúc CNN — cụ thể là **Temporal Convolutional Network (TCN)**.

TCN sử dụng tích chập giãn (dilated convolution), giúp mô hình có receptive field rộng mà không cần dùng mạng hồi quy.

TCN kết hợp được cả:

* khả năng học phụ thuộc dài hạn
* tốc độ huấn luyện nhanh
* gradient ổn định

Nên rất phù hợp cho bài toán dự báo quỹ đạo bão.


### 3. Các siêu tham số mô hình

* Input: 4 đặc trưng (lat, lon, distance, bearing)
* Hidden units: 1024
* Số lớp TCN: 2
* Learning rate: 1e-4
* Epochs: 80
* Optimizer: Adam
* Early stopping: patience = 6


### 4. Hàm Loss tổng hợp

Loss chính của mô hình được kết hợp từ:

* MSE của lat/lon
* MSE của distance/bearing
* Haversine loss (dựa trên vật lý)

Trong đó:

* λ_aux = 0.5
* λ_hav = 0.3

Cách thiết kế này giúp mô hình:

* giảm sai số tọa độ
* tôn trọng quy luật dịch chuyển vật lý
* tránh overfit vào một loại đặc trưng cụ thể

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/train.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3 : Training Process</strong>
</p>


## Evaluation

## Đánh giá mô hình

Sau khi mô hình hoàn tất quá trình huấn luyện và dừng sớm (early stopping), nhóm tiến hành đánh giá trên tập kiểm tra để xác định khả năng tổng quát hóa và mức độ sẵn sàng triển khai thực tế.

Kết quả đánh giá:

* **Total Loss:** 74.3849
* **MSE:** 0.0832
* **RMSE:** 0.2772
* **MAPE:** 0.60%
* **Haversine:** 30.75 km

Sai số vị trí trung bình khoảng **30 km** — mức hoàn toàn chấp nhận được đối với hệ thống có quy mô hàng trăm đến hàng nghìn kilomet như bão nhiệt đới. MSE nhỏ (0.08) cho thấy khả năng dự đoán tốt và ổn định.

Kết quả này cũng chứng minh rằng các mô hình convolution có thể hoạt động xuất sắc trong bài toán mô hình hóa chuỗi, không chỉ trong xử lý ảnh.

Sau khi xác thực mô hình, bước tiếp theo là **tải mô hình lên Amazon S3** và sử dụng **AWS Lambda** để thực thi mô hình khi người dùng gửi yêu cầu dự đoán.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/loss.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Figure 4 : Evaluation Metrics</strong>
</p>
