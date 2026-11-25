---
title: "Proposal"
date: 2026-08-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# EduMedia AI — Smart Lecture Storage & Search Engine on AWS

**Student:** Do Tran Minh Phuc · VNU-HCM University of Science (HCMUS)  
**Internship Period:** 17/08/2026 – 01/12/2026 · Amazon Web Services Vietnam LLC (FCAJ Bootcamp)

---

## 1. Executive Summary

**EduMedia AI** is an intelligent, multi-modal lecture cataloging and retrieval platform engineered entirely on a **Serverless** and **Generative AI** architecture using **Amazon Web Services (AWS)**.

The system addresses the fundamental limitation of traditional camera-roll search by automatically extracting multi-sensory semantics from images, audio, and video files:
* **Amazon CloudFront & Amazon S3 (Web Hosting)**: Serves the React Static SPA globally over HTTPS edge delivery backed by `agentsys-web-*`.
* **Amazon S3 (Private Media Storage)**: Secure object storage protected by *Block All Public Access*, SSE-S3 encryption, and short-lived Presigned URLs.
* **Amazon SQS & Dead-Letter Queues (DLQs)**: Decouples ingestion traffic via `agentsys-ingest-queue` and three isolated DLQs ensuring resilient zero-data-loss processing.
* **AWS Lambda, Amazon Rekognition & Amazon Transcribe**: Orchestrates workers (`agentsys-ingest`, `agentsys-aggregator`) invoking **Amazon Rekognition** (labels, scenes, Vietnamese OCR) and **Amazon Transcribe** (*vi-VN* speech-to-text via EventBridge).
* **Amazon Bedrock (Nova Pro / Nova Lite & Titan Embeddings V2)**: Acts as an AI Query Planner, chunk summarization engine, and 256-d semantic vector engine for hybrid ranking.
* **Amazon DynamoDB**: Stores media catalog metadata (`AgentSysMediaMetadata`) with **GSI `category-index`** and DynamoDB Streams for hybrid retrieval and Map-Reduce aggregation.
* **Amazon CloudWatch, Amazon SNS & AWS X-Ray**: End-to-end observability with structured JSON logs, distributed X-Ray tracing, and **7 CloudWatch Alarms** routing notifications through SNS (`agentsys-alerts-topic`).

---

## 2. Problem Statement

In academic and professional environments, students and researchers accumulate thousands of multimodal files: whiteboard notes, lecture slide snapshots, multi-hour voice recordings, and video demos.

### Major Inefficiencies in Existing Solutions:
1. **Filename-based Search Failure**: Mobile uploads default to non-descriptive names (e.g., IMG_20260826_104212.jpg or Voice_001.m4a), providing zero semantic metadata.
2. **Costly Commercial DAM Solutions**: Enterprise Digital Asset Management (DAM) platforms require 24/7 dedicated servers starting at $50–$200/month, making them impractical for students or small research teams.
3. **Weak Vietnamese Natural Language Support**: Open-source toolkits struggle with diacritics and domain-specific terminology across Mathematics, Economics, and Philosophy.

---

## 3. Goals, Success Criteria & Project Scope

| ID | Objective | Pass Condition | Responsible AWS Service |
| :--- | :--- | :--- | :--- |
| **G1** | **Ingest API** | Presigned S3 PUT (`TTL 900s`); images transition to `READY` in `< 5 seconds` (Rekognition), while audio/video initiates asynchronous `INDEXING` completed via Transcribe + EventBridge. | S3, SQS, Lambda, DynamoDB, EventBridge |
| **G2** | **Computer Vision** | Accurately extracts object labels + Vietnamese OCR text from classroom board snapshots. | Amazon Rekognition |
| **G3** | **Speech-to-Text** | Transcribes Vietnamese speech (*vi-VN*) from audio/video and synthesizes structured summaries across 5–10 minute segments. | Amazon Transcribe, Lambda Aggregator |
| **G4** | **Hybrid Search** | Natural language queries like *"find teacher review for final exam"* accurately return relevant media items. | Amazon Bedrock, DynamoDB GSI |
| **G5** | **Well-Architected & FinOps** | 100% Serverless (Scale-to-Zero), $0 idle cost, **7 CloudWatch Alarms** + SNS, AWS X-Ray active tracing, and verified Cleanup runbook. | CloudWatch, SNS, X-Ray, IAM, SAM CLI |

