---
title: "Blogs Posted"
date: 2026-11-06
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Throughout the First Cloud AI Journey (FCAJ) 2026 internship and the engineering of the EduMedia AI platform, comprehensive technical articles were authored and published to the [AWS Study Group Community (Facebook)](https://www.facebook.com/groups/awsstudygroupfcj):

---

### [Blog 1 - Serverless Media Processing: Why I Avoid Direct S3 Lambda Triggers](3.1-blog1/)
* **Publication Status:** Approved & Published — [View Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2328280037936955/)  
Deconstructs the three critical failure modes of push-based S3 direct Lambda triggers: internal unmonitored queues, concurrency exhaustion cascading downstream, and silent event drops. Details the pull-based SQS + DLQ Claim Check architecture with `ScalingConfig: MaximumConcurrency = 5` and `ReportBatchItemFailures`.

---

### [Blog 2 - DynamoDB Query Optimization: Limitations of FilterExpression and the Global Secondary Index Solution](3.2-blog2/)
* **Publication Status:** Submitted on AWS Study Group VN *(Pending Admin Approval — screenshot proof attached inside page)*  
Provides an in-depth mechanical teardown of DynamoDB storage layers: explaining why `Scan` with `FilterExpression` incurs 100% RCU disk consumption prior to RAM filtering and causes Empty Page Pagination on AWS Lambda. Demonstrates an O(1) GSI partition routing design (`category-index`) reducing RCUs by 99.98% and latencies to under 15ms.

---

### [Blog 3 - Optimizing Docker Build Time on AWS CodeBuild with Amazon ECR Remote Cache](3.3-blog3/)
* **Publication Status:** Submitted on AWS Study Group VN *(Pending Admin Approval — screenshot proof attached inside page)*  
Analyzes why AWS CodeBuild's default ephemeral `LOCAL_DOCKER_LAYER_CACHE` triggers full cold rebuilds (~10 minutes) when CI/CD runs occur hours or days apart. Demonstrates configuring Docker Buildx with a `containerd` image store during the `install` phase and enforcing `--cache-from` / `--cache-to` (`image-manifest=true`) against **Amazon ECR**—establishing a persistent multi-layer cache hierarchy that lowers incremental rebuild time to ~6 minutes and cuts build duration by up to 25% on multi-stage pipelines.