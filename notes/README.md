# Notes index

Study notes written alongside a video course with hands-on labs ([course](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/)
by Kuljot Singh Bakshi, with his [public lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F)). **"Section N" means section N of that course.**
You don't need the course to use the notes. Each file explains the idea, shows the key code, and ends with likely exam angles and a self-check.

> Unofficial and AI-assisted. Items marked *(general)*, *(my reading)*, *(inferred)* or *(outside the labs)* are inference, not fact; verify them in Anthropic's docs.
> Code excerpts are abbreviated from the instructor's lab repo (credit to the instructor). They remain the instructor's work and are **not** covered by this repo's CC BY 4.0 license.

| Note | Topic |
|---|---|
| [section-03-getting-started.md](section-03-getting-started.md) | Messages API basics: roles, **statelessness**, streaming, images; why code instead of chat |
| [section-04-model-capabilities.md](section-04-model-capabilities.md) | Reasoning effort vs thinking, structured outputs, citations, **Batches API (async)** |
| [section-05-rag.md](section-05-rag.md) | Retrieval Augmented Generation: indexing vs runtime retrieval (optional topic) |
| [section-06-tool-use.md](section-06-tool-use.md) | **The tool-use loop**, client-side vs server-side tools, Tool Runner, memory, advisor, MCP connector |
| [mcp-building-a-server.md](mcp-building-a-server.md) | Building and connecting a remote MCP server |
| [section-07-agents-sdk.md](section-07-agents-sdk.md) | Agent SDK: `query()`, permissions, subagents, skills, agent as an API |
| [section-08-managed-agents.md](section-08-managed-agents.md) | Hosted agents: sessions, multi-agent, memory stores, **vaults** |
| [section-10-context-engineering.md](section-10-context-engineering.md) | Context budgets and tool search |
| [section-11-claude-code.md](section-11-claude-code.md) | CLAUDE.md vs Skills vs prompts; build-then-verify workflow |
| [section-12-security.md](section-12-security.md) | Security scaffold: injection, guardrails, hooks, secrets (the course had no lab here) |

There is no Section 9 note: it covered a third-party framework (LangChain) that isn't named in the exam blueprint.
