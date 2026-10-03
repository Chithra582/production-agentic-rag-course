---
name: "rag-telemetry-evaluation"
description: "Tracks end-to-end RAG latency, token expenditure, and hallucination scores via Langfuse."
license: MIT
---

# RAG Telemetry & Evaluation Skill

## Overview
Provides continuous operational evaluation of retrieval quality, generation faithfulness, and API latency.

## Operational Workflow
1. Register session trace using `langfuse-evaluator`.
2. Score turn faithfulness and retrieval precision.
3. Stream telemetry dashboards to administrative monitoring consoles.
