---
title: "Week 12"
date: 2026-11-06
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

# Week 12: Workshop Documentation Packaging & Technical Verification
*Internship Period: 02/11/2026 – 06/11/2026*

### 1. Week 12 Objectives:
* Author and package comprehensive 7-chapter Hands-on Workshop guide (5.1 through 5.7) matching FCAJ standards.
* Validate automated resource teardown runbook and script (`cleanup.sh`) ensuring thorough environment purging to avoid unforeseen costs.
* Verify FinOps cost projections against AWS Pricing Calculator and actual expenses in AWS Cost Explorer / Budgets ($0.00 USD).
* Review GitHub codebase repository, complete final internship documentation, and complete technical verification of EduMedia AI.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Authored Workshop Chapter 5.1 (Architecture & FinOps overview) and Chapter 5.2 (Prerequisites & AWS SAM CLI).<br>- Detailed AWS credentials configuration and development toolchain setup. | 02/11/2026 | 02/11/2026 | [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) |
| **Tue** | - Authored Workshop Chapter 5.3 (SAM Infrastructure Deployment) and Chapter 5.4 (Media Extraction Testing).<br>- Captured screenshots of CloudFormation stack status and automated Rekognition/Transcribe data flows. | 03/11/2026 | 03/11/2026 | [Amazon Rekognition Docs](https://docs.aws.amazon.com/rekognition/) |
| **Wed** | - Authored Workshop Chapter 5.5 (Bedrock Search & Web Gallery) and Chapter 5.7 (Technical Trade-offs & Roadmap).<br>- Guided users through semantic search queries and evidence playback exploration. | 04/11/2026 | 04/11/2026 | [Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/) |
| **Thu** | - Engineered Workshop Chapter 5.6 (CloudWatch Monitoring & Clean-up Runbook).<br>- Tested automated script `cleanup.sh` purging S3 buckets, DynamoDB tables, and CloudFormation stacks in under 3 minutes. | 05/11/2026 | 05/11/2026 | [AWS CLI S3 cleanup](https://docs.aws.amazon.com/cli/latest/reference/s3/rm.html) |
| **Fri** | - Audited actual incurred costs via AWS Budgets / Cost Explorer; performed final technical verification.<br>- Polished GitHub codebase documentation and finalized the internship report. | 06/11/2026 | 06/11/2026 | [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) |

### 3. Week 12 Achievements:
* Delivered complete 7-part Workshop guide (5.1 through 5.7) with step-by-step technical instructions and actual screenshots.
* Validated `cleanup.sh` script and runbook, guaranteeing complete environment teardown within 3 minutes to avoid idle costs.
* Confirmed $0.00 USD actual incurred spend on AWS Cost Explorer, validating the serverless Scale-to-Zero model.
* Packaged complete source code, tests, and documentation on GitHub, successfully completing the FCAJ internship.

### 4. References:
* [AWS CloudFormation Stack Deletion](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html)
* [AWS Well-Architected Operational Excellence Pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/)
* [AWS Pricing Calculator - EduMedia AI Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
