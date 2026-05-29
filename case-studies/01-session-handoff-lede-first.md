# Case Study 01: Session Handoff Lede-First Redesign

> **Status:** Review
> **Date:** 2026-05-07
> **Tags:** agent-coordination, handoff, context-management, session-continuity

---

## Context

The Overwatch platform runs AI agents continuously against production
infrastructure — sometimes for hours across multiple sessions. Because each
agent session has a finite context window, work is passed between sessions
via handoff documents: a structured artifact the next session reads before
doing anything else. These handoffs are the primary inheritance mechanism for
an in-flight multi-agent operation.

The system was mid-flight on a cluster upgrade sequence spanning multiple sessions
when a recurring failure mode surfaced with enough evidence to force a redesign.
Agents were consistently taking 20-30 prompts before producing real work at the
start of a session — not because the task was unclear, but because the handoff
was sending them into the wrong investigation path.

---

## Trigger

Session-011 (2026-05-07) opened on a backlog of in-flight PRs and issues that
had been touched in sessions 009 and 010. The operator observed: *"your handoffs
suck balls because they don't do a very good job of saying where to start up."*

Two concrete failures had just landed in close succession:

- A handoff described PR #150 as "queued, gate-stuck" — but PR #150 had merged
  two hours before the handoff was written. In-session memory had not been
  re-pulled at wrap time.
- A companion issue (OPS-403) was framed as "urgent platform gate that gates ALL
  sentinel-iac PRs." The next session treated it as a critical blocker and
  investigated accordingly. It was actually cosmetic; PR #150 had merged with
  the exact null-state pattern OPS-403 said would prevent merging. The framing
  had converted a resolved side-effect into a false-critical.

Session-011 compounded the problem by reading the older, richer session-009 log
first (because it contained more narrative detail) rather than the most recent
handoff. The result was approximately 30 minutes of planning from stale state
before pivoting.

---

## Diagnosis

Three distinct failure modes converged:

**1. Stale state at write time.** Handoffs were being written from in-session
memory rather than from a live API pull at wrap time. An agent that merged a PR
three prompts before writing the handoff accurately recalled "PR #150 is about
to merge" — but didn't re-query the API to confirm it had already landed. The
handoff propagated the pre-merge belief.

**2. No observation/inference distinction.** Statements like "this PR gates all
further sentinel-iac merges" appeared without a citation to the log line or API
response that proved the claim. The next session treated inferences as
observations and acted on them at face value.

**3. Narrative-heavy structure.** Prior handoffs led with session summaries:
"We shipped X, Y, Z. Key metrics improved. Outstanding items follow." This is
the natural shape of a written summary — but it is exactly wrong for an agent
picking up mid-flight. The agent reads the top of the document, forms a mental
model from the narrative, and by the time it reaches the actual next actions it
has already anchored to a world-state that may be weeks old.

A secondary failure: multiple handoff documents of varying age existed in the
issue tracker. Without a clear convention for which one was authoritative, agents
defaulted to whichever document was longest, assuming more content meant more
useful context.

---

## Action

The redesign had three components:

**Structural: lede-first ordering.** The handoff template was rewritten so the
first section — before any narrative — is a DO-FIRST block: verbatim commands
in execution order, with the exact tool invocation, a one-sentence "why", and
the expected observable outcome. An agent can paste and execute without
re-deriving anything. Background and narrative appear only after all actionable
content.

**Epistemic: OBS/INF tagging.** Every claim in the OBSERVATIONS section must
be tagged `[OBS]` (with a source citation: a log line, an API response, a direct
quote) or `[INF]` (hypothesis, untested, with explicit instructions for how to
verify before acting). Any "blocks", "fails", "gates", or "breaks" claim
without an `[OBS]` source is flagged as an inference and should be verified
before the next session acts on it.

**Operational: live-state pull at wrap time.** The template explicitly instructs
the agent writing the handoff to re-query every in-flight PR and issue via API
immediately before writing the IN-FLIGHT table. The table includes the query
timestamp. In-session memory is not a valid source for state that could have
changed.

A companion convention was added for reading: sort the issue tracker by
sequence_id descending, find the latest `[SESSION-HANDOFF ...]` issue, and read
that first — not older session logs, regardless of length. The SESSION-HANDOFF
is the highest-fidelity inheritance point; older logs are backstops.

The redesigned template was committed to `sentinel-cache/SESSION-HANDOFF-TEMPLATE.md`
and a durable rule indexed in the agent memory store to ensure all future sessions
inherit the convention automatically.

---

## Verification

Session-012 opened against the new template. The DO-FIRST block was executed
verbatim in the first three prompts. The live-state pull at wrap time for
session-011 had caught two state discrepancies that would have been invisible
in a memory-based write: one PR had transitioned to "changes requested" between
the agent's last action and the handoff write, and one Plane issue had been
closed by an operator action the agent had not observed.

The time-to-first-real-work metric — informally tracked as "prompts before the
agent produces a commit or a meaningful API call" — dropped from the 20-30 range
observed in sessions 009-011 to under 10 in sessions that followed the new
template. The DO-FIRST block was executable without rederivation in all
subsequent sessions where it was populated.

False-critical framing (converting resolved side-effects into new blockers)
was not observed in any session following introduction of the OBS/INF tag
requirement.

---

## Generalization

**The fundamental problem is that summaries are written for readers, not for
executors.** A session summary naturally leads with what was accomplished and
ends with "here's what's next." For a human reading a status report, this is
the right shape. For an agent that must resume execution in a constrained context
window, it is backwards: by the time the executor reaches the actionable content,
the framing from the narrative has already influenced the working hypothesis.

Reverse-pyramid structure — critical action at the top, supporting context
below — is not a stylistic preference. It is load-bearing for context-limited
readers. The same principle applies to any handoff where the recipient has limited
ability to re-derive context: runbooks, incident response docs, deployment checklists.

**Live-state pull is not optional.** Any handoff system that allows an author to
write from memory about the state of external systems will eventually propagate
stale state. The cost is proportional to how long the next consumer trusts the
handoff before going to primary sources. The fix is structural: make the
live-state pull a required step in the write protocol, not a best practice.

**Epistemic tagging prevents compounding errors.** When one session writes an
untested inference as a fact, the next session may act on it, fail, write a new
inference about why it failed, and so on. Requiring an explicit `[INF]` label —
with a "verify by" instruction — breaks the chain before the first compounding.
The overhead of tagging is trivially small compared to the cost of a session that
investigates the wrong thing for an hour because an earlier session stated a
guess as a fact.

**Index by recency, not richness.** In any system with multiple overlapping
documents covering the same subject (session logs, status issues, handoffs),
consumers will default to the document that appears most detailed. This
heuristic fails when older documents are richer but less current. An explicit
convention — "find the latest SESSION-HANDOFF by sequence_id and read that
first" — eliminates the heuristic entirely.

---

*Licensed under [CC BY-SA 4.0](../LICENSE).*
