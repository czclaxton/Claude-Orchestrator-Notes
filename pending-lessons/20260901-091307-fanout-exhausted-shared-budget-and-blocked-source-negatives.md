### 2026-09-01 — Wide parallel fan-out silently exhausted a shared session-level budget, then killed every in-flight lane at once

**What happened** [verified]: a research task was decomposed into many independent lanes and
dispatched in parallel — nineteen at one point, in two waves plus later additions. Each lane's spec
told it to research thoroughly and named a target source count. The lanes complied. Collectively
they consumed a **session-wide** consumable — a capped tool-call budget shared across the whole
session, orchestrator and every lane together — and exhausted it (200 of 200) roughly a third of the
way through the run.

Nothing errored at the moment of exhaustion. Lanes dispatched after that point kept working and kept
producing full-length reports, but had lost their primary *discovery* tool and were reduced to
fetching sources whose URLs they already possessed. The reports stayed the same length. Several
lanes flagged the degradation in their own limitations sections, which is the only reason the
orchestrator learned of it at all — there was no signal from the harness, and no way to query
remaining budget before or during dispatch.

Then the second failure: a session-level rate limit terminated **nine in-flight lanes
simultaneously**. Two had already written their output files and lost only their closing summary;
seven produced nothing at all. The loss was total and correlated, because every lane was drawing on
the same pool.

Both failures share one cause: **parallel fan-out over a shared consumable has no backpressure.**
Each lane individually behaved correctly and within its spec. The spec is written per-lane, the
budget is enforced per-session, and nothing connects the two. Sequential execution would have
degraded gracefully — early lanes complete, later ones stop — where fan-out converted the same
budget into a cliff that took nine lanes down together.

**Rule concluded:** when fanning out more than a handful of lanes over a shared, capped resource,
the orchestrator owns the budget arithmetic because no lane can see it. Concretely: divide the known
cap by the planned lane count and state the per-lane allowance *in the spec* rather than asking for
"thorough" research and letting the first lanes spend freely; prefer waves over one large fan-out so
a mid-run check can observe consumption before committing the rest; and treat a lane's
self-reported limitation as the primary telemetry channel, which means specs should require lanes
to report resource exhaustion explicitly rather than leaving it to conscientiousness. Also worth
weighing at dispatch time: wide fan-out concentrates failure. Lanes that write their artifact
incrementally survive a mid-run kill; lanes that research fully and write at the end lose
everything, so specs for expensive lanes should require writing the file early and revising it.

**Finding 2 — a blocked source turned every lane's silence into a false negative** [verified]. One
major community source was network-unreachable from the environment (403 on every route, confirmed
independently by the orchestrator after a lane first reported it). Lanes researching *criticism* of
their targets therefore found little, and a report that says "few complaints found" is
indistinguishable from one that says "I could not reach the place complaints live."

Most lanes did flag this unprompted, and the ones that did wrote the distinction plainly — that
absence of found criticism was an artifact of access, not evidence of quality. That was the right
behavior and it is worth recording as a success, not only a risk. But it was not required by any
spec; it happened because individual lanes were conscientious. A lane that omitted the caveat would
have produced a clean-looking, confidently wrong report, and nothing downstream would have caught it.

This is the existing negative-claim doctrine reaching a case it was not written for. That doctrine
was framed around one specific tool's staleness — asking the authoritative side before asserting a
negative. The same failure shape appears whenever a *source* is unreachable rather than a *ref*
being stale, and the orchestrator has no general statement of it.

**Rule concluded:** generalize the negative-claim doctrine from "verify against the authoritative
side" to "a negative requires evidence that the place the positive would live was actually
reachable." Specs that ask a lane to find problems, criticism, or counter-evidence should require
the lane to state which sources it reached and which it could not, and should say explicitly that
"nothing found" is only reportable alongside that list. Cheap: one clause per spec.

**Recurrence 1 — the orchestrator asserted facts into specs and lanes corrected them** [verified].
Two specs named an attribution the orchestrator supplied from memory rather than from the
authoritative dataset it had already downloaded. Both were wrong; both lanes corrected them from
first-party data and said so. No harm resulted because the lanes checked, but the orchestrator had
the correct data on disk at the moment it wrote the incorrect spec. This is the third logged
instance of the spec-asserted-facts shape. The novel detail here is *why* it happened: the
orchestrator had authoritative data available and still reached for recall, because the fact felt
like context-setting rather than a claim. Specs should draw every factual identifier from the
dataset the lane will itself consult, so a mismatch is detectable rather than merely wrong.

**Recurrence 2 — two lanes reached opposite conclusions and nothing reconciled them** [verified].
One lane concluded a technical integration was impossible on architectural grounds; another,
researching a different target, surfaced third-party documentation describing that exact
integration working. Both were well-sourced. The orchestrator surfaced the disagreement to the user
as unresolved rather than picking a side, and dispatched a verification lane — which subsequently
established that the second lane's reading had been wrong, so the reconciliation was load-bearing
and not merely tidy. This is the second logged instance of cross-lane contradiction. Worth noting
that parallel fan-out makes this *more* likely, not less: lanes cannot see each other's findings,
so overlapping technical questions get answered independently and inconsistently by construction.
A fan-out plan that gives two lanes the same sub-question should expect to spend a third lane
reconciling them.

**Where it lives:** Finding 1 is new and belongs wherever delegation/fan-out sizing is decided — it
is the first logged case of a resource constraint that is invisible at the level the spec is written.
Finding 2 amends the existing negative-claim doctrine, widening it past its original tool-specific
framing. Recurrence 1 strengthens the existing spec-asserted-facts rule with a concrete mechanism
rather than a new rule. Recurrence 2 belongs with the existing cross-lane contradiction entry, as an
amendment noting that fan-out width increases contradiction rate.
