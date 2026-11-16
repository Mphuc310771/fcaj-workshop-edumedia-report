---
title: "Architecture Trade-offs & Enterprise Roadmap"
date: 2026-10-21
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

# 5.7. Discussion: Architecture Trade-offs & Enterprise Roadmap

During the initial Proof of Concept (PoC) phase for the First Cloud AI Journey (FCAJ), the core engineering objective for AgentSys was to build a production-grade multimodal AI media catalog while adhering strictly to a **Zero-Idle-Cost, Lean Serverless architecture**.

To strike the right balance between rapid experimental learning and cost-effectiveness, deliberate architectural trade-offs were made. Below is a rigorous technical evaluation of the 4 key limitations of the current architecture along with target enterprise-grade AWS solutions for production scale.

---

### 1. In-Memory Vector Search on Lambda RAM
* **Current PoC Status:**
  DynamoDB is a document/key-value NoSQL database without native Approximate Nearest Neighbor (ANN) vector indexing. AgentSys implements a Two-Stage Search strategy: Amazon Bedrock acts as Query Planner extracting metadata categories for sub-15ms DynamoDB GSI queries (category-index), followed by in-memory Cosine Similarity scoring over a capped candidate pool (100 newest items) within Lambda RAM.
* **Limitations:**
  While providing zero idle cost, capping the candidate pool at 100 items causes recall degradation when catalogs exceed tens of thousands of items. Conversely, unbounded table scans incur linear `O(N)` cost and latency penalties.
* **AWS Enterprise Target:**
  - Transition to **Amazon OpenSearch Serverless (Vector Engine)** or **Amazon Aurora Serverless (PostgreSQL + pgvector)**.
  - Vectors are indexed using HNSW or IVF algorithms directly in storage, achieving sub-20ms k-NN retrieval across millions of records without moving raw embeddings into Lambda RAM.

---

### 2. Vector Dimensionality & Amazon Titan Embeddings V2 Upgrade
* **Current PoC Status:**
  Earlier implementations subsampled 1536-d vectors to 256-d via uniform index striding to reduce DynamoDB item storage below 400KB limits.
* **Limitations:**
  Neural embedding latent dimensions are non-linear; uniform subsampling degrades topological semantic manifold structure.
* **AWS Enterprise Target:**
  - Upgraded to native **Amazon Titan Text Embeddings V2 (amazon.titan-embed-text-v2:0)**, which natively supports Matryoshka Representation Learning with direct dimensions: 256 and normalize: true parameterization.
  - Future roadmap includes **Amazon Titan Multimodal Embeddings** to unify image, audio, and text vectors into a single joint vector space.

---

