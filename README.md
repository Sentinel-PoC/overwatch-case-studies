# Overwatch Case Studies

Case studies and writeups from the Overwatch agentic-operations platform.
These are real incidents, real mistakes, and real wins — written to be useful
to anyone running AI agents in production, not just to ourselves.

Licensed under [CC BY-SA 4.0](LICENSE).

---

## Case Study Index

| # | Slug | Title | Status | Suggested Reading |
|---|------|-------|--------|-------------------|
| 01 | [session-handoff-lede-first](case-studies/01-session-handoff-lede-first.md) | Session Handoff Lede-First Redesign | Review | Start here — establishes core handoff vocabulary |
| 02 | [wabash-ai-community-ai-positioning](case-studies/02-wabash-ai-community-ai-positioning.md) | Wabash.ai — The Agentic Civic Site Pattern | Review | Agent-authored civic content; independence framing as engineering constraint |
| 03 | [trivy-image-pr-scope-down](case-studies/03-trivy-image-pr-scope-down.md) | Splitting an Expensive Security Scan into Fast + Nightly | Review | Fast-PR-loop + slow-nightly pattern; agent applies engineering judgment to ops cost |

Reading order: start from the top. Each entry is self-contained, but earlier
ones tend to establish vocabulary that later ones build on.

---

## Authoring a New Case Study

1. Copy `TEMPLATE.md` to `case-studies/NN-your-slug.md` (NN = next available number).
2. Fill in every section. Short and specific beats long and vague.
3. Add a row to the index table above.
4. Open a PR — see [CONTRIBUTING.md](CONTRIBUTING.md) for the full workflow.

---

## What This Repo Is

The Overwatch platform runs AI agents autonomously against production
infrastructure. This repo collects the moments worth writing up: the
surprising failures, the patterns that keep recurring, the times an agent
did something unexpected (good or bad), and what we changed as a result.

The goal is to produce material that is honest and transferable — useful
to a practitioner reading it without knowing our specific setup.
