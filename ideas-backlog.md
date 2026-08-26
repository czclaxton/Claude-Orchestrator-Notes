# Claude Orchestrator — ideas backlog

Living doc: short framework, append-only log below it. The framework gets refined *from* the log,
not the other way around — don't let this calcify into a spec nobody revisits. Modeled on
`docs/lessons.md` and `docs/browser-automation-strategy.md` from the Pepco project (inspiration,
not copied wholesale — Claude Orchestrator is a portable plugin, Pepco's docs are single-project
working notes, and that difference matters for what belongs here vs. in a per-project doc).

Nothing in this file is a committed feature. Test in real use (Connor, across other projects),
promote to the actual plugin (`agents/*.md`, `skills/orchestration/SKILL.md`) only once an idea
has proven out — mirroring the review-gap ledger's promotion discipline below.

## Ideas, not yet built

### 1. Phased workflow with checkpoints
Architect plans the task as N phases, clarifies ambiguity up front, delegates one phase at a time.
User reviews and confirms after each phase before the next starts — small-batch review instead of
one big diff at the end. **Decided:** if a phase needs mid-task guidance, that goes back through
the architect as an amended spec — not a live chat with the implementer. Reasoning: the reliability
research showed failure modes clustering around long, evolving, live-judgment sessions; turning a
bounded implementer into a live conversational partner mid-phase reintroduces exactly that risk on
a cheaper model with less headroom for it.

### 2. Persistent phase handoff file
Each implementer already returns a structured report. Extend it: also persist that report to a
per-phase file (not just chat output), so it survives a session boundary. Modeled on Pepco's
`resume-prompt.md`: single canonical file, current-only (overwritten, not accumulated), states the
immediate next action, separates closed-out from carried-forward, and — the property that matters
most — explicitly tells whoever resumes to **re-verify facts against live state**, not trust the
file blindly. A stale handoff trusted uncritically is worse than none.

### 3. Friction/lessons log — built, see `lessons.md`
Originally scoped here as a narrower "routing-decision ledger." Broadened and actually built as
`lessons.md` (developer-only, gitignored, never shipped) — general friction from real use of the
agents, not just routing outcomes, following the same promotion discipline described there (a
logged entry only becomes an actual `SKILL.md`/agent edit once it recurs or is severe enough to
justify alone). Answers the earlier "feed a failure to the next tier and dynamically adjust
routing" idea as a manual, periodic review ritual — no infrastructure here supports live automatic
learning, and none is being built. See `lessons.md` for the real mechanism; a future opt-in path
for other testers to contribute their own equivalent log is noted there too, with the privacy
problem it would raise flagged before it's needed.

### 4. Browser automation tools (Chrome DevTools MCP, Playwright MCP, Claude in Chrome)
**Recommendation: split generic from project-specific, don't hard-integrate wholesale.**

Pepco's `browser-automation-strategy.md` already worked out a solid Step-0 decision framework:
- Real external account/session needed → Claude in Chrome
- "Why slow" / "is this leaking" / Lighthouse → Chrome DevTools MCP (only one of the three that
  does real perf tracing / heap snapshots / Lighthouse)
- Repeatable scripted flow, or priming mock auth non-interactively → Playwright MCP
- One-off manual check, no real auth → Chrome DevTools MCP by default (cheaper, purpose-built)

That framework itself is already project-agnostic — worth adopting close to verbatim as a short
addendum to the orchestration doctrine or a small companion skill.

What's genuinely worth building into the plugin (a reusable safety pattern, not just a doc):
a **dedicated, tool-restricted verification subagent** specifically for Claude-in-Chrome-class
work — it's the one tool with real-session/real-account exposure, so it should be hard-scoped
(allowlist only the specific tools a check needs) rather than called directly from an implementer
or the architect. This also cuts context cost for free, same as our existing implementer lanes:
a subagent's context is forked and discarded on return.

What should **not** go into the portable plugin: project-specific specifics (ports, auth shape,
which pages, CORS config) — Pepco's own doc explicitly defers exactly this kind of thing for
exactly this reason ("deliberately not self-contained/portable yet... done once the content
stabilizes and there's a second real project to test portability against"). Each project keeps its
own usage log; only patterns that generalize across projects get promoted into Claude Orchestrator
itself — same promotion discipline as #3.

One concrete, already-evidenced default worth adopting immediately, not waiting to re-discover:
**prefer structured/text reads over screenshots** wherever these tools are used. Pepco measured
this directly — screenshot-heavy Claude in Chrome usage ate ~73% of one conversation's context, and
a stale/lagging screenshot produced a false-positive bug report that a text read would have caught
immediately. Cheap to state as a default now.

### 5. The "chief of staff" assistant — see `assistant-design-brief.md`
A full design conversation (2026-08-21/22) for a tool that manages context *and* the protocol for
relaying information and taking high-level guidance. Reframes the goal from a memory system to a
**communication protocol**, organized around four modules: Routing (already built — this plugin),
Translation (layered explanations), Continuity (the thread record), Delegation (decision boundaries
accumulated from precedent). Not built, not specced, and it is **not yet decided whether this
extends Claude Orchestrator or is a separate tool** — Connor explicitly left that open.

Kept in its own file rather than summarized here because the reasoning is the valuable part: several
positions were reversed mid-conversation by his pushback, and the brief records what was confirmed,
what was corrected, and what was only proposed. Four questions are genuinely unanswered, including
the one the session ended on — how aggressively the system may interview him, given that asking
spends the attention the tool exists to conserve.

**Relevant to this plugin regardless of where it lands:** the critique that a feedback log with no
read path is a diary (see `lessons.md`, 2026-08-21 entry), the *when X, do Y* test for whether a
logged lesson can bind behavior, and the soft-rules-vs-hooks enforcement split that overlaps the
research in notes PR #9.

## Log (append-only — one entry per real test of one of these ideas)

*(empty — nothing tested yet)*
