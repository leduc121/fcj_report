---
title : "Data Preparation"
date: 2025-09-09
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---


## Data Collection and Preparation

Data is a critical component of our project. It not only powers the **machine learning model** but is also directly displayed to end users so they can monitor the **most recent storms** in the West Pacific region. Because of this dual purpose—**model training** and **real-time visualization**—we carefully investigated multiple reliable and authoritative sources before selecting a single dataset that met all of our requirements: **Hurricane Data from NOAA**.

The **National Oceanic and Atmospheric Administration (NOAA)** is a scientific agency under the U.S. Department of Commerce. NOAA provides highly accurate, research-grade environmental data, including global weather observations, satellite imagery, and tropical cyclone records. With decades of investment in advanced technologies such as geostationary satellites, ocean buoys, radar systems, and climate monitoring networks, NOAA is widely regarded as one of the most trustworthy providers of hurricane information in the world.

For this project, we specifically use data from the **International Best Track Archive for Climate Stewardship (IBTrACS)**—a NOAA-led project and the world’s most comprehensive tropical cyclone dataset. IBTrACS consolidates and unifies historical and modern storm-track data from multiple meteorological agencies (e.g., JTWC, JMA, CMA, NHC). By merging these sources into a single consistent format, it improves inter-agency comparability and ensures researchers worldwide have access to the best available storm-track information.

The version of the dataset we use contains **226,153 rows** of hurricane observations. Each row includes a variety of valuable fields such as:

* `sid` – storm ID
* `number` – storm number
* `basin` / `subbasin` – regional classification
* `nature` – storm type (e.g., tropical storm, typhoon)
* `iso_time` – timestamp
* `lat` / `lon` – storm center coordinates
* … and many additional meteorological properties

However, for our machine learning model, we focus only on **four key columns**:
`sid`, `iso_time`, `lat`, and `lon`.
These form the essential time-series trajectory used to predict storm movement.

The dataset spans storms recorded from **1870 up to 2025**, filtered to include only those within the **West Pacific region**—our geographical focus. The raw dataset can be accessed publicly here:
[https://data.humdata.org/dataset/vnm-ibtracs-tropical-storm-tracks#](https://data.humdata.org/dataset/vnm-ibtracs-tropical-storm-tracks#)

### **Cleaning and Physics-Informed Feature Engineering**

One advantage of IBTrACS is that it is already well-maintained and consistent. Only minimal preprocessing is needed, primarily the removal of missing values.

After cleaning, we apply our first step of **physics-informed machine learning**—a technique that injects physical knowledge directly into the data pipeline. From the latitude–longitude coordinates, we compute two additional features using the **Haversine formula**:

1. **Distance** between consecutive storm points
2. **Bearing** (direction of movement)

These features are physically meaningful: they represent real-world movement patterns instead of arbitrary transformations. They enrich the dataset by giving the model more context about the storm’s momentum and direction, thereby improving learning efficiency and prediction accuracy.

<p align="center">
  <img src="/images/5-Workshop/5.2-Data Preparation/dataset.png" alt="Picture 1" />
  <br/>
  <strong style="font-size: 18px;">Figure 1 : Dataset Description</strong>
</p>


## **Data for Display**

The data used for **display** on the platform is different from the data used for **training**, even though both originate from NOAA. The training dataset is static and historical, but the display dataset must always reflect **the current storm conditions**.

To achieve this, we implement a scheduled **AWS Lambda** function that automatically retrieves the latest storm-track updates at the end of each day. This ensures that the platform always presents the most recent and accurate information to users.

The processed display data is stored as a **JSON file** in an Amazon S3 bucket. When a user accesses the website:

1. The frontend sends a request to **API Gateway**
2. API Gateway triggers the appropriate **Lambda function**
3. The Lambda function fetches the JSON from S3
4. The resulting data is returned to the user for visualization

This pipeline guarantees real-time, serverless, cost-effective data delivery.

The live dataset used for display can be accessed here:
[https://ncics.org/ibtracs/](https://ncics.org/ibtracs/)

<p align="center">
  <img src="/images/5-Workshop/5.2-Data Preparation/web_crawl.png" alt="Picture 2" />
  <br/>
  <strong style="font-size: 18px;">Figure 2 : Web to crawl data</strong>
</p>
