---
title : "Lambda Architecture"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2 </b> "
---


# Lambda Architecture - Storm Prediction AI Service

## Overview

Lambda functions are an important component of a serverless architecture. They are especially useful due to their cost-effectiveness and ease of deployment—both of which are valuable for our hurricane prediction platform.

This section presents the details of how we designed and built our Lambda architecture.

Our Lambda functions run PyTorch models for typhoon trajectory prediction and are deployed using a Docker container image.

## AWS Services Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend (Browser)                     │
└────────────────────────┬────────────────────────────────────┘
                         │ POST /predict
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Lambda Function URL (Public)                   │
│  URL: https://vill3povlzqxdyxm7ubldizobu0kdgbi...           │
│  Auth: NONE                                                 │
│  Method: POST                                               │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   Lambda Function                           │
│  Name: storm-prediction                                     │
│  Runtime: Python 3.10 (Container)                           │
│  Memory: 3008 MB                                            │
│  Timeout: 120 seconds                                       │
│  Architecture: x86_64                                       │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   ECR Repository                            │
│  Account: 339570693867                                      │
│  Region: ap-southeast-1                                     │
│  Repo: storm-prediction                                     │
│  Image: latest                                              │
│  Size: ~2 GB                                                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   S3 Buckets                                │
│  1. storm-frontend-hosting-duc-2025                         │
│     - models/lstm_totald_256_4.pt (optional)                │
│     - predictions/[storm_id]_[timestamp].json               │
│                                                             │
│  2. storm-ai-models (recommended)                           │
│     - models/lstm_totald_256_4.pt                           │
│     - models/tcn_model.pth (backup)                         │
└─────────────────────────────────────────────────────────────┘
```

## storm_prediction/ Directory Structure

```
storm_prediction/
├── app.py                          # Lambda handler (main code)
├── Dockerfile                      # Container definition
├── requirements.txt                # Python dependencies
├── cropping_storm_7304_2l.pth     # TCN model (included in image)
│
├── DEPLOY_NOW.md                   # Quick deploy guide
├── DEPLOY_CONSOLE_STEP_BY_STEP.md # AWS Console guide
├── LAMBDA_DEPLOYMENT_GUIDE.md      # Detailed deployment
├── AWS_CONSOLE_DEPLOYMENT_GUIDE.md
├── FIX_ECR_PUSH_ERROR.md          # Troubleshooting
├── FIX_UNICODE_ERROR.md           # UnicodeDecodeError fix
├── FIX_UNICODE_ERROR_SOLUTION.md  # Solution details
└── REBUILD_AND_DEPLOY.sh          # Automated script
```

## Docker Image Structure

### Dockerfile
```dockerfile
FROM public.ecr.aws/lambda/python:3.10

# Install dependencies
COPY requirements.txt .
RUN pip3 install -r requirements.txt \
    --target "${LAMBDA_TASK_ROOT}" \
    --extra-index-url https://download.pytorch.org/whl/cpu

# Copy Lambda handler
COPY app.py ${LAMBDA_TASK_ROOT}

# Copy TCN model to subdirectory (avoid .pth confusion)
RUN mkdir -p ${LAMBDA_TASK_ROOT}/models
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/models/

# Set handler
CMD [ "app.handler" ]
```

### Image Layers
```
Layer 1: AWS Lambda Python 3.10 base (~500 MB)
Layer 2: PyTorch CPU + dependencies (~1.2 GB)
Layer 3: app.py + TCN model (~300 MB)
─────────────────────────────────────────────
Total: ~2 GB
```

##  AI Models

### 1. TCN Model (Trajectory Prediction)
**File**: `cropping_storm_7304_2l.pth`
**Location**: Inside Docker image at `/var/task/models/`
**Size**: ~300 MB
**Purpose**: Predict next step (lat, lng) of typhoon trajectory

**Architecture**:
```python
class StormTCN(nn.Module):
    def __init__(self, input_dim=4, hidden_units=1024, num_layers=2):
        self.tcn = TCN(...)
        self.head_latlon = nn.Linear(hidden_units, 2)  # Predict lat, lng
        self.head_aux = nn.Linear(hidden_units, 2)     # Predict aux features
