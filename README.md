# Claude Certified Developer – Foundations (CCDV-F): A Practical Study Guide

I passed this exam on my first attempt with **926 / 1000** (pass mark 720). This repo is what I'd hand to a colleague who
asked, "How do I prepare?": the exam in brief, a map of what it measures, my notes, a study plan, and templates for tracking
your own gaps.

> **Unofficial.** Not affiliated with or endorsed by Anthropic. Everything here is my own interpretation; verify details against
> Anthropic's [Exam Guide](https://anthropic-partners.skilljar.com/claude-certified-developer-foundations-certification) and docs
> (the exam guide says it is subject to change). **No exam questions are included or discussed**; the exam is under a non-disclosure agreement.
> The notes were drafted with help from an AI assistant (Claude Code) working from course labs and Anthropic's docs. Claims marked
> *(general)*, *(my reading)* or *(inferred)* are inference, not fact. Please check them.

## Who this is for
Developers and engineers who build with Claude (APIs, tools, agents, Claude Code) and want to validate it. Anthropic recommends 1–5 years
of engineering experience, about 6 months of hands-on LLM work, and Python/TypeScript. I came from **data engineering**, so the notes lean on
analogies to ETL, batch jobs, UDFs, JDBC drivers and IAM; they help if you think that way, and are skippable if you don't.

## The exam at a glance
| | |
|---|---|
| Format | 53 items, multiple-choice and multiple-response (each item states how many answers to select) |
| Time | 120 minutes (about 2 min 15 s per item), proctored online or at a Pearson VUE test centre |
| Passing score | 720 on a scale of 100–1,000 |
| Fee | $125 per attempt; retake waits of 14 / 30 / 90 days after the 1st / 2nd / 3rd fail |
| Validity | 12 months; free non-proctored renewal if done before it lapses |
| Report | Pass/fail, scaled score, and percent-correct per domain |

## What it measures: 8 domains
| Domain | Weight |
|---|---|
| Applications and Integration | **33.1%** |
| Model Selection and Optimization | 16.8% |
| Agents and Workflows | 14.7% |
| Prompt and Context Engineering | 11.0% |
| Tools and MCPs | 10.6% |
| Security and Safety | 8.1% |
| Claude Code | 3.1% |
| Eval, Testing, and Debugging | 2.6% |

The full skill-level breakdown (24 skills) with where to study each is in **[blueprint.md](blueprint.md)**.

### Things the blueprint tells you that surprised me
1. **Claude Code matters more than its 3.1% suggests.** CLAUDE.md, `settings.json`, plugins, and "how instructions behave across Claude Code,
   Desktop, claude.ai, the API and SDKs" also appear inside Applications and Integration. Together roughly 16–20% of the exam.
2. **Eval/Testing is the smallest domain (2.6%)**, and its one skill is debugging and error handling. Don't over-invest there because a prep course does.
3. **It's a judgment exam.** It rewards choosing the right pattern for a scenario (and spotting the wrong ones), not memorising syntax.

## A study plan that worked
About **35–40 hours** in total by my estimate. I did roughly 30 minutes a day with breaks; at 5–10 h/week it's 4–8 weeks.

| Phase | What | Rough time |
|---|---|---|
| 1. Read the blueprint | Download Anthropic's Exam Guide. Skim [blueprint.md](blueprint.md). Note which skills you already use at work | 1 h |
| 2. Learn by doing | A hands-on course with labs (I used a ~9.5 h video course; Anthropic's own prep course is free). Run or read every lab | ~10–12 h |
| 3. Write notes per topic | For each topic: *when do I use this, and when don't I?* [notes/](notes/) shows the format | ongoing |
| 4. Close gaps from the docs | Use [blueprint.md](blueprint.md): the Claude Code/configuration docs first, then prompting, API/models/cost, security | ~10 h |
| 5. Practice exams, timed | **Independent** sets (not only your course's own test), closed-book, domain breakdown on. Log every miss | ~8 h |
| 6. Book and rehearse | Do a final timed run; take care of logistics | 2 h |

## How to use practice exams
- A course's own practice test mostly measures how well *that course* landed. Mine scored 94%; independent sets scored **73%, 64%, 83%, 86%**.
- **Keep a wrong-answer log** ([template](templates/wrong-answer-log.md)). For each miss, note whether it was a *concept gap*, a *misread*, a *trap* or a *guess*. That tells you whether to study more or read more slowly.
- **My booking rule** (my own heuristic, not Anthropic's): about 80% or better on two independent sets, taken **timed and closed-book**, with every domain around 75% or higher.
- Quality of practice material varies. Prefer sets that explain every answer and show a domain breakdown. Don't memorise questions.

## Decision rules worth internalising
These came up again and again in the notes and in practice material.
- Claude is **stateless**: conversation "memory" is the history you resend. Context budgets, memory tools and subagents all follow from this.
- A tool call is a **request**: Claude asks, your code (or the platform) executes. Know who owns the loop. *Any client-side tool means you own a loop.*
- **Critical rules go in code, permissions or hooks**, not in a prompt. Prompt text, a bigger model or a different `temperature` is never the fix for security or correctness.
- **Use a structured signal to end a loop**, not Claude's prose; keep an iteration cap only as a backstop.
- **Pick the simplest thing that works:** the Batches API for non-urgent bulk jobs, a fixed workflow before an agent, a few well-described tools over many.
- **Secrets never go in prompts or config**; use a vault or secret store and least-privilege tokens.
- **Treat anything the model reads** (web pages, documents, tool results) **as untrusted input.**
- **Watch for breaking changes:** for example, newer models reject `temperature`, `top_p` and `top_k`. Tutorials lag the docs.

## What's in this repo
| Path | What |
|---|---|
| [blueprint.md](blueprint.md) | The 8 domains and 24 skills, paraphrased, with where to study each |
| [notes/](notes/) | Topic notes, each ending with likely exam angles and a self-check ([index](notes/README.md)) |
| [templates/](templates/) | A wrong-answer log and a coverage matrix to copy |

## Exam-day logistics (from the Exam Guide)
- Your government photo ID must match your registration name **exactly**.
- Book through the Anthropic Partner Academy, then your Pearson VUE account. You can cancel or reschedule up to 24 hours before.
- Online proctoring: run the system check early; keep the desk clear (no notes, phones or second monitor). Accommodations must be approved *before* you schedule.
- Don't share exam content afterwards; it can invalidate your result.

## Resources
- Official: [certification page and Exam Guide](https://anthropic-partners.skilljar.com/claude-certified-developer-foundations-certification) · [free prep course](https://anthropic-partners.skilljar.com/path/claude-certified-developer-foundations) · [Claude docs](https://platform.claude.com/docs) · [Claude Code docs](https://code.claude.com/docs)
- Course I used: [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi, with his [public lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F). The course is good for building a habit and hands-on feel; it's light on a few blueprint skills (see [blueprint.md](blueprint.md)).

Found something wrong, outdated or unclear? Open an issue or pull request. The platform changes quickly, so corrections are welcome.

## License
The written material in this repo (notes, blueprint map, templates, README) is licensed under
[**Creative Commons Attribution 4.0 International (CC BY 4.0)**](https://creativecommons.org/licenses/by/4.0/): share and adapt freely, including commercially,
with credit to the author, a link to the license, and a note of any changes. See [LICENSE](LICENSE).

**Not covered by that license:** code excerpts quoted from the course instructor's lab repo (they remain the instructor's), and Anthropic's documentation,
exam guide, names and trademarks (they remain Anthropic's).

*Author: [@dpgit07](https://github.com/dpgit07)*
