---

### 2026-08-22 — The advisor validated a plan built on a false factual premise, and the premise was the only thing that mattered

**Status: both findings verified** (the premise error was re-checked directly against the artifacts
after the advisor returned; the severity miscall was re-checked against the plan text the advisor
had been given).

**Context, project-agnostically:** the orchestrator was assembling a staged install plan for two
third-party components that both modified the same set of files belonging to a host application.
Before executing the risky stage, it consulted the advisor at a commitment boundary, exactly as the
doctrine prescribes. The brief supplied a set of facts the advisor could not re-derive with its
read-only local toolset, explicitly labelled as given, and asked it to challenge the *inferences*
drawn from them.

**Finding 1 — the advisor faithfully critiqued the plan and never questioned the premise, because
the brief told it not to and its tools could not have checked anyway.** Its structural feedback was
good: it correctly identified that one stage's verification step could not detect the failure that
stage existed to catch, which was the single most valuable thing it said. But roughly half its
ranked findings — including two of three marked "Critical" — were consequences of a conflict-
resolution step that turned out not to be needed at all.

The premise was wrong. The host application resolved the two components' files in *different
load stages* rather than last-wins, so they were designed to coexist and no conflict existed. That
was settled by a single external documentation lookup performed after the advisor returned, and
then confirmed directly from the artifacts in under a minute. The entire branch of the plan the
advisor spent most of its effort hardening simply evaporated.

This is the same shape as the 2026-08-20 (second session) finding about the advisor optimizing
strictly within the constraints it was handed — but there the fixed constraint came from the *user*,
and the fix was to take the fork back to the human. Here the fixed constraint was a *factual claim
about an external system* that the orchestrator believed and stated with confidence. No human held
the correction; the correction was public information neither party had looked up.

**Rule concluded:** before spending an advisor consult on *how to handle* a constraint, spend the
cheaper check on *whether the constraint is real*. When the load-bearing risk in a decision is the
currency or correctness of a fact about an external system, an advisor restricted to reading local
files is structurally unable to help with the part that matters, however good its reasoning is.
The brief's "treat as given, challenge the inferences" framing is a reasonable instruction and was
followed correctly — which is precisely why it is dangerous: it converts an unverified premise into
a load-bearing one with a review stamp on it. Consider requiring that any premise labelled "given"
in an advisor brief be independently sourced *first*, or that the advisor be explicitly asked to
name which premises it would check if it could.

**Finding 2 — severity inflation again, in the same direction as before.** Of three findings marked
"Critical", one genuinely was. One was a documentation gap presented as a functional defect: the
advisor reported a required item as unaccounted for in the plan, when the item's owning group *was*
listed — only the item itself was not named individually. Re-checking the plan text the advisor had
been given confirmed the group was present. The distinction between "not covered" and "covered but
not spelled out" is the difference between a defect and an editorial note, and it was labelled
Critical.

This recurs with the 2026-08-21 entry on overstated severity and the 2026-08-20 (second session)
finding that lanes skew toward overstating breadth. Three sightings now, consistently in the same
direction: **lanes and the advisor inflate rather than understate**, and the inflation clusters in
claims about *scope and coverage* rather than about the mechanism itself.

**Where it lives:** here only. Finding 1 belongs in the commitment-boundaries section alongside the
2026-08-20 entry, as a second failure mode of the same underlying issue — the advisor reasons
soundly from whatever premises it is handed, and neither the doctrine nor the brief format has a
step that pressure-tests those premises. Finding 2 is now a strong enough pattern across three
sessions to justify an explicit calibration instruction in the advisor definition: state severity
in terms of demonstrated consequence, and separate "the plan does not handle X" from "the plan does
not mention X".
