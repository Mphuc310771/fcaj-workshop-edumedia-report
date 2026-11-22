---
title: "Prerequisites"
date: 2026-10-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# 5.2. Environment Setup, AWS CLI, SAM, IAM Least-Privilege & Enabling Amazon Bedrock Model Access

---

## 1. Prerequisites & Tooling Setup

Before getting started, ensure the following developer tools are configured locally:

1. **AWS Account** with AdministratorAccess or permissions to provision IAM Roles, S3, DynamoDB, Lambda, SQS, API Gateway, and CloudWatch.
2. **AWS CLI v2**: Configured via aws configure.
3. **AWS SAM CLI**: Tooling for building and deploying Serverless applications.
4. **Python 3.12** & **Node.js 20+** (for running the Web UI).

### Verify AWS CLI Configuration:
```bash
aws sts get-caller-identity
aws configure get region
```
*Ensure the returned region is **us-east-1** (N. Virginia).*

---

## 2. Setting up Cost Guardrails (AWS Budgets)

To guarantee zero unexpected charges during development:
1. Open the **AWS Management Console** → Search for **AWS Budgets**.
2. Select **Create budget** → **Cost budget**.
3. Name: My Zero-Spend Budget → Monthly budget limit: **$1.00 USD** (Zero-Spend protection).
4. Configure an email notification trigger when actual spend exceeds threshold to ensure $0.00 USD billing.

---

## 3. Amazon Bedrock Model Access Management (Automated On-Demand Enablement)

Under the latest AWS Bedrock operational policies, access management for foundation models has been streamlined into fully serverless automated enablement:

1. **Model Access Retirement Notice & Instant Availability:**
   * For Amazon Serverless Foundation Models (including **Amazon Titan Text Embeddings V2** and **Amazon Nova Lite / Nova Pro**), AWS has officially retired manual model access application workflows.
   * Models are now **automatically enabled on-demand** across all AWS commercial regions upon first invocation (`InvokeModel` or `Converse API`). Developers no longer need to submit manual use-case forms or await approval badges.

![Official AWS Bedrock notification: Model access page retired in favor of automated on-demand enablement](/images/5-Workshop/evidence/bedrock-model-access.png)

2. **IAM Least-Privilege Governance:**
   * Model invocation boundaries are now strictly governed by standard **IAM Policies** attached to Lambda execution roles (`bedrock:InvokeModel`) and Service Control Policies (SCPs), ensuring zero-trust governance directly via Infrastructure as Code (SAM).

{{% notice tip %}}
**Note:** For select third-party partner models (such as Anthropic Claude via AWS Marketplace), accounts may prompt for a one-time use-case registration prior to first invocation.
{{% /notice %}}

---

## 4. IAM Least-Privilege Architecture
 
The SAM template (`infra/template.yaml`) defines three strictly segregated execution roles following least-privilege principles, mapped 1:1 to each Lambda function's operational scope:

1. **IngestFunctionRole** (`agentsys-ingest` - lines 260-315 in `template.yaml`):
   * **S3 Access (`ReadMediaObjects`)**: `s3:GetObject` on `uploads/*` to retrieve raw media; `s3:PutObject` on `thumbs/*` and `transcripts/*` to store generated thumbnails and full Transcribe outputs.
   * **AI / ML Ingestion**: Slide label and text/formula extraction (`rekognition:DetectLabels`, `rekognition:DetectText`); speech transcription pipeline control (`transcribe:StartTranscriptionJob`, `transcribe:GetTranscriptionJob`).
   * **DynamoDB Access (`DynamoWrite`)**: Status persistence and chunk writes (`dynamodb:PutItem`, `dynamodb:GetItem`, `dynamodb:UpdateItem`, `dynamodb:Scan`, `dynamodb:Query`, `dynamodb:BatchWriteItem`, `dynamodb:DescribeTable`) on table `AgentSysMediaMetadata` and GSI `category-index`.
   * **Amazon Bedrock (`BedrockInvoke`)**: Foundation model invocation (`bedrock:InvokeModel`) for `amazon.nova-lite-v1:0`, `amazon.nova-pro-v1:0`, and `amazon.titan-embed-text-v2:0`.
   * **CloudWatch Custom Metrics (`CustomMetrics`)**: Operational telemetry publishing (`cloudwatch:PutMetricData`) strictly conditioned on `cloudwatch:namespace: AgentSys`.
   * **SQS Ingestion Event**: SQS consumer permissions on `agentsys-ingest-queue`.
