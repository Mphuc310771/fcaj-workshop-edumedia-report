---
title: "Week 5"
date: 2026-09-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Week 5: System Architecture & Proposal Finalization
*Internship Period: 14/09/2026 – 18/09/2026*

### 1. Week 5 Objectives:
* Complete comprehensive AgentSys system architecture diagram in Draw.io using official AWS Architecture Icons 2026.
* Author 7-section Project Proposal document adhering to FCAJ guidelines.
* Build an empirical FinOps cost model utilizing official AWS Pricing Calculator (calculator.aws).
* Evaluate architectural design against 6 pillars of AWS Well-Architected Framework.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Modeled AgentSys system architecture in Draw.io.<br>- Applied official AWS Architecture Icons 2026 to standardize end-to-end data flows across 5 layers. | 14/09/2026 | 14/09/2026 | [AWS Architecture Icons](https://aws.amazon.com/architecture/icons/) |
| **Tue** | - Drafted Problem Statement, objectives, and scope.<br>- Defined clear acceptance criteria across each integrated AWS service. | 15/09/2026 | 15/09/2026 | FCAJ Proposal Guidelines |
| **Wed** | - Calculated monthly operational expenses on AWS Pricing Calculator in us-east-1.<br>- Validated that the workload scales to 0 USD at idle. | 16/09/2026 | 16/09/2026 | [AWS Pricing Calculator - AgentSys Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934) |
| **Thu** | - Authored technical analysis across 6 Well-Architected pillars: Security, Reliability, Performance, Cost, Operations, Sustainability.<br>- Documented IAM least privilege and S3 SSE-S3 configurations. | 17/09/2026 | 17/09/2026 | [AWS Well-Architected](https://aws.amazon.com/architecture/well-architected/) |
| **Fri** | - Finalized bilingual Proposal documentation and committed files to report repository at content/2-Proposal/.<br>- Reviewed technical specifications ahead of codebase implementation. | 18/09/2026 | 18/09/2026 | Proposal Evaluation Guidelines |

### 3. Week 5 Achievements:
* Completed detailed AgentSys architecture diagram in Draw.io spanning 5 layers: Upload, Ingestion, AI Extraction, Storage & Querying.
* Authored bilingual Project Proposal published in content/2-Proposal/ fulfilling all 8 required sections.
* Established AWS Pricing Calculator estimate: processing 1 image costs ~0.00207 USD and 1 minute of audio costs ~0.02435 USD; the Pay-as-you-go model ensures costs scale proportionally to actual usage and drop to zero when idle.
* Conducted structured 6-pillar Well-Architected evaluation: securing storage via S3 Block Public Access, smoothing bursts with SQS load-leveling, and compressing vectors to 256-d.
* Finalized architectural blueprints ready for immediate pipeline coding in the following week.

### 4. References:
* [AWS Architecture Center & Official Icons](https://aws.amazon.com/architecture/icons/)
* [AWS Well-Architected Framework Whitepapers](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
* [AWS Pricing Calculator - AgentSys Estimate](https://calculator.aws/#/estimate?id=cd02bfb0db81283141c38707032c1bfdbc20a934)
