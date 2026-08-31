### 2026-08-30 — A verification helper read the wrong success signal, and reported the opposite of the truth without erroring

**What happened** [verified]: the orchestrator needed to know whether several branches still merged
cleanly. It ran the underlying tool once per branch and decided the answer by testing whether the
tool had produced output. The tool produces output in both cases — a plain result on success, and a
result plus conflict markers on failure — so the check returned "clean" for every branch and could
not have returned anything else. Two of them conflicted. The wrong answer was reported to the user
in a summary table, with a recommendation attached to each row.

Nothing failed. The command exited zero, printed plausible output, and the loop ran to completion.
The error surfaced only because a later step inspected one of those branches for a different reason
and found a collision the table said was not there. Re-running the same check against the tool's
**exit code** instead of its output immediately gave the correct split.

This is the third instance of the same shape and the bar is long since cleared: a verification step
whose success condition cannot distinguish success from failure. The earlier two were a spec's
verification command that could not fail, and a find-and-replace keyed on an assumed value that
exited zero after matching nothing. All three produced confident, wrong, downstream claims; none of
them errored.

**Rule concluded:** when an orchestrator writes a check rather than running one it was given, it
must first state what a *failing* run of that check looks like. If the failing case is
indistinguishable from the passing case at the signal being read, the check is inert regardless of
how much output it produces. Prefer the exit code over the presence of output, and when a tool is
documented as emitting results in both cases, that is a direct warning that output-presence is the
wrong signal. Cheap to apply: it costs one sentence at spec-writing time.

**Finding 2 — a helper artifact's own header described a safety step it did not perform, and the
description was believed for four days** [verified]. A one-off script written in an earlier session
carried a header comment stating that it preserved recovery points on the server, not merely
locally. The script created those recovery points but never pushed them. The gap was found only
because the orchestrator queried the server for them after the script ran, rather than reading the
header and reporting what it promised. Had it not checked, the destructive step that followed would
have left the only copies of the prior state on one machine.

The project already has a standing rule that a note's characterization of a file is a pointer, not
a finding — open the file before repeating the claim. This is the same rule one level down: a
comment inside an executable artifact is a claim about the code, written by someone who was not
running it at the time, and it ages exactly as badly as any other note.

**Rule concluded:** a safety property stated in a script's own comments is unverified until
observed externally. Before any irreversible step that a script claims to have made recoverable,
confirm the recovery point exists where the claim says it exists — from the authoritative side, not
from the same machine that would be lost.

**Finding 3 — a research lane returned a build recommendation without the denominator that decides
it, and asserted a configuration capability that does not exist** [verified]. A lane was dispatched
to decide whether a guardrail was worth building. Its spec asked three things: how hard, how
feasible, and how much value. It answered the first two well, with sources and evidence labels, and
returned a firm "build now" with a cost estimate of several hours.

Two problems on review. First, the recommendation never established how often the guarded event
could occur — the value half was answered from the severity of the risk alone, and the frequency
was available to the lane in the material it was given. Supplying that number inverted the verdict.
Second, one of its proposed mitigations rested on a configuration field that does not exist; the
full field list, fetched directly, does not contain it. That claim had been labelled as a
recommendation rather than as a finding, so it carried no evidence marker, and it was load-bearing
for the fallback design.

The standing rule that a lane's *characterizations* need verifying while its *observations* usually
hold was confirmed again here: everything the lane quoted from documentation was accurate, and both
errors were in what it built on top.

**Rule concluded:** a spec that asks a lane for a build-or-defer verdict must name the denominator
explicitly — how often the guarded event occurs, or how often the lane is invoked — because a lane
given only severity will price the risk and not the expected cost, and will reliably recommend
building. Separately, an evidence-labelling instruction should cover the recommendation section and
not only the findings section; the unlabelled claim in this case sat in the part of the report the
labels did not reach.

**Recurrence note** [verified]: a resume note carried a platform behavior as a permanent hazard in
its known-traps list. The platform's documented behavior had since changed, and the trap now
asserted the opposite of what was true. This is the second logged instance of a hard negative being
frozen into a resume note and going stale, and the first where the cause was external change rather
than the note's own drift. It suggests known-traps entries about *platform* behavior need a version
or date attached, where entries about the project's own behavior do not.

**Where it lives:** Finding 1 belongs in the Verification section, as the third instance of the
inert-check shape and the strongest statement of it so far — the first two were about specs handed
to lanes, this one is about a check the orchestrator wrote for itself. Finding 2 extends the
existing pointer-not-a-finding rule to executable artifacts. Finding 3 belongs in the spec-writing
section, alongside the existing five-part spec contract. The recurrence note belongs wherever the
resume-note lifecycle rules live, as an amendment rather than a new rule.
