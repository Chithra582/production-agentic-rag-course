---
name: "grounded-context-synthesis"
description: "Generates factual responses strictly grounded in retrieved evidence with citation attribution."
license: MIT
---

# Grounded Context Synthesis Skill

## Overview
Synthesizes comprehensive answers using LangGraph state graphs, enforcing strict citation binding to source document chunks.

## Operational Workflow
1. Ingest ranked context chunks via `rag-context-synthesizer`.
2. Structure reasoning prompt with citation directives.
3. Generate answer streaming tokens with verified inline citations.
4. Verify factual grounding score before returning turn.
