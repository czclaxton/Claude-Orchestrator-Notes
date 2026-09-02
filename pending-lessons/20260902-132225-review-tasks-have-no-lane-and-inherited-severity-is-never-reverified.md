### 2026-09-02 — The routing table has lanes for building and for searching, none for judging

**What happened** [verified]: a session was opened under a project instruction reading, in
substance, *"delegate all implementation, never write code yourself, delegate broad exploration
to cheap read-only agents, and get an advisor review before reporting anything done."* The
session's actual task was a code review — re-examining an external contributor's changes against
a standing findings document to determine which items had been addressed.

The session delegated nothing. Not once, across the entire task. It read the diff itself, ran its
own verification commands, and wrote its own conclusions. This was the correct outcome, and it
also directly contradicted the instruction the session was operating under.

**Why it happened, and why it is structural rather than a lapse.** Every lane in the routing
table is defined by what it *produces*: three implementation rungs that write code, and a
read-only search agent that locates code. The advisor is defined by when it is *consulted* — at
commitment boundaries and once at the end of a deliverable — and it advises rather than acting.

A review task produces neither code nor a location. It produces a judgment about existing code,
and it is the deliverable rather than a step toward one. Nothing in the table fits, so the task
falls through to the orchestrator by default. There is no lane that says "no", so the session
never has to decide anything — it simply finds itself doing the work directly, with no moment at
which the contradiction becomes visible.

**The second-order effect, which is the more expensive one.** Delegation is not just cost
control here; it is where independent verification comes from. The doctrine's own rule is
*verify evidence before accepting any lane's report*. When there is no lane, there is no report,
and therefore nothing gets verified by anyone other than the author of the claim. A review
performed entirely inside the orchestrator is the one deliverable shape that receives no second
pass at all — while being the shape whose entire value is that it is a second pass.

**Rule concluded:** the routing table needs a lane defined by *judgment about existing work*, not
by artifact produced — review, audit, re-scoring a findings document against a moved target. Until
one exists, doctrine should say plainly that review is orchestrator-executed and name that as a
deliberate exception, so a session is not silently in violation of its own instructions for the
duration of the task. Related in family to the 2026-09-01 fragment on doctrine having no
no-subagent fallback: both are cases where the doctrine describes a happy path and is simply
silent about a situation the session is actually in, and silence reads as permission.

---

### 2026-09-02 — A findings document re-scored against a moved target gets its anchors re-verified and its adjectives believed

**What happened** [verified]: the review above scored a changed branch against a findings document
written the previous day. The document was disciplined — every load-bearing claim carried a file
and line reference, which is exactly what made re-verification cheap and which caught several
things.

One finding described a failure mode as **silent**. It was not. The code had grown real error
handling — structured messages naming the affected location, with dedicated branches for four
distinct error classes — in a commit that the same document listed by name in its own changelog
section. The word "silent" had been accurate when first written, was never re-checked, and passed
through a re-review, a summary checklist, a detailed elaboration, and a drafted outbound comment
to an external party. It was caught by the human asking *"is this a bug, or are we requesting
quality of life?"* — a question about severity, not about facts. Nobody disputed the evidence.
Someone disputed the framing, and the evidence then failed on the follow-up check.

**The general shape.** When a target moves, a findings document's *anchors* get re-verified,
because a file-and-line citation announces that it is checkable and stale line numbers fail
loudly. Its *adjectives* do not: "silent", "destructive", "no validation", "breaks every
request". Those are what set severity, and severity is what the reader reacts to and acts on. A
stale adjective reads perfectly, cites a real file, and describes a real defect at the wrong
magnitude — so nothing about it looks like it needs checking.

This is the same family as the earlier citation-drift finding, inverted. There, the conclusion
survived and the provenance decayed. Here, the provenance was perfect and the severity decayed.
Both are invisible to a reviewer who confirms only that the cited location exists.

**Rules concluded:**
- **Re-verify severity, not just anchors, whenever the reviewed target has moved.** The question
  is not "is this claim still at that line" but "is this still as bad as we said."
- **Any commit in the target's history whose description plausibly touches a standing finding
  invalidates that finding until it is re-read.** The invalidating commit here was named in the
  same document as the claim it invalidated.
- **Ask "defect or quality-of-life?" of every finding before drafting it outward.** Asked once by
  a human, it removed an item that was about to be sent to an external contributor as a change
  request. It costs one sentence per finding to ask it of all of them.

---

### 2026-09-02 — A persistent shell working directory turned a search into a confident wrong negative

**What happened** [verified]: mid-task, one command was prefixed with a directory change into a
subdirectory in order to run a tool that needed to execute from there. The shell's working
directory persists between calls in this harness. Several calls later, a file search issued with
a repository-relative path returned nothing — not an error, just no results — because it was
being run from the subdirectory rather than the root. The file existed.

The session had, minutes earlier, successfully found that same file with an equivalent search.
The contradiction is what prompted a re-check; without it, the next statement would have been
"that file does not exist on this branch."

**Why this is worth logging rather than filing as a slip.** The failure produced a *negative*
result, which is the dangerous direction: a wrong positive sends someone to look and self-corrects,
while a wrong negative ends the investigation. And the mechanism is invisible at the call site —
the command was well-formed, exited zero, and its empty output is indistinguishable from a genuine
absence. This is the same shape as the earlier finding about a verification step whose success
condition cannot distinguish success from failure.

**Rule concluded:** in a harness where shell state persists across calls, treat an empty search
result as *unproven* rather than as evidence of absence, and re-issue it with an absolute path
before making any negative claim from it. Cheaper still: never leave the working directory changed
— prefix the one command that needs it and return, or address the target absolutely in the first
place.
