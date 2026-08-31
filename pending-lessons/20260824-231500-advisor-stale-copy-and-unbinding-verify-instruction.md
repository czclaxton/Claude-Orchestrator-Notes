### 2026-08-24 (second session) — The advisor was handed a stale working copy by default, ignored an explicit instruction to verify a supplied claim, and produced a false finding from an orchestrator artifact that conversation had already superseded

**Status: findings 1, 2, 3 and 5 verified** (each re-checked directly — the stale-copy gap by
comparing the working tree against the server, the ignored instruction by reading the advisor's own
citation, the superseded-source finding by reading the provenance tags in the source file, the
research corrections by re-reading the cited documentation). **Finding 4 asserted.**

**Context:** a long design session with three read-only lanes dispatched — two research lanes with
network access, one advisor review. No implementation lanes. The user caught two of the five findings
below before the system did.

**Finding 1 — a read-only lane's view of a repository is the local working copy, and nothing in the
delegation path checks whether that copy is current.** The advisor was dispatched to review a plan
against the shipped text of two commands. Its tools are file-reading only, so it would have read the
working tree — which was two commits behind the server, a state `/catch-up` had already reported at
session start and which had then sat unaddressed for the whole session. The review would have been
conducted against superseded text, and both the advisor and the orchestrator would have reported
success. This was caught only because the **user** asked whether the network-less lane needed data
supplied to it; the correct fix — extracting the current files from `origin` into a staging directory
and telling the lane explicitly to ignore the working copy — was not prompted by anything in the
doctrine.

The existing rule that the advisor cannot research covers *absence* of access. This is the inverse
and more dangerous case: the lane **has** access, to something that looks authoritative and is stale.

**Rule concluded:** when dispatching a lane to review or reason about repository content, either
verify the working copy matches the server first, or extract the reviewed files from the server ref
into a staging path and point the lane there explicitly. State in the spec which copy is
authoritative. A lane cannot detect that the files it was given are out of date.

**Finding 2 — an explicit, specific instruction to verify a supplied claim did not produce
verification.** The advisor's brief said, in substance: *do not take my claim about the promotion
ledger on faith; here are the shipped files; verify or refute it.* Its returned review cited the
orchestrator's own planning document as the support for that claim. The claim happened to be true —
independently checked before dispatch — so no harm resulted, but the instruction did not bind.

This is the third recorded instance of the advisor treating orchestrator-supplied framing as
established (2026-08-21 evening, 2026-08-22, and now). The new information is that **the previously
proposed fix does not work**: earlier entries suggested the orchestrator should ask the advisor to
question its premises. That was done here, explicitly and specifically, and the advisor complied with
every other instruction in the same brief while skipping this one.

**Rule concluded:** stop treating "tell the advisor to verify the premise" as a sufficient mitigation.
If a premise is load-bearing, the orchestrator verifies it itself before dispatch and states it in the
spec as verified — the distinction the spec contract still lacks a slot for. An instruction to a lane
to check the architect's work is a request, not a control.

**Finding 3 — an orchestrator-authored artifact went stale against a decision made in conversation,
and the staleness surfaced as a confident false finding in a review.** Mid-session the orchestrator
wrote a planning document and a compiled artifact, both derived from a long-form source file. The user
then revised one of the source file's stated preferences in conversation. Nothing propagated that
revision to the source file. A later review, correctly reading the source, flagged the compiled
artifact as unfaithful to it — a finding that was accurate about the file and wrong about the current
state of the world.

This is the second instance of the shape logged on 2026-08-24 (an agent treating an
orchestrator-written document as authority while conversation moved past it). The new element: here it
did not merely propagate a stale premise silently, it **generated a false defect report**, which costs
more — the orchestrator had to verify and then reject a finding from its own reviewer, and a less
careful pass would have "fixed" a non-defect.

**Rule concluded:** when a session revises something recorded in a durable file, the revision is not
done until the file is updated or the divergence is written down where the next reader will see it.
Where a review is dispatched against a source the session has since revised, say so in the brief.

**Finding 4 — narrowing a review's scope to exclude suggestions removed value at no saving.**
`[asserted]` The advisor brief for a second review said explicitly not to propose additions, on the
reasoning that the artifact was being cut rather than grown. The user's unprompted correction was that
suggestions are welcome and cheap, and that even rejected ones are worth recording as considered.
Scope restrictions that forbid *output* rather than forbidding *work* save nothing and can only lose
information.

**Rule concluded:** scope a review by what it should examine, not by what it may say. If a suggestion
is unwanted it can be ignored at zero cost; if it is wanted and was suppressed, it is gone.

**Finding 5 — the research lanes overturned the orchestrator's own recommendation twice, on
documented grounds, and this was the highest-value output of the session.** The orchestrator had
recommended a storage location and a content format, both from inference. Two research lanes with
network access returned documentation showing the recommended location was materially weaker than an
alternative the orchestrator had not considered, and that the recommended format measured worse than
the alternative on the one benchmark that isolates it. Both were re-checked against the cited sources
before acceptance; both held.

This is the useful counterpart to the advisor findings above. A same-family reviewer reading the same
files cannot correct an inference the orchestrator made from missing information — only a lane that
can go and get the missing information can. **The doctrine's advisor pass and a research dispatch are
not substitutes for each other**, and the current routing guidance treats "get a second opinion" as
one action.

**Rule concluded:** when a recommendation rests on inference rather than a read source, the correct
escalation is a research dispatch, not an advisor consult. The advisor checks reasoning; it cannot
supply an absent fact.

**Where it lives:** here for now, but findings 1 and 2 should not stay. Finding 1 belongs in the spec
contract as a Context clause — name the authoritative copy — and is closely tied to the
architect-asserted-facts finding logged earlier today. Finding 2 retires a mitigation two prior
entries proposed and belongs alongside them in the commitment-boundaries section. Finding 5 belongs in
the routing guidance, as the distinction between an advisor consult and a research dispatch. Findings
3 and 4 are single-instance refinements; leave them here pending recurrence.
