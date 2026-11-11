# Section 7 — Claude Agents SDK — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); code excerpts below are abbreviated from it. Credit to the instructor.
**Lab files (in the course's lab repo):** `Agents-SDK/*.py` (+ `.claude/skills/`, `claude-agent-api/`)
**Exam weight:** ⭐⭐ highest priority — "Agents and Workflows", official objectives 4 and 5.

> Source note: written from the lab code, not the videos. *(outside the labs)* = general knowledge.

---

## 1. Where the Agent SDK sits (connect the dots with Sections 3 and 6)

| Level | Who writes the loop | Who provides tools | You use it when |
|---|---|---|---|
| Messages API + hand-rolled loop (S6) | You | You (client-side) or server-side tools | You need full control: approval gates, custom logging, minimal surface |
| `tool_runner` (S6) | SDK, for *your* functions | You | Simple tool use, no special control needs |
| **Claude Agent SDK (S7)** | **The SDK — it is Claude Code's agent loop as a library** | **Built-in** (Read, Edit, Glob, WebSearch, WebFetch, …) + MCP + skills + subagents | You want an autonomous agent that works with files/web/tools out of the box |
| Managed Agents (S8) | The Anthropic platform (hosted) | Built-in toolset + MCP + skills | You want Anthropic to host the loop, sandbox and session state |

**The loop from Section 6 hasn't gone away.** Every row is the same request → tool → result →
repeat cycle; each row just moves more of it off your plate. *(Decision framing is mine.)*

---

## 2. The core call: `query()` (`Getting-Started.py`)

```python
async for message in query(
    prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
    options=ClaudeAgentOptions(
        model=claude_model_name,
        allowed_tools=["Read", "Edit", "Glob"],
        permission_mode="acceptEdits",
    ),
):
    if isinstance(message, AssistantMessage): ...   # blocks: text, tool_use (has .name)
    elif isinstance(message, ResultMessage): ...    # final: .subtype, .result
```

- `query()` is an **async generator**: it streams messages as the agent works through its
  loop. You consume them; you don't write the tool loop.
- Message types to recognise: **`AssistantMessage`** (text blocks and tool-use blocks) and
  **`ResultMessage`** (the end: `subtype` says how it finished, `result` is the final text).
- The task in the lab: `utils.py` has two crash bugs (divide-by-zero on an empty list;
  `KeyError` on a missing `"name"`). The agent **reads → finds → edits** the file itself.

### The permission model (official objective 5)
- **`allowed_tools`** = an allow-list. Only these tools are available/pre-approved →
  **least privilege**, same idea as scoping an IAM role.
- **`permission_mode="acceptEdits"`** = file edits are auto-approved without prompting.
  Convenient in a lab; in production think about *what you're pre-approving*.
- **DE analogy:** `allowed_tools` is the role's granted permissions; `permission_mode` is the
  approval policy on top.

---

## 3. MCP in the Agent SDK (`Multi-Tool.py`)

```python
mcp_servers={"MSLearnMCPServer": {"type": "http", "url": "https://learn.microsoft.com/api/mcp"}},
allowed_tools=["mcp__MSLearnMCPServer__*", "Read", "Edit", "Glob", "WebSearch", "WebFetch"]
```

- MCP tools are named **`mcp__<server-name>__<tool>`**; the `*` wildcard allows everything
  from that server. Remember the naming pattern — it's how you allow-list MCP tools.
- Compare to Section 6: there the API called the remote MCP server for you
  (`mcp_servers` + `mcp_toolset`); here the **SDK connects as an MCP client**.

---

## 4. Subagents (`subagents.py`) ⭐

```python
MARKETING_AGENTS = {
  "market-researcher": AgentDefinition(description="Expert in market research...", prompt="...", tools=[]),
  "content-creator":   AgentDefinition(description="Expert in marketing content...", prompt="...", tools=[]),
}
options=ClaudeAgentOptions(allowed_tools=["Agent"], agents=MARKETING_AGENTS)
```

- A **main agent delegates** to subagents through the built-in **`Agent`** tool (hence
  `allowed_tools=["Agent"]`). The prompt orders the work: researcher first, then writer.
- **`description` is how the orchestrator decides which subagent to use** — exactly like a
  tool description in Section 6. Vague descriptions → wrong delegation.
- Each subagent has its **own system prompt and its own tool list** (here empty:
  pure reasoning, minimum privilege).
- **Why subagents (exam angle):** *context isolation* — a subagent does messy work in its own
  context and returns a result, so the main agent's context stays small and focused
  *(general knowledge)*; plus specialisation, and limiting each worker's tools.
- **DE analogy:** coordinator + workers in a distributed pipeline; each worker has its own
  memory and returns only its output to the coordinator.

---

## 5. Skills (`claude-skills.py` + `.claude/skills/marketing-review/`)

```python
ClaudeAgentOptions(cwd=os.getcwd(), setting_sources=["project"], skills=["marketing-review"])
```
```
.claude/skills/marketing-review/
  SKILL.md                 ← frontmatter: name + description; instructions
  brand-guidelines.md, marketing-checklist.md, linkedin-template.md   ← supporting files
```

- A **skill = a folder of instructions + reference files** the agent can pull in for a type
  of task. `SKILL.md` has a `name` and a `description`; its body tells the agent which
  supporting files to read ("Read brand-guidelines.md, then…").
- `setting_sources=["project"]` tells the SDK to load project-level config from `cwd`
  (that's where `.claude/skills/` lives) — **durable project context** (official obj. 5).
- **Why skills rather than a giant system prompt:** reusable, versioned in the repo, and the
  agent loads the heavy reference material **only when the task needs it** *(general
  knowledge: description is always visible, body/files load on demand)*. That's a
  context-budget argument (Section 3's "context window = RAM").
- **DE analogy:** a reusable, parameter-free **runbook/library** in the repo that any job can
  import when needed, instead of pasting the same SQL into every job.

---

## 6. Agent as a service (`claude-agent-api/`: FastAPI + Docker)

- A `POST /chat` endpoint wraps `query(...)`, collects text and tool names, returns
  `{"response", "tools_used"}`; `Dockerfile` + `docker-compose.yml` package it.
- Shows the production shape: **agent behind an HTTP API, containerised.**
- **Judgment point (my observation, not stated in the lab):** the endpoint lets anyone's
  `prompt` drive an agent that has `Edit`, `WebFetch` and `WebSearch`. A user-controlled prompt
  plus powerful tools is a **prompt-injection / abuse surface** — apply least privilege and
  treat everything the agent fetches as untrusted. Likely Section 12 territory.

---

## 7. Likely exam angles *(my reading, not official questions)*
1. **Which layer?** Own loop vs `tool_runner` vs Agent SDK vs Managed Agents → how much control vs convenience you need.
2. **Least privilege:** narrow `allowed_tools`; give subagents only the tools they need (often none).
3. **Subagent descriptions** drive routing — write them like tool descriptions.
4. **Why subagents:** context isolation, specialisation, restricted tools — not "more agents = better". Don't build a multi-agent system when one agent or a fixed workflow does the job.
5. **Skills vs system prompt vs subagent:** reusable task knowledge loaded on demand (skill) vs always-on persona (system prompt) vs separate worker with its own context (subagent).
6. **Detect completion via the structured result** (`ResultMessage.subtype`), not by reading the prose.

## 8. Self-check
1. What does `allowed_tools=["Agent"]` enable, and where does the work happen?
2. Why give a subagent `tools=[]`?
3. What does `setting_sources=["project"]` change for skills?
4. In what way is the Agent SDK "Section 6's loop, productised"?

*Answers: (1) delegation to subagents via the Agent tool; each subagent runs with its own prompt/tools. (2) least privilege — pure reasoning needs no tools. (3) loads project config (e.g. `.claude/skills/`) from the working directory. (4) it runs the request→tool→result loop for you, with built-in tools, permissions, MCP, skills and subagents.*