### Project Scope:
* **In-Scope (PoC Phase):** Full asynchronous multimodal ingestion pipeline (images, audio, video), automated AI enrichment (Rekognition, Transcribe vi-VN, Titan Text Embeddings V2 256-d, Nova Pro / Nova Lite), hybrid search over DynamoDB GSI (`category-index`, capped Candidate Pool of 100 items), CloudFront + S3 React SPA, 143 automated Pytest unit tests, 7 CloudWatch Alarms + X-Ray tracing, and reproducible AWS SAM deployment/teardown.
* **Out-of-Scope (Enterprise Roadmap in Section 5.7):** Real-time WebSocket live streaming ASR, multi-tenant Amazon Cognito federation, and dedicated Amazon OpenSearch Serverless / Aurora pgvector clusters for multi-million-record catalogs.

---

## 4. Proposed System Architecture

![EduMedia AI AWS Architecture](/images/2-Proposal/architecture.png)

### End-to-End Data Pipeline Workflow:
1. **Frontend Edge & Presigned Upload Flow:** Users access the React Web UI SPA distributed globally via **Amazon CloudFront CDN** backed by **Amazon S3 (`agentsys-web-*`)**. The client requests a scoped S3 Presigned PUT URL (`TTL 900s`) from `agentsys-api` (Lambda) through **Amazon API Gateway HTTP API** (primary route `POST /api/v1/media/upload-url`, legacy alias `/api/upload-url`), then uploads directly to private `s3://MediaBucket/uploads/`.
2. **Asynchronous Ingestion Buffer & 3 DLQs:** S3 emits `s3:ObjectCreated:*` notifications (`uploads/` prefix only) into **Amazon SQS (`agentsys-ingest-queue`)** configured with `ReceiveMessageWaitTimeSeconds = 10` and `maxReceiveCount = 3` routing failed records to **`agentsys-ingest-dlq`** (14-day retention). Two dedicated DLQs (`agentsys-transcribe-dlq` and `agentsys-aggregator-stream-dlq`) safeguard EventBridge and DynamoDB Stream deliveries.
3. **Multimodal AI Extraction & Stream Aggregation:** `agentsys-ingest` (Lambda, `ScalingConfig: MaximumConcurrency = 5`, `BatchSize = 5`) routes by media type:
   - Images: Calls `DetectLabels` and `DetectText` (Vietnamese OCR) concurrently via **Amazon Rekognition** and generates thumbnails (`thumbs/`).
   - Audio/Video: Initiates asynchronous **Amazon Transcribe (`vi-VN`)** jobs, then processes completion/failure events via **Amazon EventBridge (`Transcribe Job State Change`)**. Transcripts exceeding **250 KB** automatically offload to `s3://MediaBucket/transcripts/raw/{file_id}.json` (retaining a 15,000-character preview in DynamoDB), while longer lectures are segmented into **5–10 minute silence-aware chunks** aggregated asynchronously via **DynamoDB Streams** and **`agentsys-aggregator`** Lambda.
4. **Metadata & 256-d Vector Storage:** Extracted content is embedded into **256-dimensional L2-normalized vectors** via **Amazon Titan Text Embeddings V2 (`amazon.titan-embed-text-v2:0`)** on **Amazon Bedrock** and persisted in **Amazon DynamoDB (`AgentSysMediaMetadata`)** under On-Demand billing (`PAY_PER_REQUEST`).
5. **Intelligent Hybrid Search & Full-Stack Observability:** Upon receiving a query (`POST /api/v1/media/agent/query`), **Amazon Bedrock (`us.amazon.nova-pro-v1:0` default or `amazon.nova-lite-v1:0`)** parses intent (reusing cached plans via `_PLAN_CACHE` LRU `256` entries, TTL `3600s`), queries **GSI `category-index`** with a capped Candidate Pool of **100 items**, and ranks results via Cosine Vector Similarity (**0.6**) + Normalized Keyword Overlap (**0.4**). All 3 Lambdas emit **AWS X-Ray (`Tracing: Active`)** traces, structured CloudWatch Logs (14-day retention), and feed **7 CloudWatch Alarms** linked to **Amazon SNS (`agentsys-alerts-topic`)**.

