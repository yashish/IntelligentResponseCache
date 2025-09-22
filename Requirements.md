# Cache strategies and algorithms
Hit cascade
1. 	Exact cache (deterministic match)
• 	Key: Hash of normalized system + developer + user prompts, model ID, params, tools, knowledge ETags.
• 	Use: Low-temp tasks, templated prompts, structured Q&A.
• 	Trade-off: Highest precision; limited recall for paraphrases.

2. 	Parametric cache (template slots)
• 	Idea: Cache at the prompt template level
• 	Key: Template fingerprint + normalized slot schema + model/params.
• 	Use: Repetitive tasks with variable inputs (e.g., “summarize in 3 bullets”).
• 	Trade-off: Requires careful templating; risk of overgeneralization if slots aren’t bounded.

3. Semantic cache (near-duplicate)
- Key: Embedding vector; ANN search returns candidates.
- Decision: If similarity ≥ τ and metadata compatible, reuse with optional light adaptation.
- Use: Paraphrased queries, similar intents.
- Trade-off: False positives/negatives; requires robust thresholds and safety guardrails.

4. Tool/function cache
- Key: Deterministic function signature + input hash + upstream ETag/version.
- Use: HTTP calls, SQL queries, retrieval results.
- Trade-off: Staleness if upstream lacks strong validators

5. Stale-while-revalidate
- Serve stale if within soft TTL and return immediately; refresh in background.
- Trade-off: Great latency; risk of showing slightly outdated info—use only for low-volatility domains.

6. Security
- Encryption: TLS in transit; KMS envelope encryption at rest; consider field-level encryption for responses.
- PII handling: Classify content; either don’t cache PII or store redacted variants with strict TTLs and access policies.
- Cache poisoning prevention: Only cache responses that pass safety filters and come from trusted agent paths

Record schema (KV):
- key: Namespaced string
- hash_key: For partitioning
- embedding_id: Link to vector row (semantic cache)
- answer: Final text (and compact structure if applicable)
- metadata: { model_id, params, locale, persona, safety_labels, quality_score, created_at, soft_ttl, hard_ttl, corpus_etags[], tool_etags[], policy_version }

