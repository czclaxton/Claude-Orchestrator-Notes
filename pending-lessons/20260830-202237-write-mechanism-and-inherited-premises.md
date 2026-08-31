### 2026-08-30 — Verification covered *what* the change contained but not *how* it was written; and a spec's premises reach the lane unchallenged

**Status: finding 1 verified** (both defects re-checked directly on disk by the orchestrator after
the review agent named them); **finding 2 partly verified** — that the orchestrator wrote a false
premise into the spec is verified from the spec text itself, while the reviewer's replacement
claim was accepted without independent re-derivation and is relayed as asserted.

**Finding 1 — an exhaustively verified change can still be delivered by an unsafe write.** The
orchestrator hand-authored a configuration change rather than driving the owning application's
own UI, then verified it hard: every mapped value cross-checked against the application's source,
every enumerated case confirmed complete, the output parsed for structural validity. All of that
was correct, and a final review agent confirmed it. What the review found instead were two defects
in the *act of writing*: the file had been edited behind an application that also owns it, leaving
the application's own settings record still describing the previous state — so the next time a
user touched the relevant control, or the app next saved, the edit would silently revert. And the
shell's default text encoding had introduced a byte-order mark and changed the line endings, in a
file that previously had neither.

Neither defect is visible from the content of the change, which is precisely why the
orchestrator's own verification missed them: every check it ran asked "is this value right?" and
none asked "does writing it this way hold?". The failure mode is specific to hand-editing state
that some other process also writes — a category that includes application config, generated
files, and anything with a UI of its own. It also has a distinctive signature: it verifies clean
and works immediately, then reverts later under a trigger unrelated to the change.

**Rule concluded:** when a spec has a lane edit state owned by another process, the verification
section should require two things the current doctrine does not ask for — that any parallel record
of that state was updated in the same pass, and that the file's encoding and line endings match
what was there before. Where the owning application exposes a way to make the change itself, that
route should be preferred over hand-editing and the spec should say so. "Prefer the owning tool;
if you hand-edit, reconcile the other copy" is the general form.

**Finding 2 — premises written into a spec are inherited by the lane, not tested by it.** The
orchestrator wrote a review brief containing a factual assertion it had carried forward from its
own earlier summarizing, unverified. The assertion was wrong. It survived into the spec because it
had been stated confidently enough in the orchestrator's own prior output to read as established.
It was caught only because that particular brief was framed adversarially — the lane was told its
job was to find errors and to check claims against source rather than confirm them — and it
disproved the premise while answering a different question. A neutrally framed spec would very
likely have reasoned *from* the false premise and returned a confident, internally consistent, and
wrong answer, with the orchestrator's own words as the corroboration.

This is the inverse of the 2026-08-20 (second session) entry. That one concerned unreliable claims
flowing *upward* from lane to orchestrator; this concerns unreliable claims flowing *downward*
from orchestrator to lane, where they are far more dangerous — a lane has no standing to doubt its
own brief, and anything the orchestrator asserts in a spec is functionally a constraint. Self-
sourced premises are the worst case, because the orchestrator will read the lane's agreement as
independent confirmation of something it told the lane in the first place.

**Rule concluded:** the spec-writing doctrine should distinguish, explicitly, between context the
lane may rely on as given and context it should verify before relying on — and the default for any
factual claim the orchestrator did not verify *in this session* should be the latter. A cheap
general mitigation, observed working here: give review specs an adversarial frame as standard
("find what is wrong with this; check claims against source, do not confirm them"), which converts
inherited premises from assumptions into targets. The stronger version is a discipline about the
orchestrator's own summaries — a claim restated from an earlier summary is a pointer, not a
finding, and putting it in a spec launders it into fact.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's spec template, as a
verification requirement for edits to externally-owned state. Finding 2 belongs alongside the
spec-writing guidance, and pairs with the 2026-08-20 (second session) entry as its mirror image —
together they argue the doctrine treats orchestrator-to-lane communication as trustworthy by
default in a way it does not treat lane-to-orchestrator communication.