### Service Selection Rationale:

To achieve optimal performance, high availability, and **Zero Idle Cost**, AWS services were chosen after rigorous evaluation against architectural alternatives:

| Selected AWS Service | Evaluated Alternative | Cost Optimization Criteria | High Availability (HA) | Serverless / Managed Model |
| :--- | :--- | :--- | :--- | :--- |
| **Amazon S3** *(Standard Storage)* | Amazon EFS / Amazon EBS | **$0.023 / GB / month**. 13x cheaper than EFS ($0.30/GB). Direct **Presigned URLs** offload client upload/download traffic entirely, saving 100% of compute and bandwidth costs on backend APIs. | Data durability of **99.999999999% (11 9s)** alongside **99.99% designed Availability (99.9% SLA commitment)**, automatically replicated across at least 3 Availability Zones (AZs) in the Region. | **Managed Object Storage**, eliminating disk volume provisioning, filesystem sizing, and storage server management. |
| **Amazon CloudFront** *(Web SPA CDN)* | EC2 Reverse Proxy (Nginx) / Public S3 Bucket | Includes **1 TB Data Transfer Out & 10M HTTP/HTTPS requests/month Always Free** for delivering the React Web SPA (`agentsys-web-*`), yielding **$0.00** idle cost. | Global edge points of presence with automatic TLS termination and burst absorption without self-managed proxy fleets. | **Managed Edge CDN**, cleanly separating static Web UI distribution from private media and API endpoints. |
| **AWS Lambda** *(Python 3.12 Runtime)* | Amazon ECS *(Fargate / EC2)* | **$0.00 idle cost (Zero Idle Cost)**. Metered strictly per millisecond and GB-second during actual event execution. 1M free requests/month under AWS Free Tier. ECS Fargate incurs continuous base charges (~$15 - $30/month/task) even with zero traffic. | Default Multi-AZ redundancy across the Region; scales instantaneously from 0 to thousands of concurrent executions without managing Auto Scaling Groups. | **Pure Serverless FaaS**, eliminating OS patching, security kernel upgrades, and container runtime daemon maintenance. |
| **Amazon API Gateway** *(HTTP API)* | Application Load Balancer (ALB) / REST API | Low cost of **$1.00 / million requests** (>70% cheaper than REST API at $3.50/M). Zero idle hourly charges compared to ALB (~$16 - $20/month base LCU cost). | Native Multi-AZ fault tolerance; protected against L3/L4 DDoS attacks via **default AWS Shield Standard** combined with **Route Throttling** (burst and steady-state rate limits) to safeguard backend Lambdas. | **Serverless HTTP API**, native CORS handling, direct zero-overhead Lambda proxy routing. API security validation is enforced at the application tier (FastAPI `verify_api_key` dependency checking `X-API-Key` header) rather than relying on native API Gateway usage plans. |
| **Amazon DynamoDB** *(On-Demand Mode)* | Amazon RDS Aurora *(PostgreSQL / MySQL)* | **$0.00 idle cost**. Metered strictly per actual read/write request units (RRU/WRU) plus 25 GB Always Free storage. Aurora Serverless v2 requires a continuous minimum ACU allocation (~$35 - $60/month). | Automatic synchronous data replication across 3 AZs; backed by a 99.99% SLA with single-digit millisecond query latency (<10ms). | **NoSQL Serverless Database**, immune to connection pool exhaustion during Lambda scale bursts; zero schema migration downtime. |
| **Amazon SQS & DLQ** *(Message Queue)* | Amazon Kinesis Data Streams / Apache Kafka (MSK) | **$0.40 / million requests**. 1 million requests free every month (Always Free). In contrast, Kinesis charges per Shard-hour (~$11/month/shard) and MSK requires continuous cluster broker fees (~$100+/month). | Highly resilient distributed message queue across Multi-AZ with up to 14 days message retention, absorbing traffic bursts without downstream AI throttling. | **Fully Managed Queue**, automatically batch-triggering Lambda Ingest workers and isolating corrupted payloads into Dead-Letter Queues (DLQ). |
| **Amazon Bedrock** *(Nova Pro / Nova Lite & Titan Embeddings V2)* | Self-hosted LLM on Amazon EC2 GPU *(G4dn / G5)* | **$0.06 / million input tokens** (Nova Lite), **$0.02 / million input tokens** (Titan Embeddings V2), plus Cross-Region Inference (`us.amazon.nova-pro-v1:0`) costing mere pennies in testing. Self-hosting LLMs on EC2 GPU (g4dn.xlarge or g5.xlarge) costs **$380 – $750 USD/month** continuously. | Foundation models hosted on AWS high-performance GPU clusters with built-in high availability and automated inference load distribution. | **Serverless GenAI API (FaaS)**, requiring zero management of GPU hardware, CUDA drivers, vLLM containers, or inference scaling clusters. |
| **Amazon Rekognition & Transcribe** | Custom self-trained ML models on EC2 | Amazon Transcribe billed at official rate **$0.024 / minute ($0.0004 / second)**. Rekognition billed at $0.001/image. Eliminates R&D, data labeling, model training, and continuous hosting overhead. | Purpose-built AWS AI services backed by 99.9% SLAs and automatic elastic concurrency. | **Pre-trained Managed AI Services**: Transcribe natively optimized for Vietnamese speech-to-text (*vi-VN*); Rekognition handles object labeling (DetectLabels) and slide text/title extraction (DetectText) out-of-the-box without custom ML training. |

