---
name: "document-ingestion-chunking"
description: "Converts raw PDFs, Markdown, and technical docs into structured chunk atoms with preserved table hierarchies."
license: MIT
---

# Document Ingestion & Chunking Skill

## Overview
Utilizes Docling to ingest multi-format enterprise documentation, preserving table layouts, document headers, and formula representations.

## Operational Workflow
1. Ingest document via `docling-chunker`.
2. Extract semantic section boundaries and table markdown tables.
3. Compute dense vector embeddings with SentenceTransformers.
4. Dispatch structured chunks and metadata into OpenSearch.
