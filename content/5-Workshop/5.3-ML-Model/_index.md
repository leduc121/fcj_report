---
title : "Machine Learning Model"
date: 2025-09-09
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

# MODEL TRAINING PROCESS

This work presents the development of a predictive model designed to estimate a storm’s next geographical position using historical observational data from its previous path. To put it simply, we use a sequence of past latitude and longitude values to predict the next latitude and longitude in the future.

## Feature Engineering

After completing the data preprocessing stage, we split the dataset into **70% training**, **10% validation**, and **20% testing**. This is done **by storm ID**, ensuring that no storm appearing in the validation or test set is included in the training set. This prevents data leakage and ensures the reliability of evaluation results.

For model training, each input consists of a **sequence of 4 consecutive time steps**, where each step represents an observation interval of **3 hours**. Thus, one input sequence covers a total of **9 hours** of storm movement.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/dataset_split.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1 : Dataset Distribution</strong>
</p>

## Applying Stepwise Temporal Fading Augmentation (STFA)

To enhance the diversity of the training data, we apply our proposed method—**Stepwise Temporal Fading Augmentation (STFA)**—to **50% of the training set**, selected based on unique storm IDs. The original sequences from these storms are replaced by their augmented counterparts, ensuring the final size of the training set remains unchanged (approximately 100% of the original size).

As introduced earlier in the proposal section, STFA modifies the *older* points in a sequence while keeping the **most recent observations unchanged**. For each 4-step sequence:

* The **latest 2 time steps** remain the same
* The **older 2 time steps** are scaled using fading coefficients: [0.98,; 0.99]

Although these values appear small, **latitude and longitude are extremely sensitive** features. Even a tiny change—such as from 6.7 to 6.8—can correspond to **tens of kilometers** of displacement in the real world. Therefore, using modest fading values is both logical and physically meaningful, ensuring the augmented data remains realistic.

### **Example of STFA on a 4-Step Sequence**

Here is a simple example showing how a 4-step time-series sequence is transformed after applying STFA:

| Row | Original (lat, lon) | Augmented (lat, lon) | Operation              |
| --- | ------------------- | -------------------- | ---------------------- |
| 1   |  [-6.8 , 107.5]     |  [-6.66 , 105.35]   | multiplied by **0.98** |
| 2   |  [-7.0 , 107.1]     |  [-6.93 , 106.03]   | multiplied by **0.99** |
| 3   |  [-7.3 , 106.7]     |  [-7.3 , 106.7]     | unchanged              |
| 4   |  [-7.5 , 106.4]     |  [-7.5 , 106.4]     | unchanged              |

This process reduces the magnitude of **older** observations while keeping the **recent** ones intact. The augmentation introduces controlled variability, helping the model generalize better for trajectory forecasting.

Earlier, we used physics-informed machine learning to compute **distance** and **bearing** between time steps using the Haversine formula.
After STFA is applied, these values are **recomputed** based on the augmented coordinates. This guarantees that all physical features remain accurate and consistent with the updated trajectory.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/storm_augmentations_grid.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2 : Comparison of Augmentation Techniques on Storm Trajectories</strong>
</p>

## Model Setting

### 1.Physics-Informed Loss

Our use of the Haversine formula does not end at feature engineering. In addition to generating distance and bearing values, we also incorporate the **Haversine distance as a custom loss function**, alongside traditional losses such as MSE, RMSE, and MAPE.

Because the Haversine formula computes the true geodesic distance between two geographical points, it serves as a natural metric for evaluating the error in the model’s predicted storm location. A larger Haversine distance indicates that the prediction is far from the true position, whereas a smaller value means the model is performing well. The optimal Haversine loss, ideally, is 0 km.

The formula was previously introduced in **Section 2: Proposal**, so we do not rewrite it here.

**Example:**

* Model prediction: [-6.72, 107.1]
* True location: [-6.8, 107.5]
* Haversine loss: 45.06 km

The value 45.06 km directly reflects the real-world positional error, making the loss physically interpretable. 