### 3. DynamoDB Schema & Chunk Scan Overhead
* **Current PoC Status:**
  AgentSysMediaMetadata uses a simple primary key (file\_id). Lecture transcript chunks are stored using composite IDs (file\_id#CHUNK#0001), requiring DynamoDB FilterExpressions during chunk aggregation.
* **Limitations:**
  FilterExpressions evaluate after disk reads, consuming Read Capacity Units (RCUs) across irrelevant records.
* **AWS Enterprise Target:**
  - Normalize to a true **DynamoDB Single-Table Design**:
    - **Partition Key (PK):** FILE#&lt;file\_id&gt;
    - **Sort Key (SK):** METADATA or CHUNK#&lt;index&gt;
  - Chunk queries become single partition reads: Query(PK='FILE#123', SK begins\_with 'CHUNK#') with exact `O(k)` complexity.

---

### 4. Distributed Map-Reduce for Long Lectures
* **Current PoC Status:**
  Upon Transcribe completion, an EventBridge rule triggers a Lambda worker that sequentially refines chunks with Bedrock and performs reduction, paired with DynamoDB Streams for race condition defense.
* **Limitations:**
  For 1-2 hour lectures (40-80 chunks), sequential LLM inference risks hitting Lambda's execution timeout limit.
* **AWS Enterprise Target:**
  - Replace sequential Lambda loops with **AWS Step Functions Distributed Map**:
    1. **Split State:** Chunk transcript and store in S3/DynamoDB.
    2. **Distributed Map:** Fan-out up to 50 concurrent Lambda workers running Bedrock chunk refinement in parallel.
    3. **Reduce State:** Fan-in and aggregate final structured summaries.
  - Reduces total processing time from minutes to under 10 seconds while eliminating timeout risks.

---

### 5. DynamoDB 400KB Item Guard (S3-Offloading) & High-Performance Warm Containers
* **Current PoC Status & Implemented Solution:**
  - **S3-Offloading (>250KB):** DynamoDB enforces a strict 400KB limit per item. Long audio lectures or multi-speaker presentations can produce transcripts exceeding 400KB–1MB. To eliminate ValidationException: Item size has exceeded the maximum allowed size of 400KB, AgentSys automatically detects transcripts larger than 250KB, writes the raw full transcript to Amazon S3 (transcripts/raw/{file\_id}.json), and stores a 15,000-character preview snippet alongside transcript\_s3\_uri and has\_extended\_transcript: true in DynamoDB. The React frontend supports in-modal viewing and direct raw JSON download.
  - **Lambda Warm Container Reuse:** Preserves Boto3 connection pools and TLS handshakes across invocations, removing cold connection teardowns and saving 50–120ms per SQS message or API request.
  - **In-Memory Query Plan Cache (\_PLAN\_CACHE):** Caches parsed search plans in Lambda RAM (LRU, up to 256 entries, 1-hour TTL). Under 50 concurrent requests, this cache provides a **3.74x** speedup (reducing mean latency from ~3,194ms down to ~855ms) and saves 100% of LLM token costs on repeated queries.
* **AWS Enterprise Target:**
  - Introduce **Amazon ElastiCache Serverless (Redis)** to share cached query plans across all horizontally scaled Lambda instances.

---

### 6. Technical Defense & FinOps Q&A Guide

To prepare rigorously for oral defense before the academic and industry review committee, the following 3 deep architectural questions and quantitative answers are compiled:

#### ❓ Question 1: "When extracting 1 keyframe every 30 seconds across 250 video minutes using ffmpeg, has this additional image volume been accounted for in the Scenario 2 Rekognition and Bedrock budget?"
* **Authoritative Answer:** 
  Yes, this cost is fully absorbed within Scenario 2's variance buffer. Specifically: 250 minutes of video sampled at 1 frame / 30 seconds yields exactly **500 keyframes**. Routing these 500 frames through Amazon Rekognition (DetectLabels + DetectText) and Amazon Bedrock (Titan Multimodal Embeddings G1) incurs:
  $$\text{Incremental Cost} = (500 \times 2 \times \$0.001) + (500 \times \$0.00006) = \$1.03\text{ USD/month}$$
  This **$1.03 USD** expense fits comfortably within the **$4.50 USD contingency margin** (between the **$9.77** base and **$14.27 USD** ceiling) defined in our FinOps model.

#### ❓ Question 2: "Why did the architecture adopt an LLM (Amazon Nova Micro) for Vietnamese query translation rather than the dedicated Amazon Translate service?"
* **Authoritative Answer:** 
  Selecting Amazon Nova Micro over Amazon Translate provides two decisive FinOps and architectural advantages:
  1. **Cost Efficiency (~1,500x Cheaper):** Amazon Translate costs **$15.00 USD / 1M characters**. In contrast, `Amazon Nova Micro` on Bedrock is priced at just **$0.035 USD / 1M input tokens** (~3.5–4M characters), delivering a ~1,500x cost reduction for translation.
  2. **Single-Pass Multi-Tasking:** Within a single unified prompt invocation, Nova Micro performs user intent classification (Intent Routing), temporal/subject metadata filtering, and visual-context query translation into English for Titan Embeddings. This eliminates cross-service network latency compared to serial pipeline calls.

#### ❓ Question 3: "Lambda uses ffmpeg to extract frames from large video files (hundreds of MBs). Lambda's default free /tmp storage is only 512MB; is there a risk of disk overflow or extra /tmp provisioning fees?"
* **Authoritative Answer:** 
  The system utilizes **Streaming Frame Extraction via S3 Presigned URLs**. Rather than downloading entire `.mp4` video files to Lambda's local `/tmp` disk, Lambda streams the presigned URL directly into the ffmpeg execution command:
  ```bash
  ffmpeg -ss <timestamp> -i "https://<bucket>.s3.amazonaws.com/<key>?<presigned_params>" -frames:v 1 -f image2 pipe:1
  ```
  Leveraging S3's native **HTTP Byte-Range Requests**, `ffmpeg` reads only the container metadata header (moov atom), seeks directly to the target byte timestamp, and streams the static frame into memory (RAM). This consumes zero `/tmp` disk space, completely prevents `No space left on device` exceptions, and completes within 1–2 seconds.

---

### Comparison Summary: PoC vs. Enterprise Production Architecture

| Technical Criteria | AgentSys PoC (Current) | Enterprise Production Target |
| :--- | :--- | :--- |
| **Vector Search Engine** | Candidate Pool (100) + In-Memory Cosine | Amazon OpenSearch Serverless / pgvector (HNSW) |
| **Embedding Space** | Titan v2 (256-d native) + v1 Fallback | Titan Multimodal Embeddings V2 (256-d / 1024-d) |
| **Data Model** | Single PK (file\_id) + FilterExpression | Composite Key (PK: FILE#id, SK: CHUNK#idx) |
| **400KB Item Size Guard**| Automated S3-Offloading for Transcripts > 250KB | S3 Intelligent-Tiering + OpenSearch Document Pointer |
| **Map-Reduce Orchestration**| EventBridge Worker + Streams Demo | AWS Step Functions (Distributed Map State) |
| **Container Lifecycle** | Warm Container Boto3 reuse + LRU \_PLAN\_CACHE | Provisioned Concurrency + ElastiCache Serverless |
| **Audio Processing Latency**| Sequential `O(N)` per chunk | Concurrent `O(1)` distributed fan-out |
| **Idle Infrastructure Cost**| **0.00 USD / month** (Zero-Idle Serverless) | ~20 – 50 USD / month (OpenSearch Serverless OCU baseline) |
| **Core Objective** | Cost optimization, rapid lab demo, PoC | Scalability to millions of records, Enterprise SLA |
