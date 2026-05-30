# Case Study 02: Wabash.ai — The Agentic Civic Site Pattern

> **Status:** Review
> **Date:** 2026-05-29
> **Tags:** agent-authoring, civic-tech, community-ai, positioning, bsides-talk-prep

---

## Context

Wabash, Indiana is a county seat of approximately 10,000 people in north-central
Indiana with an unusual innovation history. In 1880 it became the first
electrically lit city in the world, running Charles Brush carbon-arc lamps from
the courthouse dome while the rest of the country used gas. Mark Honeywell,
founder of what became the global Honeywell Corporation, developed his early
boiler-control valve prototypes in Wabash County — components that remained in
service in pre-1950s homes throughout the area well into the modern era. Wabash
National, one of the largest semi-trailer manufacturers in North America, operates
there today. The Wedcor Industrial Park holds 230-plus acres adjacent to existing
infrastructure, is within a day's drive of most U.S. markets, and was selected
for Duke Energy's 2024 Site Readiness Program for industrial power delivery up
to 50 MW.

Despite this pedigree, no community resource existed that connected Wabash County's
innovation history to AI adoption for local businesses, or that positioned the
county within Indiana's emerging AI-readiness narrative. In April 2026, Governor
Mike Braun launched the IN AI initiative — a statewide effort run through the
Central Indiana Corporate Partnership (CICP) to frame Indiana as the most
AI-ready state in the country. Wabash County had no local node in that narrative.

The operator (a Wabash-based AI implementation consultant with existing relationships
in local economic development) wanted to change that — as a community initiative,
not a personal brand play.

---

## Trigger

The operator articulated the gap directly: *"spin up a website that promotes AI
in Wabash county and Wabash county as an AI-forward location for other businesses.
I will support them but also allow other businesses that either want to use AI or
are helping to implement AI. I don't care about being the person — I just want to
promote what's good for my community."*

Two conditions created the moment:

First, Indiana's IN AI initiative had launched weeks earlier. It provided a
credible state-level umbrella under which a county-level node could position
itself — not as a grass-roots outlier but as a local expression of a governor-led
economic program. The timing mattered: a community site launched *before* the
state narrative hardened would have the most positioning latitude.

Second, the Wabash Heartland Innovation Network (WHIN) was already operating a
ten-county smart-region initiative in the area. That meant infrastructure, policy
relationships, and a receptive audience already existed. The operator's intent was
to complement and cross-link WHIN, not compete with it.

