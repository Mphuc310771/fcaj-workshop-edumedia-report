---
title: "Proposal"
date: 2026-08-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Project Proposal: EduMedia AI — Smart Lecture Storage & Search Engine on AWS

**Student:** Do Tran Minh Phuc · VNU-HCM University of Science (HCMUS)  
**Internship Period:** 17/08/2026 – 06/11/2026 · Amazon Web Services Vietnam LLC (FCAJ Bootcamp)

---

## 1. Executive Summary

**EduMedia AI** is an intelligent, multi-modal lecture cataloging and retrieval platform engineered entirely on a **Serverless** and **Generative AI** architecture using **Amazon Web Services (AWS)**.

The system addresses the fundamental limitation of traditional camera-roll search by automatically extracting multi-sensory semantics from images, audio, and video files:
* **Amazon S3**: Secure object storage with Block Public Access and SSE-S3 encryption.
* **Amazon SQS & Dead-Letter Queue (DLQ)**: Ingestion buffer ensuring resilient asynchronous processing with zero data loss.
* **AWS Lambda & Managed AI**: Orchestrating **Amazon Rekognition** (detecting objects, scenes, and Vietnamese OCR) and **Amazon Transcribe** (high-accuracy Vietnamese speech-to-text *vi-VN*).
* **Amazon Bedrock (Amazon Titan & Nova Lite)**: Serves as an AI Planner and Vector Engine converting natural language queries into optimized search plans.
* **Amazon DynamoDB**: Stores media metadata with Global Secondary Indexes (**GSI category-index**) enabling hybrid lexical and semantic vector search.
* **Amazon CloudWatch**: End-to-end observability with custom metrics, structured log streams, and automated alarm systems.

---

## 2. Problem Statement

In academic and professional environments, students and researchers accumulate thousands of multimodal files: whiteboard notes, lecture slide snapshots, multi-hour voice recordings, and video demos.

### Major Inefficiencies in Existing Solutions:
1. **Filename-based Search Failure**: Mobile uploads default to non-descriptive names (e.g., IMG_20260826_104212.jpg or Voice_001.m4a), providing zero semantic metadata.
2. **Costly Commercial DAM Solutions**: Enterprise Digital Asset Management (DAM) platforms require 24/7 dedicated servers starting at $50–$200/month, making them impractical for students or small research teams.
3. **Weak Vietnamese Natural Language Support**: Open-source toolkits struggle with diacritics and domain-specific terminology across Mathematics, Economics, and Philosophy.

---

## 3. Goals & Success Criteria

| ID | Objective | Pass Condition | Responsible AWS Service |
| :--- | :--- | :--- | :--- |
| **G1** | **Ingest API** | Presigned S3 PUT + Item transitions to READY status in DynamoDB in < 5 seconds. | S3, SQS, Lambda, DynamoDB |
| **G2** | **Computer Vision** | Accurately extracts object labels + Vietnamese OCR text from classroom board snapshots. | Amazon Rekognition |
| **G3** | **Speech-to-Text** | Transcribes Vietnamese speech (*vi-VN*) from audio/video into the transcript field. | Amazon Transcribe |
| **G4** | **Hybrid Search** | Natural language queries like *"find teacher review for final exam"* accurately return relevant media items. | Amazon Bedrock, DynamoDB GSI |
| **G5** | **Well-Architected & FinOps** | 100% Serverless (Scale-to-Zero), $0 idle cost, automated CloudWatch Alarms, and verified Cleanup runbook. | CloudWatch, IAM, SAM CLI |

---

## 4. Proposed System Architecture
AgentSys conforms strictly to **AWS Cloud-Native Serverless** blueprints:

![EduMedia AI AWS Architecture](/images/2-Proposal/architecture.png)

