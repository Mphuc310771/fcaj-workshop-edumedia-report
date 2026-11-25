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

### 3.1. Service Selection Rationale:

The architecture strictly adheres to AWS Well-Architected Framework principles, choosing Serverless and Managed Services over traditional provisioning based on clear technical justifications:

| Selected AWS Service | Evaluated Alternative | Cost Optimization Criteria | High Availability (HA) | Serverless / Managed Model |
| :--- | :--- | :--- | :--- | :--- |
| **Amazon S3** *(Standard Storage)* | Amazon EFS / Amazon EBS | **$0.023 / GB / month**. 13x cheaper than EFS ($0.30/GB). Direct **Presigned URLs** offload client upload/download traffic entirely, saving 100% of compute and bandwidth costs on backend APIs. | Data durability of **99.999999999% (11 9s)** alongside **99.99% designed Availability (99.9% SLA commitment)**, automatically replicated across at least 3 Availability Zones (AZs) in the Region. | **Managed Object Storage**, eliminating disk volume provisioning, filesystem sizing, and storage server management. |
| **Amazon CloudFront** *(Web SPA CDN)* | EC2 Reverse Proxy (Nginx) / Public S3 Bucket | Includes **1 TB Data Transfer Out & 10M HTTP/HTTPS requests/month Always Free** for delivering the React Web SPA (`agentsys-web-*`), yielding **$0.00** idle cost. | Global edge points of presence with automatic TLS termination and burst absorption without self-managed proxy fleets. | **Managed Edge CDN**, cleanly separating static Web UI distribution from private media and API endpoints. |
| **AWS Lambda** *(Python 3.12 Runtime)* | Amazon ECS *(Fargate / EC2)* | **$0.00 idle cost (Zero Idle Cost)**. Metered strictly per millisecond and GB-second during actual event execution. 1M free requests/month under AWS Free Tier. ECS Fargate incurs continuous base charges (~$15 - $30/month/task) even with zero traffic. | Default Multi-AZ redundancy across the Region; scales instantaneously from 0 to thousands of concurrent executions without managing Auto Scaling Groups. | **Pure Serverless FaaS**, eliminating OS patching, security kernel upgrades, and container runtime daemon maintenance. |
| **Amazon API Gateway** *(HTTP API)* | Application Load Balancer (ALB) / REST API | Low cost of **$1.00 / million requests** (>70% cheaper than REST API at $3.50/M). Zero idle hourly charges compared to ALB (~$16 - $20/month base LCU cost). | Native Multi-AZ fault tolerance; protected against L3/L4 DDoS attacks via **default AWS Shield Standard** and API Gateway account-level limits paired with application-layer rate limiting. | **Serverless HTTP API**, native CORS handling, direct zero-overhead Lambda proxy routing. API authentication (`verify_api_key` enforcing `X-API-Key` / JWT) and rate limiting (`slowapi` `60 reqs/min/IP`) are enforced at the FastAPI application tier. |
| **Amazon DynamoDB** *(On-Demand Mode)* | Amazon RDS Aurora *(PostgreSQL / MySQL)* | **$0.00 idle cost**. Metered strictly per actual read/write request units (RRU/WRU) plus 25 GB Always Free storage. Aurora Serverless v2 requires a continuous minimum ACU allocation (~$35 - $60/month). | Automatic synchronous data replication across 3 AZs; backed by a 99.99% SLA with single-digit millisecond query latency (<10ms). | **NoSQL Serverless Database**, immune to connection pool exhaustion during Lambda scale bursts; zero schema migration downtime. |
| **Amazon SQS & DLQ** *(Message Queue)* | Amazon Kinesis Data Streams / Apache Kafka (MSK) | **$0.40 / million requests**. 1 million requests free every month (Always Free). In contrast, Kinesis charges per Shard-hour (~$11/month/shard) and MSK requires continuous cluster broker fees (~$100+/month). | Highly resilient distributed message queue across Multi-AZ (`agentsys-ingest-queue` 24-hour retention, **3 DLQs** **14-day** retention), buffering burst loads (`MaximumConcurrency = 5`) without downstream AI throttling. | **Fully Managed Queue**, automatically batch-triggering Lambda Ingest workers and isolating corrupted payloads into Dead-Letter Queues (DLQ). |
| **Amazon Bedrock** *(Nova Pro / Nova Lite & Titan Embeddings V2)* | Self-hosted LLM on Amazon EC2 GPU *(G4dn / G5)* | **$0.06 / million input tokens** (Nova Lite), **$0.02 / million input tokens** (Titan Embeddings V2), plus Cross-Region Inference (`us.amazon.nova-pro-v1:0`) costing mere pennies in testing. Self-hosting open-source LLMs on EC2 GPU (g4dn.xlarge or g5.xlarge) costs **$380 – $750 USD/month** continuously. | Foundation models hosted on AWS high-performance GPU clusters with built-in high availability and automated inference load distribution. | **Serverless GenAI API (FaaS)**, requiring zero management of GPU hardware, CUDA drivers, vLLM containers, or inference scaling clusters. |
| **Amazon Rekognition & Transcribe** | Custom self-trained ML models on EC2 | Amazon Transcribe billed at official rate **$0.024 / minute ($0.0004 / second)**. Rekognition billed at **$0.001 / API call** (`DetectLabels` + `DetectText` = **$0.002 / image**). Eliminates R&D, data labeling, model training, and continuous hosting overhead. | Purpose-built AWS AI services backed by 99.9% SLAs and automatic elastic concurrency. | **Pre-trained Managed AI Services**: Transcribe natively optimized for Vietnamese speech-to-text (*vi-VN*); Rekognition handles object labeling (DetectLabels) and slide text/title extraction (DetectText) out-of-the-box without custom ML training. |


