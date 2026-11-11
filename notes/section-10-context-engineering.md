# Section 10 — Context Engineering and Management — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); code excerpts below are abbreviated from it. Credit to the instructor.
**Lab file (in the course's lab repo):** `Context-Engineering/tool-search.ipynb` (the only lab in the repo for this section)
**Exam weight:** ⭐ core — "Prompt and Context Engineering" and official objective 2 (plus objective 1's "context window as a fixed budget").

> **Coverage caveat:** the section has 7 lectures (46 min) but the repo holds only one lab. These notes
> cover that lab in depth. Anything marked *(general knowledge)* is not from the lab. If the lectures
> covered something else important (instruction design, context drift, summarization), check the
> lectures. Official objective 2 also names **system prompts, XML tags, few-shot examples,
> constraints and diagnosing a weak prompt**: none of these appear in the lab, so they are on
> the docs shortlist in [blueprint.md](../blueprint.md) (prompting best practices).

---

## 1. The principle behind the whole section
**The context window is a fixed budget, and *everything* competes for it:** system prompt, conversation
history, documents, tool results, **and tool definitions**. Context engineering = deciding what goes
into that budget, when, and what stays out.

**DE analogy:** the context window is RAM for a query (Section 3). Context engineering is
choosing which data to load into memory and when to spill or drop it, instead of `SELECT *` into RAM.

---

## 2. The problem the lab solves: tool definitions eat context
Every tool you declare (name + description + `input_schema`) is sent on every request and counts as
input tokens. A few tools is fine. **Dozens of tools** (an MCP server can expose many) means:
- a large share of the budget is spent *before the user asks anything*,
- more cost and latency on every call,
- worse tool selection (the "too many tools per agent" anti-pattern from Section 6).

---

## 3. Tool search: load tool definitions on demand (`tool-search.ipynb`) ⭐
```python
tools=[
  {"type": "tool_search_tool_bm25_20251119", "name": "tool_search_tool_bm25"},   # the search tool
  {"type": "mcp_toolset", "mcp_server_name": "Microsoft Learn",
   "default_config": {"enabled": True, "defer_loading": True}},                  # MCP tools, deferred
  {"name": "GetCurrentWeather", ..., "defer_loading": True},                     # custom tool, deferred
  {"name": "CalculateSum",      ..., "defer_loading": True},
]
betas=["mcp-client-2025-11-20"]
```
- **`defer_loading: True`** marks a tool as *not loaded into context up front*.
- The **tool search tool** lets Claude **look tools up by keyword when it needs a capability**, and
  only the matching tool definitions are pulled in. (BM25 is a classic keyword-relevance ranking
  algorithm, the same family used in text search engines.) *(mechanism inferred from the names and
  config; confirm against the lecture)*
- Tools you set `defer_loading` on can be **a mix of MCP and your own custom tools**, as the lab shows.
- **The loop rules from Section 6 still apply:** `GetCurrentWeather` and `CalculateSum` are
  **client-side**, so the lab still has a `while True` loop that runs them and returns `tool_result`s.
  The MCP tools are resolved server-side (`# Skip MCP tools`). *Any client-side tool = you own a loop.*

**DE analogy:** a **data catalog**. Instead of mounting every dataset in memory, you keep a
searchable index, find the one you need, then load only that. Lazy loading instead of eager loading.

**When to use / when not to** *(my framing)*
| Use tool search when… | Don't bother when… |
|---|---|
| You have many tools (dozens) or an MCP server exposing a large catalog | You have a handful of tools |
| Tool definitions are long, so they consume a real share of the budget | The tools are always needed on every call (keep those *non*-deferred) |
| Claude is picking the wrong tool among many similar ones | Latency matters more than tokens (the search adds a step) |

Trade-off to remember: you save context and gain selection quality, but add a lookup step, and a
bad search can miss the tool you needed.

---

## 4. Other context-management levers from earlier sections *(connect the dots)*
| Lever | Where | What it manages |
|---|---|---|
| Resend only what's needed in `messages` | S3 | History growth: every call resends everything |
| **Subagents** | S7 | Context isolation: messy work happens in a separate context; only the result returns |
| **Memory tool / memory store** | S6, S8 | Persist facts *outside* the window and across conversations |
| **RAG** | S5 | Retrieve only the relevant chunk instead of stuffing a whole corpus in |
| **Document citations** | S4 | Ground answers in supplied text without trusting recall |
| **Tool search** | S10 | Keep tool definitions out until needed |
| **Skills** | S7, S8 | Reference material loaded only when the task needs it |
| Prompt caching *(general knowledge; see the docs shortlist in blueprint.md)* | docs | Make repeated prefixes cheaper/faster |

**Pattern:** each lever answers "how do I avoid paying for context I don't need *right now*?"

---

## 5. Likely exam angles *(my reading)*
1. **"Too many tools, bloated context, wrong picks"** → tool search / `defer_loading`, or split into specialised subagents, not "bigger context" or a longer prompt.
2. **Which lever for which problem?** facts needed across sessions → memory; one big corpus → RAG; isolated heavy subtask → subagent; huge tool catalog → tool search.
3. **Statelessness consequence:** history grows and is resent each call, so costs and the risk of hitting the limit grow too. Plan for it.
4. **Defer, don't delete:** deferred tools are still available; they're just not loaded until searched.
5. **Always-needed tools shouldn't be deferred.**

## 6. Self-check
1. Why does declaring 40 tools hurt even if the user's question needs only one?
2. What does `defer_loading: True` do, and what must also be in `tools` for it to work?
3. In the lab, why is there still a `while True` loop?
4. Name three other ways to keep irrelevant material out of the context window.

*Answers: (1) their definitions are sent and counted on every request, and a crowded tool list degrades selection. (2) the tool isn't loaded up front; the tool search tool must be included so Claude can find it. (3) the weather and sum tools are client-side, so you execute them and return results. (4) e.g. subagents, RAG, memory store/tool, skills.*
