---
title : "Kiến trúc Lambda"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2 </b> "
---



# Kiến trúc Lambda - Dịch vụ AI Dự báo Bão

## Tổng quan

Hàm Lambda là một thành phần quan trọng trong kiến trúc serverless. Chúng đặc biệt hữu ích nhờ chi phí vận hành thấp và khả năng triển khai dễ dàng—những yếu tố rất phù hợp với nền tảng dự đoán bão của nhóm.

Phần này trình bày chi tiết cách chúng tôi thiết kế và xây dựng kiến trúc Lambda.

Các hàm Lambda của chúng em chạy mô hình PyTorch để dự đoán quỹ đạo bão và được triển khai thông qua Docker container image.

## Kiến trúc các dịch vụ AWS

```text
┌─────────────────────────────────────────────────────────────┐
│                   Frontend (Trình duyệt)                    │
└────────────────────────┬────────────────────────────────────┘
                         │ POST /predict
                         ▼
┌─────────────────────────────────────────────────────────────┐
│            Lambda Function URL (Công khai)                  │
│  URL: https://vill3povlzqxdyxm7ubldizobu0kdgbi...           │
│  Auth: NONE (không xác thực)                                │
│  Method: POST                                               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                       Lambda Function                       │
│  Tên: storm-prediction                                      │
│  Runtime: Python 3.10 (Container)                           │
│  Bộ nhớ: 3008 MB                                            │
│  Timeout: 120 giây                                          │
│  Kiến trúc: x86_64                                          │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                        ECR Repository                       │
│  Account: 339570693867                                      │
│  Region: ap-southeast-1                                     │
│  Repo: storm-prediction                                     │
│  Image: latest                                              │
│  Size: ~2 GB                                                │
└────────────────────────┬────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                          S3 Buckets                         │
│  1. storm-frontend-hosting-duc-2025                         │
│     - models/lstm_totald_256_4.pt (tùy chọn)                │
│     - predictions/[storm_id]_[timestamp].json               │
│                                                             │
│  2. storm-ai-models (khuyến nghị)                           │
│     - models/lstm_totald_256_4.pt                           │
│     - models/tcn_model.pth (backup)                         │
└─────────────────────────────────────────────────────────────┘
```


## Cấu trúc thư mục storm_prediction/

```text
storm_prediction/
├── app.py                          # Lambda handler (mã chính)
├── Dockerfile                      # Định nghĩa container
├── requirements.txt                # Thư viện Python phụ thuộc
├── cropping_storm_7304_2l.pth     # Mô hình TCN (đóng kèm trong image)
│
├── DEPLOY_NOW.md                   # Hướng dẫn deploy nhanh
├── DEPLOY_CONSOLE_STEP_BY_STEP.md # Hướng dẫn AWS Console từng bước
├── LAMBDA_DEPLOYMENT_GUIDE.md      # Hướng dẫn triển khai chi tiết
├── AWS_CONSOLE_DEPLOYMENT_GUIDE.md
├── FIX_ECR_PUSH_ERROR.md          # Tài liệu xử lý lỗi ECR
├── FIX_UNICODE_ERROR.md           # Sửa lỗi UnicodeDecodeError
├── FIX_UNICODE_ERROR_SOLUTION.md  # Chi tiết giải pháp
└── REBUILD_AND_DEPLOY.sh          # Script tự động build & deploy
```


## Cấu trúc Docker Image

### Dockerfile
```dockerfile
FROM public.ecr.aws/lambda/python:3.10

# Cài đặt dependencies
COPY requirements.txt .
RUN pip3 install -r requirements.txt \
    --target "${LAMBDA_TASK_ROOT}" \
    --extra-index-url https://download.pytorch.org/whl/cpu

# Copy Lambda handler
COPY app.py ${LAMBDA_TASK_ROOT}

# Copy mô hình TCN vào thư mục con (tránh nhầm file .pth)
RUN mkdir -p ${LAMBDA_TASK_ROOT}/models
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/models/

# Đặt handler
CMD [ "app.handler" ]
```

### Các lớp (Image Layers)
```text
Layer 1: AWS Lambda Python 3.10 base (~500 MB)
Layer 2: PyTorch CPU + dependencies (~1.2 GB)
Layer 3: app.py + mô hình TCN (~300 MB)
─────────────────────────────────────────────
Tổng dung lượng: ~2 GB
```