2. **AggregatorFunctionRole** (`agentsys-aggregator` - lines 355-402 in `template.yaml`):
   * **DynamoDB Streams & Table (`DynamoDBStreamAndTableAccess`)**: Stream event processing (`dynamodb:DescribeStream`, `dynamodb:GetRecords`, `dynamodb:GetShardIterator`, `dynamodb:ListStreams`); keyed chunk reading and updates (`dynamodb:GetItem`, `dynamodb:BatchGetItem`, `dynamodb:UpdateItem`, `dynamodb:Query`, `dynamodb:Scan`, `dynamodb:DescribeTable`) on table `AgentSysMediaMetadata`.
   * **Amazon Bedrock (`BedrockInvocation`)**: Summary synthesis invocation (`bedrock:InvokeModel`) on `amazon.nova-lite-v1:0` and `amazon.nova-pro-v1:0`.
   * **SQS Stream DLQ (`SQSStreamDLQAccess`)**: Forwarding unrecoverable stream events (`sqs:SendMessage`) to `AggregatorStreamDLQ`.
   * **CloudWatch Custom Metrics (`CustomMetrics`)**: Emitting operational metrics (`cloudwatch:PutMetricData`) to namespace `AgentSys`.
3. **ApiFunctionRole** (`agentsys-api` - lines 435-502 in `template.yaml`):
   * **S3 Presigned Access (`PresignUploads`)**: Generating Presigned PUT URLs for direct uploads (`uploads/*`, `direct/*`), Presigned GET URLs for secure media/thumbnail delivery (`thumbs/*`, `uploads/*`), and `s3:DeleteObject` for asset cleanup.
   * **DynamoDB Access (`DynamoAccess`)**: Media metadata CRUD and catalog querying (`dynamodb:GetItem`, `dynamodb:PutItem`, `dynamodb:DeleteItem`, `dynamodb:Scan`, `dynamodb:Query`, `dynamodb:BatchWriteItem`) across `AgentSysMediaMetadata` and GSI `category-index`.
   * **Amazon Bedrock (`BedrockInvoke`)**: Foundation model inference permissions (`bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`) for `amazon.nova-lite-v1:0`, `amazon.nova-pro-v1:0`, and `amazon.titan-embed-text-v2:0` powering AI Search Planner and Converse API endpoints.
   * **SQS Ingestion Dispatch (`IngestQueueSend`)**: Queue dispatching permissions (`sqs:SendMessage`) on `agentsys-ingest-queue` when clients invoke media processing or reprocessing routes (`/process`, `/reprocess`).
   * **CloudWatch Custom Metrics (`CustomMetrics`)**: Publishing operational telemetry (`cloudwatch:PutMetricData`) to namespace `AgentSys`.
   * **Health Check & Diagnostics (`HealthPings`, `HealthBucket`, `HealthTable`)**: Subsystem diagnostics for the `/health` endpoint (`sts:GetCallerIdentity`, `rekognition:ListCollections`, `transcribe:ListTranscriptionJobs`, `bedrock:GetFoundationModel`, `bedrock:GetInferenceProfile`, `s3:ListBucket` on `MediaBucket`, `dynamodb:DescribeTable` on `MetadataTable`).

---

## 5. Running Automated Unit Tests

Run the local unit test suite with Pytest to verify core modules (ingestion pipeline, API authentication, and Bedrock query handling) before deploying to AWS:

```bash
# 1. Clone the EduMedia AI repository
git clone https://github.com/Mphuc310771/edumedia-ai.git
cd edumedia-ai

# 2. Create and activate Python virtual environment
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

# 3. Install dependencies and run 143 unit test cases
pip install -r requirements.txt
pytest -v
```
*Expected: Test suite passes, confirming business logic and IAM access assumptions function as designed.*

---

## 6. Building and Deploying with AWS SAM

From the `edumedia-ai` project directory:

```bash
sam build -t infra/template.yaml
sam deploy --guided
```

### Guided Deployment Parameters:
| Parameter | Recommended Value | Purpose |
| :--- | :--- | :--- |
| **Stack Name** | edumedia-ai | CloudFormation Stack Name (or agentsys) |
| **AWS Region** | us-east-1 | Target AWS Region |
| **Parameter ApiKey** | dev-local-api-key | API Protection Key (X-API-Key header) |
| **Parameter FrontendOriginUrl** | \* (or your Vercel/CloudFront domain) | Allowed Origin for CORS |
| **Confirm changes before deploy** | Y | Review CloudFormation changeset |
| **Allow SAM CLI IAM role creation** | Y | Grant SAM permission to create IAM roles |

After deployment succeeds, record the values in **Outputs**:
* ApiEndpoint: HTTP API Gateway Invoke URL (e.g., https://xxxx.execute-api.us-east-1.amazonaws.com - *note: HTTP API uses the $default stage, so do NOT append /Prod*)
* MediaBucketName: S3 Media Bucket Name
* MetadataTableName: AgentSysMediaMetadata
