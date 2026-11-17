---
title: "Week 1"
date: 2026-08-21
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Week 1: FCAJ Onboarding & Environment Setup
*Internship Period: 17/08/2026 – 21/08/2026*

### 1. Week 1 Objectives:
* Complete onboarding for the FCAJ 2026 program, register HCM-Portal account, and review office guidelines.
* Install local development toolchain on Ubuntu 24.04: Python 3.12, AWS CLI v2, AWS SAM CLI, and Docker Engine.
* Set up AWS account with MFA enabled and configure AWS CLI named profile using dedicated IAM user credentials.
* Initialize EduMedia AI project repository structure on GitHub and outline serverless event-driven architecture approach.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Attended FCAJ 2026 internship orientation session.<br>- Activated HCM-Portal account and reviewed program milestones, attendance, and office conduct regulations. | 17/08/2026 | 17/08/2026 | [FCAJ HCM Rules & Guidelines](https://hcm-rules.awsfcaj.com/) |
| **Tue** | - Installed development toolchain on Ubuntu 24.04: Python 3.12, AWS CLI v2, AWS SAM CLI, and Docker.<br>- Verified Docker daemon execution for local Lambda testing. | 18/08/2026 | 18/08/2026 | [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/) |
| **Wed** | - Configured AWS account security, enabled MFA, and created dedicated IAM User (avoiding Root Account).<br>- Configured named profile in ~/.aws/credentials and verified connectivity via aws sts get-caller-identity. | 19/08/2026 | 19/08/2026 | [AWS Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) |
| **Thu** | - Initialized EduMedia AI repository on GitHub with baseline directory layout (infra, src, tests).<br>- Structured bilingual project documentation outlines. | 20/08/2026 | 20/08/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/) |
| **Fri** | - Evaluated AWS services for multimodal ingestion: S3, Lambda, Rekognition, Transcribe, DynamoDB, and Bedrock.<br>- Drafted initial asynchronous event-driven workflow for project EduMedia AI. | 21/08/2026 | 21/08/2026 | [AWS Serverless Architecture](https://aws.amazon.com/serverless/) |

### 3. Week 1 Achievements:
* Activated HCM-Portal account and clarified internship milestones and evaluation criteria.
* Completed local dev environment configuration with AWS CLI credentials targeting region us-east-1.
* Secured AWS account with MFA and adopted dedicated IAM user credentials following AWS security best practices.
* Established GitHub repository with organized source code structure and bilingual documentation hierarchy.
* Outlined core architecture for AgentSys: adopted a Pay-as-you-go Serverless model (Lambda, S3, DynamoDB On-Demand) that scales to zero during idle periods, eliminating recurring server maintenance expenses.

### 4. References:
* [FCAJ HCM Rules & Guidelines Portal](https://hcm-rules.awsfcaj.com/)
* [AWS Command Line Interface User Guide v2](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
* [AWS Serverless Application Model (SAM) Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
* [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
* [AWS Serverless Architecture Overview](https://aws.amazon.com/serverless/)