The critical constraint was this: the initiative was explicitly *not* sanctioned
by Grow Wabash County, the county government, WHIN, Honeywell, or IN AI. It was
one person's community effort. The site would need to reference those organizations
respectfully and link to them, while making unmistakably clear that it was an
independent community initiative. The operator had existing relationships with
economic-development contacts (including Grow Wabash County's executive director)
and could not allow the site to appear to speak for those organizations.

---

## Diagnosis

The core design problem was positioning, not engineering.

A typical "AI for local business" site makes one of two errors. Either it is too
generic — essentially a rebranded vendor pitch deck with "Wabash County" inserted
into the intro — and offers nothing a local business couldn't find anywhere else.
Or it is too technical — an index of tools and APIs that assumes the audience
already knows what large language models are and how to use them.

The Wabash case required a third path: *civic narrative anchoring*. The pitch to
a Wabash County business owner or site-selection consultant is not "AI is the
future" (they've seen that). It is "the county where Honeywell invented temperature
control and where the lights came on 145 years ago is doing it again, and here
is how you participate."

Three design constraints emerged from this diagnosis:

**Heritage accuracy matters.** The Honeywell connection, the 1880 electrification,
the Wedcor site-readiness certification — these are verifiable claims, not marketing
language. Getting them wrong would undermine credibility with exactly the audience
(economic-development professionals, local business owners with long memories) who
would notice.

**Independence framing is not optional.** Any reasonable reader seeing the Honeywell
name and the IN AI logo on the same page would assume official affiliation. The
site needed explicit, prominent language: "independent community initiative, not
affiliated with" followed by a list of the organizations referenced. This is not
a legal formality; it is a relationship-preservation constraint with practical
stakes.

**The audience is not the BSides audience.** The case study is written for a BSides
talk audience — practitioners running AI agents in production infrastructure. The
*site itself* is not written for them. The pattern being demonstrated (using AI
agents to produce locally-informed, heritage-anchored civic content) is what the
BSides audience is interested in, and the wabash.ai instance is the concrete
example.

---

## Action

The operator registered the `wabash.ai` domain via Cloudflare. The Overwatch
platform — the same multi-agent infrastructure running CI/CD pipelines and
compliance automation for production systems — was turned toward the task of
building the site.

An agent session read the operator's articulation of the vision (quoted above),
the memory store's notes on Wabash County heritage anchors, and the IN AI
initiative's public materials. From these it drafted a site structure: a landing
page with the heritage narrative and the community positioning, a resources section
for local businesses new to AI, a directory path for businesses and consultants
operating in the area, and a clear "about this site / independent initiative"
footer on every page.

The content generation process was not generic. The agent was working from
operator-verified facts (the Honeywell prototype valves being in service locally,
the 1880 electrification date, the Wedcor Duke Energy certification) and explicit
operator framing ("I don't care about being the person"). The output reflected
that specificity — the site reads as Wabash-specific because the agent was
constrained to Wabash-specific inputs, not because a generic template was
search-and-replaced.

A static site on Cloudflare Pages was the implementation choice: low operational
cost, zero server maintenance, fast global delivery, and trivially auditable
content (it is just files). The same infrastructure that manages production IaC
can manage a static site without proportional operational overhead.

The site was registered as a Plane issue (OPS-408) and tracked through the same
work cycle — PLAN, CHANGE, VERIFICATION — as any other platform deliverable.
This was intentional: a community site built on a whim, outside the tracking
system, would be a one-session effort that drifted into disrepair. A tracked
deliverable with acceptance criteria gets the same attention as a CI pipeline fix.

The independent-initiative language was drafted by the agent and reviewed by the
operator before publication. The final form: a footer on every page reading
"Independent community initiative. Not affiliated with, endorsed by, or
representing Grow Wabash County, WHIN, Honeywell, IN AI, or any county, state,
or corporate entity referenced on this site."

---

## Verification

Verification for a civic positioning site is not a unit test. The signal sought
was: does a reasonable Wabash County business owner, reading this site without
context, understand what it is offering and who is behind it?

The operator reviewed the deployed site against three criteria: (1) heritage
claims are accurate and sourced, (2) the independence framing is prominent and
unambiguous, (3) the resources and directory path are actually useful to a local
business considering AI adoption rather than aspirational filler.

The Wedcor industrial-capacity figures (230+ acres, Duke Energy Site Readiness
for 2-50 MW delivery) were flagged during review as figures that should be
verified with Grow Wabash County before being represented as current. They were
moved from the body copy to a "learn more" reference pointing to Grow Wabash
County's contact information. This is the correct editorial posture for facts
that may change with economic-development timelines.

The site launched under OPS-408 tracking. The BSides talk-prep use case (OPS-415)
references it as a concrete example of the "agentic civic site" pattern — a
community-facing deliverable produced by the same multi-agent infrastructure that
runs production platform operations.

---

## Generalization

**AI agents can produce locally-specific content when they are given locally-specific
inputs.** The failure mode for AI-generated community content is generic output:
civic boilerplate that could apply to any county in any state. The antidote is not
better prompting — it is better source material. The agent that produced wabash.ai
content was working from operator-verified facts about a specific place. The output
was specific because the inputs were specific.

**The "agentic civic site" pattern.** A small region with a distinct heritage and
an economic-development gap can use AI agents to produce positioning content that
would otherwise require a PR agency engagement. The pattern has four components:
(1) a heritage narrative anchor (something verifiable and specific to the place),
(2) a current economic-development hook (a state initiative, a program, a certified
site), (3) an independent-initiative frame that protects existing relationships,
and (4) tracking the site as a platform deliverable so it does not drift into
disrepair.

**The same infrastructure, different workload.** The Overwatch platform runs
production infrastructure automation and a community civic site on the same
foundation — the same Vault credential management, the same Plane issue tracking,
the same work cycle discipline. This is not efficiency theater. It means that the
civic site gets the same operational rigor as the production pipeline, and that
the platform's capabilities are demonstrated on a workload that is legible to a
non-technical audience. For a BSides talk about AI agents in production, having a
real community site as a concrete artifact is more persuasive than an internal
metrics dashboard.

**Independence framing is an engineering constraint, not a legal afterthought.**
When an agent-built site references real organizations by name, the question of
affiliation is not a compliance checkbox. It is a relationship-preservation
constraint with real consequences for a real person. The site that says
"independent community initiative, not affiliated with [organization]" prominently
is not covering liability — it is preserving the operator's ability to walk into
a meeting with those organizations without a misunderstanding already in play.
Agents building content that references third parties should treat independence
framing as a first-class acceptance criterion, not a footer to add at the end.

---

*Licensed under [CC BY-SA 4.0](../LICENSE).*
