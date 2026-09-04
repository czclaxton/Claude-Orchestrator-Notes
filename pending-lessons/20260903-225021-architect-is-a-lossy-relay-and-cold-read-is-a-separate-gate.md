### 2026-09-03 — The architect is a lossy relay: a subagent's qualified finding became a false absolute one hop downstream

**What happened** [verified]: an exploration agent reported, precisely, that a particular UI pattern
existed in the codebase but that in the one place it appeared, both code paths produced the same
outcome — so the pattern had never actually had to separate two behaviours. The orchestrator then
wrote a spec for a downstream agent asserting that the pattern *had never been used at all*. The
downstream agent checked, found the pattern in a named file with line numbers, and corrected the
orchestrator. Re-verified afterwards: the exploration agent was exactly right and the orchestrator's
restatement was false.

Nothing was fabricated. The qualifier was dropped in transit. The exploration agent's finding had
the shape "X exists, but with a caveat that means it hasn't been stress-tested"; the orchestrator
compressed it to "X doesn't exist."

**The general shape.** This pattern makes the orchestrator a relay between agents that never speak
to each other, and every relay is lossy in a specific direction. Qualifiers compress away first
because they read as hedging, and the absolute form is more persuasive and shorter — so the error
travels better than the truth did. It is distinct from the usual "verify agent claims" failure,
where the agent was wrong; here the agent was right and the orchestrator degraded a correct finding
while summarizing it into a spec.

It is also self-concealing. The orchestrator has no reason to re-check something it just read, and
the downstream agent has no way to know a qualifier was stripped unless it happens to check the
underlying claim for its own reasons. In this instance it was caught only because the downstream
task made the claim load-bearing.

**Rule concluded:** when relaying a subagent's finding into another agent's spec, quote the finding
rather than paraphrasing it, especially where it carries a qualifier. A finding that reads "X, but
only under condition Y" must arrive downstream with condition Y attached or not at all. On the
receiving side, treat a summary from the orchestrator as a claim to verify, not as established
ground — including when the orchestrator is the one directing the work.

---

### 2026-09-03 — Narrowing an ambiguity is a scope decision, and a documentation agent will make it silently

**What happened** [verified]: an agent was dispatched to amend a planning document, including
resolving passages that contradicted each other. It found a scope statement that was genuinely
ambiguous — a phrase that could be read two ways, one of which contradicted the document's own build
steps. It resolved the ambiguity in the reasonable direction and marked the change as a
clarification.

In doing so it converted a vague phrase into an explicit exclusion of a feature. The user had never
agreed to that exclusion, discovered it later while looking at the running software, and asked how
it had happened. There was no decision record for it anywhere; it lived only inside the document, in
a paragraph dated the same day it was written.

Two more instances of the same family surfaced in the same session: a documented "default" recorded
for a question that an approved design in a different document had already answered, and a
consequence of a user ruling written directly beneath that ruling's heading, which gave it the
grammar of a decision the user never made. When the user challenged the third one, the record showed
he was right — the option he supposedly chose had never been put to him.

**The general shape.** All three have one shape: **a consequence recorded in the grammar of a
decision.** Each individual inference was reasonable. That is what makes it dangerous — *reasonable*
is not the test, because a reasonable inference recorded as a ruling is indistinguishable from a
ruling once the session that made it is gone. The next reader has no way to tell what was decided
from what was derived, and the derived thing inherits the authority of the decision it sits under.

Documentation-editing agents are especially prone to this because their whole job is resolving
inconsistency, and the boundary between "this passage is unclear" and "this passage understates the
scope" is invisible from inside the task.

**Rule concluded:** three things, all cheap. When a documentation agent's clarification changes what
is in or out of scope, that is a decision and must surface to the user rather than settling inside a
document. When recording a consequence of a user's ruling, mark it as a consequence and name the
ruling it derives from, so a later reader can separate the two. And before recording any new
"default," grep the existing record for an answer that already exists — in this session that grep
paid for itself within minutes of being written down, surfacing a document that had answered the
question weeks earlier.

