# Section 8 — Claude Managed Agents — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); the code below is my own adaptation of standard SDK patterns shown in the labs, with generic example names. Credit to the instructor.
**Lab files (in the course's lab repo):** `Managed-Agents/*.ipynb` (7 notebooks + `skills/`)
**Exam weight:** ⭐⭐ highest priority — "Agents and Workflows", plus objectives 4, 5, 7.

> Source note: written from the lab code, not the videos. *(outside the labs)* = general knowledge.
> Managed Agents is a **beta** API (`client.beta.…`) — know the concepts; don't memorise the strings.

---

## 1. The big shift: the platform hosts the loop *and the state*

| | Messages API / Agent SDK | **Managed Agents** |
|---|---|---|
| Who runs the loop | You (or the SDK in *your* process) | **Anthropic's platform** |
| Where tools execute | Your machine / your sandbox | A **cloud environment** Anthropic provides |
| Conversation state | **You** resend history (stateless) | **The session holds it** — you just send the next message |

This is the single most important contrast with Section 3. In `Getting-Started.ipynb` you
send each new user message as an event to the **same session** and never resend history.
**DE analogy:** Messages API = running your own Spark cluster and managing job state;
Managed Agents = a **managed job service** with a persistent run you submit events to.

---

## 2. The four building blocks (`Getting-Started.ipynb`)

| Resource | What it is | Key code |
|---|---|---|
| **Agent** | Reusable, **versioned** config: model, system prompt, tools, skills, MCP servers | `client.beta.agents.create(name, model, system, tools=[{"type": "agent_toolset_20260401"}])` → `agent.id`, `agent.version` |
| **Environment** | The cloud container the agent runs in (networking etc.) | `client.beta.environments.create(config={"type": "cloud", "networking": {"type": "unrestricted"}})` |
| **Session** | A running instance = agent + environment (+ resources, vaults) | `client.beta.sessions.create(agent=…, environment_id=…)` |
| **Events** | The messages going in and out | in: `user.message`; out: `agent.message`, `agent.tool_use`, `session.status_idle` |

```python
with client.beta.sessions.events.stream(sess.id) as feed:
    client.beta.sessions.events.send(sess.id, events=[{"type": "user.message", "content": [...]}])
    for event in feed:
        match event.type:
            case "agent.message": ...
            case "agent.tool_use": ...
            case "session.status_idle": break      # the agent has finished its turn
```

- **Termination = a structured event** (`session.status_idle`), not text parsing — same
  principle as `stop_reason` in Section 6.
- `agent_toolset_20260401` = the **built-in toolset** (versioned type string — don't memorise).
- **DE analogy:** Agent = job *definition*; Environment = *cluster/runtime config*; Session = one
  *run*; Events = the *log stream* you tail until the run goes idle.

---

## 3. Multi-agent orchestration (`Multi-Agent.ipynb`) ⭐
```python
coordinator = client.beta.agents.create(name="Coordinator", ...,
    multiagent={"type": "coordinator", "agents": [{"type": "agent", "id": researcher.id},
                                                   {"type": "agent", "id": writer.id}]})
```
- A **coordinator** agent lists the agents it can delegate to (Researcher *with* tools, Writer
  *without*). You run a session against the **coordinator**; it delegates.
- The prompt tells it "use only one researcher instance, keep it quick/short" → a **cost/latency
  control** — multi-agent runs multiply token usage, so constrain them.
- Same coordinator-worker pattern as Section 7 subagents, but hosted.

---

## 4. Persistent memory store (`persistent-memory.ipynb`) ⭐
```python
store = client.beta.memory_stores.create(name="User preferences", description="...")
resources=[{"type": "memory_store", "memory_store_id": store.id, "access": "read_write",
            "instructions": "User preferences... Check before starting any task."}]
```
- Session #1 (Agent #1) is told the user's preferences → saved in the store. **Session #2 uses a
  different agent** and still recalls them, then **updates** them (Python → C#) and uses the new
  preference.
- **Memory outlives sessions *and* agents.** It's the managed version of Section 6's local
  memory tool. `instructions` tells the agent how/when to use it.
- **Three kinds of "memory" to keep straight:**
  1. `messages` list — within one stateless conversation (S3)
  2. Memory tool / memory store — across conversations (S6 local, S8 managed)
  3. Session state — the platform remembering the running conversation (S8)
- **DE analogy:** session = temp table; memory store = a persistent table shared by many jobs.

---

## 5. Vaults — secrets done right (`Vaults.ipynb`) ⭐ (official objective 7)
```python
vault = client.beta.vaults.create(display_name="Service token vault")
client.beta.vaults.credentials.create(vault_id=vault.id,
    auth={"type": "static_bearer", "mcp_server_url": "https://api.githubcopilot.com/mcp/", "token": service_token})
session = client.beta.sessions.create(agent=agent.id, environment_id=env.id, vault_ids=[vault.id])
```
- The **agent definition names the MCP server but holds no secret**. The credential sits in a
  **vault** and is attached **at session time** via `vault_ids`.
- Result: **the token never appears in a prompt, a system prompt, the agent config, or the
  transcript.** This is the exam answer to "connect to enterprise systems without leaking credentials".
- **Judgment points:** the lab sets `"permission_policy": {"type": "always_allow"}` on the GitHub
  toolset — convenient, but it pre-approves everything that toolset can do. Use a **least-privilege
  token** and consider requiring approval for risky actions *(approval modes aren't shown in the lab)*.
  Likewise `networking: unrestricted` is a lab convenience; restrict egress in production *(general knowledge)*.
- **DE analogy:** a secrets manager (Vault/Key Vault) referenced by a job at runtime, instead of
  a password hard-coded in the DAG file.

---

## 6. Skills, MCP, files

- **Agent Skills (`Agent-Skills.ipynb`):** upload a custom skill (`client.beta.skills.create(files_from_dir(...))`),
  attach to the agent: `skills=[{"type": "anthropic", "skill_id": "docx"}, {"type": "custom", "skill_id": skill.id, "version": …}]`.
  **Anthropic-provided skills** (like `docx`) vs **your custom skills** (versioned). Outputs
  (the `.docx`) are fetched afterwards via `files.list(scope_id=session.id)` + `download`.
- **MCP connector (`MCP-Connector.ipynb`):** `mcp_servers=[…]` on the agent + `{"type": "mcp_toolset", "mcp_server_name": …}` in `tools` — same shape as Section 6.
- **PDF support (`PDF-Support.ipynb`):** upload with the Files API, then reference it in the message as
  `{"type": "document", "source": {"type": "file", "file_id": …}}`. Claude analyses text *and images* in the PDF.

---

## 7. Choosing the right layer *(my framing — the exam is "when to do what")*

| Situation | Reach for |
|---|---|
| Need full control (approval gate, custom tracing), few tools | Messages API + own loop |
| Simple tool calling, little control needed | `tool_runner` |
| Autonomous agent working on files/web in **your** infrastructure | Agent SDK |
| Want hosted loop + sandbox + persistent sessions/memory/vaults, less ops | Managed Agents |
| One-shot, fixed steps (classify → route → respond) | **A fixed workflow — not an agent at all** |

**The anti-over-engineering rule:** an agent is for open-ended work where the next step depends
on what's found. If you can draw the steps in advance, use a workflow.

---

## 8. Likely exam angles *(my reading)*
1. **Stateless vs stateful:** Messages API needs history resent; Managed Agents sessions keep it.
2. **Secrets belong in vaults**, attached at session time — never in prompts or agent config.
3. **Memory store** for cross-session persistence; know the three kinds of memory above.
4. **Coordinator + specialists:** give workers narrow tools; constrain instances/cost.
5. **Completion = structured event** (`session.status_idle`), not prose.
6. **Permission policy / networking** choices are security decisions, not defaults to copy.

## 9. Self-check
1. Why doesn't `Getting-Started.ipynb` resend the conversation each turn?
2. Session #2 recalls preferences from Session #1 using a *different agent*. What makes that possible?
3. Where does the GitHub token live, and why does it matter?
4. What signals that the agent has finished its turn?
5. When would you pick a plain workflow over a coordinator agent?

*Answers: (1) the session keeps state. (2) the shared memory store attached via `resources`. (3) in a vault, injected via `vault_ids` — never in prompts/agent config. (4) the `session.status_idle` event. (5) when the steps are known in advance.*
