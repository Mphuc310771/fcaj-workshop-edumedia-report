---
title: "Self-Evaluation"
date: 2026-11-05
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# 6. Self-Assessment

### Internship Information

| Field | Details |
| :--- | :--- |
| **Student** | Do Tran Minh Phuc (Student ID: `23120156` — VNU-HCM University of Science) |
| **Organization** | Amazon Web Services Vietnam Co., Ltd. |
| **Program** | Workforce Bootcamp — First Cloud AI Journey (FCAJ) |
| **Duration** | August 17, 2026 – December 01, 2026 |
| **Project** | EduMedia AI — Smart Lecture Storage & Search Engine on AWS |
| **Project Submission Deadline** | November 30, 2026 |
| **Mentor** | Lữ Hoàn Thiện — Program Manager |
| **Mentor Contact** | `thienlh@awsfcaj.com` · `0986 479 292` |

---

## 1. Detailed Self-Evaluation (12 Criteria)

During my internship at **Amazon Web Services Vietnam Co., Ltd.** from **August 17, 2026** to **December 01, 2026**, I had the opportunity to apply university computer science foundations to a real-world cloud engineering environment. By architecting and deploying **EduMedia AI** end-to-end on AWS Serverless and Generative AI services, I strengthened my technical depth in IaC (AWS SAM), event-driven pipelines, FinOps cost modeling, automated testing, and technical documentation.

To objectively reflect on my internship period, I evaluate myself across the following 12 criteria:

| No. | Criteria | Evidence & Self-Reflection | Good | Fair | Average |
| :---: | :--- | :--- | :---: | :---: | :---: |
| 1 | **Professional knowledge & skills** | Mastered the AWS Serverless & GenAI stack: S3, SQS + 3 DLQs, Lambda, EventBridge, Rekognition, Transcribe, DynamoDB GSI/Streams, Bedrock, CloudFront, CloudWatch, and X-Ray. | ✅ | ☐ | ☐ |
| 2 | **Ability to learn** | Rapidly absorbed new cloud patterns from official AWS documentation; mastered AWS SAM, 256-d Titan vector embeddings, and FinOps cost optimization principles. | ✅ | ☐ | ☐ |
| 3 | **Proactiveness** | Proactively built a 143-test Pytest suite (84% coverage), enforced IAM Least-Privilege across 3 Lambda roles, created automated cleanup runbooks, and audited 0 HRIs in AWS Well-Architected Tool. | ✅ | ☐ | ☐ |
| 4 | **Sense of responsibility** | Completed all 12 weekly worklogs, live production deployment, bilingual 7-part workshop, and FCAJ Submission package on schedule with high engineering rigor. | ✅ | ☐ | ☐ |
| 5 | **Discipline** | Strictly adhered to program regulations and Bitexco office sessions (<mark style="background-color: #fef08a; color: #854d0e; padding: 2px 6px; border-radius: 4px; font-weight: 600; border: 1px solid #fde047;">1 sick day on 27/08/2026 documented with medical prescription in <a href="../1-worklog/1.2-week2/#4-medical-excuse-documentation-27082026" style="text-decoration: underline; font-weight: bold; color: #854d0e;">Week 2</a></mark>). | ☐ | ✅ | ☐ |
| 6 | **Progressive mindset** | Actively incorporated review feedback on architecture consistency, FinOps reconciliation, and security hardening (S3-offloading guard, NoEcho API key). | ✅ | ☐ | ☐ |
| 7 | **Communication** | Authored full bilingual documentation (EN & VI) and 3 technical blogs; still improving concise verbal presentation of complex distributed-system trade-offs. | ☐ | ✅ | ☐ |
| 8 | **Teamwork** | Collaborated effectively within Group 3P on shared Proposal, Workshop, and peer-reviewed technical blogs, and exchanged cloud learnings with fellow FCAJ interns. | ✅ | ☐ | ☐ |
| 9 | **Professional conduct** | Maintained respectful, courteous conduct toward mentors, program administrators, fellow interns, and office property at Bitexco Floor 26. | ✅ | ☐ | ☐ |
| 10 | **Problem-solving skills** | Eliminated S3-Transcribe recursive loops via `uploads/` vs `transcripts/` segregation; compressed vectors to 256-d; automated S3-offloading (>250KB) to prevent DynamoDB 400KB overflow. | ✅ | ☐ | ☐ |
| 11 | **Contribution to project/team** | Delivered live CloudFront SPA, FastAPI/Lambda backend, 7 CloudWatch Alarms + SNS, 7-part hands-on workshop guide, and production data-protection guardrails (HTTP 403 lock, ≤25MB cap). | ✅ | ☐ | ☐ |
| 12 | **Overall** | Achieved all technical, Well-Architected, and FinOps objectives of the FCAJ Workforce Bootcamp with measurable production deliverables. | ✅ | ☐ | ☐ |

---

## 2. Needs Improvement

* **Earlier Load Testing & Concurrency Benchmarking:** Conduct synthetic stress testing earlier in the development cycle to measure P95/P99 latency under high concurrent upload and search bursts.
* **Enterprise-Scale Vector & Identity Architecture:** Deepen hands-on experience with multi-tenant **Amazon Cognito** federation and managed ANN vector databases (**Amazon OpenSearch Serverless** / **Aurora pgvector HNSW**) for catalogs exceeding tens of thousands of items.
* **Schedule & Health Balance:** Better pace university coursework alongside on-site office schedules to avoid last-minute deadline pressure and maintain consistent energy throughout long engineering sprints.

---

## 3. Technical Challenges Overcome

1. **Handling Asynchronous Transcribe Latency:** Direct synchronous polling in Lambda risked 120s timeouts on large audio files. Resolved by architecting an asynchronous state machine (`INDEXING` → `READY`) backed by SQS buffering and EventBridge callbacks.
2. **Optimizing Vector Storage Costs:** Persisting high-dimensional embeddings (1024–1536 dimensions) significantly increases DynamoDB item size. Adopted **Amazon Titan Text Embeddings V2 (`256-d native` with L2 normalization)**, cutting vector storage by **75%** versus default 1024-d (and **83%** versus Titan V1 1536-d) while preserving semantic retrieval quality.
3. **Data Protection Guardrails & DynamoDB 400KB Item Guard:** Proactively engineered guardrails (HTTP 403 on protected file deletion, 25MB upload cap) and automated S3-offloading for transcripts exceeding 250KB, eliminating DynamoDB `ValidationException` risks.

---

## 4. Future Roadmap

* Implement automated CD deployment pipelines using **AWS CodePipeline / GitHub Actions** with staged Dev/Prod SAM environments.
* Integrate multi-user **Amazon Cognito** authentication onto the CloudFront + S3 React Web UI and evaluate **Amazon OpenSearch Serverless / Aurora pgvector (HNSW Index)** as catalog scale grows past tens of thousands of media items.