## Các mô hình AI

### 1. Mô hình TCN (Dự đoán quỹ đạo)
**File**: `cropping_storm_7304_2l.pth`  
**Vị trí**: Bên trong Docker image tại `/var/task/models/`  
**Dung lượng**: ~300 MB  
**Mục đích**: Dự đoán bước tiếp theo (lat, lng) của quỹ đạo bão

**Kiến trúc**:
```python
class StormTCN(nn.Module):
    def __init__(self, input_dim=4, hidden_units=1024, num_layers=2):
        self.tcn = TCN(...)
        self.head_latlon = nn.Linear(hidden_units, 2)  # Dự đoán lat, lng
        self.head_aux = nn.Linear(hidden_units, 2)     # Dự đoán đặc trưng phụ
```

**Input**: `[batch, sequence, 4]` - (lat, lng, distance, bearing)  
**Output**:
- `pred_latlon`: (lat, lng) kế tiếp
- `pred_aux`: đặc trưng phụ

### 2. Mô hình LSTM (Dự đoán tổng quãng đường)
**File**: `lstm_totald_256_4.pt`  
**Vị trí**: S3 bucket (tải về ở lần chạy đầu tiên)  
**Dung lượng**: ~50 MB  
**Mục đích**: Dự đoán tổng quãng đường bão sẽ di chuyển

**Kiến trúc**:
```python
class StormLSTM(nn.Module):
    def __init__(self, input_size=4, hidden_size=256, num_layers=2):
        self.lstm = nn.LSTM(...)
        self.fc = nn.Sequential(
            nn.Linear(hidden_size, hidden_size // 2),
            nn.ReLU(),
            nn.Linear(hidden_size // 2, 1)  # Dự đoán tổng quãng đường
        )
```

**Input**: Tổng hợp theo ngày `[batch, days, 4]` - (day, daily_dist, avg_speed, motion_type)  
**Output**: Tổng quãng đường (km)


## Luồng xử lý request

### 1. Nhận request
```text
POST /predict
Content-Type: application/json

{
  "history": [
    {"lat": 15.0, "lng": 120.0},
    {"lat": 15.1, "lng": 120.1},
    ...  // Tối thiểu 9 điểm
  ],
  "storm_name": "Test Storm",
  "storm_id": "TEST001"  // Tùy chọn
}
```

### 2. Tải mô hình (chỉ lần gọi đầu tiên)
```python
def load_models():
    global LSTM_MODEL, TCN_MODEL
    
    # Tải LSTM từ S3 (nếu có)
    if not os.path.exists('/tmp/lstm_model.pt'):
        s3_client.download_file(
            MODEL_BUCKET,
            'models/lstm_totald_256_4.pt',
            '/tmp/lstm_model.pt'
        )
    LSTM_MODEL = StormLSTM(...)
    LSTM_MODEL.load_state_dict(torch.load('/tmp/lstm_model.pt'))
    
    # Tải TCN từ local (đã có sẵn trong image)
    TCN_MODEL = StormTCN(...)
    TCN_MODEL.load_state_dict(
        torch.load('/var/task/models/cropping_storm_7304_2l.pth')
    )
```


### 4. Dự đoán tổng quãng đường (LSTM)
```python
def predict_total_distance(record_tensor):
    if LSTM_MODEL is None:
        # Dự phòng (fallback): avg_distance * 24 bước
        return fallback_distance
    
    # Gom theo ngày (9 điểm/ngày)
    # Chạy dự đoán bằng LSTM
    with torch.no_grad():
        pred = LSTM_MODEL(summary_tensor, lengths)
    
    return pred.item()  # km
```


