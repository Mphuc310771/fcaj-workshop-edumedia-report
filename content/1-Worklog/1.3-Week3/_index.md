---
title: "Week 3"
date: 2026-09-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Week 3: Asynchronous Processing with SQS & Lambda
*Internship Period: 31/08/2026 – 04/09/2026*

### 1. Week 3 Objectives:
* Develop a Python 3.12 Lambda prototype to process S3 object creation events.
* Evaluate concurrency exhaustion risks under the direct S3-to-Lambda push invocation pattern.
* Integrate Amazon SQS for load leveling, enforcing MaximumConcurrency and partial batch processing via ReportBatchItemFailures.
* Configure an SQS Dead-Letter Queue (DLQ) and resolve visibility timeout mismatches.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Built a Python 3.12 Lambda prototype consuming s3:ObjectCreated:* events.<br>- Verified payload delivery logs on CloudWatch Logs. | 31/08/2026 | 31/08/2026 | [AWS Lambda with Python](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) |
| **Tue** | - Executed a burst upload test simulating 50 concurrent file uploads.<br>- Analyzed rapid unthrottled Lambda scaling impacting regional account concurrency limits. | 01/09/2026 | 01/09/2026 | [Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html) |
| **Wed** | - Placed Amazon SQS between S3 and Lambda to regulate data flow.<br>- Enforced MaximumConcurrency = 5 on Event Source Mapping to cap worker scaling. | 02/09/2026 | 02/09/2026 | [SQS as Event Source](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html) |
| **Thu** | - Enabled ReportBatchItemFailures for granular retry handling.<br>- Configured an SQS Dead-Letter Queue (DLQ) with maxReceiveCount = 3. | 03/09/2026 | 03/09/2026 | [Batch Item Failures](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html#services-sqs-batchfailurereporting) |
| **Fri** | - Identified duplicate processing caused by visibility timeouts shorter than Lambda execution duration.<br>- Increased SQS Visibility Timeout to 360 seconds (6x Lambda duration). | 04/09/2026 | 04/09/2026 | [SQS Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) |

### 3. Week 3 Achievements:
* Successfully transitioned ingestion from a push model to a decoupled pull architecture via SQS, safely absorbing 50-file bursts without concurrency spikes.
* Capped ingestion concurrency to a maximum ceiling of 5 concurrent workers via MaximumConcurrency, protecting downstream databases from throttling.
* Verified partial batch failure reporting via ReportBatchItemFailures, preventing redundant retries of already successful items.
* Completely eliminated duplicate processing: identified that a 30s default visibility timeout expired during 60s Lambda executions, re-queueing in-flight messages; raising visibility timeout to 360s permanently resolved the race condition.
* Secured pipeline with an SQS Dead-Letter Queue providing 14-day retention for poisoned messages.

### 4. References:
* [Using AWS Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
* [Amazon SQS Visibility Timeout Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
* [AWS Lambda Concurrency Management](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
