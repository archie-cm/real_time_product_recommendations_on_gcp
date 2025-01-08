# Real-Time Product Recommendations with Pub/Sub and Dataproc

This project demonstrates how to build a real-time product recommendation system using Pub/Sub Lite and Apache Spark with Dataproc. The system leverages the Alternating Least Squares (ALS) machine learning algorithm for collaborative filtering to recommend products in real time.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Pipeline Process](#pipeline-process)
  - [1. Upload Source Files](#1-upload-source-files)
  - [2. Set Up Pub/Sub Lite](#2-set-up-pubsublite)
  - [3. Train Recommendation Model](#3-train-recommendation-model)
  - [4. Publish Events to Pub/Sub Lite](#4-publish-events-to-pubsublite)
  - [5. Set Up Dataproc](#5-set-up-dataproc)
  - [6. Model Inference](#6-model-inference)
- [License](#license)

---

## Overview
This project uses:
1. Google Pub/Sub Lite for real-time data ingestion.
2. Dataproc with Spark for training and deploying a recommendation model.
3. ALS (Alternating Least Squares) from Apache Spark MLlib for collaborative filtering.

## Prerequisites
- Google Cloud account.
- GCP SDK installed.
- `gcloud` CLI configured.
- Python 3.x installed locally.
- Google Cloud Storage bucket for uploading files.

## Pipeline Process

### 1. Upload Source Files
Upload the source files (`users.csv`, `products.csv`, `order_items.csv`) to a Google Cloud Storage bucket:
```bash
gsutil cp users.csv gs://<your-bucket-name>/
gsutil cp products.csv gs://<your-bucket-name>/
gsutil cp order_items.csv gs://<your-bucket-name>/
```

### 2. Set Up Pub/Sub Lite
1. Enable Pub/Sub Lite API:
   ```bash
   gcloud services enable pubsublite.googleapis.com
   ```
2. Create a topic:
   ```bash
   gcloud pubsublite topics create recommendations-topic \
       --location=us-central1-a \
       --partitions=1 \
       --per-partition-bytes=30GiB \
       --retention-period=24h
   ```
3. Create a subscription:
   ```bash
   gcloud pubsublite subscriptions create recommendations-subscription \
       --location=us-central1-a \
       --topic=recommendations-topic \
       --delivery-requirement=deliver-after-stored
   ```

### 3. Train Recommendation Model
Train the ALS recommendation model using Apache Spark and save the model to Google Cloud Storage:
```bash
spark-submit model-training.py
```
Make sure the script reads data from the uploaded files in GCS and saves the trained model back to GCS.

### 4. Publish Events to Pub/Sub Lite
Run the `publish-pubsublite-events.py` script to publish events to Pub/Sub Lite:
```bash
python publish-pubsublite-events.py
```
This script reads data from your source files and simulates real-time events by publishing them to the Pub/Sub Lite topic.

### 5. Set Up Dataproc
Submit a PySpark job to Dataproc to stream recommendations in real time:
```bash
gcloud dataproc jobs submit pyspark streaming-recommendations.py \
    --cluster=spark-streaming \
    --region=us-central1 \
    --jars=gs://spark-lib/pubsublite/pubsublite-spark-sql-streaming-LATEST-with-dependencies.jar
```
This job processes incoming events from Pub/Sub Lite and provides real-time recommendations.

### 6. Model Inference
Run the `model-inference.py` script to use the trained model for inference:
```bash
python model-inference.py
```
This script performs batch or real-time inference using the trained ALS model to recommend products for users based on their activity.

## License
This project is licensed under the MIT License. See the LICENSE file for details.

