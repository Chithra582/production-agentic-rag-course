# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Production Agentic RAG Course** (`production-agentic-rag-course`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Production Agentic RAG Course (`production-agentic-rag-course`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Data & Analytics / Production Agentic RAG, Hybrid Retrieval & Observability  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Production Agentic RAG Course operates an intelligent, five-stage deterministic operational pipeline designed to transform enterprise documentation into high-precision, hallucination-free answers. The system orchestrates document ingestion, hybrid vector/BM25 retrieval, reciprocal rank fusion, grounded synthesis, and automated observability evaluation.

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
[ Inbound User Query / Document Ingestion Stream ]
                         │
                         ▼
[Stage 1: Intent Classification & Query Vectorization Gate]
  - Parses query tokens, strips control delimiters, and extracts session ID
  - Dispatches dense vector embedding to SentenceTransformers model
  - Generates sparse BM25 lexical keyword query terms
                         ▼
[Stage 2: Hybrid OpenSearch Retrieval & RRF Ranking Gate]
  - Executes parallel KNN cosine vector similarity search
  - Computes BM25 lexical relevance over chunk document indexes
  - Fuses rankings using Reciprocal Rank Fusion (RRF) formulation
                         ▼
[Stage 3: Relevance Filtering & Context Fitting Gate]
  - Computes composite relevance score S_retrieval
  - Enforces strict token budget ceiling (T_budget <= 4000 tokens)
  - Prunes sub-threshold chunks and structures citation markers
                         ▼
[Stage 4: LangGraph Grounded Synthesis & Citation Gate]
  - Injects ranked context chunks into LangGraph state graph
  - Synthesizes factual answer with mandatory inline source citations
  - Evaluates semantic grounding score G_ground against context
                         ▼
[Stage 5: Langfuse Observability & Telemetry Sealing Gate]
  - Evaluates retrieval precision, hallucination index, and latency
  - Logs end-to-end execution trace to Langfuse monitoring backend
  - Seals turn record and streams verified answer to caller
                         ▼
