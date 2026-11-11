# Section 4 — Claude Model Capabilities — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); code excerpts below are abbreviated from it. Credit to the instructor.
**Lab files (in the course's lab repo):** `Model-Capabilities/*.ipynb`

---

## 1. Reasoning vs. Thinking — two *different* mechanisms, don't confuse them

| | Reasoning Effort | Extended/Adaptive Thinking |
|---|---|---|
| What it is | A simple dial: `low` / `medium` / `high` | A separate mode producing a *visible*, inspectable reasoning trace |
| Code | `output_config={"effort": "medium"}` | `thinking={"type": "adaptive", "display": "summarized"}` |
| What you get back | Just the final answer, tuned by effort level | Two block types: `"thinking"` **and** `"text"` — Claude's reasoning is literally readable before its answer |
| DE analogy | Choosing query optimizer aggressiveness — more effort, better plan, more cost | `EXPLAIN ANALYZE` — you see the reasoning steps, not just the result |

**Exam-testable distinction:** one controls *how hard* Claude thinks (invisible), the other controls whether you *see* the thinking (visible trace).

## 2. Lab: Configure Reasoning Efforts

- Key line: `output_config = {"effort": "medium"}` inside `messages.create()`
- `message.usage` is always inspectable on the response — token usage tracking for cost (Model Selection & Optimization domain)
- Remember the *concept* (tunable cost/quality knob), not the exact dict key.

## 3. Lab: Configure Adaptive Thinking

- Non-streaming: loop `response.content`, check `block.type == "thinking"` vs. `"text"` separately.
- **Streaming here is different from Section 3's plain streaming** — Section 3 was just `stream.text_stream`. Here, because there are *two* content types, you iterate raw `event`s and check `event.delta.type`: `"thinking_delta"` vs. `"text_delta"`.
- Rule: more content types in a response → more granular event-based streaming needed.

## 4. Lab: Derive Structured Outputs ⭐ (important one)

| Method | Code | When to use |
|---|---|---|
| Raw JSON Schema | `output_config={"format": {"type": "json_schema", "schema": {...}}}` | You control exact schema shape; response is text you parse yourself |
| Pydantic model | `client.messages.parse(..., output_format=CustomerLead)` → `response.parsed_output` | Higher-level — get a **typed Python object back directly**, no manual parsing |

DE analogy: raw JSON schema ≈ hand-writing SQL DDL; Pydantic ≈ using an ORM — same destination, less plumbing.

`additionalProperties: False` + `required: [...]` = no extra/missing fields allowed, like a strict table schema rejecting malformed rows.

## 5. Lab: Document Citations

- Docs passed as content blocks: `{"type": "document", "source": {...}, "title": ..., "context": ..., "citations": {"enabled": True}}`
- Response text blocks come back with `.citations` — each has `document_title` + `cited_text`: Claude tells you exactly which source backed each claim.
- Why it matters: a grounding/reliability mechanism — makes hallucination *checkable*, not just hoped-for. Feeds into RAG (Section 5) and production reliability.
- DE analogy: a query result with lineage metadata — not just the number, but which source row produced it.

## 6. Lab: Batch Processing ⭐ (batch vs. real-time, made concrete)

- `client.messages.batches.create(requests=[Request(custom_id=..., params=...), ...])` — submit many independent requests as one job.
- `custom_id` = your correlation key to match results back to inputs.
- **Async**: get a batch ID back immediately, then **poll**: `client.messages.batches.retrieve(batch_id)` until `processing_status == "ended"`.
- Results per-request: `succeeded` / `errored` (sub-types: `invalid_request_error` = fix & resend, vs. server error = safe to retry as-is) / `expired`.

DE analogy: exactly a Spark batch job / async ETL run — submit → get a job ID → poll status → collect results by ID. `custom_id` is your join key back to the source row.

Why exam-relevant: this **is** the literal "batch vs. real-time" tradeoff from the Applications & Integration domain — batch trades latency for cost/throughput at scale.

---

## Takeaways

- **Why write API code instead of just chatting?** The API is how *your software* talks to Claude — unattended, at scale, integrated into a system. Chat needs a human per message; the API doesn't. This is the entire premise of the Developer cert: making Claude a dependable component of a system, not "a better chat."
- **What to memorize vs. offload to Claude Code:** memorize *concepts* (why/when for each capability). Offload exact parameter syntax to tooling — same split already made with SQL/pandas syntax. Always read generated code enough to explain what it does; that's what the (multiple-choice) exam actually tests.
