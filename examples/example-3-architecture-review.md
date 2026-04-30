# Example 3 — Architecture review: "Review my RAG system design"

When the user brings an existing design and wants pushback. SD leads, AIE chimes in, risks get explicit callouts.

---

## Prompt

> I'm building a RAG system over our company's internal docs (~50k pages of PDFs and Confluence pages). Here's my plan:
>
> - Chunk everything into 512-token windows
> - Embed with `text-embedding-3-small`
> - Store in Pinecone
> - At query time: embed query, top-5 retrieval, stuff into Claude prompt
> - Single endpoint, no caching
>
> What am I missing?

## Response

**[PM] — Quick scope check**

Before reviewing the design: who's the user, and what's the success metric? "Good RAG over internal docs" can mean very different things for engineering onboarding vs. customer support vs. sales enablement. I'll review against a generic "internal Q&A" target — flag if your use case is different.

**[SD] — Review of the design**

The skeleton is fine. Three things are missing or under-spec'd:

1. **Chunking strategy is too naive for mixed PDF + Confluence content.** Fixed 512-token windows shred tables, code blocks, and section hierarchies. Use semantic chunking that respects headings — `unstructured` or LlamaIndex's `MarkdownNodeParser` for Confluence, layout-aware parsing (`pymupdf4llm`) for PDFs. Keep chunks ~300-800 tokens with 10-15% overlap.

2. **Top-5 retrieval with no reranking is a known weak point.** `text-embedding-3-small` is fine for recall, bad for precision at small k. Either go top-20 + rerank with `cohere-rerank-3` down to top-5, or use hybrid retrieval (BM25 + dense) with reciprocal rank fusion. Reranking gives the bigger lift for less effort.

3. **No evaluation harness.** This is the most important gap. Build a test set of 30-50 (question, expected_doc, expected_answer) tuples *before* you ship. Otherwise you can't tell whether changes are improvements.

**[AIE] — RAG-specific notes**

- **Citation handling**: stuff the retrieved chunks with stable IDs (`<doc id="conf-1234">...</doc>`) and instruct Claude to cite them inline. Without this, hallucinated citations are nearly guaranteed.
- **Prompt structure**: put retrieved docs *before* the question, not after. Recall improves measurably ([Anthropic's long-context docs](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/long-context-tips)).
- **Failure mode**: when retrieval misses, the model still confidently answers. Add an explicit "if the documents don't contain the answer, say so" instruction and an "I don't know" eval case.

**[BE] — Infra notes**

- **Caching matters more than you think.** Most internal-docs systems see heavy repeat-query patterns. Cache (query → answer) with a 24h TTL keyed on the query embedding (cosine > 0.97 = cache hit). Cuts 30-50% of LLM cost in practice.
- **Pinecone is fine but consider pgvector** if you already run Postgres. For 50k docs (~500k chunks), it's well within pgvector's comfortable range, and you skip a vendor.
- **Ingestion needs to be incremental.** Confluence changes daily. Set up a webhook or nightly sync — don't re-embed the corpus from scratch.

**[SD] — Risks (ranked)**

1. **No eval harness** → you'll ship and have no idea if it's good. Build this first.
2. **Permissions leak** → if your Confluence has restricted spaces, your RAG must respect ACLs at retrieval time. Filter by user permissions in the vector query, not in post-processing. This is a real security incident waiting to happen if you skip it.
3. **PDF parse quality** → 50k pages will include scanned PDFs that `pypdf` mangles silently. Audit the corpus before trusting the index.

**[PM] — Recommended order**

If I were sequencing the work:

1. Build the eval harness (30 questions, expected docs, expected answers) — 1 day.
2. Implement permissions-aware retrieval — 1 day.
3. Ship the naive version (your current design) and run the eval — 0.5 day.
4. Add reranking and re-run eval — 0.5 day.
5. Improve chunking on whichever doc type fails most — 1-2 days.

Resist the urge to optimize before you have the eval. You'll waste a week tuning the wrong knob.

---

## What the skill did well here

- **PM didn't ceremony** — just flagged the missing context (use case) and moved on.
- **SD prioritized** the three gaps instead of listing every possible improvement.
- **Risks ranked**, with the security risk (permissions leak) called out explicitly even though the user didn't ask about it.
- **Sequenced the work** at the end so the user has an order of operations, not just a wishlist.