---

## 5. AWS Well-Architected Framework Evaluation (6 Pillars)

The architecture is comprehensively evaluated against the 6 pillars of the **AWS Well-Architected Framework** and the **Serverless Lens**:

| Audit Parameter | Verification Details on AWS |
| :--- | :--- |
| **Assessment Tool** | AWS Well-Architected Tool |
| **Workload Name** | `EduMedia AI` |
| **Workload ID** | `13f51f4fbcb8f084f550fe07020d295e` (Region: `us-east-1`) |
| **Review Owner** | Do Tran Minh Phuc (Student ID: `23120156` · Account: `181040156515`) |
| **Applied Lenses** | AWS Well-Architected Framework & Serverless Lens |
| **High-Risk Issues (HRIs)** | **0 High Risks** (100% Best Practice Compliance) |


### 1. Security
* **IAM Least-Privilege**: Execution roles are strictly decoupled across all 3 Lambda functions (`agentsys-ingest` scoped to S3 `uploads/` & Rekognition/Transcribe; `agentsys-aggregator` scoped to DynamoDB Streams & Bedrock refinement; `agentsys-api` scoped to DynamoDB GSI & Bedrock query planning).
* **Storage Protection**: S3 enforces *Block All Public Access*, *SSE-S3 (AES-256)* server-side encryption, and scoped short-lived Presigned URLs (`PUT: 900s / 15 min`, `GET: 3600s / 60 min`).
* **Secret Management**: Zero hardcoded credentials; API authentication uses `X-API-Key` secured via CloudFormation `NoEcho` alongside JWT Bearer token validation.

### 2. Reliability
* **Traffic Decoupling**: SQS (`agentsys-ingest-queue`) buffers burst ingestion traffic (`MaximumConcurrency = 5`), preventing Lambda concurrency exhaustion.
* **Failure Isolation**: Failed payloads are routed after 3 retries into three dedicated DLQs (`agentsys-ingest-dlq`, `agentsys-transcribe-dlq`, `agentsys-aggregator-stream-dlq`), triggering immediate CloudWatch Alarms via SNS.
* **DynamoDB 400KB Item Guard (S3-Offloading)**: Transcripts exceeding 250KB are automatically offloaded to Amazon S3 (`transcripts/raw/{file_id}.json`), storing a 15,000-character preview and S3 URI in DynamoDB to safely maintain items well below the 400KB limit while preserving instant UI access.

