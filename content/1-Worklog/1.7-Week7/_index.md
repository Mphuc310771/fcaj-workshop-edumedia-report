---
title: "Week 7"
date: 2026-10-02
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Week 7: Rekognition OCR & Asynchronous Transcribe Integration
*Internship Period: 28/09/2026 – 02/10/2026*

### 1. Week 7 Objectives:
* Integrate Amazon Rekognition (DetectLabels, DetectText) into Ingestion Lambda worker to automate image classification and OCR.
* Architect an asynchronous state-decoupled pipeline for Amazon Transcribe: dispatching jobs and updating database status to INDEXING.
* Configure an Amazon EventBridge Rule capturing Transcribe Job State Change (COMPLETED) events to trigger finalization workers.
* Author a timestamp-preserving chunking algorithm segmenting lecture transcripts into seekable intervals (start_time, end_time).

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Integrated Rekognition OCR and label extraction into apps/workers/ingestion.py.<br>- Applied minimum confidence cutoff of 65.0 to discard visual artifacts. | 28/09/2026 | 28/09/2026 | [Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/) |
| **Tue** | - Implemented State Separation for audio media.<br>- Worker initiates start_transcription_job and sets DynamoDB item status to INDEXING. | 29/09/2026 | 29/09/2026 | [Transcribe Job Status](https://docs.aws.amazon.com/transcribe/latest/dg/how-it-works.html) |
| **Wed** | - Configured EventBridge Rule in SAM template.<br>- Listened for Transcribe Job State Change with status COMPLETED routed to Ingestion Lambda. | 30/09/2026 | 30/09/2026 | [EventBridge with Transcribe](https://docs.aws.amazon.com/transcribe/latest/dg/monitoring-events.html) |
| **Thu** | - Implemented transcript segmenting into 30–60 second intervals.<br>- Annotated chunks with start_time and end_time to enable media player playback synchronization. | 01/10/2026 | 01/10/2026 | Python Text Chunking Patterns |
| **Fri** | - Executed end-to-end integration tests on a 10-minute lecture audio recording.<br>- Validated automated INDEXING to READY transitions upon EventBridge completion events. | 02/10/2026 | 02/10/2026 | [AWS EventBridge Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) |

### 3. Week 7 Achievements:
* Fully automated image labeling and Vietnamese text OCR pipeline for all image uploads, synthesizing unified search text.
* Established an asynchronous EventBridge architecture for Transcribe: worker exits after ~1.2s compute duration, reducing Lambda execution costs by ~95%.
* Timestamped chunking logic verified across 10–30 minute lecture recordings, supporting precise playback seeking.
* Verified seamless state transitions from INDEXING to READY upon EventBridge COMPLETED signals.

### 4. References:
* [Amazon EventBridge Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)
* [Monitoring Transcribe with Amazon EventBridge](https://docs.aws.amazon.com/transcribe/latest/dg/monitoring-events.html)
* [Amazon Rekognition Text Detection Guide](https://docs.aws.amazon.com/rekognition/latest/dg/text-detection.html)
