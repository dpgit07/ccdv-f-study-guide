# Section 11 — Claude Code for Developers — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); the code below is my own adaptation of standard SDK patterns shown in the labs, with generic example names. Credit to the instructor.
**Lab files (in the course's lab repo):** `Claude-Code/` — three markdown files: `Claude.md`, `Testing-Skill.md`, `web-api-development.md`
**Exam weight:** the unofficial estimate puts the Claude Code domain at only ~3% (about 2 questions), but official
objective 5 ("run Claude Code under a permission model, give it durable project context, package workflows as Skills
and plugins") is broader. Light pass, but know the vocabulary.

> Source note: this section has **no Python lab**. The repo holds the instructions to paste into Claude Code, so these
> notes are conceptual. *(general knowledge)* = not shown in the files.

---

## 1. The workflow the section teaches (build → verify)
```
1. Put project rules in CLAUDE.md                  (durable context)
2. Prompt: "Using the project requirements defined in CLAUDE.md, build ..."   (task prompt points at the context)
3. Add a Skill (api-testing) for a repeatable task type
4. "Write pytest tests for every endpoint ..."      (Claude uses the skill)
5. pip install pytest httpx  →  pytest             (YOU run the tests: verification)
```
Project built: a Task Management REST API (FastAPI) with `GET/POST/PUT/DELETE /tasks`, in-memory storage.

---

## 2. CLAUDE.md = durable project context ⭐
The lab's `CLAUDE.md` holds stable, project-wide rules:
- stack: Python 3.13, FastAPI; Pydantic validation; type hints; PEP 8; REST best practices; JSON responses
- behaviour: **"Explain important implementation decisions"** and **"Ask for clarification whenever requirements are ambiguous"**

**What it is:** a file in the project that Claude Code loads at the start of a session, so you don't repeat the same
instructions in every prompt *(general knowledge)*. Notice how the task prompt just says "using the requirements in
CLAUDE.md", which is exactly the point: **put what's always true in CLAUDE.md, put what's true for this task in the prompt.**

**DE analogy:** CLAUDE.md is a pipeline's **shared config / coding standards file**; the prompt is the **job parameters**.

**Two judgment rules to remember**
1. **CLAUDE.md is guidance, not enforcement.** It *asks* Claude to follow conventions; it can't *guarantee* anything.
   Anything that must hold (don't touch production files, never run destructive commands) belongs in **permissions,
   hooks or code**, not in a markdown instruction. Same anti-pattern as "prompt-based enforcement of business rules".
2. **Keep it short and stable, and never put secrets in it.** It's loaded into context every session (context budget),
   and it usually lives in the repo.

---

## 3. Skills = on-demand procedures (`Testing-Skill.md`) ⭐
```
.claude/skills/api-testing/SKILL.md
---
name: api-testing
description: Write pytest test suites for a web API (use when asked to add or improve endpoint tests).
---
When to use: the user asks for endpoint tests, better coverage, or checks for invalid input.
Steps: 1. use the framework's test client  2. cover the happy path  3. cover invalid input and edge cases
       4. check status codes and response bodies  5. list any scenarios still untested
```
- Same structure as the skills in Sections 7 and 8: a folder with `SKILL.md` (`name` + `description` + instructions).
- **The `description` decides when the skill gets used**, just like tool descriptions and subagent descriptions.
  Vague description → skill not picked when it should be.

**CLAUDE.md vs Skill vs prompt: the distinction to be able to state**
| | Loaded | Holds | Example |
|---|---|---|---|
| **CLAUDE.md** | Every session (always-on) | Stable project facts and conventions | "Use FastAPI, PEP 8, type hints" |
| **Skill** | When the task matches its description *(general knowledge)* | A repeatable procedure for a task type | "How to write our pytest suites" |
| **Prompt** | This request only | The specific job | "Build the /tasks endpoints" |

---

## 4. Permissions (official objective 5): carried over from Section 7
This section's files don't show permissions, but the objective names them. Covered earlier:
- Agent SDK: `allowed_tools=[...]` (allow-list) and `permission_mode="acceptEdits"` (Section 7).
- From the Anthropic docs: modes `default`, `dontAsk`, `acceptEdits`, `bypassPermissions`, `plan`; a `canUseTool`
  callback for human approval; a **`PreToolUse` hook** to gate every tool call.
- **Principle:** least privilege. Pre-approve only what the task needs; require approval for anything risky.
  `bypassPermissions` is the "skip all prompts" end of the spectrum, so be deliberate about when it's ever acceptable.

**Not covered by any lab (check the docs if time allows):** plugins, and hooks beyond the one-line mention.

---

## 5. Likely exam angles *(my reading)*
1. **Where does this instruction belong?** Always-true project convention → CLAUDE.md; reusable task procedure → Skill; one-off → prompt; **must-never-be-violated rule → permissions/hooks/code**.
2. **CLAUDE.md can't enforce.** Distractor: "add 'never delete files' to CLAUDE.md to make it safe".
3. **Skill discovery is driven by its description.**
4. **You still verify.** Claude writes the tests; *you* run `pytest` and read the result.
5. **Ask-for-clarification instructions** reduce wrong assumptions on ambiguous requirements.

## 6. Self-check
1. A rule says "never modify files in `/prod`". Put it in CLAUDE.md, a Skill, or somewhere else? Why?
2. How does the task prompt in `web-api-development.md` avoid repeating the project's requirements?
3. What makes Claude choose to use the `api-testing` skill?
4. What's the difference between an `allowed_tools` allow-list and a `PreToolUse` hook?

*Answers: (1) somewhere enforceable (permissions/hook/code), because CLAUDE.md and Skills are guidance the model may not follow. (2) it points at CLAUDE.md, which Claude loads as durable context. (3) the skill's `description` (and its "use this when..." text) matching the request. (4) the allow-list pre-approves which tools can run; a hook is custom code that runs on each tool call and can allow, deny or modify it.*