### 3. Performance Efficiency
* **Vector Subsampling (Titan V2 Native 256-d)**: Configures `dimensions = 256` with L2 normalization natively on Amazon Titan Text Embeddings V2—achieving a **75% storage reduction** compared to Titan V2's default 1024 dimensions (and **83% reduction** vs Titan V1's 1536 dimensions) while preserving semantic recall.
* **DynamoDB GSI Partitioning (`category-index`)**: GSI partition routing paired with a bounded 100-item Candidate Pool eliminates full-table scans, reducing RCU cost and latency by over 90% (<40ms).
* **Lambda Warm-Container Reuse**: Boto3 client connections and TLS handshakes persist across warm Lambda invocations, avoiding 50–120ms reconnection overhead on every SQS message or API request.
* **In-Memory Query Plan Cache (`_PLAN_CACHE`)**: LRU caching (max 256 entries, 1h TTL) of parsed search plans speeds up repeated query responses by **3.7x** and saves 100% of LLM token costs.

### 4. Cost Optimization (FinOps)
* **Scale-to-Zero Model**: Idle infrastructure costs exactly **$0.00 USD**.
* **Bedrock LRU Caching**: In-memory query caching eliminates token costs for recurring search queries.
* **S3 Lifecycle Rules**: Automatic expiration for temporary files in `transcripts/` after **7 days** (`ExpirationInDays: 7`).

### 5. Operational Excellence
* **Infrastructure as Code (IaC)**: The entire stack is provisioned via **AWS SAM (`agent_sys/infra/template.yaml`)**, enabling single-command teardown via `scripts/cleanup.sh` and `sam delete`.
* **Observability & Tracing**: All 3 Lambdas enable **AWS X-Ray (`Tracing: Active`)**, emit structured JSON CloudWatch Logs (**14-day retention**), publish custom metrics (`MediaIngested`, `MediaIngestFailed`), and enforce **7 CloudWatch Alarms** (`agentsys-api-errors`, `agentsys-ingest-errors`, `agentsys-aggregator-errors`, `agentsys-ingest-failed`, `agentsys-ingest-dlq`, `agentsys-aggregator-stream-dlq`, `agentsys-transcribe-dlq`) linked to SNS Topic `agentsys-alerts-topic`.

### 6. Sustainability
* Fully leverages shared multi-tenant AWS Managed Services, eliminating energy waste from idle GPU compute.

---

## 6. AWS FinOps Cost Modeling & Production Workload Estimates (Region: us-east-1)

The system adheres to a strict **Scale-to-Zero** FinOps model. All infrastructure resources (S3, DynamoDB, Lambda, SQS, API Gateway, CloudWatch) are centrally provisioned in **AWS Region: us-east-1 (N. Virginia)**. For the Generative AI layer (Amazon Bedrock), the system leverages Cross-Region Inference Profiles with prefix `us.` (`arn:aws:bedrock:*:${AWS::AccountId}:inference-profile/us.*`) for dynamic inference routing across US data centers while all orchestration assets remain strictly within `us-east-1`. All operational costs have been modeled and calculated using the official **AWS Pricing Calculator**:
* **Official AWS Public Estimate:** [EduMedia AI Workload Estimate (AWS Pricing Calculator)](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)

