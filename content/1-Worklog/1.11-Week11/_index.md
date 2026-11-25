---
title: "Week 11"
date: 2026-10-30
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

# Week 11: Automated Testing Suite, CloudWatch Alarms & Technical Blogs
*Internship Period: 26/10/2026 – 30/10/2026*

### 1. Week 11 Objectives:
* Construct an automated unit test suite in Pytest mocking AWS SDK operations using moto and unittest.mock, achieving 143 passing tests with 83.69% codebase coverage.
* Execute a burst stress benchmark simulating 50 concurrent uploads to validate load leveling resilience and SQS buffer queuing.
* Configure 7 CloudWatch Alarms linked to SNS Topic agentsys-alerts-topic satisfying production operational criteria.
* Author 2 personal technical blogs (S3-SQS-Lambda, DynamoDB GSI) and coordinate Blog 3 (AWS CodeBuild + Amazon ECR Remote Cache, authored by Nguyen Thanh Phuc) for submission to the AWS Study Group community.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Configured tests/ directory structure.<br>- Authored unit tests for SQS consumption and format routing mocking S3 and DynamoDB via moto. | 26/10/2026 | 26/10/2026 | [Moto Library](http://docs.getmoto.org/) |
| **Tue** | - Expanded test suite across edge scenarios: corrupt payloads, Bedrock latency timeouts, and S3-offloading.<br>- Reached 143 passing tests (83.69% coverage). | 27/10/2026 | 27/10/2026 | [Pytest-cov Guide](https://pytest-cov.readthedocs.io/) |
| **Wed** | - Authored 7 CloudWatch Alarms in SAM template and enabled SNS alert topic.<br>- Emitted custom metrics (MediaIngested, MediaIngestFailed) and executed 50-file burst stress benchmark. | 28/10/2026 | 28/10/2026 | [CloudWatch Alarms Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) |
| **Thu** | - Authored Blog 1: Analyzing S3 to SQS to Lambda buffering pattern (Claim Check, DLQ).<br>- Authored Blog 2: DynamoDB Query Optimization: Why replace Scan with GSI category-index?<br>- Reviewed & packaged Blog 3 (authored by Nguyen Thanh Phuc): Optimizing Docker Build Time on AWS CodeBuild via Amazon ECR Remote Cache (`containerd` + Buildx). | 29/10/2026 | 29/10/2026 | [Section 3 - Blogs Posted](../../3-blogsposted/) |
| **Fri** | - Finalized article formatting and submitted 3 technical blogs to the AWS Study Group community (Blog 1 approved publicly; Blogs 2 & 3 documented with pending approval screenshots).<br>- Engaged in discussions and reviewed feedback from community peers. | 30/10/2026 | 30/10/2026 | [Blog 1 Post](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2328280037936955/), [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) |

### 3. Week 11 Achievements:
* Delivered complete automated test suite comprising 143 unit tests achieving 83.69% coverage, guaranteeing zero regressions across the codebase.
* Provisioned 7 CloudWatch Alarms and SNS alert routing on AWS for real-time fault detection and DLQ monitoring.
* Validated system elasticity under a 50-file burst test confirming zero data loss and smooth queue drainage.
* Authored 2 personal technical blogs (Blog 1 published; Blog 2 pending approval) and coordinated Blog 3 submission (pending approval) on the AWS Study Group community.

### 4. References:
* [Testing AWS Lambda with Moto](http://docs.getmoto.org/en/latest/)
* [Amazon CloudWatch Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
* [AWS Study Group Facebook Community](https://www.facebook.com/groups/awsstudygroupfcj)
* [Amazon DynamoDB Core Components](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)
