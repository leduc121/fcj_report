---
title : "Lamda-Deployment"
date: 2025-09-09
weight : 4 
chapter : false
pre : " <b> 5.4.2.3 </b> "
---

# Step to deploy our PyTorch Model on storm prediction on AWS Lambda

AWS Lambda deployment plays a critical role in our website development pipeline. In this section, we document the process we followed to successfully complete the deployment.

### Step 1: Prepare the Code

#### 1.1. Update app.py
```python
import json
import torch
import numpy as np
from typing import List, Dict

MODEL_PATH = "model.pth"
device = torch.device("cpu")  
model = None

def load_model():
    global model
    if model is None:
        print(f"Loading model from {MODEL_PATH}...")
        model = torch.load(MODEL_PATH, map_location=device)
        model.eval()
        print("Model loaded successfully!")
    return model

def prepare_features(history: List[Dict]) -> torch.Tensor:
    """
    Convert history into a tensor for the model
    history: [{"lat": 15.0, "lng": 107.0}, ...]
    """
    # TODO: Implement feature engineering based on your model
    lats = [p["lat"] for p in history]
    lngs = [p["lng"] for p in history]
    
    features = np.array([lats + lngs])  # Shape: (1, 18)
    return torch.tensor(features, dtype=torch.float32)

def format_predictions(predictions: torch.Tensor, storm_name: str) -> Dict:
    """
    Format output to match what the frontend expects
    """
    pred_array = predictions.detach().cpu().numpy()[0]
    
    forecast = []
    base_timestamp = int(time.time() * 1000)
    
    for i in range(0, len(pred_array), 2):
        if i + 1 < len(pred_array):
            forecast.append({
                "lat": float(pred_array[i]),
                "lng": float(pred_array[i + 1]),
                "timestamp": base_timestamp + (i // 2) * 3600000,  # +1 hour each
                "windSpeed": 120.0,  # TODO: Predict from model
                "pressure": 980.0,   # TODO: Predict from model
                "category": "Category 3",  # TODO: Classify from windSpeed
                "confidence": 0.85
            })
    
    return {
        "storm_name": storm_name,
        "forecast": forecast
    }

def handler(event, context):
    """
    Lambda handler function
    """
    try:
        # Parse input
        body = json.loads(event.get('body', '{}'))
        history = body.get('history', [])
        storm_name = body.get('storm_name', 'Unknown Storm')
        
        # Validate input
        if len(history) < 9:
            return {
                'statusCode': 400,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*'
                },
                'body': json.dumps({
                    'error': f'Need at least 9 positions, got {len(history)}'
                })
            }
        
        # Load model
        model = load_model()
        
        # Prepare features
        X = prepare_features(history)
        
        # Predict
        with torch.no_grad():
            predictions = model(X)
        
        # Format output
        result = format_predictions(predictions, storm_name)
        
        return {
            'statusCode': 200,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps(result)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({
                'error': str(e)
            })
        }
```

#### 1.2. Update Dockerfile
```dockerfile
FROM public.ecr.aws/lambda/python:3.11

# Copy requirements and install
COPY requirements.txt ${LAMBDA_TASK_ROOT}
RUN pip install --no-cache-dir -r requirements.txt

# Copy model (rename to model.pth)
COPY cropping_storm_7304_2l.pth ${LAMBDA_TASK_ROOT}/model.pth

# Copy code
COPY app.py ${LAMBDA_TASK_ROOT}

# Set handler
CMD ["app.handler"]
```

#### 1.3. Verify requirements.txt
```txt
torch==2.1.0
numpy==1.24.3
```

### Step 2: Build the Docker Image
```bash
cd storm_prediction

# Build image
docker build -t storm-prediction-model .

# Test locally (optional)
docker run -p 9000:8080 storm-prediction-model

# Test with curl
curl -X POST "http://localhost:9000/2015-03-31/functions/function/invocations" \
  -d '{
    "body": "{\"history\": [{\"lat\": 15.0, \"lng\": 107.0}, {\"lat\": 15.1, \"lng\": 107.1}, {\"lat\": 15.2, \"lng\": 107.2}, {\"lat\": 15.3, \"lng\": 107.3}, {\"lat\": 15.4, \"lng\": 107.4}, {\"lat\": 15.5, \"lng\": 107.5}, {\"lat\": 15.6, \"lng\": 107.6}, {\"lat\": 15.7, \"lng\": 107.7}, {\"lat\": 15.8, \"lng\": 107.8}], \"storm_name\": \"Test Storm\"}"
  }'
```

### Step 3: Upload to AWS ECR
```bash
# 1. Create ECR repository
aws ecr create-repository \
  --repository-name storm-prediction-model \
  --region ap-southeast-1

# 2. Login Docker to ECR
aws ecr get-login-password --region ap-southeast-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com

# 3. Tag image
docker tag storm-prediction-model:latest \
  <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction-model:latest

# 4. Push image
docker push <account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction-model:latest
```

### Step 4: Create the Lambda Function

#### 4.1. Create Lambda from Console
1. Open AWS Lambda Console
2. Click “Create function”
3. Choose “Container image”
4. Function name: `storm-prediction`
5. Container image URI: select the image you pushed to ECR
6. Architecture: x86_64
7. Click “Create function”

