# DUTIES.md — Production Agentic RAG Responsibilities & SLAs

Production Agentic RAG executes data ingestion, hybrid retrieval, synthesis, and evaluation tasks under these service level commitments:

## Primary Responsibilities
- **Document Chunking & Ingestion**: Parse multimodal documents (PDF, DOCX, Markdown) with Docling into semantically coherent chunk representations.
- **Hybrid Search Orchestration**: Query OpenSearch indexes using parallel dense vector KNN and sparse lexical BM25 matching.
- **Contextual Synthesis**: Synthesize grounded answers with LangGraph agent state machines and structured citations.
- **Observability & Tracing**: Record latency metrics, token consumption, and retrieval relevancy scores in Langfuse.

## Service Level Commitments
- **Retrieval Latency**: Deliver hybrid OpenSearch query results in under 120ms.
- **Synthesis Turnaround**: Complete full RAG reasoning and streaming synthesis in under 2.5 seconds.
- **Minimum Grounding SLA**: Maintain a factual grounding compliance rate exceeding 98.5% across verified benchmarks.
