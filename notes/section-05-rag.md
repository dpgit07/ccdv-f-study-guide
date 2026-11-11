# Section 5 — RAG (Retrieval Augmented Generation) — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); the code below is my own adaptation of standard SDK patterns shown in the labs, with generic example names. Credit to the instructor.
**Lab files (in the course's lab repo):** `RAG/load-vectors.ipynb`, `RAG/RAG.ipynb`

> ⚠️ **Marked optional in the study plan** — this section is not named in Anthropic's
> official CCDV-F curriculum. Still genuinely useful: RAG is an extremely common
> real-world pattern under the "Applications and Integration" domain, even if not
> called out by name.

---

## The core idea, in one sentence

Claude doesn't know your private/proprietary data (a company's travel brochures, internal
docs, etc.) — RAG **finds the relevant chunk of your data first**, then hands it to
Claude as context so it can answer *using* that specific information instead of guessing
from general training knowledge.

## Two separate jobs — don't conflate them

| Notebook | When it runs | What it does |
|---|---|---|
| `load-vectors.ipynb` | **Once** (or periodically, when data changes) — "indexing" | PDF → extract text → generate embedding vector → store in Qdrant |
| `RAG.ipynb` | **Every user query** — "serving" | User question → embed it → search Qdrant for closest match → inject that text into the Claude prompt → get answer |

**DE analogy — this maps directly onto familiar data-engineering ground:** this is exactly an **ETL load job vs. a query-time lookup**. `load-vectors.ipynb` is the batch "load" step that builds a search index once. `RAG.ipynb` is the "query" step that does an indexed lookup at request time, every single call. You'd never re-run the full indexing job just to answer one question — same instinct as not re-running a full ETL load just to serve one read.

---

## `load-vectors.ipynb` — the indexing pipeline

```python
# 1. Extract text from PDFs
reader = PdfReader(file)
content = "".join(page.extract_text() for page in reader.pages)

# 2. Generate an embedding vector for each document (OpenAI, not Claude)
result = embed_client.embeddings.create(model="text-embedding-3-small", input=doc_text)
vector = result.data[0].embedding

# 3. Store vectors in Qdrant (a vector database)
qdrant_client.create_collection(collection_name=..., vectors_config=VectorParams(size=1536, distance=Distance.DOT))
qdrant_client.upsert(collection_name=..., points=[PointStruct(id=..., vector=..., payload={"content": ...})])
```

**Key concept — what's an embedding?** A vector (list of numbers) that represents the
*meaning* of a piece of text. Similar meanings → similar vectors → close together in
vector space. This is what makes "search by meaning" possible instead of exact keyword
matching.

**DE analogy:** an embedding is like a **hash of meaning** instead of a hash of exact
bytes — two different sentences with the same meaning land near each other, the way two
different-looking rows can share a join key.

**Qdrant = the vector database** — same *role* in this pipeline as any index/database is
in yours: a place optimized to answer "give me the closest matches to X" fast, at scale.

---

## `RAG.ipynb` — the runtime query pipeline

```python
def embed_text(question):
    return openai_client.embeddings.create(model="text-embedding-3-small", input=question).data[0].embedding

def retrieve_context(question):
    query_vector = embed_text(question)
    hits = qdrant_client.query_points(
        collection_name="docs_embeddings",
        query=query_vector,
        limit=2
    ).points
    return hits[0].payload.get("content")  # top match only

context = retrieve_context(question)

# Feed the retrieved text to Claude as context
messages=[{
    "role": "user",
    "content": f"Answer the question using only the context below.\n\nQuestion: {question}\n\nContext:\n{context}"
}]
```

**The pattern, step by step:**
1. Embed the *user's question* the same way documents were embedded (same model — this must match, or the vectors aren't comparable)
2. Search the vector DB for the closest document(s) — `limit=2` fetches the top 2, but the code only uses `hits[0]` (the single best match) — worth noticing, since that's a subtle "why only the top one?" design choice you could imagine being asked about
3. Manually build a prompt that says "here's the question, here's supporting text, answer using it"
4. Send to Claude via the normal Messages API — **RAG is not a special API call**, it's a *pattern*: retrieve first, then call the regular `messages.create`/`messages.stream` calls from Section 3

**This is the key exam-relevant insight:** RAG isn't a new Claude feature — it's an
architectural pattern you build *around* the same Messages API from Section 3. The
"retrieval" part (embeddings + vector search) happens entirely outside Claude, in your
own code/infrastructure, before Claude ever sees anything.

---

## Connects back to Section 4

This pairs naturally with **Document Citations** (Section 4) — you could combine them:
retrieve the relevant chunk via RAG, then pass it as a `"document"` content block with
`citations: {"enabled": True}` so Claude's answer comes back with exact source
attribution instead of just prose. Retrieval finds the right haystack; citations prove
which straw the answer came from.

---

## One-sentence summary to keep

**RAG = your own search/indexing pipeline (embeddings + vector DB) that runs *before*
a normal Claude API call, so Claude answers using retrieved facts instead of guessing.**