---

### 2026-09-03 — Cold-read execution testing and quality review are different gates and find disjoint defects

**What happened** [verified]: a long planning document was put through two separate review agents.
The first was a quality review — is this good practice for the language and framework, is it
consistent with how the codebase already does things. It returned a "not safe to implement" verdict
with seven substantive findings, several with runnable reproductions.

The second was a cold-read execution test: a fresh agent with no session context, handed the
document and asked what it would actually *do*, step by step, and what it found ambiguous. It found
a different set of problems entirely — a file the document referred to but never named, a step
described as mechanical that required a design decision, a summary line that contradicted a step's
own exit condition, and a retrieval failure in how retired content was marked.

Neither review found the other's findings. The overlap was approximately zero.

**The general shape.** The two lenses ask genuinely different questions. Quality review asks whether
the described solution is correct and idiomatic; it reads with domain expertise and naturally
assumes competence at following the document. Execution testing asks whether the description is
*followable by someone who was not there*; it reads without context and trips on exactly the things
expertise papers over — an unnamed file, an unstated dependency, a phrase that reads as an
instruction but is history.

The second is the one that maps to how this pattern actually consumes documents. Plans written in
one session get executed by agents in another, with no shared memory. A plan can be entirely correct
and still unexecutable, and a quality review will not notice, because a competent reviewer fills the
gaps automatically without registering that it did.

**Rule concluded:** treat these as two gates, not one, and run both before handing a plan off for
execution. The execution test should be genuinely cold — a fresh agent, no session context — and
should be asked to *do* the plan rather than assess it: name every file you would touch, what
happens if a user arrives by this other path, what did you have to guess at. Ask directly what was
ambiguous; that question produces the highest-value findings and reviewers do not volunteer them
unprompted. Consider running the execution test on a mid-capability model rather than the strongest
available — a stronger model infers past gaps that a weaker one trips on, and tripping is the signal
being bought.

---

### 2026-09-03 — Marking retired content in place does not survive arrival-by-search; only separation closes it

**What happened** [verified]: a long-lived planning document accumulated two build orders — one
already executed, one live — alongside a disciplined convention for marking superseded content in
place rather than deleting it. A cold-read reviewer confirmed the convention worked: every apparent
contradiction it chased resolved correctly, and it praised the markers as unusually rigorous.

It then demonstrated the failure anyway. Grepping for an instruction-shaped phrase returned a line
deep inside the retired section with no marker on that line. To know it was history, a reader had to
work out that the line sat inside a span opened hundreds of lines earlier.

The first fix tagged each retired step's own heading. A second cold reader confirmed that closed
arrival-by-step — and then reproduced the same failure one level down, on sub-bullets inside those
steps, twice. Its summary: the risk in the document was not contradiction, it was retrieval. The fix
that actually worked was moving the retired content into a separate file, leaving a pointer. A grep
of the live document can then never return a retired instruction at any nesting depth.

**The general shape.** In-place supersession markers are designed for a reader moving through a
document sequentially, and they work well for that reader. Agents frequently do not read that way —
they grep for a phrase and act on what comes back, and a single line of grep output carries no
information about the span it sits inside. Every level of tagging fixes the level above it and
leaves the level below exposed, so the effort is unbounded while the exposure never reaches zero.

The convention itself is sound and worth keeping; recording superseded reasoning rather than
deleting it repeatedly proved its value in the same session. The error is assuming co-location is
free. Retired content and live content in one file are only separable by a reader who knows where
the boundaries are.

**Rule concluded:** mark superseded *reasoning* in place, since that is context a reader wants
beside the thing it explains. But move superseded *instructions* — anything imperative that an agent
could act on — into a separate file with a pointer left behind. The test is not "is it marked" but
"does one line of grep output, alone, tell a reader this is not live." If the answer is no, tagging
harder will not fix it, because the next nesting level has the same problem.