[ Verified Factual Answer Delivered to User ]
```

### 2. Decision Logic & Routing Formulations

When retrieving candidate document chunks and synthesizing answers, Production Agentic RAG Course evaluates two deterministic mathematical formulations:

1. **Reciprocal Rank Fusion Hybrid Score ($S_{\text{hybrid}}$)**:
   $$S_{\text{hybrid}}(d) = \frac{\alpha}{k + \text{rank}_{\text{dense}}(d)} + \frac{1 - \alpha}{k + \text{rank}_{\text{sparse}}(d)}$$
   Where:
   - $\text{rank}_{\text{dense}}(d)$: Ordinal ranking of chunk $d$ in dense cosine vector KNN search.
   - $\text{rank}_{\text{sparse}}(d)$: Ordinal ranking of chunk $d$ in sparse BM25 lexical search.
   - $k = 60$: Standard smoothing constant preventing ranking bias on top-1 candidates.
   - $\alpha = 0.60$: Dense vector weighting relative to sparse BM25 keyword matching ($1 - \alpha = 0.40$).
   - Retrieval eligibility: Chunks are eligible for context inclusion only if $S_{\text{hybrid}}(d) \ge \tau_{\text{retrieve}} = 0.012$.

2. **Factual Grounding & Hallucination Index ($G_{\text{ground}}$)**:
   $$G_{\text{ground}} = w_e \cdot E_{\text{entailment}} + w_c \cdot C_{\text{citation}} + w_s \cdot (1 - S_{\text{novelty}})$$
   Where:
   - $E_{\text{entailment}} \in [0, 1]$: Natural language inference (NLI) entailment score between generated claims and source chunks.
   - $C_{\text{citation}} \in [0, 1]$: Precision of inline citation tags mapping to valid retrieved chunk IDs.
   - $S_{\text{novelty}} \in [0, 1]$: Unsupported named entity introduction penalty.
   - Parameter weights: $w_e = 0.50$, $w_c = 0.30$, $w_s = 0.20$ ($\sum w_i = 1.0$).
   - Generation approval: Answers are cleared for emission only when $G_{\text{ground}} \ge 0.70$.

### 3. Thresholding & Refusal Decision Criteria

Production Agentic RAG Course enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_INSUFFICIENT_RETRIEVAL_CONTEXT**: Hybrid search results with scores below threshold ($S_{\text{hybrid}} < 0.012$) halt with code `ERR_INSUFFICIENT_RETRIEVAL_CONTEXT`.
- **Refusal on ERR_GROUNDING_CONFIDENCE_LOW**: Synthesized answers with grounding score below cutoff ($G_{\text{ground}} < 0.70$) halt with code `ERR_GROUNDING_CONFIDENCE_LOW`.
- **Refusal on ERR_TOKEN_BUDGET_EXCEEDED**: Query context payloads exceeding turn token limits ($T_{\text{context}} > 4000$) halt with code `ERR_TOKEN_BUDGET_EXCEEDED`.
- **Refusal on ERR_DOCUMENT_PARSING_FAILURE**: Corrupted or password-protected document files trigger refusal with code `ERR_DOCUMENT_PARSING_FAILURE`.
- **Refusal on ERR_OPENSEARCH_CONNECTION_TIMEOUT**: Vector cluster queries exceeding 500ms timeout halt with code `ERR_OPENSEARCH_CONNECTION_TIMEOUT`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Sparse BM25 Fallback**: If SentenceTransformers dense vector inference is unavailable or times out, retrieval gracefully degrades to pure OpenSearch BM25 keyword matching.
- **Safe Out-of-Domain Refusal Fallback**: If retrieved chunks lack sufficient facts to answer the user query, the agent falls back to a transparent refusal statement without fabricating facts.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Ingestion Catalog Review**: Administrators review and approve new document sources before indexing into production OpenSearch namespaces.
- **Hallucination Alert Escalation**: Turns triggering low grounding flags ($G_{\text{ground}} < 0.70$) are dispatched to the Langfuse audit queue for human expert verification.
- **Cluster & Access Control Administration**: Operators manage tenant access control lists (ACL), vector index mappings, and cryptographic API tokens.

---

## The Data It Uses

Production Agentic RAG Course operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Enterprise Source Documents**: Technical documentation, PDF manuals, markdown guides, and tabular CSV spreadsheets.
- **User Query Payloads**: Natural language questions, conversational follow-ups, and session identifier tokens.
- **Telemetry & Traces**: OpenSearch query execution times, retrieval scores, and Langfuse audit traces.

### 2. Configuration & Reference Data

- **Vector Index Schemas**: OpenSearch index mappings, KNN vector dimension definitions (384/768), and BM25 analyzer settings.
- **Docling Chunking Configurations**: Token window boundaries, table extraction rules, and metadata tagging schemas.
- **Tenant Isolation Policies**: Organization IDs, access control lists, and collection visibility filters.

### 3. Base Model & Inference Lineage

- **Embedding Models**: Local `sentence-transformers/all-MiniLM-L6-v2` and `bge-small-en-v1.5` dense encoders.
- **Reasoning Engines**: LangGraph agent workflows orchestrating foundation models via standardized REST adapters.
- **Storage Infrastructure**: OpenSearch 3.0+, PostgreSQL 16 with SQLAlchemy 2.0, and Redis cache controllers.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection in ingested PDFs, cross-tenant data leakage, and API key exposure.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Production Agentic RAG Course is essential for effective deployment.

### 1. Tabular Layout Extraction Complexity
- **Limitation**: Highly nested or borderless financial tables in scanned PDFs can experience chunk boundary fragmentation.
- **Mitigation**: Deploy Docling structural table parsing with explicit HTML table markdown conversion before chunking.

### 2. Semantic Drift Across Domain Vocabularies
- **Limitation**: Dense embeddings trained on general web corpora may misjudge semantic similarity on rare enterprise jargon.
- **Mitigation**: Combine dense vectors with BM25 sparse matching via Reciprocal Rank Fusion to ensure exact term matching.

### 3. Memory Pressure During Bulk Document Ingestion
- **Limitation**: Concurrently indexing thousands of large multi-page PDF documents can overwhelm worker memory.
- **Mitigation**: Enforce Celery/Airflow asynchronous queueing with bounded worker concurrency and batch vector inserts.

### 4. Cold-Start Latency on Local Embeddings
- **Limitation**: Initial model loading of local SentenceTransformers weights can add latency to the first query turn.
- **Mitigation**: Pre-warm model instances in FastAPI application startup hooks and maintain hot in-memory worker pools.

### 5. Multi-Hop Reasoning Across Disjoint Documents
- **Limitation**: Answering complex queries requiring synthesis across disparate documents can fail if initial retrieval top-k is narrow.
- **Mitigation**: Deploy iterative query expansion and multi-step sub-query decomposition in LangGraph workflows.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Tabular Layout Extraction Complexity | Section 1 | Verified |
| - Semantic Drift Across Domain Vocabularies | Section 2 | Verified |
| - Memory Pressure During Bulk Document Ingestion | Section 3 | Verified |
| - Cold-Start Latency on Local Embeddings | Section 4 | Verified |
| - Multi-Hop Reasoning Across Disjoint Documents | Section 5 | Verified |
