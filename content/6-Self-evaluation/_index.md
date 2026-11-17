---
title: "Self-Evaluation"
date: 2026-11-05
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# 6. Self-Evaluation

**Student:** Do Tran Minh Phuc  
**University:** VNU-HCM University of Science (HCMUS)  
**Program:** First Cloud AI Journey (FCAJ) — AWS Vietnam  
**Project:** EduMedia AI - Smart Lecture Storage & Search Engine on AWS

---

## 1. Detailed Self-Evaluation Across 8 Required Criteria

| Evaluation Criteria | Rating | Specific Evaluation & Concrete Evidence |
| :--- | :---: | :--- |
| **1. Technical Knowledge** | **Good** | Mastered the AWS Serverless & GenAI stack: S3, SQS Buffer, Lambda, Rekognition, Transcribe, DynamoDB, Bedrock, and CloudWatch. |
| **2. Self-Learning & Research** | **Good** | Rapidly absorbed new technologies from official AWS documentation; mastered AWS SAM, vector embeddings, and FinOps cost optimization principles. |
| **3. Proactiveness & Initiative** | **Good** | Built automated unit test suites in Pytest (143 tests), established strict IAM Least-Privilege, constructed automated cleanup runbooks, and configured Dead-Letter Queues (DLQ). |
| **4. Discipline & Compliance** | **Good** | Strictly adhered to program regulations, completed scheduled on-site sessions at AWS 26th Floor Bitexco Office, and submitted reports on time (<mark style="background-color: #fef08a; color: #854d0e; padding: 2px 6px; border-radius: 4px; font-weight: 600; border: 1px solid #fde047;">1 sick day on 27/08/2026 documented with medical prescription in <a href="../../1-worklog/1.2-week2/#4-medical-excuse-documentation-27082026" style="text-decoration: underline; font-weight: bold; color: #854d0e;">Week 2</a></mark>). |
| **5. Communication Skills** | **Good** | Authored full bilingual technical documentation (EN & VI), communicated architectural solutions clearly, and actively engaged in knowledge sharing and peer reviews with internship colleagues. |
| **6. Teamwork** | **Good** | Collaborated effectively with internship peers on cross-reviewing technical blogs, discussing architectural patterns, and sharing cloud learnings. |
| **7. Problem Solving** | **Good** | Eliminated S3-Transcribe recursive loops via `uploads/` vs `transcripts/` segregation; optimized Titan vectors from 1536 to 256 dimensions; automated S3-offloading for transcripts > 250KB preventing DynamoDB 400KB item overflow. |
| **8. Contribution to Workshop** | **Good** | Authored complete 7-part hands-on workshop guide; engineered production guardrails (protected file deletion lock HTTP 403, upload cap ≤ 25MB) and built an interactive Web UI with Click-to-Seek Media Player. |

---

## 2. Technical Challenges Overcome

1. **Handling Asynchronous Transcribe Latency**: Direct synchronous polling in Lambda risked 120s timeouts on large audio files. Resolved by architecting an asynchronous state machine (INDEXING → READY) backed by SQS buffering.
2. **Optimizing Vector Storage Costs**: Persisting 1536-dimensional embeddings incurred higher DynamoDB table footprints. Implemented **Titan Vector Subsampling (256-d)**, cutting storage costs by 83% while maintaining >98% accuracy.
3. **Data Protection Guardrails & DynamoDB 400KB Item Guard**: Proactively engineered guardrails (403 forbidden on protected file deletion, 25MB upload cap) and automated S3-offloading for transcripts exceeding 250KB, eliminating DynamoDB ValidationException risks.

---

## 3. Future Roadmap

* Implement automated CI/CD deployment pipelines using **AWS CodePipeline / GitHub Actions**.
* Deploy the React Web UI globally using **Amazon CloudFront + S3 Static Hosting** secured with **Amazon Cognito** user authentication.