### 6.1. Unit Economics (AWS Region: us-east-1)
* **1 Lecture Slide Image Processing:** Amazon Rekognition DetectLabels ($0.0010) + DetectText OCR ($0.0010) = **$0.0020 USD / image** (~51 VND).
* **1 Audio Minute Transcription:** Amazon Transcribe Standard Batch vi-VN = **$0.0240 USD / minute** ($0.0004 USD / second official AWS rate card, equivalent to ~612 VND/min).
* **1 Intelligent AI Query:** Amazon Titan Text Embeddings v2 ($0.00002/1k tokens) + Amazon Bedrock Nova Pro (1,000 in + 300 out tokens) = **~$0.00178 USD / query** (~45.4 VND).
* **AI Agent Reasoning & Intent Parsing:** Amazon Nova Lite ($0.06/1M in + $0.24/1M out tokens) acts as the autonomous Agent orchestrator parsing queries, translating Vietnamese intent, and summarizing chunks = **~$0.08 – $0.15 USD / month** for 2,000 queries.
* **Amazon S3 Standard Storage:** **$0.0230 USD / GB / month** (~586.5 VND/GB). (API Requests: PUT $0.005 / 1k reqs, GET $0.0004 / 1k reqs).
* **Amazon CloudFront Distribution:** **1 TB (1,000 GB) Data Transfer Out** and **10,000,000 requests/month** free forever (Always Free Tier).
* **AWS Lambda (conservative 1024MB, 1.5s estimate profile; deployed SAM default is 512MB):** $0.20/1M requests + $0.0000166667 / GB-s (x86) or $0.0000133334 / GB-s (ARM Graviton2) = **$0.000020 / invocation**. (1M requests + 400,000 GB-s/month free forever).

---

### 6.2. Scenario 1: Self-hosted Workshop & POC Evaluation (1 Tester)
Workload: 50 slide images, 5–10 short demo video clips (15–20 total audio minutes), 100 Bedrock AI queries, 1 GB S3 Standard, ~2,000 Lambda & DynamoDB invocations.

| AWS Service | Evaluation Workload | Gross Cost (Without Free Tier) | Net Invoiced Cost (With Free Tier) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 50 images × 2 APIs = 100 calls | $0.1000 USD | **$0.0000 USD** (1,000 calls/mo Free Tier) |
| **Amazon Transcribe** | 15–20 minutes vi-VN audio (Batch × $0.024/min) | $0.3600 – $0.4800 USD | **$0.0000 USD** (60 minutes/mo Free Tier) |
| **Amazon Bedrock** | 100 queries Nova Pro / Nova Lite + Titan Embed | $0.1780 USD | **$0.1780 USD** *(No quantity Free Tier)* |
| **Amazon S3** | 1 GB Standard + PUT/GET requests | $0.0240 USD | **$0.0000 USD** (5 GB Free Tier allowance) |
| **AWS Lambda & SQS** | 2,000 invocations (1024MB estimate, 1.5s = 3,000 GB-s) | $0.0528 USD | **$0.0000 USD** (Always Free 1M req, 400k GB-s) |
| **Amazon DynamoDB** | On-Demand (2,000 writes + 2,000 reads) | $0.0015 USD | **$0.0015 USD** (25 GB Storage Always Free) |
| **Amazon CloudFront** | Static Web UI distribution (< 1 GB) | $0.0425 USD | **$0.0000 USD** (Always Free 1 TB/mo) |
| **TOTAL (POC)** | **Exchange Rate: 1 USD = 25,500 VND** | **$0.76 – $0.88 USD** *(~19,400 – 22,400 VND)* | **$0.18 USD** *(~4,500 VND - Or $0.00 with Credits)* |

---

### 6.3. Scenario 2: Department Class / Study Club Workload (100 – 200 Students / Month)
Monthly workload: 1,500 lecture slides, 250 audio minutes, 2,000 Bedrock queries, 20 GB S3 storage, 15,000 Lambda & SQS invocations, 15,000 writes + 60,000 reads DynamoDB, 10 GB CloudFront DTO.