This is what we refer to as physics-informed machine learning loss, where physical laws and domain knowledge guide the model’s optimization.

### 2.Model Architect

Sequence modeling tasks are traditionally handled by recurrent architectures such as RNNs, LSTMs, or GRUs. However, recent research has shown that convolution-based approaches can outperform these methods in many time-series applications.

While CNNs do not inherently model sequence order, they excel at:
* extracting local spatial/temporal features
* parallel processing
* faster training
* stable gradients
* efficient scaling

For our project, we adopt a convolution-based architecture as the foundation. Specifically, our main model is a Temporal Convolutional Network (TCN).

#### Temporal Convolutional Network (TCN)

A TCN uses 1D dilated convolutions, enabling the model to have a receptive field that expands over time, allowing it to “see” far into the past without requiring recurrence.

**Example (sequence = [a, b, c, d])**:

* Layer 1 (dilation = 1): model sees `[d]`
* Layer 2 (dilation = 2): model sees `[b, d]`
* Layer 3 (dilation = 4): model can see `[a, b, c, d]`

By stacking dilated convolutions, the network learns long-range dependencies while retaining all the benefits of fast convolution operations.
Thus, TCNs combine **memory of the past** with **computational efficiency**, making them an ideal choice for storm trajectory forecasting.

### 3. Model Hyperparameters

Below are the key hyperparameters used in our training configuration:

* **Input dimensions:** 4 (latitude, longitude, distance, bearing)
* **Hidden units:** 1024
* **Number of TCN layers:** 2
* **Learning rate:** 1e-4
* **Epochs:** 80
* **Optimizer:** Adam
* **Early stopping:** patience = 6 (based on overall loss)

### 4. Composite Loss Function

Our main training loss is a weighted combination of multiple components:
* MSE on latitude and longitude
* MSE on distance and bearing (auxiliary features)
* Haversine loss (physics-informed component)

The contribution of each auxiliary loss is controlled by weighting factors:

* λ_aux = 0.5 (for distance and bearing MSE)
* λ_hav = 0.3 (for the Haversine loss)

This design ensures that the model:

* learns to minimize coordinate errors,
* respects physical displacement,
* and does not overfit to any single feature.

By integrating physics-informed loss terms with data-driven learning, the model becomes more stable, consistent, and aligned with real-world storm dynamics.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/train.png" alt="Picture 3" />
  <br/>
  <strong style="font-size: 18px;">Figure 3 : Training Process</strong>
</p>


## Evaluation 

After the training process concludes and the model triggers early stopping, performing an evaluation on the test set is necessary to determine whether the model generalizes well and is ready for real-world deployment. We evaluate the model using Overall Loss, MSE, RMSE, MAPE, and Haversine distance to gain deeper insight into its performance:

* **Total Loss:** 74.3849
* **MSE:** 0.0832
* **RMSE:** 0.2772
* **MAPE:** 0.60%
* **Haversine (km):** 30.75

From these results, we observe that the average positional error is approximately 30 km from the true location. This level of error is acceptable because hurricanes are extremely large systems, often spanning hundreds to thousands of kilometers. The MSE of only 0.08 for latitude and longitude also indicates strong predictive accuracy, suggesting that the model can effectively estimate realistic future storm trajectories with minimal error.

These results further demonstrate that convolutional computations can perform very well on sequence-modeling tasks, and should be considered a strong alternative to traditional sequential architectures such as RNNs, LSTMs, or GRUs—not only for images but also for structured spatiotemporal data.

With the model validated, the next step is to **upload the trained model to an Amazon S3 bucket** and use an AWS **Lambda function** to load and execute it in response to user inference requests.

The following sections will focus on how we design and deploy our online prediction platform, making the hurricane trajectory model accessible for public use.

<p align="center">
  <img src="/images/5-Workshop/5.3-Model Training/loss.png" alt="Picture 4" />
  <br/>
  <strong style="font-size: 18px;">Figure 4 : Evaluation Metrics</strong>
</p>