### 5. Dự đoán đường đi (TCN)
```python
def predict_storm_path(record_tensor, total_distance, history):
    seq = record_tensor.clone()
    gone_distance = 0
    predicted_points = []
    
    while gone_distance < total_distance:
        # Dự đoán vị trí tiếp theo
        pred_latlon, pred_aux = TCN_MODEL(seq)
        new_lat = pred_latlon[0, -1, 0].item()
        new_lng = pred_latlon[0, -1, 1].item()
        
        # Tính khoảng cách & hướng di chuyển
        step_distance = haversine(last_lat, last_lng, new_lat, new_lng)
        
        # Ước lượng tốc độ gió (giảm dần theo thời gian)
        estimated_wind = max(avg_wind * (0.98 ** step), 30)
        
        predicted_points.append({
            'lat': new_lat,
            'lng': new_lng,
            'timestamp': base_timestamp + (step * 3 * 3600 * 1000),
            'windSpeed': estimated_wind,
            'pressure': 980.0,
            'category': calculate_category(estimated_wind)
        })
        
        # Cập nhật chuỗi (sliding window)
        seq = torch.cat([seq[:, 1:, :], next_point.unsqueeze(1)], dim=1)
        gone_distance += step_distance
        step += 1
    
    return predicted_points
```

### 6. Trả về response
```python
result = {
    'storm_id': storm_id,
    'storm_name': storm_name,
    'prediction_time': datetime.now().isoformat(),
    'totalDistance': 500.5,
    'actualDistance': 520.3,
    'lifespan': 72,
    'forecastHours': 72,
    'forecast': [
        {
            'lat': 15.1,
            'lng': 106.99,
            'timestamp': 1765015351626,
            'windSpeed': 65,
            'pressure': 980,
            'category': 'Typhoon'
        },
        ...
    ]
}

return {
    'statusCode': 200,
    'headers': {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
    },
    'body': json.dumps(result)
}
```

## Quy trình Build & Deploy

### Bước 1: Build Docker Image
```bash
cd storm_prediction

docker build \
  --provenance=false \
  --platform linux/amd64 \
  -t storm-prediction-model .
```

Giải thích flags:
- `--provenance=false`: Giảm kích thước image (không kèm metadata build)
- `--platform linux/amd64`: Lambda chỉ hỗ trợ x86_64
- `-t storm-prediction-model`: Tên tag của image


### Bước 2: Tag để push lên ECR
```bash
docker tag storm-prediction-model:latest \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

### Bước 3: Đăng nhập ECR
```bash
aws ecr get-login-password --region ap-southeast-1 | \
  docker login --username AWS --password-stdin \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com
```

### Bước 4: Push image lên ECR
```bash
docker push \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

Thời gian: ~5–10 phút (upload ~2GB)


### Bước 5: Cập nhật Lambda Function
Trên AWS Console:
1. Lambda → storm-prediction
2. Tab **Image** → **Deploy new image**
3. Chọn image `latest`
4. Nhấn **Save**


## Cấu hình Lambda

### Thiết lập Function
```text
Name: storm-prediction
Runtime: Container image
Architecture: x86_64
Memory: 3008 MB
Timeout: 120 seconds
Ephemeral storage: 512 MB
```

### Biến môi trường
```text
MODEL_BUCKET=storm-frontend-hosting-duc-2025
DATA_BUCKET=storm-frontend-hosting-duc-2025
```


### Function URL
```text
URL: https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws
Auth type: NONE
CORS: Enabled
  - Allow origins: *
  - Allow methods: POST, OPTIONS
  - Allow headers: Content-Type
```