| AWS Service | Production Workload (100 - 200 Students) | Gross Cost (Without Free Tier) | Standard Production Bill (With Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1,500 images × 2 APIs = 3,000 calls | **$3.00 USD** | **$3.00 USD** *(Year 1: $2.00)* |
| **Amazon Transcribe** | 250 lecture minutes vi-VN | **$6.00 USD** *(Rate Card $0.024/min)* | **$6.00 USD** *(Baseline Calculator: $1.50)* |
| **Amazon Bedrock** | 2,000 queries Nova Pro / Nova Lite (1K in + 300 out) + Embed | **$3.56 USD** | **$3.56 USD** |
| **Amazon S3** | 20 GB Standard storage + requests | **$0.46 USD** *(+ requests: $0.53)* | **$0.46 USD** *(Year 1: $0.35)* |
| **AWS Lambda & SQS** | 15,000 invocations (22,500 GB-s) + SQS | **$0.38 USD** | **$0.00 USD** *(Always Free Tier)* |
| **Amazon DynamoDB** | On-Demand (15,000 writes + 60,000 reads) | **$0.02 USD** | **$0.02 USD** *(< 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB/mo)* |
| **TOTAL** | **Full System for 100 - 200 Students** | **$14.27 USD / month** *(Baseline Calculator: $9.77)* | **$13.04 – $13.09 USD / month** *(Baseline: $8.54)* |
| **CURRENCY CONVERSION** | **Exchange Rate: 1 USD = 25,500 VND** | **~364,000 VND / month** *(Baseline: ~249,000 VND)* | **~333,000 – 334,000 VND / month** *(Baseline: ~217,000 VND)* |
| **PER-STUDENT COST** | **Scale of 100 – 200 Students** | **1,819 – 3,639 VND / student / month** | **1,663 – 3,338 VND / student / month** |

> **AWS Pricing Calculator vs Rate Card Reconciliation:**
> * On the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934), base monthly cost displays **4.96 USD / month** (Rekognition 3.00 USD for 3,000 images, Transcribe 1.50 USD baseline trial workload, S3 20GB 0.46 USD, Lambda 0.00 USD Always Free).
> * Scaling up to the full production workload of **250 audio minutes/month for 100–200 students (Amazon Transcribe Standard totaling 6.00 USD)** plus Bedrock Inference (Nova Lite / Nova Pro + Titan Embeddings) and S3 API / CloudWatch Logs, monthly production spend ranges between **13.04 – 14.27 USD / month** (~**1,663 – 3,639 VND / student / month**).
> * Over **82% of operational spend resides in AI Inference**. The entire Serverless compute/storage tier (Lambda, SQS, S3, DynamoDB, CloudFront) costs under **1.50 USD / month**, automatically scaling to 0.00 USD when idle.
> * **Architecture Diagram vs Cost Model Reconciliation:** The architecture diagram captures the complete end-to-end production topology—spanning Frontend Edge delivery via **Amazon CloudFront CDN** backed by **Amazon S3 Static SPA Hosting** (`agentsys-web-*`, validated at `d27yx88v3wm0yt.cloudfront.net` in section 5.5) alongside the Backend Serverless ingestion and AI processing pipeline in `us-east-1`—aligning 100% with every line item modeled in the FinOps cost breakdown.

---

## 7. Project Timeline

* **Phase 1 (Month 1 — Weeks 1–4):** Requirements analysis, AWS AI evaluation, AWS SAM skeleton, S3/SQS setup, and automated Pytest unit testing suite.
* **Phase 2 (Month 2 — Weeks 5–8):** Ingestion pipeline development (Rekognition Labels/OCR + Transcribe vi-VN + EventBridge + Lambda Aggregator), DynamoDB GSI, and 7 CloudWatch Alarms.
* **Phase 3 (Month 3 — Weeks 9–12):** Amazon Bedrock hybrid search, React Web UI on CloudFront + S3, FinOps optimization, and final workshop documentation.

---

## 8. Risks & Mitigations