### Data Pipeline Workflow:
1. **Upload & Presign Flow**: The client requests a Presigned URL from agentsys-api (Lambda) via API Gateway (endpoint POST /api/upload-url), then uploads files directly to s3://MediaBucket/uploads/ via HTTPS PUT.
2. **Asynchronous Ingestion Buffer**: S3 emits s3:ObjectCreated:\* notifications to **Amazon SQS**. The buffer holds messages up to 14 days with a **Dead-Letter Queue (DLQ)** configured at maxReceiveCount=3.
3. **AI Metadata Extraction**: agentsys-ingest (Lambda) consumes message batches:
   - Images: Invokes DetectLabels and DetectText (OCR) concurrently via **Amazon Rekognition**.
   - Audio/Video: Starts asynchronous Vietnamese transcription (*vi-VN*) via **Amazon Transcribe**.
4. **Metadata & Embedding Aggregation**: Extracted content is converted into a 256-dimensional semantic embedding via **Amazon Titan Text Embeddings V2** (amazon.titan-embed-text-v2:0) on **Amazon Bedrock** and persisted in **DynamoDB**.
5. **Intelligent Query Execution**: Upon receiving a user query, **Amazon Bedrock (Amazon Nova Lite: amazon.nova-lite-v1:0)** parses intent and coordinates DynamoDB GSI filtering (category-index), capped Candidate Pool retrieval (100 items), Cosine Vector Similarity (0.6 weight), and Normalized Keyword Overlap (0.4 weight).

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
| **Official Audit Report** | [Download Official Well-Architected Review Report (90-Page PDF)](/files/EduMedia_AI_Well_Architected_Report.pdf) |

### 1. Security
* **IAM Least-Privilege**: Ingestion and API Lambda execution roles are strictly decoupled with resource-level scoping.
* **Storage Protection**: S3 features *Block All Public Access*, *SSE-S3 (AES-256)* server-side encryption, and short-lived Presigned URLs.
* **Secret Management**: Zero hardcoded credentials; API authentication uses X-API-Key secured via CloudFormation NoEcho.

### 2. Reliability
* **Traffic Decoupling**: SQS buffers burst ingestion traffic, preventing Lambda concurrency exhaustion.
* **Failure Isolation**: Malformed files are automatically routed to the DLQ after 3 retries, triggering CloudWatch Alarms.
* **DynamoDB 400KB Item Guard (S3-Offloading)**: Transcripts exceeding 250KB are automatically offloaded to Amazon S3 (transcripts/raw/{file_id}.json), storing a 15,000-character preview and S3 URI in DynamoDB to safely maintain items well below the 400KB limit while preserving instant UI access.

### 3. Performance Efficiency
* **Vector Subsampling**: Embedding dimensions are compressed from 1536 to **256 dimensions**, reducing DynamoDB storage footprint by 83% while retaining >98% retrieval accuracy.
* **DynamoDB GSI Partitioning**: GSI queries on category-index eliminate full-table scans, reducing latency by over 90%.
* **Lambda Warm-Container Reuse**: Boto3 client connections and TLS handshakes persist across warm Lambda invocations, avoiding 50–120ms reconnection overhead on every SQS message.
* **In-Memory Query Plan Cache (\_PLAN\_CACHE)**: LRU caching of parsed search plans speeds up repeated query responses by **3.7x** and saves 100% of LLM token costs.

### 4. Cost Optimization (FinOps)
* **Scale-to-Zero Model**: Idle infrastructure costs exactly **$0.00 USD**.
* **Bedrock LRU Caching**: In-memory query caching eliminates token costs for recurring search queries.
* **S3 Lifecycle Rules**: Automatic expiration for temporary files in transcripts/ after 7 days.

### 5. Operational Excellence
* **Infrastructure as Code (IaC)**: The entire stack is provisioned via **AWS SAM (infra/template.yaml)**.
* **Observability**: Structured JSON logging in CloudWatch with automated alarms for agentsys-ingest-errors and agentsys-ingest-dlq.

### 6. Sustainability
* Fully leverages shared multi-tenant AWS Managed Services, eliminating energy waste from idle GPU compute.

---