### Quyền IAM Role
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::storm-frontend-hosting-duc-2025/*",
        "arn:aws:s3:::storm-ai-models/*"
      ]
    }
  ]
}
```


## Giám sát & Logs

### CloudWatch Logs
**Log Group**: `/aws/lambda/storm-prediction`

Các log quan trọng:
```text
Loading LSTM model...
Downloaded LSTM from S3
LSTM loaded successfully

Loading TCN model...
Checking: /var/task/models/cropping_storm_7304_2l.pth
Found TCN at /var/task/models/cropping_storm_7304_2l.pth
TCN loaded successfully

Processing: Test Storm (TEST001)
Input points: 9
Predicted total distance: 500.50 km
Generated 24 predictions (72 hours)
Saved to S3: predictions/TEST001_1733486400.json
```


### Metrics
CloudWatch Metrics:
- Invocations
- Duration (trung bình ~5–10 giây)
- Errors
- Throttles
- Memory used (~500–800 MB)


## Lỗi thường gặp & cách xử lý

### 1. UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80
Triệu chứng:
```text
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 64
```

Nguyên nhân: file model `.pth` ở thư mục gốc bị Lambda hiểu nhầm như file cấu hình Python

Cách xử lý: chuyển model vào thư mục con
```dockerfile
RUN mkdir -p ${LAMBDA_TASK_ROOT}/models
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/models/
```

### 2. 502 Bad Gateway
Triệu chứng: Frontend nhận lỗi 502

Nguyên nhân có thể:
- Lambda timeout (quá 120s)
- Lambda crash (hết bộ nhớ)
- Load model thất bại

Cách xử lý:
1. Kiểm tra CloudWatch Logs
2. Tăng memory nếu cần
3. Tăng timeout nếu cần

### 3. LSTM Fallback
Đặc điểm: log có " Using fallback distance"

Nguyên nhân: chưa có model LSTM trên S3

Cách xử lý: upload `lstm_totald_256_4.pt` lên S3
```bash
aws s3 cp lstm_totald_256_4.pt \
  s3://storm-frontend-hosting-duc-2025/models/
```

### 4. ECR Push 403 Forbidden
Đặc điểm: `403 Forbidden` khi push image

Nguyên nhân có thể:
- Hết hạn đăng nhập ECR
- Sai account ID
- Repo chưa tồn tại

Cách xử lý:
```bash
# Đăng nhập lại
aws ecr get-login-password --region ap-southeast-1 | \
  docker login --username AWS --password-stdin \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com

# Tạo repository nếu cần
aws ecr create-repository \
  --repository-name storm-prediction \
  --region ap-southeast-1
```

## Kiểm thử

### Test local (nếu có thể)
```python
# Chạy local
python app.py

# Test event
test_event = {
    "history": [
        {"lat": 15.0, "lng": 120.0},
        {"lat": 15.1, "lng": 120.1},
        ...
    ],
    "storm_name": "Test Storm"
}

result = handler(test_event, None)
print(result)
```

### Test trực tiếp trên Lambda
Trên AWS Console:
1. Lambda → tab Test
2. Tạo test event:
```json
{
  "history": [
    {"lat": 15.0, "lng": 120.0},
    {"lat": 15.1, "lng": 120.1},
    {"lat": 15.2, "lng": 120.2},
    {"lat": 15.3, "lng": 120.3},
    {"lat": 15.4, "lng": 120.4},
    {"lat": 15.5, "lng": 120.5},
    {"lat": 15.6, "lng": 120.6},
    {"lat": 15.7, "lng": 120.7},
    {"lat": 15.8, "lng": 120.8}
  ],
  "storm_name": "Test Storm"
}
```
3. Nhấn **Test**
4. Kiểm tra response

### Test bằng cURL
`bash
curl -X POST \
  "https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "history": [
      {"lat": 15.0, "lng": 120.0},
      {"lat": 15.1, "lng": 120.1},
      {"lat": 15.2, "lng": 120.2},
      {"lat": 15.3, "lng": 120.3},
      {"lat": 15.4, "lng": 120.4},
      {"lat": 15.5, "lng": 120.5},
      {"lat": 15.6, "lng": 120.6},
      {"lat": 15.7, "lng": 120.7},
      {"lat": 15.8, "lng": 120.8}
    ],
    "storm_name": "Test Storm"
  }'
`

## Checklist triển khai

- File model `cropping_storm_7304_2l.pth` tồn tại
- (Tùy chọn) Upload model LSTM lên S3
- Build Docker image thành công
- Tag image đúng account ID (339570693867)
- Login ECR thành công
- Push image lên ECR
- Update Lambda function với image mới
- Kiểm tra cấu hình Lambda (memory, timeout)
- Test Lambda với test event
- Test qua Function URL bằng cURL
- Test từ frontend
- Kiểm tra CloudWatch Logs
- Xác nhận kết quả dự đoán hiển thị trên bản đồ

## Ảnh chụp

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-13.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Hình 1</strong>
</p>

#### Cấu hình (Configuration)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-14.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Hình 2</strong>
</p>

#### Biến môi trường (Environment Variables)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-15.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Hình 3</strong>
</p>

## ECR

### Kho lưu trữ (Repository)

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-16.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Hình 4</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-17.png" alt="Picture 5" />
  <br/>
  <strong style="font-size: 18px;">Hình 5</strong>
</p>
