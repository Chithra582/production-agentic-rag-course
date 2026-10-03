---
name: "hybrid-vector-lexical-retrieval"
description: "Coordinates hybrid OpenSearch queries combining vector similarity and BM25 lexical ranking."
license: MIT
---

# Hybrid Vector & Lexical Retrieval Skill

## Overview
Executes dual-channel retrieval over OpenSearch, balancing semantic intent understanding with exact keyword and identifier matching.

## Operational Workflow
1. Generate query embedding and parse search terms.
2. Execute concurrent vector KNN and BM25 queries via `opensearch-hybrid-retriever`.
3. Compute Reciprocal Rank Fusion (RRF) to merge candidate lists.
4. Filter out items below similarity threshold.
