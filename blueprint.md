# Exam Blueprint Map: 8 domains, 24 skills, and where to study each

Domain and skill **weights come from Anthropic's *Claude Certified Developer – Foundations Exam Guide* v1.0 (effective July 2026)**.
The guide says it is subject to change, so check the current version. Descriptions below are **my own short paraphrases**, not the guide's
wording. "Study with" points to [notes/](notes/) (section numbers refer to the video course in the notes' headers) and to Anthropic's docs
(links checked November 2026; the docs move, so search by title if one breaks).

**Where the marks are:** Applications and Integration is a third of the exam on its own. Claude Code, configuration and "how Claude behaves across
interfaces" also sit inside it, so they count for far more than the 3.1% Claude Code domain suggests.

## Docs shortlist (grouped by topic)
- **Claude Code and configuration:** [Memory / CLAUDE.md](https://code.claude.com/docs/en/memory) · [Settings](https://code.claude.com/docs/en/settings-reference) · [.claude directory](https://code.claude.com/docs/en/claude-directory) · [Commands](https://code.claude.com/docs/en/commands) · [Extend Claude Code](https://code.claude.com/docs/en/features-overview) · [Glossary](https://code.claude.com/docs/en/glossary) · [Hooks guide](https://code.claude.com/docs/en/hooks-guide) · [Agent SDK + Claude Code features](https://code.claude.com/docs/en/agent-sdk/claude-code-features)
- **Prompting:** [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- **API, models, cost:** [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Rate limits](https://platform.claude.com/docs/en/api/rate-limits) · [Choosing a model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model) · [Optimizing for cost and intelligence](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence) · [Opus 5.5 migration guide](https://platform.claude.com/docs/en/models/opus-5-5/migration-guide) · [Claude in Amazon Bedrock](https://platform.claude.com/docs/en/build-with-claude/claude-in-amazon-bedrock) · [Claude on Vertex AI](https://platform.claude.com/docs/en/build-with-claude/claude-on-vertex-ai)
- **Security and permissions:** [Hooks guide](https://code.claude.com/docs/en/hooks-guide) · [Agent SDK permissions](https://platform.claude.com/docs/en/agent-sdk/permissions) · [Approvals and user input](https://code.claude.com/docs/en/agent-sdk/user-input)
- **Evals and errors:** [Define success criteria](https://platform.claude.com/docs/en/test-and-evaluate/define-success) · [Build evaluations](https://platform.claude.com/docs/en/test-and-evaluate/develop-tests)

---

## Domain 1: Agents and Workflows (14.7%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Agent Architecture | 4.5% | When to use a workflow vs an agent; supervisor/worker hierarchies; the role of subagents | [S7](notes/section-07-agents-sdk.md), [S8](notes/section-08-managed-agents.md) |
| Agent Construction with Claude | 5.3% | Agent SDK; custom loops/harnesses; hosted vs self-hosted agents; hooks for deterministic steps | [S6](notes/section-06-tool-use.md), S7, S8, hooks guide |
| Agent Patterns and Frameworks | 4.9% | Tool loops, subagents, memory, context management; abstraction frameworks (the guide names Strands, LangGraph and PydanticAI as examples) | S6–S8, [S10](notes/section-10-context-engineering.md); framework overviews |

## Domain 2: Applications and Integration (33.1%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Understanding Requirements | 3.4% | Turning business needs into functional and infrastructure requirements | general engineering experience |
| Systems Life Cycle | 2.8% | SDLC concepts and frameworks | general engineering experience |
| Claude API Mechanics | 6.8% | Messages, tools, streaming, vision, thinking, caching, third-party platforms (Bedrock/Vertex), batch vs realtime | [S3](notes/section-03-getting-started.md), [S4](notes/section-04-model-capabilities.md), S6; caching, Bedrock, Vertex docs |
| Software Engineering Foundations | 7.4% | REST, JSON, async programming, version control, code review, refactoring | general engineering experience |
| Claude Application Design | 8.6% | How instructions behave across Claude Code / Desktop / claude.ai / API / SDKs; content boundaries; schema design; session hygiene; plugin management | S3, S6 (schemas, statelessness); Claude Code docs cluster |
| Configuration Management | 4.1% | CLAUDE.md, `settings.json`, model-version pinning, prompt versioning, plugin dependencies | [S11](notes/section-11-claude-code.md); settings, memory and `.claude` docs |

## Domain 3: Claude Code (3.1%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Claude Code Operation | 3.1% | Rules, skills, commands, agents, agent memory; sessions; slash commands; non-interactive (headless) mode; CLAUDE.md hierarchy; settings | S11; Claude Code docs cluster |

## Domain 4: Eval, Testing, and Debugging (2.6%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Debugging and Error Handling | 2.6% | Identify error types; choose a recovery strategy; trace analysis; isolate integration faults from model-output faults | S4 (batch error types), S6 (tool errors); rate-limit and evals docs |

## Domain 5: Model Selection and Optimization (16.8%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| LLM Fundamentals | 5.2% | Tokens, context, sampling, non-determinism; fast mode, extended/adaptive thinking, effort; zero/single/multi-shot prompting | S3, S4; migration guide (temperature) |
| Technical Fundamentals | 6.1% | SDKs that wrap REST APIs; websockets | S3; SDK docs |
| Model Selection and Tradeoffs | 2.7% | Opus vs Sonnet vs Haiku; adaptive-thinking support; quality/latency/cost; breaking changes between releases | model docs; [S6](notes/section-06-tool-use.md) (advisor tool) |
| Cost and Token Management | 2.8% | Token budgets, usage tracking, cost modelling, prompt caching | S4 (batch, usage); caching and cost docs |

## Domain 6: Prompt and Context Engineering (11.0%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Context Engineering | 3.8% | Window management; preventing drift and bloat (prune tool output, compaction); isolation with subagents | [S10](notes/section-10-context-engineering.md), S7, S8 |
| Prompt Engineering | 4.6% | Clear instructions, few-shot, system vs user placement, constraints, iteration, input sanitisation | prompting best-practices doc |
| Output Handling | 2.6% | Structured outputs, validation, defensive parsing, scepticism toward confident output | S4 |

## Domain 7: Security and Safety (8.1%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| AI Application Security | 3.2% | Prompt injection, jailbreaks, untrusted input, data leakage, PII, authN/authZ, confidentiality | [S12](notes/section-12-security.md) (scaffold), docs |
| Guardrails and Safe Deployment | 2.3% | Content policy, layered guardrails, secure-by-design, IAM, least privilege | S12, S7 |
| Claude Hooks | 1.0% | Hooks that block destructive actions | hooks guide |
| Identity, Secrets, and Key Management | 1.6% | Secrets and keys across dev and prod; identity validation; access approval and monitoring | [S8](notes/section-08-managed-agents.md) (vaults) |

## Domain 8: Tools and MCPs (10.6%)
| Skill | Wt | What it covers | Study with |
|---|---|---|---|
| Tool Implementation | 4.4% | Function calling; tool descriptions; error handling; client vs server-side tools; approval patterns; tool-set design | S6 |
| MCP Server Development | 2.1% | Authoring and deploying servers; resources, tools and prompts; stdio vs HTTP transports | [mcp-building-a-server](notes/mcp-building-a-server.md) |
| Agentic Customization | 4.1% | Trade-offs among built-in tools, custom tools, Skills and MCP | S6–S8, S11 |

---
**Make it yours:** copy [templates/coverage-matrix.md](templates/coverage-matrix.md) and rate yourself 0–100% per skill. Study the skills with the most
weight and the least confidence first.
