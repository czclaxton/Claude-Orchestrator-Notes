### 2026-09-01 — The escalation chain caught orchestrator briefs that were *wrong*, not merely incomplete — twice

**What happened** [verified]: a six-step unattended build ran with the standard escalation
chain in every implementer brief — *the implementer does not fill gaps; it halts and
escalates rather than choosing.* Five halts occurred across the run and all five found real
specification defects. That much is the doctrine working as designed.

The notable part is the two halts that were not about gaps at all. In both, the orchestrator
had issued a **positive instruction that was factually false about library behaviour**, and
the lane halted rather than complying:

- One brief said to fix a layout overflow using one of two named mechanisms. The lane
  rendered both, proved the first was a measurable no-op (the library derived the relevant
  dimension from the *smaller* of two axes, so the instructed knob could not move it), showed
  the second would require a ~50% reduction that would degrade every non-affected case, and
  reported that the stated success criterion was unsatisfiable as written. The defect turned
  out not to be in the component at all — it was a layout constraint the orchestrator had
  misdiagnosed.
- Another brief said to pass a specific formatter to a display component. The lane found the
  formatter would be applied to *every* row rather than the intended one, and that on the
  other row it produced a plausible-looking but meaningless value. Following the instruction
  would have replaced one visibly wrong output with two, and the result would have looked
  deliberate.

**Why this matters and is not just "the chain worked."** The escalation chain is written and
justified entirely in terms of *missing* information — "from the inside, a gap does not feel
like a gap." That framing is correct but undersells it. A confidently wrong instruction is
strictly more dangerous than a gap: a gap produces a question, a wrong instruction produces
clean, compliant, wrong work with nothing to halt on. The chain turns out to catch this
second class too, but **only because the briefs happened to also say "verify library claims
against the installed library rather than trusting prose, including mine."** Without that
sentence, both lanes would most likely have complied.

**Rule concluded:** the escalation chain's stated purpose should explicitly include halting
on instructions the lane can demonstrate are wrong, not only on instructions that are absent.
And the "trust the library over the document" clause should be part of the standard brief
rather than something an orchestrator remembers to add — it is what converts the chain from a
gap-detector into a wrongness-detector. Candidate home: the escalation-chain section of the
orchestration doctrine, plus the spec contract's verification part.

---

### 2026-09-01 — A lane produced a better halt trigger than the doctrine's own wording

**What happened** [verified]: early in the same run, a lane shipped an unspecified detail
rather than halting, reasoning that the item was too small to justify stopping a multi-step
unattended run. When corrected, it articulated its own error better than the doctrine does:

> *If I catch myself weighing options, that is the halt signal regardless of the item's size.
> Proportionality is not a valid exception to the STOP rule — judging the cost of halting is
> substituting my judgment for the rule's judgment about when to halt.*

That wording was then carried verbatim into every subsequent brief in the run, and the next
lane halted correctly on three separate items, all of which were genuine specification
defects.

**Why the doctrine's own phrasing is weaker.** "Halt if the specification does not determine
what to build" asks the lane to classify the *specification* — an assessment it makes from
inside the gap, which is exactly the position the doctrine elsewhere says is unreliable. "Halt
if you notice yourself deliberating" asks it to observe its **own behaviour**, which is
directly observable and needs no judgment about severity. It also closes the proportionality
loophole explicitly, which the current wording leaves open by omission.

**Rule concluded:** adopt the deliberation-detection phrasing as the operational trigger
alongside the existing definition, and state the proportionality exclusion outright. Cheap
change, and one run's evidence suggests it converts silently-filled gaps into visible
questions. Not yet promoted — worth one more run's confirmation before rewriting the doctrine
text, since this is a single session's observation.

---

### 2026-09-01 — Review *kind* mattered far more than review count; a never-run precondition found a class four reviews structurally could not

**What happened** [verified]: a specification had survived four independent cold-read passes
before implementation began. The governing protocol lists several preconditions, one of which
— *trace the plan both ways against its requirements source* — had never actually been run.
Running it before the build, as two independent passes (one per direction), found roughly
fifteen items, of which several had real build consequence: a documented justification that
was factually false, a specified behaviour that contradicted the very source it cited, and a
requirement satisfied for one of the four cases that motivated it.

None of the four cold reads could have found any of them. A cold reader is handed the
specification and asked whether it is implementable — and it was. The specification could have
been perfectly implementable while silently omitting a third of what the source required, and
nothing in that review shape would surface it.

**The counterintuitive part, worth recording precisely.** The instinct on seeing "fifteen more
findings after four reviews" is that the planning process is poor. It is the wrong read.
Findings-per-pass measures *what a given review shape can see*, not document quality. The
useful signal was severity across passes, which dropped monotonically: the first review found
a core signature ambiguity that would have been built wrong several times over; the last found
cosmetic issues inside an already-acknowledged placeholder. A traceability pass finding
nothing would have meant the check was useless, not that the document was good.

**Two structural observations from the same run:**
- Splitting the protocol's single "verifier" role into **spec conformance** and **rendering
  verification**, run in parallel as separate agents, worked well. They found disjoint sets:
  conformance found a false claim in a code comment that no amount of looking at a screen
  would reveal; rendering found four visual defects invisible to any amount of reading. It
  also let the expensive model go only at the judgment-heavy half.
- The final advisor, asked directly to *name anything the orchestrator appears to believe is
  right that isn't*, returned a systematic gap in a rule the orchestrator had authored and was
  therefore worst placed to audit. That prompt shape produced materially more than a general
  "review this diff" would have.

**Rule concluded:** when a review has already run several times without finding much, change
the *kind* of check rather than adding another of the same. And when consulting the advisor at
a commitment boundary, ask it explicitly to attack the orchestrator's own beliefs — not just
to review the artifact. Candidate home: the advisor guidance in the orchestration doctrine.

---

### 2026-09-01 — Instructing lanes to produce *evidence* rather than assertions changed what they caught

**What happened** [verified]: the doctrine's verification guidance is aimed at the
orchestrator — *reports are claims, not evidence; read the diff and re-run the command.* That
held up well across this run and caught real issues. But the higher-leverage move turned out
to be the inverse: instructing the **lanes** to generate checkable evidence rather than
statements, and telling them a specific technique for doing so without a browser.

Once briefs said to verify library behaviour by rendering the component in a headless
server-render harness and reporting measured values, lanes started returning numbers instead
of claims — computed radii matched against a closed-form expectation, before/after coordinates
for a clipping fix, byte-identical output proving a prop was inert before removing it. One
lane reproduced an independent browser reviewer's measurement *exactly* before changing
anything, which validated its model before it touched code.

Every false claim about library behaviour in this run — there were several, across multiple
documents — was caught by opening the installed library or rendering it. **None was ever
caught by re-reading the prose**, including prose that had been reviewed four times.

**A related failure of the orchestrator's own, worth recording as the general shape**
[verified]: the orchestrator ruled that a component was exempt from a fix because it could not
inherit a value from its parent. That reasoning was correct and the conclusion was wrong — it
verified the *stated mechanism* and never asked what the outcome actually was, which turned out
to be a different hardcoded value with the same practical defect. A later lane caught it by
rendering. Verifying that a mechanism does not apply is not the same as verifying the outcome
is acceptable.

**Rule concluded:** the spec contract's verification part should ask lanes for measurements
where measurement is possible, not just for a passing command. And the orchestrator's own
verification discipline needs a companion rule: check the *outcome*, not only the mechanism
you reasoned about. Candidate home: the spec contract and the verification section of the
orchestration doctrine.
