---
title: "Week 10"
date: 2026-10-23
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

# Week 10: Web UI Engineering & Client Integration
*Internship Period: 19/10/2026 – 23/10/2026*

### 1. Week 10 Objectives:
* Bootstrap and design a modern Web Gallery interface (React, Tailwind CSS, Vite) directly integrated with AgentSys REST backend APIs.
* Implement an interactive Media Player with Click-to-Seek capability synchronized to timestamp evidence extracted from lecture transcripts.
* Establish system guardrails and input validation: enforce deletion protection on system media (HTTP 403), upload size restriction (<= 25MB), and API Gateway CORS policy.
* Integrate category chips and subject filtering calling the semantic query endpoint `POST /api/v1/media/agent/query`.

### 2. Daily Tasks & Schedule:
| Day | Detailed Tasks | Start Date | Completion Date | Reference Documentation |
|:---:|---|:---:|:---:|---|
| **Mon** | - Bootstrapped frontend project using Vite, React, and Tailwind CSS.<br>- Designed responsive grid layout displaying multimodal media cards. | 19/10/2026 | 19/10/2026 | [Vite & React Guide](https://vitejs.dev/) |
| **Tue** | - Built API Gateway client integration connecting GET /api/v1/media.<br>- Rendered image thumbnails, label badges, and OCR text extracted by Rekognition. | 20/10/2026 | 20/10/2026 | [Amazon API Gateway Guide](https://docs.aws.amazon.com/apigateway/) |
| **Wed** | - Developed Click-to-Seek interactive Media Player.<br>- Clicking transcript segments jumps audio and video playback directly to exact speech timestamps. | 21/10/2026 | 21/10/2026 | [HTML5 Media Audio and Video API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement) |
| **Thu** | - Implemented client-side and API guardrails.<br>- Enforced system asset deletion protection (HTTP 403), 25MB upload file size caps, and API Gateway CORS FrontendOriginUrl configuration. | 22/10/2026 | 22/10/2026 | [API Gateway CORS Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html) |
| **Fri** | - Conducted local client testing and component render optimization.<br>- Connected UI search bar to Bedrock Agent Query API rendering structured Search Plans. | 23/10/2026 | 23/10/2026 | [React Hooks Documentation](https://react.dev/) |

### 3. Week 10 Achievements:
* Delivered complete Web Gallery frontend seamlessly connected to AgentSys Serverless APIs.
* Click-to-Seek player operates accurately across audio and video lectures, enhancing study material retrieval.
* Input validation and resource guardrails safeguard system integrity against accidental deletion and oversized payloads.
* Search interface clearly visualizes query classifications, subjects, and timestamped evidence citations.

### 4. References:
* [Vite Getting Started Guide](https://vitejs.dev/guide/)
* [Tailwind CSS Documentation](https://tailwindcss.com/docs)
* [Amazon API Gateway Enabling CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)
* [HTML5 Media Audio and Video DOM Reference](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
