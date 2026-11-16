---
title: "Week 4"
date: 2026-09-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Week 4: Amazon Rekognition & Transcribe Exploration
*Internship Period: 07/09/2026 – 11/09/2026*

### 1. Week 4 Objectives:
* Explore Amazon Rekognition Computer Vision APIs (DetectLabels, DetectText) across lecture slides and handwritten whiteboard captures.
* Evaluate Vietnamese speech-to-text accuracy (vi-VN) using Amazon Transcribe via StartTranscriptionJob.
* Benchmark processing latencies and payload footprints across media formats to furnish empirical FinOps data.
* Overcome payload size limits when invoking Amazon Rekognition directly.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Scripted Boto3 calls testing Amazon Rekognition DetectLabels API.<br>- Evaluated object classification accuracy across 20 IT lecture slides. | 07/09/2026 | 07/09/2026 | [Rekognition Labels](https://docs.aws.amazon.com/rekognition/latest/dg/labels.html) |
| **Tue** | - Experimented with DetectText for Vietnamese and whiteboard handwriting OCR.<br>- Calibrated confidence thresholds to filter out optical noise. | 08/09/2026 | 08/09/2026 | [Rekognition Text](https://docs.aws.amazon.com/rekognition/latest/dg/text-detection.html) |
| **Wed** | - Tested Amazon Transcribe StartTranscriptionJob with language code vi-VN.<br>- Transcribed 5 audio lectures (3–5 mins each) and benchmarked turnaround times. | 09/09/2026 | 09/09/2026 | [Amazon Transcribe](https://docs.aws.amazon.com/transcribe/latest/dg/) |
| **Thu** | - Analyzed payload limits: identified Rekognition rejecting images > 5MB when passed via Bytes.<br>- Refactored calls to pass S3Object references, expanding ceiling to 15MB. | 10/09/2026 | 10/09/2026 | [Rekognition Limits](https://docs.aws.amazon.com/rekognition/latest/dg/limits.html) |
| **Fri** | - Synthesized benchmark data covering processing latency and payload sizing across media types.<br>- Formulated empirical input metrics for FinOps modeling. | 11/09/2026 | 11/09/2026 | [AWS Architecture Center](https://aws.amazon.com/architecture/) |

### 3. Week 4 Achievements:
* Mastered core Rekognition and Transcribe vi-VN APIs, evaluating extraction accuracy on technical terminology.
* Resolved image payload limit exceptions: identified that Rekognition enforces a strict 5MB limit for direct byte payloads; migrated to S3Object references to safely process high-resolution images up to 15MB.
* Calibrated optimal confidence filters: Confidence >= 65.0 for object labels and >= 70.0 for OCR text, eliminating false-positive optical artifacts.
* Established an empirical latency and cost reference table serving as the foundation for architectural design.

### 4. References:
* [Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html)
* [Amazon Transcribe Developer Guide](https://docs.aws.amazon.com/transcribe/latest/dg/what-is-transcribe.html)
* [Amazon Rekognition Service Quotas](https://docs.aws.amazon.com/general/latest/gr/rekognition.html)