| No | Identified Risk | Impact | Mitigation Strategy |
| :---: | :--- | :---: | :--- |
| **1** | **DynamoDB Vector Scan Amplification & Repeated LLM Query Costs:** DynamoDB lacks native ANN/HNSW indices; unbounded Full Table Scans push RRUs and Lambda execution times linearly `O(N)` per query. | High | **Two-Stage Hybrid Search Architecture:** Route queries through **DynamoDB GSI (`category-index`)** into a **capped Candidate Pool (`CANDIDATE_POOL_LIMIT = 100` newest items)** inside Lambda RAM for hybrid Cosine (`0.6`) + Keyword (`0.4`) scoring, paired with compact **256-d Titan V2** vectors and an in-memory **LRU `_PLAN_CACHE` (256 entries, 1h TTL)** saving 100% of repeat query tokens. For catalog scales beyond tens of thousands of items, Section 5.7 details migration to OpenSearch Serverless / pgvector. |
| **2** | **VPC NAT Gateway Cost Trap:** Placing Lambda inside a VPC private subnet without private RDS incurs a fixed NAT Gateway charge of $32.85 USD/month even when idle. | High | Zero-VPC design: keep Lambda outside VPC as all interactions communicate with AWS managed public APIs (S3, Bedrock, Rekognition, DynamoDB) strictly secured via IAM Least-Privilege roles and HTTPS TLS 1.2+ encryption. |
| **3** | **Infinite S3-to-Lambda Triggers & SQS Retry Storms:** Recursive event loops on file generation exhausting AI API quotas. | High | Strict S3 prefix segregation (`uploads/` trigger only, writing artifacts to `thumbs/` and `transcripts/`); configure three Dead-Letter Queues (`agentsys-ingest-dlq`, `agentsys-transcribe-dlq`, `agentsys-aggregator-stream-dlq`) with `maxReceiveCount = 3` to isolate failing messages. |
| **4** | **Rekognition Video Cost Trap ($0.10/min) & Long Lecture Processing:** Rekognition Video is extremely costly ($0.10/min/feature = $50 for 250 mins), while long voice recordings risk lengthy synchronous processing. | High | **Intelligent Multi-Modal Tiering:** Route video/audio files asynchronously to **Amazon Transcribe (`vi-VN`, $0.024/min)**; split transcripts via silence-aware `pipeline/chunking.py` into **5–10 minute logical segments** refined by **`agentsys-aggregator`** over **DynamoDB Streams** and embedded with **Titan Text Embeddings V2** (supporting up to 8,192 tokens, emitting compact 256-d vectors); reserve **Rekognition Image ($0.001/image)** for static slides/boards (>95% cost savings). |
| **5** | **Lambda Synchronous Wait on Batch Transcription:** Batch transcription takes 1–3 minutes. Polling synchronously with `sleep()` inside Lambda inflates GB-sec runtime by 100x. | High | 100% Asynchronous Event-Driven Design: `agentsys-ingest` invokes `StartTranscriptionJob()` and exits immediately. **Amazon EventBridge** captures `Transcribe Job State Change (COMPLETED | FAILED)` events to trigger downstream aggregation and indexing. Polling wait cost = $0.00 USD. |
| **6** | **S3 Egress Costs & Private Media Access Control:** Directly hosting Web SPA bundles from uncached buckets or exposing raw media risks S3 egress fees ($0.09/GB past 100GB) and security leaks. | Medium | Separate Web UI and private media storage cleanly: serve the **React Static Web SPA (`agentsys-web-*`)** via **Amazon CloudFront CDN** (leveraging CloudFront's Always Free **1 TB Data Transfer Out & 10M requests/month**), while keeping **`MediaBucket`** blocked from public access (`Block All Public Access = true`) behind scoped **S3 Presigned URLs (`PUT: 900s / GET: 3600s`)**. |
| **7** | **DynamoDB 400KB Item Limit & Logs/Polling Overhead:** Lengthy transcripts exceeding DynamoDB 400KB single-item limits; unbounded logs or aggressive SQS polling inflating baseline bills. | Medium | Automatic S3-offloading for transcript payloads exceeding **250 KB** (`s3://MediaBucket/transcripts/raw/{file_id}.json`), storing a 15,000-character preview and URI in DynamoDB. Enforce **SQS Long Polling (`ReceiveMessageWaitTimeSeconds = 10`)**, **CloudWatch Log Retention = 14 days**, and **S3 Lifecycle expiration (7 days)** on `transcripts/`. |
