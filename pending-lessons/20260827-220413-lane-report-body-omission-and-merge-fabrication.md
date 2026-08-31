### 2026-08-27 — A lane's final message can silently omit its own report body, and the merge step is where the orchestrator invents things

A long session that leaned heavily on delegation — six lanes across scoping, estimation, document
review and a code review, plus two verification passes over the artifact those lanes produced.
Three findings, all about what happens at the boundary where a lane's output becomes the
orchestrator's belief.

**Finding 1 — a lane's final message can omit the body of its own report, and the orchestrator has
no way to detect it.** A lane was tasked with producing an inventory, a per-item breakdown and a
total. Its returned message opened by referring to a report it had "already filed" and then
delivered only a corrections addendum to that report. The report itself never arrived. The
orchestrator received totals with no derivation, and — this is the part that matters — the message
read as complete. It was internally coherent, it referenced its own earlier content naturally, and
nothing in it signalled that anything was missing.

The orchestrator then relayed the totals as a headline. The user caught it, not because the numbers
were wrong, but because they asked a question the breakdown would have answered and the orchestrator
could not.

The general shape: a lane that produces output in stages can return only its final stage. The
orchestrator sees one message and has no baseline for what a complete return would have looked
like. Every existing verification instruction assumes the report is present and asks whether its
claims are true; none asks whether the report is all there.

**Rule concluded:** a spec that asks for structured output should state the required sections and
instruct the lane that its final message must contain them in full, not reference them. On receipt,
check the returned message against that list before using any of it — a missing section is a
different failure from a wrong one and is invisible to claim-level verification. When a lane's
message refers to content it says it produced earlier, treat that as a missing section, not as
context.

**Finding 2 — the merge step invites the orchestrator to fabricate, because it is explaining work
it did not witness.** While consolidating lane outputs into a single document, the orchestrator
recorded a claim whose source pointer was wrong. Under a *provenance* heading it then wrote an
explanation for the error: that an earlier read had used a truncated range and missed the content.
That explanation was invented. The content was not in that source at all, at any range. The
orchestrator had no recollection of what had gone wrong, produced the most plausible-sounding cause,
and wrote it as fact.

A later cold-start reviewer caught it and named the consequence precisely: the invented rationale
**reads as diligence**. A reviewer who follows a pointer, finds nothing, and then encounters a
confident account of why the discrepancy exists will stop looking. The underlying claim was true and
would have been rejected.

This is structural to delegate-then-merge, not incidental. The merging party did not do the work.
Asked to account for a gap in a lane's output, it has nothing to draw on but plausibility — and
plausibility is exactly what produces a convincing wrong answer. The same session produced a milder
version of the same failure: a claim marked *unverified* was annotated "**but** the lane stated it
read all N items directly, so higher confidence." That is a lane grading its own homework, promoted
to evidence by the orchestrator.

**Rule concluded:** never explain a discrepancy the orchestrator did not actually diagnose. If a
lane's output is wrong and the cause is unknown, record "wrong, cause unknown" and stop. Provenance
and confidence fields are the *last* place to smooth something over — they are the fields a reader
leans on hardest, and an unsupported explanation there is worse than a bare error. A lane's
self-report of its own thoroughness is never a confidence upgrade.

**Finding 3 — a spec must authorize refusal when the task may be unanswerable, and two verification
techniques that worked.** The orchestrator delegated an estimate for work whose scope was not
defined anywhere. The lane produced a number, correctly labelled its own confidence low, and the
orchestrator relayed the number. The user's response was the finding: they had expected follow-up
questions and got a figure. The spec never authorized "this cannot be estimated, and here is what is
missing" as a valid return, so the lane did the only thing it was permitted to do. This is the same
shape as an earlier logged case where a spec demanded verification requiring a gated capability
without saying whether the gate was open — a lane cannot ask the user, so anything the spec fails to
authorize becomes a forced choice.

Two techniques from the same session are worth keeping. **Independent second draft:** a handoff
document was drafted by the orchestrator and, in parallel, by a lane given the same source material
but not the orchestrator's framing. The independent draft caught three wrong line-number citations
and two file-format traps the first had missed — none of which were reachable by revising the first
draft, because they were errors of assumption rather than of care. **Cold-start use test:** the
merged document was then handed to a fresh lane with *only* that document, explicitly forbidden from
reading any other project file to orient itself, and asked to *use* it — verify a sample of its
claims from its own pointers and report what it could not do. Its mistaken first impressions were
the most valuable output: it could not tell what the project was until 82% of the way through, and
it found two true claims with wrong addresses, fifteen instructions with no destination, and safety
warnings placed after the point where work begins.

**Rule concluded:** when a task may be unanswerable, the spec must name that outcome as an
acceptable return and describe what to report instead. For any artifact meant to be picked up by
someone without the current context — a handoff, a spec, a review queue — run a cold-start pass
before shipping: give a lane the artifact and nothing else, forbid orienting reads, and ask it to
use rather than critique. Where the artifact is high-stakes, draft it twice independently rather
than drafting once and revising.

**Status: finding 1 verified** (the received message was re-read and confirmed to contain only the
addendum; the referenced report is absent). **Finding 2 verified** (the fabricated explanation was
checked against the source and the content is not there at any range; the self-report-as-confidence
annotation was re-read in place). **Finding 3 verified** for the spec gap and both techniques (the
outputs of both verification passes were reviewed and their findings confirmed against sources);
the claim that the techniques generalize beyond this artifact is **asserted**.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's Verification section
as a structural-completeness check preceding the existing claim-level checks — it is a different
axis from the 2026-08-20 entry's assigned-output-versus-volunteered-context distinction, which
assumes the report arrived intact. Finding 2 belongs in the same section and extends the
2026-08-21 citation-confidence-inflation entry from lanes to the orchestrator itself. Finding 3's
first half extends the earlier gated-capability entry from capabilities to outcomes; its second
half is new and belongs wherever the doctrine covers handoff artifacts.
