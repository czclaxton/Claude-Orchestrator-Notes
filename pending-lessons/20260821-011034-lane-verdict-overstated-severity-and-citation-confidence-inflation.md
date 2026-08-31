### 2026-08-21 — A lane's volunteered synthesis overstated severity again, while its assigned output self-corrected the orchestrator three times

**Status: finding 1 verified** (the lane's comparative claim was re-checked directly against the
source files and found to invert the actual relationship); **finding 2 verified** (the three
corrections are in the lane's returned report and each was re-confirmed against the files);
**finding 3 verified** (the over-broad confidence header was written, then had to be dismantled
claim-by-claim after the user challenged the framing).

One read-only audit lane was dispatched to analyse a large body of third-party configuration files
and report findings against a set of prior assumptions. Single lane, no ladder escalation, no
advisor consult — so this entry is about lane output quality and how it propagates, not routing.

**Finding 1 — the 2026-08-20 "incidental claims" failure recurred, in the same direction.** The
lane's assigned output (locate values, cite file and line, verify or contradict specific claims)
was excellent. Its closing *verdict* section — synthesis it volunteered, not work it was asked for
— contained a comparative claim asserting that one side of a system could not close a gap against
another. Re-reading the files directly showed the opposite: the scaling factor the lane treated as
applying to only one side applied to both, so the gap it described did not exist in the form
stated. The lane's own earlier section had actually reported the correct underlying facts; the
error appeared only when it summarised them into a verdict.

Notably, the orchestrator relayed that claim to the user verbatim before checking it, and had to
retract it a turn later. This is the second consecutive session where a lane's volunteered
context skewed toward **overstating severity** — the same direction recorded on 2026-08-20. Two
occurrences in a row in the same direction is starting to look like a property, not noise.

**Rule concluded (reinforces, doesn't extend, the 2026-08-20 rule):** a lane's concluding
verdict/synthesis section should be treated as the *lowest*-confidence part of its report, below
even its incidental mid-report asides, because synthesis is where it compresses correct facts into
a directional story. The existing rule says verify volunteered context before relaying. The
addition worth making: **when a lane's summary and its own body disagree, the body is right** —
and the orchestrator should read the body rather than trusting the lane's compression of it.

**Finding 2 — explicitly instructing a lane that contradicting the orchestrator outranks agreeing
produced three real corrections.** The dispatch spec listed the orchestrator's prior assumptions as
claims to verify, and stated plainly that a claim being wrong mattered more than confirmation. The
lane returned a verdict table marking one assumption misattributed (right number, wrong causal
mechanism — it had been credited to a variable that did not control the behaviour), one off by
roughly 25%, and one not findable in the supplied material at all. All three held up on re-check.
Without that instruction the likely outcome is a report that confirms the framing it was handed.

**Rule concluded:** when a lane is dispatched to check the orchestrator's own prior claims, the
spec should name those claims explicitly *and* state that contradiction is the higher-value result.
Worth considering as a standard element of verification-shaped specs, alongside the existing
five-part structure. Cheap to add, and it converts the lane from a confirmer into a check.

**Finding 3 — file-and-line citations from a lane inflate the orchestrator's downstream confidence,
and the inflation compounds when written up.** After receiving a report dense with precise
citations, the orchestrator wrote a summary page opening with a blanket header asserting everything
below was verified from source. It wasn't. The page mixed: values genuinely read from files;
comparisons against baseline values that were themselves only a third party's undated annotations;
figures the orchestrator had *calculated* from those values and never observed; and predictions
about real-world behaviour that no one had tested. Only the first grade deserved the label.

The mechanism is worth naming: precise citations feel like evidence about *the world*, when they
are only evidence about *the file*. A lane reporting `variable = 30 at path:line` has established
what a file contains, not what happens when the system runs. That gap is invisible in a report
formatted as a verification table.

**Rule concluded:** a lane returning citations upgrades confidence about the artifact it read, not
about the artifact's effects. When an orchestrator writes up lane findings, per-claim grading is
required and a page- or section-level confidence header should be treated as an anti-pattern — it
necessarily takes the grade of the *strongest* claim and applies it to the weakest, which is
precisely backwards for a reader trying to find the soft spots.

**Where it lives:** here only. Finding 1 reinforces the existing Verification-section change queued
on 2026-08-20 and should be folded into it rather than logged separately. Finding 2 is a candidate
addition to spec guidance for verification-shaped tasks. Finding 3 is new and concerns what the
orchestrator does *after* a lane reports — an area the doctrine currently says little about, since
it focuses on dispatch and verification but not on write-up.
