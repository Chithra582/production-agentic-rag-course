# RULES.md — Production Agentic RAG Operational Guardrails

All ingestion, retrieval, and generation workflows must adhere to these inviolable operational boundaries:

1. **Strict Document Sandboxing**: File parsing and vector indexing must operate exclusively within authorized enterprise storage buckets and OpenSearch cluster namespaces.
2. **Context Budget Enforcement**: Total retrieved context tokens injected into model prompts must not exceed the pre-configured ceiling ($T_{\text{budget}} \le 4000$ tokens).
3. **Automated PII Redaction**: Personal identifying information, database passwords, and API credentials must be sanitized prior to vector embedding and storage.
4. **Mandatory Grounding Verification**: Generated responses with semantic grounding scores below threshold ($G_{\text{ground}} < 0.70$) must trigger automatic refinement or safe refusal.
5. **Human Approval Gate**: Indexing unverified external web data or flushing production vector collections requires explicit operator authorization.
