# Section 3 — Claude AI Foundations and Fundamentals — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); the code below is my own adaptation of standard SDK patterns shown in the labs, with generic example names. Credit to the instructor.
**Lab file (in the course's lab repo):** `Getting-Started/messages-api.ipynb`

---

## Setup pattern (same in every lab going forward)

```python
%pip install anthropic python-dotenv
load_dotenv()
client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
```

Same boilerplate across the whole course — install SDK, load `.env`, create one client, reuse it.

---

## The core call

```python
message = client.messages.create(
    model=MODEL_NAME,
    system="You are a helpful travel-desk assistant.",
    messages=[{"role": "user", "content": "..."}],
    max_tokens=20000
)
for block in message.content:
    if block.type == "text":
        print(block.text)
```

| Part | What it is |
|---|---|
| `system=` | Standing instruction shaping behavior — not part of the conversation turns |
| `messages=[...]` | Ordered list of turns, each `{"role": ..., "content": ...}` |
| `max_tokens` | Caps **output** length only (not input) |
| `message.content` | A **list of content blocks** (text, thinking, etc.) — not a plain string. Always filter by `block.type`. |

---

## ⭐ THE key concept: statelessness

**Claude has zero memory between API calls.** Each call is a total stranger to the last one.

- `role: "user"` = what the human said
- `role: "assistant"` = what Claude said (its own past voice)
- The **entire ordered message list, together, IS "the conversation so far"** — there's no separate "history" flag on any message. Position + strictly alternating roles (user → assistant → user...) is the *only* signal. The list always ends on a `user` turn, and Claude's job is simply "continue this transcript."
- If you want Claude to "remember" something, **you must resend the full history yourself, every call.**

**DE analogy:** exactly like a stateless Lambda / stored procedure with no session state. There's no server-side memory — the message list *is* the state, and the caller owns it. Also like an event log ordered by timestamp: no `is_history` column needed, order alone tells you what's "past."

**Why this matters downstream:** this is the root cause of context-window management, memory scope, and summarization strategy — all of it exists *because* Claude is stateless and conversations grow every turn.

---

## Streaming

```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

Prints tokens as they're generated instead of waiting for the full response. DE analogy: streaming query results row-by-row vs. waiting for the full result set.

---

## Images (two input methods)

| Method | Code | When to use |
|---|---|---|
| URL | `"source": {"type": "url", "url": image_url}` | Image is already publicly reachable |
| Base64 | fetch bytes yourself, `base64.standard_b64encode(...)` | Local/private files Claude can't fetch itself |

DE analogy: passing a file *reference* (S3 URL) vs. passing file *contents* inline in a payload.

---

## Bigger-picture reframe: why use the API instead of chat?

**Why write code instead of just using Claude.ai chat?** Because the API is how *your software* talks to Claude — unattended, at scale, integrated into a system. Chat needs a human in the loop for every message; the API doesn't.

- Chat ≈ manually running a query in a SQL GUI client
- API ≈ a script running that query programmatically, on a schedule, over millions of rows, no human watching

**The whole exam is really about:** "how do I make Claude a dependable, controllable component of a system I'm building" — not "how do I get a better chat reply."

**What to actually memorize vs. offload to Claude Code:** memorize the *concepts* (why statelessness matters, when to stream, when to use structured output). Offload exact parameter syntax — the same split data engineers already make with SQL/pandas syntax nobody keeps memorized. But always **read** generated code enough to explain what it does — that's the actual skill the (multiple-choice, code-reading) exam tests.
