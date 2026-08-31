### 2026-08-28 — A subagent reported a deliverable "as delivered" that the parent never received; and asking research agents to judge *desirability*, not just feasibility, changes what comes back

Session shape: a long research-and-audit session in a non-code project. Around ten background
agents across two waves, all read-only investigation rather than implementation, plus one
cross-session relay.

**Finding 1 — a subagent's completion notification asserted it had delivered a full report; the
parent had only ever received a short summary. VERIFIED (observed directly in the parent's own
context).** An agent was resumed mid-task via a message relaying information it had asked a peer
for. Its final notification opened by saying the relayed content matched what it had already
incorporated, then stated that "the report stands as delivered" with a list of corrections applied
on top of it. No report body ever reached the parent — only that corrections summary. The parent
was left holding deltas against a document it did not have, and had to tell the user plainly that
it could not supply the underlying procedures.

The mechanism is worth naming because it is not the agent lying. A resumed agent appears to treat
work it did earlier in its own transcript as having been *communicated*, when only its final
message crosses the boundary. The longer the agent runs and the more it is resumed, the wider that
gap gets — the agent's sense of "what I have told you" tracks its own transcript, not the parent's.

**Rule concluded:** a resumed agent's final message must restate its deliverable in full, not
diff against earlier internal state. Worth putting in the agent definitions as an explicit
instruction ("your final message is the only thing that reaches the caller; do not reference
earlier work as though it was received"), and worth an orchestrator-side habit of noticing when a
completion summary references a document by allusion rather than containing it. The failure is
silent and reads as competence — a tidy corrections list looks like a *more* finished report than a
raw dump.

**Finding 2 — instructing research agents to answer two separate questions, "is it possible" and
"do we want it", produced decisions where the same agents would otherwise have produced
inventories. ASSERTED (I wrote the prompts and read the outputs; no controlled comparison).** The
user observed that their own project notes recorded exploratory thinking in the same confident
register as settled decisions, and asked for a re-audit on both axes. Prompts were written to give
each agent the project's stated design philosophy as the standard, and to explicitly license a
DROP recommendation. Agents then recommended dropping several technically-viable items on
philosophy grounds, and — more usefully — surfaced cases where a stated blocker was false but the
item still deserved deferring for a different and stronger reason.

The generalisable part: a research agent asked only "is X feasible" reliably answers that and
stops, because feasibility is the question. Handing it the constraints it should judge against, and
saying plainly that "drop this" is an allowed answer, is what converts a survey into a
recommendation. Without the explicit license, agents appear reluctant to recommend against
something they were asked to evaluate.

**Finding 3 — recurrence of the 2026-08-20 volunteered-context finding, in a new variant:
mis-framed *desirability*, not overstated breadth. VERIFIED (a second agent whose task actually
covered the area settled it with evidence).** An agent doing a health/integrity audit flagged a
configuration value as a problem, reasoning from what the system "is for". A different agent, whose
assigned scope was that configuration, established the value was a deliberate setting matching the
project's documented intent — the correct state. The first agent had no access to the design
context and offered a judgement that only design context could support.

This extends the existing entry rather than replacing it. The prior finding was that volunteered
claims skew toward overstating breadth; this one is an agent volunteering a *value* judgement
outside its evidence. Both share a root: the agent's assigned scope bounds what it can verify, but
nothing bounds what it will comment on.

**Rule concluded:** when fanning out audits over one system, expect scope-boundary contradictions
and plan for them — the orchestrator should hold a finding from agent A that lands in agent B's
scope until B reports, rather than relaying it. In this session that was done by instinct and the
instinct was right; it should be doctrine. Practically: a fan-out spec should tell each agent to
mark any observation outside its own scope as a lead for another lane, not a finding.

**Where it lives:** Finding 1 in the agent definitions (final-message contract) and in the
orchestration skill's section on resuming agents. Finding 2 in spec-writing guidance — a
research-task spec should name the standard to judge against and state which verdicts are
permitted. Finding 3 extends the 2026-08-20 entry's Verification section rule; the fan-out
scope-boundary rule is new and belongs beside it.
