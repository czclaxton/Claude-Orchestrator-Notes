# Pending lessons — 2026-09-01

Session shape: one long orchestrator session hardening a build plan for unattended execution. No
implementation code written. Two requirements traces, four cold reads, five amendment rounds, one
external research pass, several implementer lanes running documentation edits.

---

## 1. `catch-up` cannot detect a concurrent session in the same repo

**verified** — re-checked against live on-disk session state during the session.

Two sessions ran in one repository on independent workstreams. The second's existence was noticed
only at hour six, by accident, when a file's modification time did not match anything this session
had written. Until then this session believed it held the whole picture, and was about to overwrite
a shared handoff file.

`catch-up` reads git state and the resume note. Neither says anything about other live sessions.
Its negative-claim discipline is rigorous about the *remote* — it insists on querying the server
before asserting nothing has been pushed — and has no equivalent for the local machine.

**A cheap fix exists.** The host maintains live per-session state on disk including working
directory, a human-readable session name, and timestamps. Filtering by working directory and
liveness yields the other sessions in the repo, by name. Three implementation notes, all learned by
trial:

- On Windows, the POSIX shell's `kill -0 <pid>` does **not** work against native PIDs — it reports
  live processes as dead. Node's `process.kill(pid, 0)` works.
- The "last updated" timestamp does not tick for an idle session, so freshness is not liveness.
- Stale records from other projects persist; filter by working directory, not by file presence.

**The caveat matters more than the feature.** This is undocumented host internal state and may
change. It must degrade to *"peer status unknown"*, never to a confident *"no other sessions"*. A
false negative there is exactly the failure being fixed.

---

## 2. `wrap-up`'s "overwrite, don't append" is hostile to multi-session use

**verified** — read the command text.

The command instructs, verbatim: *"Write (overwrite, don't append) `RESUME-PROMPT.md`."* With two
sessions in one repo that is a clobber instruction.

Worse, it is the instruction a project-level fix would have to actively defend against. A local
design considered this session required a hook whose sole purpose was to block the plugin's own
step. **A local patch fighting the tool it patches is a signal the fix belongs upstream.** Written
up as an ideas-backlog entry rather than built locally, for that reason.

---

## 3. The cold-read pass should be a first-class orchestrator practice

**verified** — ran four times this session, with measurably different results from briefed reviews.

A plan was reviewed three times by agents briefed with session context. All three passed it. A
fourth agent, given the document and nothing else, immediately found a defect that would have
propagated through four components — an analogy to another file that pointed at a different
function signature than the author intended.

**This has a name and a literature.** *Unconscious disambiguation*: a reader silently resolves an
ambiguity to the first meaning that occurs to them, unaware others exist. The resulting defect is
*persistent ambiguity* — it survives review precisely because every reviewer resolved it,
invisibly, possibly differently. **A reviewer holding the author's context disambiguates the way the
author did, and identical disambiguation is indistinguishable from clarity.**

Across four rounds the cold read found the highest-cost defect every time, including two the
orchestrator itself had written.

**Refinements from established practice, worth building in rather than rediscovering:**

- **Give the reader a procedure, not an open question.** Perspective-Based Reading found procedural
  scenarios beat ad-hoc reading, with the largest gains for readers unfamiliar with the domain. "As
  the implementer of step N, write the signature you would produce and list every input whose type
  or default you had to invent" beats "what would you guess?"
- **Two readers, then diff their interpretations.** One reader's guesses do not reveal whether the
  guess was forced. Divergence between two independent statements localises the ambiguity exactly.
- Every *"I would have to decide this myself"* is a defect in the document, not the reader. Fix the
  document; never brief the reader.

---

## 4. Implementer lanes caught the orchestrator's errors three times — encode what produced it

**verified** — all three are in the session transcript with the lanes' own reasoning.

Implementer lanes caught three orchestrator mistakes rather than following them:

- A ruling placing a file in a directory that does not exist, against the repo's own convention.
  The lane applied the ruling but flagged the deviation.
- An instruction to cite a document entry that did not exist. The lane refused to write a false
  citation and said why.
- A stated count that contradicted the source document. The lane declined to change a ruling the
  orchestrator had asserted was correct, and flagged the contradiction instead.

**What produced this:** briefs saying, in substance, *"every item below is a ruling; apply it, and
tell me if any is wrong against the document's own evidence."* That single clause converts a lane
from a compliance engine into a check on the orchestrator. It costs one sentence.

**Worth considering as standard spec boilerplate.** The orchestrator is the least-checked actor in
the system — nothing reviews its briefs before a lane acts on them.

---

## 5. Document review has a crossover point, and there is a signal for it

**verified** — the finding came from a lane's own render output, reproduced in its report.

A plan passed four cold reads. The last real defect was found not by reading but by an implementer
rendering a component server-side against the installed library: labels were being drawn in their
own container's colour, on top of it, invisible. The instruction that produced this was accurate,
cited, and verified against library source. **Nothing about it was wrong on the page.**

**The general shape.** Document review finds missing information, internal contradiction, and
claims that do not match their source. It cannot find a defect that requires the code to exist.

**The signal that you have crossed over: the last real finding came from execution rather than
inspection.** At that point further reads look in the wrong place, and the correct move is to build.
This is a usable stopping rule for a review cycle that would otherwise have no natural end — which
matters, because every round genuinely did find real things, and "it found something" is not by
itself a reason to run another.

**Related, and cheap:** verifying a library claim against its source is necessary but not
sufficient. Where a claim concerns *composition* — what happens when two library pieces are used
together — render it. Server-side rendering against the installed package needs no dev server, no
browser, and no test framework.

---

## 6. A confidently wrong instruction is the one defect a stop-and-ask rule cannot catch

**verified** — the error and its correction are both in the session, with library source checked
before and after.

A plan carried a STOP rule: halt and escalate rather than fill any gap. It was well drafted, and a
cold reader judged it credible.

The same hardening pass then introduced a factual error about library behaviour — stated
confidently, in a document that instructs the implementer not to substitute judgment.

**A gap produces a question. A confident wrong statement produces none.** There is nothing to halt
on, nothing that feels uncertain, and no downstream check separating "followed the plan" from
"followed the plan and got the wrong answer". Every other control — trace both ways, cold read,
bounded extension licence, escalation chain — is aimed at *missing* information.

**Also worth recording: the error was written while closing an ambiguity a reviewer had found.** The
pressure to close an open question is itself a source of confident wrongness. Closing a decision and
verifying the fact underneath it are separate actions, and only the first feels like progress.

**Mitigation that partly works:** tell the executor what to do when the document is *wrong*, not
only when it is incomplete — if a stated library behaviour does not match the library, the document
is wrong; trust the library, stop, report. This does not catch the error, but it gives an
implementer who happens to check somewhere to go.

---

## 7. Over-specification is the failure mode that feels like virtue

**asserted** — the trend is clear in the session, but I did not measure whether length
independently degraded any outcome.

Closing ambiguity adds bulk. Across five amendment rounds a plan grew roughly threefold for a
package of about nine files, and one round that deliberately deleted material still ended longer
than it started. Every individual addition looked justified.

External research found this measured elsewhere: instruction-following degrades multiplicatively
with the number of simultaneous instructions, and one documented case put a very large specification
through to produce a small amount of code, at substantial review cost, and still shipped a bug.

**What the cold reads actually said, which cut against my assumption:** asked directly whether the
document was too long, they said the length earned its keep and that **ordering, not volume, was the
problem** — instructions buried far from the step that needed them. The fix that worked was per-step
reading lists, not cutting.

Carry as a caution rather than a rule: watch bulk, but diagnose before cutting.
