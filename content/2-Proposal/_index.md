
---
title: "Proposal"
date: "2025-09-09"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# 🌪️ Typhoon Path Prediction Website (AWS Serverless Architecture)

## I. SYSTEM OVERVIEW

The *Typhoon Path Prediction Website* is a *serverless web application* built entirely on *Amazon Web Services (AWS)*.  
It predicts storm trajectories using *real-time weather data* collected from external APIs and a *pre-trained Machine Learning (ML) model* stored in *Amazon S3*.

### System Components:
- *Frontend Layer* hosted on *Amazon S3* and delivered via *CloudFront*.  
- *Backend Layer* powered by *AWS Lambda* and *Amazon API Gateway*.  
- *Automation & Data Scheduling* managed by *EventBridge*.  
- *Security & Monitoring* handled by *Secrets Manager* and *CloudWatch*.  

✅ This architecture ensures *low cost*, *auto scalability*, and *zero server maintenance*.

---

## II. SYSTEM ARCHITECTURE

### 🧭 1. Frontend & Content Delivery

| AWS Service | Role | Description |
|--------------|------|-------------|
| *Amazon Route 53* | DNS management | Routes user traffic to CloudFront for web delivery. |
| *Amazon CloudFront* | CDN | Delivers static content globally with low latency. |
| *Amazon S3 (Static Files)* | Static hosting | Hosts frontend files (HTML, CSS, JS). |

---

### ⚙️ 2. Backend & API Processing

| AWS Service | Role | Description |
|--------------|------|-------------|
| *Amazon API Gateway* | Entry point | Handles HTTP API requests from the frontend. |
| *AWS Lambda (Storm Prediction)* | ML function | Loads ML model from S3 and predicts storm movement. |
| *AWS Lambda (Fetch Recent Storm Data)* | Data fetcher | Retrieves recent storm information from S3. |
| *Amazon S3 (Model Bucket)* | ML model storage | Stores .h5 or .pkl files used by the prediction Lambda. |
| *Amazon S3 (Recent Storms Bucket)* | Weather data storage | Stores data fetched by the crawler Lambda. |

---

### ⚙️ 3. Automation & Data Crawling

| AWS Service | Role | Description |
|--------------|------|-------------|
| *Amazon EventBridge* | Scheduler | Triggers Lambda (Web Crawler) every Sunday. |
| *AWS Lambda (Web Crawler)* | Data automation | Fetches new storm data from external APIs. |
| *AWS Secrets Manager* | Secure storage | Stores API keys for the external weather service. |

---

### 🔐 4. Security & Monitoring

| AWS Service | Role | Description |
|--------------|------|-------------|
| *Amazon CloudWatch* | Logs and metrics | Collects Lambda execution logs and system metrics. |
| *AWS IAM Roles* | Permissions | Assigns least-privilege access for Lambda functions. |

---

### 👩‍💻 5. Development Workflow

| Component | Role | Description |
|------------|------|-------------|
| *GitHub Repository* | Version control | Stores source code for frontend and Lambda functions. |
| *Developer (Dev)* | Maintenance | Uploads new ML models to S3 and pushes code to GitHub. |

---

## III. SYSTEM FLOW (Simplified Overview)

1. User accesses the website via *Route 53 → CloudFront → S3 (static files)*.  
2. Frontend calls *API Gateway* for storm prediction or recent data.  
3. API Gateway invokes Lambda functions:
   - *Storm Prediction* → loads model from S3 and predicts typhoon path.  
   - *Fetch Recent Data* → retrieves last known storm records from S3.  
4. *EventBridge* triggers *Web Crawler Lambda* weekly to fetch new data.  
5. *Crawler Lambda*:
   - Reads API key from *Secrets Manager*.
   - Calls external weather API.
   - Stores results in *S3*.
6. *CloudWatch* collects logs and performance metrics.  
7. *Developer* uploads new ML models to S3 when updates are ready.

---

## IV. MONTHLY COST BREAKDOWN (AWS Pricing Calculator)

*Region:* ap-southeast-1 (Singapore)

---

### 💻 1. Frontend & Content Delivery

| Service | Usage | Monthly Cost (USD) |
|----------|--------|--------------------|
| Amazon S3 (Static Files) | 5 GB storage, 10 GB transfer | $0.50 |
| Amazon CloudFront | 50 GB data transfer, 1M requests | $4.25 |
| Amazon Route 53 | 1 hosted zone, 1M queries | $0.50 |
| AWS Certificate Manager (ACM) | TLS certificate | $0.00 |

*Subtotal:*
*$5.25 / month*

---

### ⚙️ 2. Backend (API & ML Processing)

| Service | Usage | Monthly Cost (USD) |
|----------|--------|--------------------|
| Amazon API Gateway | 1M API requests/month | $3.50 |
| Lambda (Storm Prediction) | 1,000/day × 2GB × 3s | $5.00 |
| Lambda (Fetch Recent Storm Data) | 500/day × 512MB × 1s | $0.30 |

*Subtotal:*
*$8.80 / month*

---

### 🕒 3. Automation & Data Crawling

| Service | Usage | Monthly Cost (USD) |
|----------|--------|--------------------|
| EventBridge | 1 cron trigger/week | $0.00 (Free Tier) |
| Lambda (Web Crawler) | 24/day × 512MB × 30s | $0.50 |
| Secrets Manager | 5 secrets | $2.00 |

*Subtotal:*
*$2.50 / month*

---

### 📊 4. Monitoring & Logging

| Service | Usage | Monthly Cost (USD) |
|----------|--------|--------------------|
| CloudWatch Logs | 5 GB ingestion + 1 GB retention | $2.50 |
| CloudWatch Metrics | 20 metrics | $0.60 |

*Subtotal:*
*$3.10 / month*

---

### 💾 5. Storage & Data Transfer

| Service | Usage | Monthly Cost (USD) |
|----------|--------|--------------------|
| S3 (Model Bucket) | 5 GB storage | $0.50 |
| S3 (Recent Storms Bucket) | 20 GB storage | $1.00 |
| Outbound Data Transfer | 20 GB/month | $1.80 |

*Subtotal:*
*$3.30 / month*

---

## 💰 TOTAL ESTIMATED MONTHLY COST

| Category | Monthly Cost (USD) |
|-----------|--------------------|
| Frontend & CDN | $5.25 |
| Backend (API + ML) | $8.80 |
| Automation (Crawler + Secrets) | $2.50 |
| Monitoring & Logging | $3.10 |
| Storage & Data Transfer | $3.30 |

### ✅ *TOTAL: ≈ $22.95 / month*