## 6. AWS FinOps Cost Modeling & Production Workload Estimates (Region: us-east-1)

The system adheres to a strict **Scale-to-Zero** FinOps model. All operational costs have been modeled and calculated using the official **AWS Pricing Calculator**:
* **Official AWS Public Estimate:** [EduMedia AI Workload Estimate (AWS Pricing Calculator)](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
* **Official Exported Estimate (PDF):** [Download AWS Pricing Calculator Estimate (PDF)](/files/EduMedia_AI_AWS_Pricing_Calculator_Estimate.pdf)

### 6.1. Unit Economics (AWS Region: us-east-1)
* **1 Lecture Slide Image Processing:** Amazon Rekognition DetectLabels ($0.0010) + DetectText OCR ($0.0010) = **$0.0020 USD / image** (~51 VND).
* **1 Audio Minute Transcription:** Amazon Transcribe Standard Batch vi-VN = **$0.0060 USD / minute** (~153 VND/min current AWS Calculator rate at $0.0001/sec, or $0.0240 USD / min ceiling list price).
* **1 Intelligent AI Query:** Amazon Titan Text Embeddings v2 ($0.00002/1k tokens) + Amazon Bedrock Nova Pro (1,000 in + 300 out tokens) = **~$0.00178 USD / query** (~45.4 VND). (With Claude 3.5 Haiku: ~$0.00202 USD).
* **AI Agent Reasoning & Intent Parsing:** Amazon Nova Micro ($0.035/1M in + $0.14/1M out tokens) acts as the autonomous Agent orchestrator parsing queries, translating Vietnamese intent, and invoking tools = **~$0.05 – $0.10 USD / month** for 2,000 queries.
* **Amazon S3 Standard Storage:** **$0.0230 USD / GB / month** (~586.5 VND/GB). (API Requests: PUT $0.005 / 1k reqs, GET $0.0004 / 1k reqs).
* **Amazon CloudFront Distribution:** **1 TB (1,000 GB) Data Transfer Out** and **10,000,000 requests/month** free forever (Always Free Tier).
* **AWS Lambda (1024MB, 1.5s):** $0.20/1M requests + $0.0000166667 / GB-s (x86) or $0.0000133334 / GB-s (ARM Graviton2) = **$0.000020 / invocation**. (1M requests + 400,000 GB-s/month free forever).

---

### 6.2. Scenario 1: Self-hosted Workshop & POC Evaluation (1 Tester)
Workload: 50 slide images, 5–10 short demo video clips (15–20 total audio minutes), 100 Bedrock AI queries, 1 GB S3 Standard, ~2,000 Lambda & DynamoDB invocations.

| AWS Service | Evaluation Workload | Gross Cost (Without Free Tier) | Net Invoiced Cost (With Free Tier) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 50 images $\times$ 2 APIs = 100 calls | $0.1000 USD | **$0.0000 USD** (1,000 calls/mo Free Tier) |
| **Amazon Transcribe** | 15–20 minutes vi-VN audio (Batch) | $0.0900 – $0.1200 USD *(Rate card: $0.36 – $0.48)* | **$0.0000 USD** (60 minutes/mo Free Tier) |
| **Amazon Bedrock** | 100 queries Nova Pro/Micro + Titan Embed | $0.1780 USD *(Haiku: $0.2020)* | **$0.1780 USD** *(No quantity Free Tier)* |
| **Amazon S3** | 1 GB Standard + PUT/GET requests | $0.0240 USD | **$0.0000 USD** (5 GB Free Tier allowance) |
| **AWS Lambda & SQS** | 2,000 invocations (1024MB, 1.5s = 3,000 GB-s) | $0.0528 USD | **$0.0000 USD** (Always Free 1M req, 400k GB-s) |
| **Amazon DynamoDB** | On-Demand (2,000 writes + 2,000 reads) | $0.0015 USD | **$0.0015 USD** (25 GB Storage Always Free) |
| **Amazon CloudFront** | Static Web UI distribution (< 1 GB) | $0.0425 USD | **$0.0000 USD** (Always Free 1 TB/mo) |
| **TOTAL (POC)** | **Exchange Rate: 1 USD = 25,500 VND** | **$0.48 – $0.84 USD** *(~12,000 – 21,000 VND)* | **$0.18 USD** *(~4,500 VND - Or $0.00 with Credits)* |

---

### 6.3. Scenario 2: Department Class / Study Club Workload (100 – 200 Students / Month)
Monthly workload: 1,500 lecture slides, 250 audio minutes, 2,000 Bedrock queries, 20 GB S3 storage, 15,000 Lambda & SQS invocations, 15,000 writes + 60,000 reads DynamoDB, 10 GB CloudFront DTO.

| AWS Service | Production Workload (100 - 200 Students) | Gross Cost (Without Free Tier) | Standard Production Bill (With Always Free) |
| :--- | :--- | :---: | :---: |
| **Amazon Rekognition** | 1,500 images $\times$ 2 APIs = 3,000 calls | **$3.00 USD** | **$3.00 USD** *(Year 1: $2.00)* |
| **Amazon Transcribe** | 250 lecture minutes vi-VN | **$1.50 USD** *(Rate card ceiling: $6.00)* | **$1.50 USD** *(Rate card ceiling: $6.00)* |
| **Amazon Bedrock** | 2,000 queries Nova Pro/Micro (1K in + 300 out) + Embed | **$3.56 USD** *(Haiku: $4.04)* | **$3.56 USD** *(Haiku: $4.04)* |
| **Amazon S3** | 20 GB Standard storage + requests | **$0.46 USD** *(+ requests: $0.53)* | **$0.46 USD** *(Year 1: $0.35)* |
| **AWS Lambda & SQS** | 15,000 invocations (22,500 GB-s) + SQS | **$0.38 USD** | **$0.00 USD** *(Always Free Tier)* |
| **Amazon DynamoDB** | On-Demand (15,000 writes + 60,000 reads) | **$0.02 USD** | **$0.02 USD** *(< 25 GB Always Free)* |
| **Amazon CloudFront** | 10 GB Data Transfer Out | **$0.85 USD** | **$0.00 USD** *(Always Free 1 TB/mo)* |
| **TOTAL** | **Full System for 100 - 200 Students** | **$9.77 – $14.27 USD / month** | **$8.54 – $13.09 USD / month** |
| **CURRENCY CONVERSION** | **Exchange Rate: 1 USD = 25,500 VND** | **~249,000 – 364,000 VND / month** | **~217,000 – 334,000 VND / month** |
| **PER-STUDENT COST** | **Scale of 100 – 200 Students** | **1,246 – 3,639 VND / student / month** | **1,088 – 2,177 VND / student / month** |

> **AWS Pricing Calculator vs Rate Card Reconciliation:**
> * On the official live [AWS Pricing Calculator Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934), base monthly cost totals **$4.96 USD / month** (Rekognition $3.00, Transcribe Batch $1.50 at $0.0001/sec, S3 20GB $0.46, Lambda $0.00 Always Free).
> * Calculated against the **Worst-Case Public Rate Card ($0.024/min for Transcribe)** plus Bedrock Inference (Nova Micro + Titan Embeddings) and S3 API / CloudWatch Logs, monthly production spend ranges between **$9.77 – $14.27 USD / month**.
> * Over **82% of operational spend resides in AI Inference**. The entire Serverless compute/storage tier (Lambda, SQS, S3, DynamoDB, CloudFront) costs under **$1.50 USD / month**, automatically scaling to $0.00 USD when idle.

---

## 7. Project Timeline

* **Phase 1 (Month 1):** Requirements analysis, AWS SAM skeleton, S3/SQS configuration, and unit testing suite.
* **Phase 2 (Month 2):** Ingestion pipeline development (Rekognition & Transcribe), DynamoDB GSI, and CloudWatch Alarms.
* **Phase 3 (Month 3):** Amazon Bedrock hybrid search, React Web UI, FinOps optimization, and workshop documentation.

---

## 8. Risks & Mitigations

| No | Identified Risk | Impact | Mitigation Strategy |
| :---: | :--- | :---: | :--- |
| **1** | **DynamoDB Vector Scan Amplification & S3 Race Conditions:** DynamoDB lacks native ANN/HNSW indices; full table scans spike RRUs exponentially. Furthermore, S3 lacks native append/lock, causing concurrent Lambda ingest workers to overwrite `index.bin`. | High | Segregated architecture: Ingestion workers store lecture metadata and raw vectors atomically into DynamoDB via `PutItem`. A scheduled compaction job or single-concurrency Lambda Indexer (`ReservedConcurrency = 1` with `If-Match ETag`) builds a consolidated binary vector index on S3. Search Lambdas cache the index in RAM checking `HeadObject` (TTL 60s), eliminating 99% of DynamoDB reads and preventing concurrent write data loss. |
| **2** | **VPC NAT Gateway Cost Trap:** Placing Lambda inside a VPC private subnet without private RDS incurs a fixed NAT Gateway charge of $32.85 USD/month even when idle. | High | Zero-VPC design: keep Lambda outside VPC as all interactions communicate with AWS managed public APIs (S3, Bedrock, Rekognition, DynamoDB) strictly secured via IAM Least-Privilege roles and TLS 1.3 encryption. |
| **3** | **Infinite S3-to-Lambda Triggers & SQS Retry Storms:** Recursive event loops on file generation exhausting AI API quotas. | High | Strict S3 prefix segregation (`uploads/` trigger only, writing transcripts to `transcripts/` or dedicated bucket); configure SQS Dead-Letter Queue (DLQ) with `maxReceiveCount = 3` to isolate failing messages. |
| **4** | **Rekognition Video Cost Trap ($0.10/min) & Token Caps:** Rekognition Video is extremely costly ($0.10/min/feature = $50 for 250 mins). Moreover, Titan Multimodal Embeddings G1 only accepts text up to 128 tokens and static images, rejecting direct video inputs. | High | Intelligent Multi-Modal Tiering: Video files are routed strictly to Amazon Transcribe for audio extraction and text chunking (≤ 128 tokens) for Titan Embeddings. For visual search, `ffmpeg` on a Lambda Layer extracts static keyframes (1 frame / 30 seconds) routed to Rekognition Image ($0.001/call), achieving 99% cost reduction over Rekognition Video. |
| **5** | **Lambda Synchronous Wait on Batch Transcription:** Batch transcription takes 1–3 minutes. Polling synchronously with `sleep()` inside Lambda inflates GB-sec runtime by 100x. | High | 100% Asynchronous Event-Driven Design: The Ingestion Lambda invokes `StartTranscriptionJob()` and exits immediately in 0.5s. Amazon EventBridge captures `Transcribe Job State Change (COMPLETED)` (or S3 ObjectCreated) to trigger the downstream indexing Lambda. Polling wait cost = $0.00 USD. |
| **6** | **S3 Data Transfer Out (Egress) Spikes:** Hundreds of students streaming lecture videos from S3 via presigned URLs incur $0.09/GB egress fees beyond 100GB. | Medium | Deploy Amazon CloudFront with Origin Access Control (OAC) in front of S3, capitalizing on CloudFront's Always Free 1 TB (1,000 GB) Data Transfer Out and 10,000,000 requests/month, completely eliminating egress risks. |
| **7** | **DynamoDB 400KB Item Limit & SQS Empty Receives:** Lengthy transcripts exceeding DynamoDB 400KB single-item limits; redundant SQS polling costs. | Medium | Automatic S3-offloading for transcript payloads exceeding 250KB, maintaining only pointers in DynamoDB. Configure SQS Long Polling (20s) and set CloudWatch Log Retention to 7 days to eliminate silent idle billing. |