```

**Input**: `[batch, sequence, 4]` - (lat, lng, distance, bearing)
**Output**: 
- `pred_latlon`: Next (lat, lng)
- `pred_aux`: Auxiliary features

### 2. LSTM Model (Total Distance Prediction)
**File**: `lstm_totald_256_4.pt`
**Location**: S3 bucket (downloaded on first use)
**Size**: ~50 MB
**Purpose**: Predict total distance typhoon will travel

**Architecture**:
```python
class StormLSTM(nn.Module):
    def __init__(self, input_size=4, hidden_size=256, num_layers=2):
        self.lstm = nn.LSTM(...)
        self.fc = nn.Sequential(
            nn.Linear(hidden_size, hidden_size // 2),
            nn.ReLU(),
            nn.Linear(hidden_size // 2, 1)  # Predict total distance
        )
```

**Input**: Daily summary `[batch, days, 4]` - (day, daily_dist, avg_speed, motion_type)
**Output**: Total distance (km)

## Request Flow

### 1. Receive Request
```
POST /predict
Content-Type: application/json

{
  "history": [
    {"lat": 15.0, "lng": 120.0},
    {"lat": 15.1, "lng": 120.1},
    ...  // Min 9 points
  ],
  "storm_name": "Test Storm",
  "storm_id": "TEST001"  // Optional
}
```

### 2. Load Models (First Invocation Only)
```python
def load_models():
    global LSTM_MODEL, TCN_MODEL
    
    # Load LSTM from S3 (if available)
    if not os.path.exists('/tmp/lstm_model.pt'):
        s3_client.download_file(
            MODEL_BUCKET,
            'models/lstm_totald_256_4.pt',
            '/tmp/lstm_model.pt'
        )
    LSTM_MODEL = StormLSTM(...)
    LSTM_MODEL.load_state_dict(torch.load('/tmp/lstm_model.pt'))
    
    # Load TCN from local (already in image)
    TCN_MODEL = StormTCN(...)
    TCN_MODEL.load_state_dict(
        torch.load('/var/task/models/cropping_storm_7304_2l.pth')
    )
```

### 3. Preprocess Input
```python
def preprocess_history(history):
    # Convert to tensor [1, sequence_length, 4]
    # Features: [lat, lng, distance, bearing]
    
    processed = []
    for i in range(len(history)):
        if i == 0:
            processed.append([lat, lng, 0.0, 0.0])
        else:
            dist = haversine(prev_lat, prev_lng, lat, lng)
            brng = bearing(prev_lat, prev_lng, lat, lng)
            processed.append([lat, lng, dist, brng])
    
    return torch.tensor(processed).unsqueeze(0)
```

### 4. Predict Total Distance (LSTM)
```python
def predict_total_distance(record_tensor):
    if LSTM_MODEL is None:
        # Fallback: avg_distance * 24 steps
        return fallback_distance
    
    # Group by day (9 points/day)
    # Run LSTM prediction
    with torch.no_grad():
        pred = LSTM_MODEL(summary_tensor, lengths)
    
    return pred.item()  # km
```

### 5. Predict Path (TCN)
```python
def predict_storm_path(record_tensor, total_distance, history):
    seq = record_tensor.clone()
    gone_distance = 0
    predicted_points = []
    
    while gone_distance < total_distance:
        # Predict next position
        pred_latlon, pred_aux = TCN_MODEL(seq)
        new_lat = pred_latlon[0, -1, 0].item()
        new_lng = pred_latlon[0, -1, 1].item()
        
        # Calculate distance & bearing
        step_distance = haversine(last_lat, last_lng, new_lat, new_lng)
        
        # Estimate windspeed (decay over time)
        estimated_wind = max(avg_wind * (0.98 ** step), 30)
        
        predicted_points.append({
            'lat': new_lat,
            'lng': new_lng,
            'timestamp': base_timestamp + (step * 3 * 3600 * 1000),
            'windSpeed': estimated_wind,
            'pressure': 980.0,
            'category': calculate_category(estimated_wind)
        })
        
        # Update sequence (sliding window)
        seq = torch.cat([seq[:, 1:, :], next_point.unsqueeze(1)], dim=1)
        gone_distance += step_distance
        step += 1
    
    return predicted_points
```

### 6. Return Response
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


## Build & Deploy Process

### Step 1: Build Docker Image
```bash
cd storm_prediction

docker build \
  --provenance=false \
  --platform linux/amd64 \
  -t storm-prediction-model .
```

**Flags**:
- `--provenance=false`: Reduce image size (no build metadata)
- `--platform linux/amd64`: Lambda only supports x86_64
- `-t storm-prediction-model`: Tag name

### Step 2: Tag for ECR
```bash
docker tag storm-prediction-model:latest \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

### Step 3: Login to ECR
```bash
aws ecr get-login-password --region ap-southeast-1 | \
  docker login --username AWS --password-stdin \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com
```

### Step 4: Push to ECR
```bash
docker push \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

**Time**: ~5-10 minutes (2GB upload)

### Step 5: Update Lambda Function
**AWS Console**:
1. Lambda → storm-prediction
2. Tab **Image** → **Deploy new image**
3. Select latest image
4. Click **Save**


## Lambda Configuration

### Function Settings
```
Name: storm-prediction
Runtime: Container image
Architecture: x86_64
Memory: 3008 MB
Timeout: 120 seconds
Ephemeral storage: 512 MB
```

### Environment Variables
```
MODEL_BUCKET=storm-frontend-hosting-duc-2025
DATA_BUCKET=storm-frontend-hosting-duc-2025
```

### Function URL
```
URL: https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws
Auth type: NONE
CORS: Enabled
  - Allow origins: *
  - Allow methods: POST, OPTIONS
  - Allow headers: Content-Type
```

### IAM Role Permissions
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

## Monitoring & Logs

### CloudWatch Logs
**Log Group**: `/aws/lambda/storm-prediction`

**Key Log Messages**:
```
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

**Screenshots needed:**
- [ ] CloudWatch → Log groups → /aws/lambda/storm-prediction
- [ ] Log stream → Recent logs with emojis
- [ ] Logs → Duration, Memory used

### Metrics
**CloudWatch Metrics**:
- Invocations
- Duration (avg ~5-10 seconds)
- Errors
- Throttles
- Memory used (~500-800 MB)

**Screenshots needed:**
- [ ] Lambda → Monitor → Metrics
- [ ] CloudWatch → Metrics → Lambda → Function metrics

## Common Issues & Solutions

### 1. UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80
**Symptom**:
```
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 64
```

**Cause**: Model `.pth` file at root mistaken as Python config file

**Solution**: Move model to subdirectory
```dockerfile
RUN mkdir -p ${LAMBDA_TASK_ROOT}/models
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/models/
```

### 2. 502 Bad Gateway
**Symptom**: Frontend receives 502 error

**Causes**:
- Lambda timeout (exceeds 120s)
- Lambda crash (out of memory)
- Model failed to load

**Solutions**:
1. Check CloudWatch Logs
2. Increase memory if needed
3. Increase timeout if needed

### 3. LSTM Fallback
**Symptom**: Log shows "⚠️ Using fallback distance"

**Cause**: LSTM model not on S3

**Solution**: Upload `lstm_totald_256_4.pt` to S3:
```bash
aws s3 cp lstm_totald_256_4.pt \
  s3://storm-frontend-hosting-duc-2025/models/
```

### 4. ECR Push 403 Forbidden
**Symptom**: `403 Forbidden` when pushing image

**Causes**: 
- ECR login expired
- Wrong account ID
- Repository doesn't exist

**Solutions**:
```bash
# Re-login
aws ecr get-login-password --region ap-southeast-1 | \
  docker login --username AWS --password-stdin \
  339570693867.dkr.ecr.ap-southeast-1.amazonaws.com

# Create repository if needed
aws ecr create-repository \
  --repository-name storm-prediction \
  --region ap-southeast-1
```


## Testing

### Local Test (if possible)
```python
# Run locally
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

### Lambda Test
**AWS Console**:
1. Lambda → Test tab
2. Create test event:
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
3. Click **Test**
4. Check response


### cURL Test
```bash
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
```

## Deployment Checklist

- Model file `cropping_storm_7304_2l.pth` exists
- (Optional) Upload LSTM model to S3
- Build Docker image successfully
- Tag image with correct account ID (339570693867)
- Login to ECR successfully
- Push image to ECR
- Update Lambda function with new image
- Check Lambda configuration (memory, timeout)
- Test Lambda with test event
- Test via Function URL with cURL
- Test from frontend
- Check CloudWatch Logs
- Verify prediction results on map

## Screenshots

#### Function

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-13.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1</strong>
</p>

#### Configuration

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-14.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2</strong>
</p>

#### Environment Variables

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-15.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3</strong>
</p>

## ECR

### Repository

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-16.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Figure 4</strong>
</p>

<p align="center">
  <img src="/images/5-Workshop/5.4-FB_END/5.4.2-Lambda/image-17.png" alt="Picture 5" />
  <br/>
  <strong style="font-size: 18px;">Figure 5</strong>
</p>
