# First real validation run — Hang-Up Log, sandboxed off Pepco

Written before the run (2026-08-19), so it measures evidence instead of a vibe.
See `C:\Users\Zane\.claude\plans\resume-imperative-nygaard.md` for the full plan
this brief supports — this file is only the capture checklist.

## Why this run

Claude Orchestrator has never been used on real work. `lessons.md` has two
entries, both about tooling. `ideas-backlog.md`'s log is empty. Four open design
questions all reduce to "how does this behave under real delegation?" — this run
is the first chance to answer any of them with evidence instead of reasoning.

**Context:** Connor upgraded to Max 5x on 2026-08-19, mid-planning — this run
validates the full four-agent ladder (Sonnet/Opus/Fable ×2), not a Pro-degraded
two-rung version. Fable is included up to 50% of the weekly limit.

**Setup:** cloned off Pepco's `develop`, no `origin`, Son-of-Anton doctrine
stripped out (Pepco itself runs SoA — mined for ideas per Connor's call, not
adopted; irrelevant to this sandbox). Task: the Hang-Up Log grouped/duplicates
view, split into a well-specified build (3a) and a genuinely open spec session
(3b) — see the plan for the four parked design questions.

## What to capture during the run

### Routing decisions (log every one)
- Task → lane chosen → stated reason → correct in hindsight? (verification
  agreed, or a corrected spec/escalation was needed)
- Any lane that failed its spec: what happened, whether the corrected-spec →
  escalate-on-second-failure rule actually fired as written, how many round
  trips it took (plan sets a hard cap of 3 before forced escalation)

### The spec contract
- Was the five-part spec (objective/files/interfaces/constraints/verification)
  awkward or insufficient anywhere — especially for `critical-implementer`
  (Fable)? This is the direct test of the open "loosen it for Fable" question.
  Anthropic's own migration docs already suggest over-prescriptive prompts hurt
  Fable's output quality — does that show up here?

### The planning phase (never exercised before this run)
- On the Hang-Up Log's 4 open questions (count-vs-addressed, resolved-replaces-
  time-window, per-call-vs-per-number, notes-is-same-feature): did the architect
  ask good leading questions, or quietly guess? Did the output correctly split
  three ways — decided now / needs the client / needs Conrad — rather than
  silently deciding something that wasn't the architect's to decide?

### Phase boundaries (evidence for backlog #1/#2 — don't build either preemptively)
- Any point where a checkpoint before continuing would have caught a wrong
  direction earlier, or where a persisted handoff file would have mattered

### Model pins under real multi-step delegation
- Closes out the 2026-08-11 smoke test, which was one trivial single-turn
  prompt per agent — does each lane actually run on its pinned model across a
  longer, real task? (advisor + critical-implementer → Fable, complex-implementer
  → Opus, routine-implementer → Sonnet)

### Fable budget (new this run, thanks to the Max upgrade)
- Running estimate of Fable share of the weekly limit consumed by this one run
  — the 50% cap is the real-world version of the cost-discipline doctrine's
  "spend the premium only where it changes outcomes." If a single validation run
  eats a large chunk of it, that's a finding about the doctrine's real-world
  headroom, not just an inconvenience.

### The final review
- Did `advisor`'s mandatory end-of-deliverable pass catch anything the architect
  missed? Or was it a rubber stamp — also worth knowing honestly.

## After the run

Write the actual `lessons.md` entries here from this checklist, then re-open the
four design questions in `project_claude_orchestrator.md` with real evidence
behind them instead of secondhand research. Decide separately whether Max 5x is
worth it for Connor's general daily use (likely yes) versus whether this plugin
specifically needed it (this run is the test).
