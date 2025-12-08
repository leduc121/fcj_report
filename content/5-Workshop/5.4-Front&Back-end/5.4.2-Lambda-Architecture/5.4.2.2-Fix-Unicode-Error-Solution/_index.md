---
title : "Fix-Unicode-Error"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2.2 </b> "
---

# Fixing UnicodeDecodeError in Lambda

There is a series error we want to talk about during the process of development, and this section will talk about it.

## Issue
```text
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80 in position 64: invalid start byte
```

## Root Cause
- The PyTorch model uses the `.pth` extension (a binary file).
- The Python runtime also uses `.pth` files for path configuration (text files).
- If the model `.pth` is placed directly in `LAMBDA_TASK_ROOT`, Python may try to read it as text → causing the error.

## Applied Fix

### 1. Update Dockerfile
Move the model into a subdirectory `models/`:

```dockerfile
# Copy model file to subdirectory to avoid Python .pth file confusion
RUN mkdir -p ${LAMBDA_TASK_ROOT}/models
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/models/
```

### 2. Update app.py
Update the model search paths:

```python
possible_paths = [
    '/var/task/models/cropping_storm_7304_2l.pth',  # Primary location in Lambda
    'models/cropping_storm_7304_2l.pth',
    tcn_path
]
```

## Rebuild & Deploy Steps

### Step 1: Build the Docker image
```bash
cd storm_prediction
docker build --provenance=false --platform linux/amd64 -t storm-prediction-model .
```

Note: `--provenance=false` helps reduce image size for pushing to ECR.

### Step 2: Tag the image
```bash
docker tag storm-prediction-model:latest 211125445874.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

### Step 3: Login to ECR
```bash
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 211125445874.dkr.ecr.ap-southeast-1.amazonaws.com
```

### Step 4: Push to ECR
```bash
docker push 211125445874.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction:latest
```

### Step 5: Update Lambda
1. AWS Console → Lambda → storm-prediction  
2. Open the **Image** tab  
3. Click **Deploy new image**  
4. Select the latest image  
5. Click **Save**

### Step 6: Test
```bash
curl -X POST "https://vill3povlzqxdyxm7ubldizobu0kdgbi.lambda-url.ap-southeast-1.on.aws/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "history": [
      {"lat": 14.5, "lng": 121.0},
      {"lat": 14.6, "lng": 121.1},
      {"lat": 14.7, "lng": 121.2},
      {"lat": 14.8, "lng": 121.3},
      {"lat": 14.9, "lng": 121.4},
      {"lat": 15.0, "lng": 121.5},
      {"lat": 15.1, "lng": 121.6},
      {"lat": 15.2, "lng": 121.7},
      {"lat": 15.3, "lng": 121.8}
    ],
    "storm_name": "Test Storm"
  }'
```

## Check Logs After Deploy
```bash
aws logs tail /aws/lambda/storm-prediction --region ap-southeast-1 --follow
```

## Summary
- Before: The `.pth` model file was in the root → Python mistook it for a config `.pth` file → UnicodeDecodeError
- After: The `.pth` model file is inside `models/` → Python ignores it → Lambda works ✅
