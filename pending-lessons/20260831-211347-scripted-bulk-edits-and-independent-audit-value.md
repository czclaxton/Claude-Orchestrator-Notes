### 2026-08-31 — Scripted bulk edits are a distinct orchestrator hazard, and both failure modes report success

**Status: verified** — both failures were reproduced and their damage confirmed by an independent
audit before repair, and the repairs were re-checked by re-reading the affected records.

**What happened:** the orchestrating session needed to reconcile ~23 records inside one structured
document so a status field matched a summary table above it. It wrote a script rather than editing
by hand. The script failed twice, in two independent ways, and **both times it reported success.**

*First failure — mutating while iterating.* The script collected match offsets from the original
string, then edited that same string inside the loop. After the first edit every subsequent offset
was stale, so it silently skipped 9 of 23 targets. This was caught only because a remaining-count
check did not reach zero. Had the skipped count happened to be zero, the run would have been
reported as complete.

*Second failure — matching by position instead of by ID.* The rewrite paired verdicts to records
positionally. Six records received a neighbour's verdict. Three of those had a status meaning
"recorded but never approved" and came out reading "approved by the user" — **the script
manufactured user approvals that were never given**, in a document whose own convention makes that
distinction load-bearing.

*Collateral — structural damage invisible to content checks.* The same script consumed the blank
line before 22 section dividers. In Markdown, text immediately followed by a horizontal rule becomes
a setext heading — so 22 status lines silently became headings and 22 dividers disappeared. At one
site it consumed the newline before a real heading and destroyed it.

**Why the orchestrator's own summary was useless here:** the report it would have written was "zero
remaining, counts consistent, no text lost." Every clause was true. None of them touched the actual
failures. Content-level verification cannot see positional misassignment or whitespace-dependent
structure.

**Fix / rule now in place:**
- **Never match records by position in a bulk edit. Match by identifier, every time.** If the data
  has no identifiers, that is a signal not to script the edit.
- **Never mutate a string while iterating over match offsets taken from it.** Re-scan after each
  edit, or collect all edits and apply them back-to-front.
- **A scripted edit to a structured document must be checked for structural damage, not just
  content damage.** "No text was lost" is not "the document still parses as it did." Whitespace
  before a divider or heading is load-bearing.
- **A clean summary from the process that did the work is not verification.** This is the specific
  case where an orchestrator's self-report is worth nothing.

---

### 2026-08-31 — Independent audit of the orchestrator's own writing found errors at a consistent, high rate

**Status: verified** — five separate audit passes, each dispatched read-only against work the
orchestrating session had just produced.

**What happened:** across one long session the orchestrator wrote a substantial volume of prose
containing cross-references, counts, and citations into a body of documentation. Five independent
audit agents were dispatched at intervals to check that writing. They found roughly **19, 12, 9, 12
and 20 problems** respectively.

The errors clustered into three repeatable kinds, none of which self-review would surface:

1. **Self-generated tallies that were wrong and internally inconsistent** — counts the orchestrator
   produced itself, in one case three different wrong values for the same quantity across a single
   paragraph. Nobody fed it those numbers; it generated and then trusted them.
2. **Citations that went stale during the same session that wrote them** — line-number references
   into files the orchestrator was concurrently editing. One "correction" to a stale pointer went
   stale again *during the run of the agent applying it*, which flagged rather than silently
   re-fixed it.
3. **Load-bearing claims asserted without checking** — including one that credited a colleague with
   fixing something they had not touched, and one that mis-attributed a decision's provenance in a
   way that would have made downstream work rest on unverifiable evidence.

**What made the audits effective, beyond simply existing:**
- Instructing the auditor to **re-derive rather than re-read** — recompute the number, re-run the
  measurement, open the primary source — rather than confirming that the text says what it says.
- Naming the **specific failure modes already observed** in the prompt, so the auditor knew what
  shape of error to hunt.
- Telling the auditor the author's **prior error rate**. Later audits were given "four prior audits
  of this author found 19, 12, 9 and 12 errors." Those runs went visibly deeper. *(Asserted — no
  controlled comparison was run; the prompts differed in other ways too.)*

**Fix / rule now in place:** for any orchestrated work whose output is prose containing counts,
cross-references, or citations — not just code — dispatch an independent verifier as a matter of
course rather than as a judgment call. The orchestrator is structurally the worst reader of its own
writing, and the observed rate is high enough that "this pass was probably fine" is not a safe
default.

---

### 2026-08-31 — "Flag rather than guess" produced better agent outcomes than any amount of task detail

**Status: verified** — three separate agents behaved this way, each catching something a
guess would have buried.

**What happened:** several dispatched agents were given an explicit instruction of the form: *if a
target does not match its description, do not guess and do not fix it — record it as unapplied and
report what you actually found.* All three that received it used it, and each catch was real:

- One applying a specified list of corrections found that a pointer it had just been told to write
  had **gone stale during its own run**, because its earlier edits shifted the target. It refused to
  silently re-fix a value it had not been given, and reported it.
- One found that a count in its brief was understated by roughly threefold, applied the operative
  instruction rather than the stated number, and said so.
- One caught a summary line, outside its assigned list, whose components did not sum — and left it
  alone while reporting it.

**The contrast worth noting:** the same session's *own* scripted edit had no such guard and
fabricated approvals (see the first entry above). The difference was not capability — it was that
the agents had been told what to do when reality disagreed with the brief, and the script had not.

**Fix / rule now in place:** include an explicit disagreement clause in every implementation or
edit-applying spec — what to do when the target does not match its description, stated as *report,
do not resolve*. This appears to buy more than additional detail about the task itself, because it
covers the case the spec author did not anticipate.

---

### 2026-08-31 — Subagents corrected the orchestrator's brief premise twice, unprompted

**Status: verified** — both corrections were independently re-checked against primary sources and
both held.

**What happened:** two dispatched agents contradicted premises stated as fact in their own briefs
rather than working around them.

- One was told a piece of state was orphaned by a planned removal. It found a live consumer, said
  the brief's premise did not hold, and named the consumer.
- One was told a filter existed in a particular place. It reported the filter absent — correctly for
  the scope it had searched — which surfaced that the brief had understated the search scope rather
  than that the filter did not exist.

**Why this is worth logging rather than assumed:** an agent that silently works around a wrong
premise produces output that looks correct and quietly encodes the error. Both of these instead
returned the contradiction as a finding. Neither prompt asked for premise-checking explicitly,
though both did ask the agent to cite evidence for every claim — which may be what produced it, by
making an uncitable premise visible. *(Asserted — the causal link is an impression, not tested.)*

---

### 2026-08-31 — A background agent survived host-process termination and resumed by message

**Status: verified** — the resume was performed and the agent delivered its full report.

**What happened:** a dispatched background agent was still running when the host session terminated
for an unrelated reason. The orchestrator received a notification that no completion record existed
and that the agent may have been running at exit.

Sending the agent a message addressed to its ID resumed it from its saved transcript, and it
returned a complete report. The resume message explicitly told it to carry forward already-completed
work rather than re-derive it, and to deliver a partial result if it was mid-way rather than going
quiet again.

**Worth knowing:** the recovery path exists and works, so an interrupted background agent is not
lost work. The instruction to prefer a partial delivered result over a complete undelivered one is
worth including in the resume message — an agent resumed without it may restart rather than
continue.
