# Case Study 03: Splitting an Expensive Security Scan into Fast + Nightly

> **Status:** Review
> **Date:** 2026-05-29
> **Tags:** ci-optimization, security-scanning, agent-engineering-judgment, cost-reduction

---

## Context

An agentic operations platform runs a multi-job CI security pipeline on every
pull request against its infrastructure-as-code repository. One job —
a container image vulnerability scan — was sweeping an entire private registry
on every PR, regardless of whether the PR touched any image references at all.
The scan was a required status check; nothing could merge until it passed.
At 16 minutes and 14 seconds average runtime, it was the longest job in the
pipeline by a wide margin, and it ran identically whether the PR changed one
YAML line or a hundred image tags.

---

## Trigger

The operator's dashboard showed PR CI times consistently above 16 minutes.
An agent reviewing the pipeline configuration traced the bottleneck to a single
job: a full registry sweep that called a shell script iterating over every
project in a private Harbor instance. The job had been written when the
registry was small and the CI pipeline was new. As the registry grew, the
job's runtime grew with it — but the job's trigger condition never changed.

No alert fired. The signal was the wall clock: every engineer on every PR
waited the same 16 minutes, even when the PR had nothing to do with container
images.

---

## Diagnosis

The agent read the CI workflow file and the shell script it called. The key
observation was structural: the job had one code path regardless of inputs.
It always ran the full registry sweep. There was no early-exit, no scope
check, no way to short-circuit.

The agent then asked: what does a typical PR actually change? In the
infrastructure-as-code repo, most PRs touch Ansible roles, Kubernetes
manifests, or Helm chart values. Only a minority reference container image
tags directly. The full registry sweep was providing accurate security
coverage for those image-referencing PRs, but was providing redundant
coverage — at full cost — for all the others.

The root cause was not a bug. It was a design assumption that had aged out:
"every PR might change an image" was true when the repo was small and mixed,
but was no longer a useful proxy for "this PR actually changed an image."

No dead ends. The diagnosis was clear from the code: unconditional full sweep,
no diff inspection.

---

## Action

The agent split the single job into two jobs with different trigger conditions:

**Job 1 — fast PR scan (`trivy-image-pr`):**
- Triggers on `pull_request` events only.
- Before scanning, runs a `git diff` against the PR's changed files.
- Inspects only YAML files under the paths where image references live
  (`apps/`, `ansible/`, `deployments/`, `charts/`, `manifests/`).
- Extracts Harbor image references from those diffs using a short inline
  script.
- If no Harbor image refs appear in the diff, the job prints a one-line
  notice and exits with success. This is the skip path.
- If refs are found, it scans only those specific images — not the full
  registry — with a 5-minute timeout, reporting HIGH and CRITICAL
  severity only.
- No upload to the defect-tracking system (that is reserved for the
  authoritative nightly run).

**Job 2 — full nightly scan (`trivy-image-nightly`):**
- Triggers on scheduled cron (`0 2 * * *` UTC) and on pushes to the
  default branch.
- Identical behavior to the original job: full registry sweep via the
  existing shell script, 10-minute timeout, upload to the defect-tracking
  system.
- This is the authoritative vulnerability record. Nothing changed here.

The branch-protection required-status-check was updated to track the new
PR-targeted job name instead of the old one. This was a necessary mechanical
step: the old job name no longer existed, so without this update the PR gate
would have been broken for all subsequent PRs.

The workflow file changed by +105 lines and -2 lines. The shell script that
performs the actual scanning was not modified.

---

## Verification

The PR CI run produced a concrete timing artifact: the new `trivy-image-pr`
job completed in **11 seconds** on a PR whose diff contained no Harbor image
references — the skip path exercised correctly. The old job's baseline for
the same class of PR was **16 minutes and 14 seconds**, an 88x reduction for
the common case.

The agent verified three things by inspection before the PR was opened:

1. The workflow YAML parsed without errors (`python3 -c 'import yaml; yaml.safe_load(...)'`).
2. A simulated diff extraction against a known runbook-only PR correctly
   short-circuited with "no scoped YAML changes."
3. No remaining references to the old job name existed in the workflow file.

The judge workflow ran a full CI pass against the PR branch, confirmed 11
required checks passed (4 lint, 7 security scans including the new
`trivy-image-pr` context), and approved the merge. The defect-tracking health
check dependency chain was updated automatically as part of the same commit.

---

## Generalization

**The pattern: fast PR loop + slow scheduled sweep.**

Any expensive scan that has an "authoritative" result (the kind you upload to
a compliance or tracking system) and a "gating" result (the kind that blocks
a PR) is a candidate for this split. The two roles have different requirements:

- The PR gate needs to be fast and scoped. It only needs to answer: "does
  *this change* introduce a new problem?" It does not need to certify the
  entire system is clean.
- The nightly sweep needs to be thorough and unconditional. It answers:
  "is anything in the current state of the system vulnerable?" It is the
  authoritative artifact that matters for compliance and tracking.

Running the authoritative sweep on every PR is common and understandable —
it is the conservative choice when you first set up the pipeline. But it
conflates two distinct questions and charges the cost of the thorough answer
every time you only need the scoped one.

The split pays off when: (a) most PRs do not touch the scanned artifact, and
(b) the scan's runtime scales with the size of the artifact, not the size
of the diff. Both conditions held here. If your scan always takes the same
time regardless of what changed, the split adds complexity without saving time.

**On the branch-protection update:** when a required job is renamed, the
protection rule must be updated atomically with the rename — or a gap exists
where PRs either block forever (old name still required, new name never
appears) or merge without the check (if the requirement was removed before
the new name was added). In this case the agent included the protection
update as a documented step in the PR description and executed it immediately
after merge. A safer pattern is to run old and new jobs in parallel for one
cycle before removing the old one, so the required-check list can be updated
to the new name while the old name still exists. That approach costs one
extra CI run but eliminates the gap entirely.

**Agent engineering judgment, not checklist execution:** the agent did not
receive a ticket saying "split this job in two." It received a ticket saying
"this job is slow." The structural observation — that the job's trigger
condition was decoupled from the actual scope of what most PRs change —
required reading the code and reasoning about the system's use patterns. This
is the kind of judgment that distinguishes useful automation from automation
that merely executes what it is told.

---

*Licensed under [CC BY-SA 4.0](../LICENSE).*
