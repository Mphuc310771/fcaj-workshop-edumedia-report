---
title: "Week 8"
date: 2026-10-09
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Week 8: Metadata Database & DynamoDB GSI Optimization
*Internship Period: 05/10/2026 – 09/10/2026*

### 1. Week 8 Objectives:
* Design a NoSQL catalog schema on Amazon DynamoDB (AgentSysMediaMetadata) with a single primary partition key file_id.
* Analyze and eliminate high-latency, costly Scan + FilterExpression operations when filtering media items by category.
* Implement a Global Secondary Index (category-index) transitioning operational complexity from O(N) full-table scans to O(1) partition queries.
* Construct an automated S3-Offloading pattern routing transcripts > 250KB to S3, neutralizing DynamoDB's 400KB item size limit.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Provisioned AgentSysMediaMetadata on SAM.<br>- Configured Partition Key file_id (String) under On-Demand (Pay-Per-Request) billing. | 05/10/2026 | 05/10/2026 | [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/) |
| **Tue** | - Benchmarked category-based filtering APIs.<br>- Observed Scan + FilterExpression consuming unneeded RCUs across entire table partitions and causing Empty Pages. | 06/10/2026 | 06/10/2026 | [DynamoDB Scan vs Query](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html) |
| **Wed** | - Added Global Secondary Index (GSI) category-index (PK: category, SK: created_at, Projection: INCLUDE) to infra/template.yaml. | 07/10/2026 | 07/10/2026 | [DynamoDB GSI Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html) |
| **Thu** | - Built S3-Offloading logic in worker.<br>- Automatically redirected raw transcripts exceeding 250KB to s3://MediaBucket/transcripts/raw/{file_id}.json. | 08/10/2026 | 08/10/2026 | [DynamoDB Item Sizes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html#limits-items) |
| **Fri** | - Measured performance metrics.<br>- Confirmed GSI reduced RCU consumption by 99.98% and reduced latency from seconds to under 30ms. | 09/10/2026 | 09/10/2026 | [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html) |

### 3. Week 8 Achievements:
* Established AgentSysMediaMetadata table following NoSQL architectural best practices.
* Replaced full-table scans with GSI partition queries, decreasing read RCU costs by 99.98%.
* Eliminated Empty Page Pagination loops, reducing category query latencies to sub-30ms.
* Validated S3-Offloading logic, ensuring item sizes never breach the 400KB limit.

### 4. References:
* [DynamoDB Secondary Indexes Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html)
* [DynamoDB Best Practices for Querying and Scanning](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-query-scan.html)
* [Handling Large Items and Attributes in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-use-s3-too.html)