#### 4.2. Configure Lambda
```bash
# Or use AWS CLI
aws lambda create-function \
  --function-name storm-prediction \
  --package-type Image \
  --code ImageUri=<account-id>.dkr.ecr.ap-southeast-1.amazonaws.com/storm-prediction-model:latest \
  --role arn:aws:iam::<account-id>:role/lambda-execution-role \
  --timeout 60 \
  --memory-size 3008 \
  --region ap-southeast-1
```

Important configuration:
- Memory: 3008 MB (PyTorch models need RAM)
- Timeout: 60 seconds (inference can take 10–30s)
- Ephemeral storage: 512 MB (default; increase if needed)

#### 4.3. Create IAM Role
Lambda needs a role with permissions:
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
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```


### Step 5: Create API Gateway
```bash
# 1. Create REST API
aws apigateway create-rest-api \
  --name storm-prediction-api \
  --region ap-southeast-1

# 2. Get API ID and Root Resource ID
API_ID=<your-api-id>
ROOT_ID=<your-root-resource-id>

# 3. Create resource /predict
aws apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_ID \
  --path-part predict

# 4. Create POST method
RESOURCE_ID=<predict-resource-id>
aws apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --authorization-type NONE

# 5. Integrate with Lambda
aws apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:ap-southeast-1:lambda:path/2015-03-31/functions/arn:aws:lambda:ap-southeast-1:<account-id>:function:storm-prediction/invocations

# 6. Deploy API
aws apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name prod
```

API URL: `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod/predict`


### Step 6: Update the Frontend

#### 6.1. Update `.env.production`
```env
VITE_PREDICTION_API_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/prod
```

#### 6.2. Build & deploy frontend
```bash
cd frontend
npm run build
# Deploy dist/ to S3/CloudFront
```


##  Optimizations

### 1. Reduce Cold Start
Provisioned Concurrency:
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name storm-prediction \
  --provisioned-concurrent-executions 1 \
  --qualifier $LATEST
```

### 2. Reduce Image Size

Use PyTorch CPU-only:
```txt
# requirements.txt
torch==2.1.0+cpu --extra-index-url https://download.pytorch.org/whl/cpu
numpy==1.24.3
```

Multi-stage build:
```dockerfile
# Stage 1: Build
FROM python:3.11-slim as builder
COPY requirements.txt .
RUN pip install --target /packages -r requirements.txt

# Stage 2: Runtime
FROM public.ecr.aws/lambda/python:3.11
COPY --from=builder /packages ${LAMBDA_RUNTIME_DIR}
COPY model.pth ${LAMBDA_TASK_ROOT}/
COPY app.py ${LAMBDA_TASK_ROOT}/
CMD ["app.handler"]
```

### 3. Cache the Model in /tmp
```python
import os

MODEL_PATH = "/tmp/model.pth" if os.path.exists("/tmp/model.pth") else "model.pth"

def load_model():
    global model
    if model is None:
        # Copy to /tmp for faster access
        if not os.path.exists("/tmp/model.pth"):
            import shutil
            shutil.copy("model.pth", "/tmp/model.pth")
        
        model = torch.load("/tmp/model.pth", map_location=device)
        model.eval()
    return model
```


##  Monitoring

### CloudWatch Logs
`bash
# View logs
aws logs tail /aws/lambda/storm-prediction --follow
`

### CloudWatch Metrics
- Invocations: number of calls
- Duration: runtime
- Errors: error count
- Throttles: throttled invocations

### Alerts
```bash
# Create an alarm for errors
aws cloudwatch put-metric-alarm \
  --alarm-name storm-prediction-errors \
  --alarm-description "Alert when Lambda has errors" \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --statistic Sum \
  --period 300 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=FunctionName,Value=storm-prediction
```


##  Troubleshooting

- Error: "Task timed out after 3.00 seconds"  
  Fix: Increase timeout to 60s

- Error: "Runtime exited with error: signal: killed"  
  Fix: Increase memory to 3008 MB

- Error: "No module named 'torch'"  
  Fix: Check requirements.txt and rebuild the image

- Error: Model cannot be loaded  
  Fix: Verify the model filename in Dockerfile and app.py match


##  Estimated Cost

* Lambda:
- Free tier: 1M requests/month, 400,000 GB-seconds
- After that: $0.20 per 1M requests + $0.0000166667 per GB-second

* Example: 10,000 requests/month, each request 10s, 3GB RAM
- Compute: 10,000 × 10s × 3GB × $0.0000166667 = ~$5/month
- Requests: 10,000 × $0.20/1M = ~$0.002/month
- Total: ~$5/month

* API Gateway:
- $3.50 per million requests
- 10,000 requests = ~$0.035/month

* ECR:
- $0.10 per GB/month storage
- Image ~2GB = ~$0.20/month

Estimated total: ~$5.25/month for 10,000 predictions


##  Final Checklist

- [ ] Fix the model filename mismatch in app.py or Dockerfile
- [ ] Test the Docker image locally
- [ ] Push the image to ECR
- [ ] Create Lambda with 3008MB memory, 60s timeout
- [ ] Create API Gateway and integrate with Lambda
- [ ] Test the API with Postman/curl
- [ ] Update `VITE_PREDICTION_API_URL` in the frontend
- [ ] Build and deploy the frontend
- [ ] Test the prediction form on the web UI
- [ ] Set up CloudWatch alerts
- [ ] Monitor logs and performance
