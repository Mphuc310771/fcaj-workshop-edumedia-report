---
title: "Overview"
date: 2026-10-02
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# 5.1. Problem Overview, Serverless Architecture & FinOps Estimation

---

## 1. Problem Statement

Modern learners and researchers manage thousands of unstructured multimodal assets (whiteboard photos, lecture slides, audio recordings, video demos).
* Default filenames (IMG\_091522.jpg, voice\_record\_01.m4a) convey zero semantic context.
* Manually searching for specific moments like *"teacher reviewing matrix exercises"* or *"integration formula on board"* requires tedious manual review.

---

## 2. Workshop Objectives

Build and deploy **EduMedia AI** following an **Event-Driven Serverless** architecture on AWS:
1. Upload files directly to **Amazon S3** (Private Bucket, SSE-S3 encryption).
2. Buffer upload events via **Amazon SQS** with an automated **Dead-Letter Queue (DLQ)**.
3. Orchestrate **AWS Lambda** processing:
   * Images: **Amazon Rekognition** extracts object labels and Vietnamese OCR.
   * Audio/Video: **Amazon Transcribe** generates Vietnamese speech-to-text (*vi-VN*).
4. Generate 256-dimensional semantic embeddings using **Amazon Titan Text Embeddings V2** and persist catalog in **Amazon DynamoDB**.
5. Natural language Vietnamese query routing powered by **Amazon Bedrock (Nova Lite)**.
6. End-to-end monitoring with **Amazon CloudWatch Logs & Alarms**.

---

## 3. System Architecture Diagram

![EduMedia AI AWS Architecture](/images/5-Workshop/architecture.png)


---

## 4. FinOps Cost Modeling & Production Workload Estimates (AWS Region: us-east-1)

Within the FCAJ project framework, operational expenditures are engineered following **FinOps (Cloud Financial Management)** principles based on two realistic deployment scenarios modeled via the **AWS Pricing Calculator**:
* **Official AWS Public Estimate:** [Official EduMedia AI Public Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)

### 4.1. Scenario 1: Self-hosted Workshop & POC Evaluation (1 Tester)
* **Evaluation Workload:** 50 slide images, 5 audio files (20 total minutes), 100 Bedrock AI queries, 1 GB S3 storage, 2,000 Lambda & DynamoDB invocations.
* **Gross Incurred Cost (Without Free Tier):** **$0.48 – $0.84 USD** (~12,000 – 21,000 VND for completing the entire lab).
* **Net Invoiced Cost (With Free Tier):** Pay only for Bedrock On-Demand (~**$0.18 USD / ~4,500 VND**), or **$0.00 USD** if using an account eligible for AWS Free Tier Credits.

### 4.2. Scenario 2: Department Class / Study Club Workload (100 – 200 Students / Month)
Monthly workload: 1,500 lecture slides, 250 audio minutes, 2,000 Bedrock queries, 20 GB S3 storage, 15,000 Lambda invocations, DynamoDB On-Demand, 10 GB CloudFront DTO.

| AWS Service | Production Workload (100 - 200 Students) | Gross Cost (Without Free Tier) | Standard Production Bill (With Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1,500 images × 2 APIs = 3,000 calls | **$3.00 USD** | **$3.00 USD** *(Year 1: $2.00)* |
| **Amazon Transcribe** | 250 lecture minutes vi-VN | **$1.50 USD** *(legacy: $6.00)* | **$1.50 USD** *(legacy: $6.00)* |
| **Amazon Bedrock** | 2,000 queries Nova Pro / Nova Lite (1K in + 300 out) + Embed | **$3.56 USD** | **$3.56 USD** |
| **Amazon S3** | 20 GB Standard storage + requests | **$0.46 USD** *(+ requests: $0.53)* | **$0.46 USD** *(Year 1: $0.35)* |
| **AWS Lambda & SQS** | 15,000 invocations (22,500 GB-s) + SQS | **$0.38 USD** | **$0.00 USD** *(Always Free Tier)* |
| **Amazon DynamoDB** | On-Demand (15,000 writes + 60,000 reads) | **$0.02 USD** | **$0.02 USD** *(< 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB/mo)* |
| **TOTAL** | **Full System for 100 - 200 Students** | **$9.77 – $14.27 USD / month** | **$8.54 – $13.09 USD / month** |
| **CURRENCY CONVERSION** | **Exchange Rate: 1 USD = 25,500 VND** | **~249,000 – 364,000 VND / month** | **~217,000 – 334,000 VND / month** |
| **PER-STUDENT COST** | **Scale of 100 – 200 Students** | **1,246 – 3,639 VND / student / month** | **1,088 – 2,177 VND / student / month** |

> **AWS Pricing Calculator vs Rate Card Reconciliation:**
> * On the official live [AWS Pricing Calculator Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934), base monthly cost totals **$4.96 USD / month** (Rekognition $3.00, Transcribe Batch $1.50 at $0.0001/sec, S3 20GB $0.46, Lambda $0.00 Always Free).
> * Calculated against the **Worst-Case Public Rate Card ($0.024/min for Transcribe)** plus Bedrock Inference (Nova Lite / Nova Pro + Titan Embeddings) and S3 API / CloudWatch Logs, monthly production spend ranges between **$9.77 – $14.27 USD / month** (~1,200 – 2,500 VND / student / month).
> * Over **82% of operational spend resides in AI Inference**. The entire Serverless compute/storage tier (Lambda, SQS, S3, DynamoDB, CloudFront) costs under **$1.50 USD / month**, automatically scaling to $0.00 USD when idle.

---

## 5. Technical Acceptance & Verification Criteria

Before proceeding to evaluation or resource cleanup, the workshop implementation is verified against the following concrete engineering deliverables:

| No | Test Deliverable | Verification Method | Expected Engineering Outcome | AWS Services |
| :---: | :--- | :--- | :--- | :--- |
| **1** | Infrastructure as Code | `sam deploy` or CloudFormation Console | Stack `edumedia-ai` (or `agentsys`) reaches `CREATE_COMPLETE` or `UPDATE_COMPLETE`, exporting all Outputs (ApiEndpoint, S3 Bucket, TableName). | AWS SAM, CloudFormation |
| **2** | Health Check API | `GET /health` | Returns HTTP 200 with discrete health diagnostics for S3, DynamoDB, Rekognition, and Bedrock. | API Gateway, Lambda |
| **3** | Visual Document Processing | Upload study slide via Web / API | SQS buffers event, triggers Lambda Ingest to call Rekognition Labels and OCR text extraction; classifies subject and category. | S3, SQS, Rekognition, DynamoDB |
| **4** | Audio Lecture Ingestion | Upload lecture audio (`.m4a`, `.mp3`) | Triggers Speech Processing, tracks job state through EventBridge, and writes transcript to S3 / DynamoDB. | S3, SQS, Transcribe, EventBridge |
| **5** | Intelligent Query Routing | `POST /api/v1/media/agent/query` | Generates a structured Search Plan (keywords, category, subject) and returns matching records with evidence timestamps. | Amazon Bedrock, DynamoDB GSI |
| **6** | Observability & Alarms | Synthetic traffic or intentional error injection | Structured JSON logs emitted (`MediaIngested`, `MediaIngestFailed`), 7 CloudWatch Alarms active in OK/ALARM states. | CloudWatch Logs, Alarms, SNS |
