---
title: "Week 6"
date: 2026-09-25
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

# Week 6: Ingestion Pipeline & S3 Storage Architecture
*Internship Period: 21/09/2026 – 25/09/2026*

### 1. Week 6 Objectives:
* Implement ingestion worker Lambda module (apps/workers/ingestion.py) in Python 3.12.
* Optimize Lambda container execution by reusing Boto3 connection pools and TLS sessions outside handler scope.
* Author input classification router to distinguish media types based on MIME extensions.
* Package and deploy baseline Serverless stack to AWS via sam build and sam deploy.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Structured agent_sys/ project hierarchy.<br>- Authored apps/workers/ingestion.py consuming SQS messages. | 21/09/2026 | 21/09/2026 | [AWS Lambda Programming Model](https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model.html) |
| **Tue** | - Optimized container warm-up behavior.<br>- Initialized Boto3 clients (S3, Rekognition, Transcribe) in global scope outside handler. | 22/09/2026 | 22/09/2026 | [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) |
| **Wed** | - Implemented detect_media_type router.<br>- Differentiated images (JPG, PNG, WEBP), audio (MP3, WAV, M4A), and video (MP4, MOV). | 23/09/2026 | 23/09/2026 | Python mimetypes Module |
| **Thu** | - Authored initial pytest unit test suites.<br>- Verified media routing logic across diverse and unrecognized file extensions. | 24/09/2026 | 24/09/2026 | [Pytest Documentation](https://docs.pytest.org/) |
| **Fri** | - Ran sam build and executed initial deployment to AWS in us-east-1.<br>- Verified SQS to Lambda execution logs in CloudWatch Logs confirming successful processing. | 25/09/2026 | 25/09/2026 | [SAM CLI Deploy](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html) |

### 3. Week 6 Achievements:
* Implemented modular apps/workers/ingestion.py executing under Python 3.12 runtime.
* Optimized container execution: global client instantiation enables TCP connection pooling and TLS session reuse, shaving 50–120ms off warm invocations.
* Validated media routing logic across all target categories (Images, Audio, Video), preventing mismatched payloads from reaching AI services.
* Created baseline unit tests verifying fallback behaviors on unmapped MIME extensions.
* Deployed agentsys-stack to AWS us-east-1 via AWS SAM; CloudWatch Logs confirmed end-to-end event propagation from S3 upload through SQS into Lambda.

### 4. References:
* [Optimizing AWS Lambda performance](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
* [Building Serverless Applications with SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started.html)
* [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
