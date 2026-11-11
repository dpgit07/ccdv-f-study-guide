# Section 6 — Use Tools with the Claude API — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); code excerpts below are abbreviated from it. Credit to the instructor.
**Lab files (in the course's lab repo):** `Tool-Use/*.ipynb` (9 notebooks)
**Exam weight:** ⭐⭐ highest priority — feeds "Tools and MCPs", "Agents and Workflows", and official Module 2 objective 3 (*"define tool schemas, construct the tool-use loop"*).

> Source note: written from the lab notebooks, not the video lectures. Items marked
> *(outside the labs)* are from general API knowledge, not shown in the repo code.

---

## 1. The one idea that connects everything

**A tool is a function Claude can *ask* to have run. Claude never runs anything itself.**
Claude only emits a *request* ("call GetCurrentWeather with location=London"). Somebody
then has to execute it. Every tool in this section differs in just one thing — **who that
somebody is.**

| Bucket | Who executes | What YOU write | Labs |
|---|---|---|---|
| **Client-side tools** | **Your code** | Schema + the function + (the loop, or let the SDK run it) | user-defined-tool, Tool-Runner, Memory-Tool |
| **Server-side tools** | **Anthropic's platform** | Just declare it in `tools=[...]` — no function, no loop | Web-Search, Web-Fetch, Code-Execution, Advisor |
| **MCP connector** | **Anthropic's API calls a remote MCP server for you** | `mcp_servers` + `mcp_toolset` — no loop for those tools | MCP, Multi-Tool |

**DE analogy:** client-side tools = **UDFs you host and run yourself** (you own the code,
the runtime, the errors). Server-side tools = **built-in functions of a managed
warehouse** (`SELECT` calls them; you don't deploy anything). MCP = a **JDBC-style
connector** to somebody else's system, pre-wired for you.

**Decision rule to remember:** *Need your own business logic or your own systems?* →
client-side tool. *Need a generic capability (search, fetch a page, run Python)?* →
server-side tool exists, use it instead of rebuilding it.

---

## 2. The tool-use loop (the heart of the section — know this cold)

From `user-defined-tool.ipynb`. Trace it as the `messages` list changes:

```
messages = [ user: "What's the weather in London?" ]
        │
        ▼  client.messages.create(tools=tools, messages=messages)
Claude returns:  assistant: [ text?, tool_use{ id:"toolu_1", name:"GetCurrentWeather", input:{location:"London"} } ]
        │
        ▼  YOU append that assistant response to messages        ← statelessness (Section 3)!
        ▼  YOU run get_current_weather("London")
        ▼  YOU append  user: [ tool_result{ tool_use_id:"toolu_1", content:"...json..." } ]
        │
        ▼  call the API again with the longer messages list
Claude returns:  assistant: [ text: "It's 14°C and cloudy in London..." ]   ← no tool_use → loop ends
```

Things the loop teaches, each of which is exam-shaped:

1. **`tool_use` is a request, `tool_result` is the answer.** They're linked by `id` ↔ `tool_use_id`.
2. **`tool_result` goes in a `role: "user"` message.** That's why Section 3's alternating
   user/assistant rule matters — your tool output comes back *as the user's turn*.
3. **You must append Claude's response (with its `tool_use` blocks) to the history before
   adding the result.** Claude is stateless; skip this and the result refers to a request
   Claude can't see. Append the **whole** `response.content`, not just the text — that
   also keeps `thinking` blocks intact when thinking is on *(outside the labs)*.
4. **The loop ends when Claude stops asking for tools.** The lab detects that by "no
   `tool_use` block in this response". The canonical structured signal is
   `stop_reason == "tool_use"` (keep going) vs `"end_turn"` (done) *(outside the labs)*.
   Either way: **a structured signal, never parsing Claude's prose** ("if the text says
   'I'm done'…") — that's a named anti-pattern. Add a max-iterations cap only as a
   *backstop*, not as the primary stop.
5. **One response can contain several `tool_use` blocks** (parallel calls — the lab's
   `for block in response.content` handles that). The safe pattern is to send **all**
   results for that turn back together in one user message *(outside the labs; the lab
   appends them one by one, which is fine for a single call)*.

**DE analogy:** the loop is an **orchestrator (think Airflow scheduler)**: the model is the
planner that says "run task X with these args", your code is the executor, and the
result is fed back until the planner says there's nothing left to do.

---

## 3. Anatomy of a tool definition

```python
{
  "name": "GetCurrentWeather",
  "description": "Retrieve the current weather ... using the wttr.in service.",
  "input_schema": { "type": "object",
                    "properties": { "location": {"type": "string", "description": "City ..."} },
                    "required": ["location"] }
}
```

- **`description` is what Claude uses to decide *whether* and *when* to call the tool** —
  it's the most important field. Vague description → wrong or missed tool calls.
  (Exam angle: the fix for "Claude keeps picking the wrong tool" is usually a better
  description, not more prompt text.)
- **`input_schema` is plain JSON Schema** — same skill as the structured-outputs lab,
  and as defining a strict function signature / API contract. Property-level
  `description`s help Claude fill arguments correctly.
- **`tool_choice`** controls whether Claude *may* use tools: `auto` (shown in the lab —
  Claude decides), plus forcing a specific tool / forcing any tool / disabling tools
  *(outside the labs; the exam domain explicitly mentions "tool-choice control")*.
- **Keep tool sets small and focused.** Dozens of tools degrade selection quality
  (anti-pattern: "18+ tools per agent") — split into specialised agents instead.

---

## 4. Tool Runner — same loop, written for you (`Tool-Runner.ipynb`)

```python
@beta_tool
def get_weather(location: str) -> str:
    """Get the current weather in a given location.
    Args: location: The city ..."""

runner = client.beta.messages.tool_runner(model=..., tools=[get_weather, calculate_sum], messages=[...])
for message in runner: ...
```

- `@beta_tool` **generates the schema for you** from the type hints + docstring (so the
  docstring *is* the description — write it well).
- `tool_runner` **runs the loop for you**: calls Claude → executes the function →
  sends the result → repeats until done. The demo asks for weather *and* 15+27 in one
  prompt and it handles both tools.
- **Hand-rolled loop vs Tool Runner:** runner = less code, good for straightforward
  cases. Hand-rolled = you control every step → needed for **human-approval before an
  irreversible action, custom logging/tracing, retries, per-tool error handling**.
  *(Official objective 4: "human-in-the-loop checkpoints for irreversible actions" —
  that's the situation where you want your own loop.)*
- **DE analogy:** hand-rolled loop = writing your own scheduler logic; Tool Runner = using
  an orchestration framework's built-in operator.

---

## 5. Server-side tools — declare and forget

| Tool | Declaration (versioned `type` string — don't memorise the dates) | What comes back |
|---|---|---|
| Web search | `{"type": "web_search_20260318", "name": "web_search"}` | `web_search_tool_result` block: `title`, `url`, `page_age` per hit, plus Claude's answer |
| Web fetch | `{"type": "web_fetch_20250910", "name": "web_fetch", "max_uses": 5}` | Page content folded into the answer; **`max_uses` caps cost/runaway** |
| Code execution | `{"type": "code_execution_20250825", "name": "code_execution"}` | Runs Python in an **Anthropic-hosted sandbox**; outputs in `bash_code_execution_tool_result` |

- **No `tool_result` handling in your code** — the platform did the work and put the
  results straight into the response. That's the giveaway that it's server-side.
- **Code execution + files:** upload with `client.beta.files.upload(...)`, attach to the
  prompt as a `container_upload` block (needs the `files-api` beta), then **download
  generated files** (e.g. the Matplotlib chart) via the `file_id`s in the result.
  Use it when Claude should *compute* (stats, charts) instead of *guessing* numbers —
  LLMs are unreliable at arithmetic; running code is the reliable path.
- **DE analogy:** web search/fetch = calling a managed external API you didn't build;
  code execution = a **managed sandboxed compute** (like a serverless notebook runner).

---

## 6. Memory tool — beating statelessness *across* conversations (`Memory-Tool.ipynb`)

```python
memory = BetaLocalFilesystemMemoryTool(base_path="./memory")
runner = client.beta.messages.tool_runner(..., tools=[memory], messages=[...])
```

- Conversation 1: "Remember my name, company, preferred tone…" → Claude **writes files**
  into `./memory`. Conversation 2 starts with a **brand-new, empty `messages` list**:
  "Write a LinkedIn post" → Claude **reads memory first** and writes it in the right
  tone for the right audience.
- It is a **client-side tool**: Claude issues file operations, *your* process performs
  them against local storage. (You own where memory lives — and its security.)
- **Dot to connect:** Section 3 taught that the `messages` list is memory *within* one
  conversation and that Claude remembers nothing between calls. The memory tool is the
  official answer to "how do I persist things *between* conversations?"
- **DE analogy:** `messages` = a **session/temp table** that vanishes; memory tool = a
  **persistent table** that survives the session.

---

## 7. Advisor tool — model selection inside one call (`Advisor-Tool.ipynb`)

```python
model="claude-sonnet-5",
tools=[{"type": "advisor_20260301", "name": "advisor", "model": "claude-opus-4-8"}],
betas=["advisor-tool-2026-03-01"]
```

- A main ("executor") model does the work and can **consult a stronger advisor model**
  for the hard parts. Dot to connect → **Model Selection & Optimization** domain: you
  don't have to choose "all Opus (expensive)" vs "all Sonnet (maybe not enough)".
- **DE analogy:** run on the **cheap warehouse by default and escalate only the
  expensive step** to the big one.
- *(Behaviour inferred from the lab config; if the lecture or the docs describe it
  differently, trust those.)*

---

## 8. MCP connector and mixing tool types (`MCP.ipynb`, `Multi-Tool.ipynb`)

```python
mcp_servers=[{"type": "url", "url": "https://learn.microsoft.com/api/mcp", "name": "MSLearnMCPServer"}],
tools=[{"type": "mcp_toolset", "mcp_server_name": "MSLearnMCPServer"}],
betas=["mcp-client-2025-11-20"]
```

- MCP = a **standard interface** so Claude can use tools exposed by another system
  without custom glue per integration. Here the **API itself connects to the remote
  server** — so `MCP.ipynb` has **no loop**.
- `Multi-Tool.ipynb` **mixes** an MCP toolset *and* your own client-side
  `GetCurrentWeather` in one call: the MCP calls resolve server-side, but because a
  client tool is present **you still need the loop** to handle *that* tool.
  **Rule: the moment any tool is client-side, you own a loop.**
- Credentials/secrets for MCP must never be pasted into prompts or schemas — the
  course's "GitHub MCP + Vault secret" lab is about exactly that (official Module 3:
  *"connect Claude to enterprise systems through MCP without leaking credentials"*).
- *(Building your own MCP server is covered in [mcp-building-a-server.md](mcp-building-a-server.md).)*

---

## 9. Recognition cheat-sheet (read code, don't memorise it)

| You see… | It means… |
|---|---|
| `client.beta.messages…` / `betas=[…]` | Feature still in beta (MCP connector, files, advisor, tool runner, memory) |
| `block.type == "tool_use"` | Claude is *requesting* a client-side tool call |
| `{"type": "tool_result", "tool_use_id": …}` in a **user** message | You're returning the result |
| A tool with a dated `type` (`web_search_2026…`) and no `input_schema` | Server-side tool — no function from you |
| `mcp_servers` + `mcp_toolset` | MCP connector — platform talks to the MCP server |
| `@beta_tool` + `tool_runner` | SDK is running the loop for you |
| `max_uses` | Spend/runaway guardrail on a server tool |

---

## 10. Likely exam angles (my reading of the domain — not official questions)

1. **"Claude calls the function directly"** → wrong for client tools; Claude only requests, your code executes.
2. **Missing/incorrect history** (result sent without the assistant `tool_use` turn) → statelessness bug.
3. **Loop termination** → structured signal (`stop_reason`/no `tool_use`), cap as backstop; never parse prose.
4. **Wrong tool chosen / tool never chosen** → fix the `description` and schema first; trim the tool list.
5. **Which tool type?** needs own logic/system → client tool; generic search/fetch/compute → server tool; third-party system → MCP.
6. **Irreversible action** (refund, delete, send) → human approval gate in *your* loop; don't let an auto-runner fire it.
7. **Reliability:** the labs are happy-path (`raise_for_status()` crashes on failure). Production returns a *clear, structured error* in the `tool_result` so Claude can recover (anti-patterns: generic errors; swallowing errors as "success").
8. **Math/data analysis** → code execution tool, not "ask Claude to calculate".
9. **Persistence across sessions** → memory tool (or your own store), not a longer prompt.

---

## 11. How this connects backwards and forwards

- **Section 3 (statelessness)** → why you must append the assistant turn and why results ride in a user message.
- **Section 4 (structured outputs)** → `input_schema` is the same JSON-Schema skill, aimed at *inputs to your function* instead of *Claude's output*.
- **Section 4 (batch / thinking)** → thinking blocks must survive the loop; tool loops are real-time, not batch.
- **Section 7–8 (Agents SDK, Managed Agents)** → an agent **is** this loop, productised (plus memory, orchestration, guardrails). If the loop in §2 is clear, agents are "the loop with more machinery".
- **Evals and security (see the docs shortlist in [blueprint.md](../blueprint.md))** → test that the right tool is called with valid args; defend against prompt injection arriving via tool results (web fetch/search content is **untrusted input**).

---

## 12. Self-check — can you answer these without scrolling up?

1. Who executes a `GetCurrentWeather` tool in `user-defined-tool.ipynb`, and what does Claude actually return?
2. Why does the `tool_result` go in a `user` message, and what must you append *before* it?
3. Which of web search / code execution / memory / your own function needs you to write `tool_result` handling? Why?
4. When would you hand-roll the loop instead of using `tool_runner`?
5. A call mixes an MCP toolset with one custom function. Do you need a loop? Why?

*Answers: (1) your Python function; Claude returns a `tool_use` request. (2) Claude's output returns as the next turn; the assistant response containing the `tool_use`. (3) Only your own function and memory (client-side); web search/code execution are server-side. (4) Approval gates, custom logging/retries/error handling. (5) Yes — the custom function is client-side.*
