---
title : "Introduction"
date: 2025-09-09
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

# ONLINE PLATFORM FOR TRACKING AND FORECASTING HURRICANE TRAJECTORY

In this workshop, we will present how we created an online platform that allows internet users to freely check, track, and even predict the path of ongoing storms in the West Pacific region. This platform helps users better prepare for upcoming natural disasters and reduces the potential damage they may cause.

The platform provides **two main functionalities**:

1. **Showing Recent Storms** – Allows users to view the path, intensity, wind speed, and other characteristics of recent storms in the West Pacific region.
2. **Predicting Hurricane Trajectories** – Allows users to input past storm locations (latitude and longitude; at least 9 data points) to obtain predictions about the storm’s near-future movement, intensity changes, and potential path.

Following the flow of this workshop, we will discuss the **datasets**, **pre-processing steps**, **model-training pipeline**, and the process of **building the online platform using AWS services**. We will also demonstrate our proposed augmentation techniques—**Stepwise Temporal Fading Augmentation (STFA)** and **Plausible Geodesic Bearing Augmentation (PGBA)**—along with the use of **physics-informed machine learning**. These approaches enhance the realism of the training data and significantly improve prediction accuracy for storm trajectories, lifetime estimates, and total travel distance.

<p align="center">
  <img src="/images/5-Workshop/5.1-Workshop-overview/trainning_process.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1 : Model pipeline</strong>
</p>

Once the model-training process is completed, we move to building the online platform using a **serverless architecture**. This architecture is cost-efficient, scalable, and easy to maintain/deploy—making it an ideal choice for our project. Below are the main AWS services used:

* **AWS Lambda** – Executes the ML models and handles backend logic
* **Amazon S3** – Stores static files, trained models, and storm data
* **Amazon API Gateway** – Routes user requests to the appropriate Lambda functions depending on whether they are viewing recent storms or running predictions
* **Amazon CloudFront** – Speeds up content delivery through edge locations
* **AWS Secrets Manager** – Stores API keys and other sensitive information
* **…** – Additional supporting services as needed

<p align="center">
  <img src="/images/5-Workshop/5.1-Workshop-overview/skynet_diagram.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2 : Platform Architecture</strong>
</p>

