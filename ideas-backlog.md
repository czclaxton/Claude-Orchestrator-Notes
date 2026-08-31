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

### 7. Personas / profiles — preset modes that change how the session communicates

**Status: closed — step 0 came back "yes, the platform already does this." Not built, and not to be built.** Kept because the risk analysis below applies to the output-style mechanism now in use. Originally recorded
now because the seed evidence is concrete and would otherwise be lost.

**The idea.** Named preset personas the user toggles into, each changing how the session explains
itself. First and clearest case: a **layman's-terms mode** — plain language, analogies, no jargon —
for when the user is learning something complex or being caught up on unfamiliar ground. Other
personas to be discovered from real use rather than invented up front.

**Seed evidence (2026-08-20).** During a plugin-maintenance session the user asked, mid-thread, for
a technical status report to be re-delivered "in layman's terms as if I'm a non-technical investor."
The re-delivery was materially more useful to them than the original. That is a real, observed
demand for a register shift — not a hypothetical. It is also the entire case for the feature, and
one data point is one data point.

**Step 0 before building anything: find out whether the platform already does this.** Claude Code
may already ship a built-in mechanism for exactly this (output styles, or similar). Verify
empirically in a scratch session — do not assume in either direction, per the standing rule that
burned several confident-but-wrong assertions in this repo already. If a built-in covers it, the
plugin should document the built-in and build nothing. Duplicating a platform feature inside a
plugin is the worst outcome available here: ongoing maintenance for zero differentiation.

**Why it might be counter-productive — the risks, in priority order:**

1. **Register/substance conflation. This is the one that matters.** A prompt that says "be simple"
   reliably produces more than simple *vocabulary* — it drops caveats, flattens uncertainty, and
   sounds more confident than the evidence supports. For a user whose stated purpose is *learning*,
   that is the worst possible failure: they walk away with a clean, memorable, wrong model and no
   signal that anything was elided. Directly parallels the `/catch-up` remote-state finding — a
   confident simplification terminates further questioning exactly like a stale negative does.
   **Any persona must be able to change how something is said and never what is true, what was
   verified, or which caveats survive.**
2. **Analogy drift.** A persona instructed to use "lots of analogies" will manufacture them where
   none fit, and a bad analogy is worse than no analogy because it is memorable and load-bearing in
   the reader's head. The rule should be *analogy when it genuinely maps, plain language always* —
   never an analogy quota.
3. **Invisible mode state.** A toggle that silently changes behavior is a problem when the user
   forgets what is active, or when context compaction drops the fact mid-session. Note this repo
   already solved exactly this shape for testing mode (explicit toggle command + marker file +
   `SessionStart` hook injecting the active state into context). Reuse that pattern rather than
   re-deriving it.
4. **Proliferation.** "More personas as we go" is how this becomes twelve half-specified overlapping
   modes that nobody uses. Same promotion discipline as everything else here: one persona, proven in
   real use across multiple sessions, before a second exists.
5. **Collision with the orchestration doctrine.** The doctrine mandates evidence-bearing reporting —
   command output, verification transcripts, explicit unverified/verified labels. A communication
   persona must sit *above* that layer and cannot be allowed to suppress it. Worth stating as an
   explicit precedence rule if this is ever built, since the repo has already been bitten twice by
   two of its own instructions colliding with no stated precedence.

**Why it might genuinely be worth it.** Everything above is achievable today by simply asking, so
the feature buys three things and should be judged only on them: **persistence** (set once, applies
all session, no re-asking every turn), **consistency** (a stable register instead of drift back to
default after a few exchanges), and **composability** (a persona could plausibly alter more than
prose — a learning mode might also mean narrate routing decisions instead of delegating silently,
and prefer showing the work over reporting the result). The third is the only one that could not be
replicated by a saved snippet of text, so it is the real test of whether this deserves to be a
plugin feature rather than a habit.

**How to implement, if step 0 says build it.** Mirror the testing-mode toggle exactly — it is built,
shipped, and validated: a `/claude-orchestrator:persona <name|off|status>` command writing a marker
file, plus `SessionStart` hook injection so the active persona is visible in context every session.
Session-scoped rather than per-project, since the need is situational (the same user wants plain
language while learning and dense evidence while reviewing, in the same repo, hours apart).

**The falsifiable test for risk 1**, which should gate promotion: take a substantive technical answer,
produce it in default mode and in persona mode, and diff them for *dropped facts, dropped hedges, and
confidence inflation* — not for readability. If the persona version loses caveats rather than
jargon, the persona is broken regardless of how good it reads. Readability is the easy half and the
half that will look fine.

**Step 0 — resolved 2026-08-31: yes, Claude Code already does this.** Output styles (`~/.claude/output-styles/<name>.md`, selected via `"outputStyle"` in `~/.claude/settings.json`) are the built-in mechanism, and one is in active use on this account. Per this entry's own rule — *if a built-in covers it, document the built-in and build nothing* — the feature is closed. What survives is the analysis: **risk 1 (register/substance conflation) applies to the installed output style exactly as it would have to a custom persona**, and the falsifiable test at the end of this entry is still the right gate. Output styles are also documented as not reaching subagents, which bounds what any such mechanism can fix.

## Log (append-only — one entry per real test of one of these ideas)

*(empty — nothing tested yet)*
