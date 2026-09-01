### 2026-09-01 — Catch-up reconstructs internal state but has no notion of what was already communicated outward

**What happened** [verified]: a session opened with `/catch-up` to review an external
developer's open pull requests and check whether prior feedback had been addressed. Every
local artifact needed was present and current — resume note, canonical status docs, per-branch
review files, git state. The session still could not answer the first question the task
required: *which of our findings has the other party actually seen?*

The local review documents held a large set of findings. A much smaller subset had ever been
sent. Nothing in any document distinguished them. One doc contained an incidental sentence
("the two comments the user left on the PR are what produced these") that implied the
distinction existed, without recording it. The review platform was not reachable from the
environment, so there was no way to read it from the other side either.

The cost was not small. The session's entire first phase went to establishing that the
question was unanswerable, and the eventual resolution was to ask the human. Later in the same
session, working through six review threads one at a time, the same gap recurred twice more:
two standing findings were traced to an earlier review round with no record of whether either
had ever been raised.

**The general shape.** Resume notes and catch-up capture two things well — *what we know* and
*what is next*. They have no slot for *what has been communicated*, and that third category
behaves differently from the other two: it lives partly outside the repository, it cannot be
reconstructed from git or from the working tree, and it decays silently because nothing in the
project contradicts a wrong assumption about it. Any project whose output is consumed by
someone outside the session — pull request comments, tickets, email, a design doc handed to
another team — has this gap. It is invisible in single-player work and immediately expensive
in collaborative work.

**Rule concluded:** outbound communication is first-class session state and should be recorded
as deliberately as findings are. The fix applied in-project was a section recording what was
actually posted — anchor, substance, and what was deliberately withheld and why — kept
separate from the findings list. Worth considering whether `wrap-up`'s resume note should
prompt for it directly, since it is the one category of state the next session cannot recover
on its own.

---

### 2026-09-01 — The orchestration doctrine has no defined fallback when subagents are unavailable

**What happened** [verified]: the project's `CLAUDE.md` carried the standard orchestrator
framing — delegate all implementation, never write code directly, get an advisor review before
reporting any deliverable done. The session was simultaneously configured with an instruction
not to spawn subagents unless the user explicitly asked.

These are not reconcilable. Two of the doctrine's core obligations become unperformable, and
nothing in the skill or the agent definitions says what should happen. The session flagged the
conflict once, proceeded without delegation, and mentioned the skipped advisor pass when
reporting a deliverable — but that was an improvised judgment call, not a documented path.

The work in question was review and documentation rather than implementation, so the cost was
low here. It would not have been low for a build task: "never type code yourself" with no
lanes available leaves no legal move at all.

**Rule concluded:** not yet promoted — logged. The doctrine currently assumes lane availability
as a given. Worth deciding explicitly what degrades and what holds when it is not: plausibly
the advisor review becomes "state that it was skipped and why", and the delegation rule
becomes "do it directly, and say so", rather than both silently lapsing. The failure mode to
avoid is an orchestrator that either freezes or quietly abandons the doctrine without telling
anyone.

---

### 2026-09-01 — Testing-mode findings had one sink, gated behind a command that could not run

**What happened** [verified]: testing mode's session banner instructs the model to capture
friction and sweep it via `wrap-up` before the next `/clear`. This session ran alongside a
second session in the same project, and that session owned the resume note. Running `wrap-up`
here would have written over another session's work, so it was correctly not run — and the
banner's only named path for the findings went with it.

The findings survived only because the user asked twice, unprompted, whether anything still
needed writing down. A `pending-lessons/` directory already exists and is exactly the
write-anytime sink for this case, but nothing in the testing-mode instruction mentions it; it
was found by inspecting the notes repository directly.

**Rule concluded:** the testing-mode banner should name `pending-lessons/` as the sink that
does not require `wrap-up`, so findings can be written the moment they are noticed rather than
accumulating against a command that may never run. The current phrasing makes an optional
end-of-session ritual the single point of failure for the whole feedback loop.

---

### 2026-09-01 — Nothing addresses two sessions working the same project concurrently

**What happened** [verified]: the user explicitly ran two sessions in parallel on one
repository, telling this one to take a separate task. The other session owned
`RESUME-PROMPT.md`. This session avoided that file, confined its writes to documents the other
was not touching, and said so — again, judgment rather than guidance.

`catch-up` and `wrap-up` both assume a single session owns the project's notes. There is no
ownership convention, no staleness check on read, and no mention of concurrency anywhere. The
risk is concrete: both commands read and rewrite the same canonical files, so two sessions
wrapping up in sequence can silently discard each other's updates.

**Rule concluded:** not yet promoted — logged, and lower confidence than the entries above
since it rests on one occurrence. Worth watching for a second instance before acting. If it
recurs, the cheapest mitigation is probably a re-read-before-write on the resume note plus an
explicit line in `catch-up`'s output naming which files this session intends to own.

---

### 2026-09-01 — Verification doctrine covers "is the claim true", not "is the behaviour wrong"

**What happened** [verified]: a single review finding produced three reversed recommendations
in one conversation. Pass one dropped it against the user's triage bar. The user pushed back on
a weak hedge, an empirical test proved the bad state was genuinely reachable, and pass two
reversed to "raise it". The user then asked whether it was needed to meet the client's
requirements — and checking the client's own spreadsheet formula showed the existing code
already matched it, and the proposed fix would have diverged from it. Pass three dropped it
again, for a reason neither earlier pass had looked for.

The empirical test in the middle round was correct and irrelevant. It established that the
state could occur; it said nothing about whether handling it differently was right. The
authority on that was a source file sitting in the project the whole time.

**The general shape.** For findings about a *metric* — a count, a rate, a grouping — "the code
does X" and "X is wrong" are separate claims. Verification doctrine, as written, is entirely
about the first: confirm the claim, cite the evidence, do not relay an unverified assertion. It
has no step for confirming that the alleged defect is actually a defect against whatever
external source defines correct. Proving reachability is cheap, satisfying, and answers the
lesser question convincingly enough to move a verdict that was already right.

The same check later *strengthened* a different finding on the same branch, so this is not a
bias toward dropping things — it is the step that decides either way.

**Rule concluded:** not yet promoted — arguably a reviewing lesson rather than an orchestration
one, and it lands in the same territory as the existing entry about volunteered context being
treated as a lead rather than a finding. Logged because the shape is distinct: that entry is
about not over-trusting a lane's claims, this is about the architect's own severity
classification having an unexamined premise. Also worth noting separately: when a
recommendation reverses, the reversal itself has a cost the individual verdicts do not show —
the human re-runs the same decision each time — so the new evidence that caused the change
should be stated explicitly rather than the new position being presented as if it were the
first.
