---
title: "Week 9"
date: 2026-10-16
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

# Week 9: Amazon Bedrock & Hybrid Search Integration
*Internship Period: 12/10/2026 – 16/10/2026*

### 1. Week 9 Objectives:
* Integrate Amazon Bedrock SDK into API Lambda layer connecting foundation LLMs and embedding models.
* Employ Amazon Nova Lite (amazon.nova-lite-v1:0) as an AI Query Planner converting Vietnamese natural queries into structured search plans.
* Neutralize json.loads runtime parsing exceptions triggered by markdown code fencing blocks.
* Integrate Amazon Titan Text Embeddings V2 (amazon.titan-embed-text-v2:0) configured with native 256-d subsampling and L2 normalization.
* Implement a Hybrid Search algorithm combining Vector Cosine Similarity (weight 0.6) and Normalized Keyword Overlap (weight 0.4).

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Configured Boto3 Bedrock Runtime client in apps/api/search.py.<br>- Validated bedrock:InvokeModel IAM permissions. | 12/10/2026 | 12/10/2026 | [Bedrock Runtime API](https://docs.aws.amazon.com/bedrock/latest/userguide/api-methods-run.html) |
| **Tue** | - Engineered prompts for Amazon Nova Lite.<br>- Extracted categories, lexical tokens, and semantic intents from Vietnamese natural queries. | 13/10/2026 | 13/10/2026 | [Amazon Nova Lite Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-nova.html) |
| **Wed** | - Handled JSON parsing exceptions from markdown code blocks.<br>- Developed regex pre-sanitizer clean_llm_json_output prior to deserialization. | 14/10/2026 | 14/10/2026 | Python re Module |
| **Thu** | - Invoked Amazon Titan Text Embeddings V2 passing dimensions: 256 and normalize: True.<br>- Generated normalized 256-d query embeddings. | 15/10/2026 | 15/10/2026 | [Titan Embeddings V2 Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html) |
| **Fri** | - Implemented weighted Hybrid Search scoring combining Vector Cosine Similarity and Lexical Overlap.<br>- Benchmarked ranking precision across complex multi-format queries. | 16/10/2026 | 16/10/2026 | Vector Similarity Algorithms |

### 3. Week 9 Achievements:
* Verified Nova Lite query planning across conversational Vietnamese queries.
* Eliminated JSON parsing runtime exceptions via resilient regex sanitization.
* Integrated Titan Text Embeddings V2 producing normalized 256-d vectors, reducing storage footprint by 83% while preserving high recall.
* Verified Hybrid Search ranking precision against multimodal academic assets.

### 4. References:
* [Amazon Bedrock Converse API Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html)
* [Amazon Titan Text Embeddings V2 Technical Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html)
* [Hybrid Search Architecture Patterns on AWS](https://aws.amazon.com/blogs/machine-learning/)