---

## 4. FinOps Cost Modeling & Production Workload Estimates (AWS Region: us-east-1)

Within the FCAJ project framework, operational expenditures are engineered following **FinOps (Cloud Financial Management)** principles based on two realistic deployment scenarios modeled via the **AWS Pricing Calculator**:
* **Official AWS Public Estimate:** [Official EduMedia AI Public Estimate (`us-east-1` N. Virginia)](https://calculator.aws/#/estimate?id=ecbb5a062896472eb768e1aa807cb0a645a65aea)

### 4.1. Scenario 1: Self-hosted Workshop & POC Evaluation (1 Tester)
* **Evaluation Workload:** 50 slide images, 5 audio files (20 total minutes), 100 Bedrock AI queries, 1 GB S3 storage, 2,000 Lambda & DynamoDB invocations.
* **Gross Incurred Cost (Without Free Tier):** **$0.76 – $0.88 USD** (~19,400 – 22,400 VND for completing the entire lab at Transcribe's $0.024/min rate card).
* **Net Invoiced Cost (With Free Tier):** Pay only for Bedrock On-Demand (~**$0.18 USD / ~4,500 VND**), or **$0.00 USD** if using an account eligible for AWS Free Tier Credits.

### 4.2. Scenario 2: Department Class / Study Club Workload (100 – 200 Students / Month)
Monthly workload: 1,500 lecture slides, 250 audio minutes, 2,000 Bedrock queries, 20 GB S3 storage + PUT/GET requests, 15,000 Lambda invocations, 50,000 SQS messages, 15,000 HTTP API Gateway requests, DynamoDB On-Demand (15k W / 60k R), 10 GB CloudFront DTO, 2 custom metrics + 0.5 GB logs + 7 CloudWatch Alarms.

| AWS Service | Production Workload (100 - 200 Students) | Gross Cost (Without Free Tier) | Standard Production Bill (With Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1,500 images × 2 APIs = 3,000 calls | **$3.00 USD** | **$3.00 USD** *(Year 1: $2.00)* |
| **Amazon Transcribe** | 250 lecture minutes vi-VN | **$6.00 USD** *(Rate Card $0.024/min)* | **$6.00 USD** *(Calculator: $1.50)* |
| **Amazon Bedrock** | 2,000 queries Nova Pro (2.0M in + 0.6M out) + Titan Embed V2 (2.0M) | **$3.56 USD** *(Nova Pro $3.52 + Embed $0.04)* | **$3.56 USD** |
| **Amazon S3** | 20 GB Standard + PUT (2,000) / GET (10,000) requests | **$0.47 USD** *(Storage $0.46 + Reqs $0.01)* | **$0.47 USD** *(Year 1: $0.35)* |
| **AWS Lambda & SQS** | 15,000 invocations (512MB, 1.5s) + 50,000 SQS msgs | **$0.21 USD** *(Lambda $0.19 + SQS $0.02)* | **$0.00 USD** *(Always Free: 1M Lambda + 1M SQS)* |
| **Amazon API Gateway** | HTTP API v2 (15,000 requests/mo) | **$0.02 USD** | **$0.02 USD** |
| **Amazon DynamoDB** | On-Demand (15,000 writes + 60,000 reads, 0.02 GB) | **$0.02 USD** | **$0.02 USD** *(< 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out + 15,000 HTTPS reqs | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB / 10M reqs)* |
| **Amazon CloudWatch** | 2 custom metrics ($0.60) + 0.5 GB logs ($0.25) + 7 alarms ($0.70) | **$1.55 USD** | **$0.00 USD** *(Always Free: 10 metrics / 10 alarms / 5 GB)* |
| **TOTAL** | **Full System for 100 - 200 Students** | **$15.68 USD / month** *(Calculator Transcribe: $11.18)* | **$13.07 USD / month** *(Calculator Direct: $10.14; Net: $8.57)* |
| **CURRENCY CONVERSION** | **Exchange Rate: 1 USD = 25,500 VND** | **~400,000 VND / month** *(Calculator Transcribe: ~285,000 VND)* | **~333,000 VND / month** *(Calculator Direct: ~259,000 VND; Net: ~219,000 VND)* |
| **PER-STUDENT COST** | **Scale of 100 – 200 Students** | **1,999 – 3,998 VND / student / month** | **1,666 – 3,333 VND / student / month** |

> **AWS Pricing Calculator vs Rate Card Reconciliation:**
> * On the official [AWS Pricing Calculator (`us-east-1` N. Virginia)](https://calculator.aws/#/estimate?id=ecbb5a062896472eb768e1aa807cb0a645a65aea), all **11 resource entries across 10 AWS services** total **10.14 USD / month** (**121.68 USD / 12 months**): Rekognition **$3.00**, Transcribe **$1.50**, Bedrock Nova Pro **$3.52**, Bedrock Titan Embeddings V2 **$0.04**, S3 **$0.47**, DynamoDB **$0.02**, SQS **$0.02** *(pre-Free Tier)*, API Gateway **$0.02**, Lambda **$0.00**, CloudFront **$0.00**, and CloudWatch **$1.55** *(pre-Free Tier)*.
> * Subtracting the Always Free allowances excluded by CloudWatch & SQS sections in AWS Calculator (**$1.55 + $0.02 = $1.57 USD**) yields **$8.57 USD / month** on Calculator pricing, or **$13.07 USD / month** (**~1,666 – 3,333 VND / student / month**) when conservatively pricing 250 Transcribe minutes at the standard **$0.024/min ($6.00 USD)** rate card instead of Calculator's **$1.50 USD** (`$0.006/min`).
> * Roughly **80% – 96% of operational spend resides in AI Inference**. Excluding AI APIs, the entire Serverless compute, queue, database, CDN, and monitoring tier totals only **$0.51 USD / month** after Always Free allowances, scaling automatically to **$0.00 USD** when idle.
> * **Architecture Diagram vs Cost Model Reconciliation:** The architecture diagram captures the complete end-to-end production topology—spanning Frontend Edge delivery via **Amazon CloudFront CDN** backed by **Amazon S3 Static SPA Hosting** (`agentsys-web-*`, validated at `d27yx88v3wm0yt.cloudfront.net` in section 5.5) alongside the Backend Serverless ingestion and AI processing pipeline in `us-east-1`—aligning 100% with every line item modeled in the AWS Pricing Calculator estimate.

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
