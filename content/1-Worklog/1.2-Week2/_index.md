---
title: "Week 2"
date: 2026-08-28
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Week 2: S3 Storage Design, IAM Least-Privilege & SAM Baseline
*Internship Period: 24/08/2026 – 28/08/2026*

### 1. Week 2 Objectives:
* Design object storage topology on Amazon S3 featuring Block Public Access and default SSE-S3 encryption.
* Architect a prefix-separation model to eliminate the risk of recursive invocation loops.
* Define IAM Execution Roles for Lambda functions following least-privilege principles.
* Initialize infrastructure template (infra/template.yaml) using AWS SAM and request foundation model access on Amazon Bedrock.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Designed S3 bucket agentsys-mediabucket.<br>- Enforced Block All Public Access and default Server-Side Encryption (SSE-S3 AES-256). | 24/08/2026 | 24/08/2026 | [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/) |
| **Tue** | - Defined S3 prefix partitions: uploads/ (incoming files) and transcripts/ (ASR outputs).<br>- Configured S3 Event Notification prefix filters to prevent recursive trigger loops. | 25/08/2026 | 25/08/2026 | [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html) |
| **Wed** | - Authored IAM Execution Roles in SAM template.<br>- Decoupled permissions between background ingestion worker (agentsys-ingest-role) and query API (agentsys-api-role). | 26/08/2026 | 26/08/2026 | [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| **Thu** | - <mark style="background-color: #fef08a; color: #854d0e; padding: 2px 6px; border-radius: 4px; font-weight: 600; border: 1px solid #fde047;">Sick leave (medical prescription attached below)</mark>. | 27/08/2026 | 27/08/2026 | [AWS SAM Specification](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification.html) |
| **Fri** | - Requested Foundation Model Access on AWS Bedrock Console for Titan Text Embeddings V2 and Nova Lite.<br>- Reviewed technical guides in preparation for AI integration. | 28/08/2026 | 28/08/2026 | [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/) |

### 3. Week 2 Achievements:
* Established S3 bucket baseline with 100% Block Public Access and SSE-S3 AES-256 encryption.
* Enforced prefix-filtering on uploads/, guaranteeing zero risk of recursive invocation cascades when writing transcripts back to storage.
* Successfully segregated execution permissions between ingestion and query layers under least-privilege standards.
* SAM template validated cleanly with 0 syntax or linting errors via sam validate --lint.
* Foundation model access granted on Amazon Bedrock for both embedding generation and query planning models.

### 4. Medical Excuse Documentation (27/08/2026):

On 27/08/2026, the student was unable to attend the office due to a sudden fever and visited the dormitory clinic for examination. The Portal attendance record and medical prescription are attached below:

![August 2026 FCAJ Portal Attendance Calendar Record](/images/1-Worklog/attendance-portal-august-2026.png)
*Figure: FCAJ Portal attendance record for August 2026.*

![Official Medical Examination Slip and Prescription from VNU-HCM Health Clinic](/images/1-Worklog/medical-prescription-2026-08-27.png)
*Figure: Medical prescription dated 27/08/2026.*

### 5. References:
* [Amazon S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
* [AWS Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
* [Amazon Bedrock Model Access Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
