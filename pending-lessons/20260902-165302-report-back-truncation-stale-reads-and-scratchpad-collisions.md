### 2026-09-02 — Multi-item report-back lists get truncated to the last item (**verified**)

**What happened:** across a long orchestration session, five `routine-implementer` agents were
each dispatched with a numbered "report back" list of four to six items. **Three of the five
returned only the final item** — in each case the trivial one (a `git status` confirmation) — and
silently dropped the substantive items above it: counts, the payload list of findings, and the
"anything that made this uncertain" self-report.

The deliverables themselves were complete and correct every time. Only the agent's *account of its
own work* was lost. That is the worst thing to lose, because the orchestrator's whole reason for
asking "what made this hard" is to learn where not to trust the output.

Detected only because the orchestrator independently opened the output files and found substantial
work that the report had not mentioned. An orchestrator that trusted the report-back would have
concluded three of five agents had produced almost nothing.

**Fix/rule now in place:** do not treat an agent's report-back as a reliable channel for anything
load-bearing. Put the payload in the written artifact and read it. If a report-back list is used,
**put the most important item first, not last** — the observed truncation kept the tail, but the
ordering assumption is unverified and the safe move is not to depend on position at all.

**Where it lives:** logged here. Candidate doctrine change: the standard spec's "report back"
section currently reads as though it is a dependable return channel, and on this evidence it is
not.

---

### 2026-09-02 — Long-running agents read a stale snapshot when the orchestrator edits the same file mid-flight (**verified**)

**What happened:** twice in one session, an agent reported that a document "still contains" a claim
that the orchestrator had already corrected while that agent was running. Both reports were written
in good faith and were false by the time they arrived. In one case the agent's report explicitly
recommended work that had already been done.

This is structural, not a bug in the agent. Agents run for minutes; a document read at dispatch is
a snapshot. The orchestrator, working in parallel, is the thing most likely to invalidate it.

The second-order risk is worse than the wasted turn: a report saying "X is still broken" invites
re-doing a fix, and a fresh agent following it may distrust a correction it can plainly see,
because an authoritative-looking note says it does not exist.

**Fix/rule now in place:** treat any agent finding about *file contents* as timestamped at dispatch,
not at report. Before acting on one, re-check the file. And avoid editing a file that a running
agent was told to read — either wait, or send the agent a correction mid-run (which worked cleanly
twice this session when a decision was retracted while two agents were writing it up).

**Where it lives:** logged here. Relates to the same class as the freshness rule already applied to
comparison artifacts — an artifact must be verified current *at the moment of use*, and that
applies to an agent's report as much as to a checked-out repository.

---

### 2026-09-02 — Parallel agents sharing one scratchpad root overwrite each other's files silently (**verified**)

**What happened:** five agents were dispatched simultaneously and all pointed at the same scratchpad
directory. Two independently wrote helper scripts with the same generic filename, and one was
overwritten mid-run by the other. The affected agent noticed, confirmed its already-captured data
was unharmed, and reported it rather than staying quiet — but nothing in the dispatch prevented it,
and a less careful agent would not have caught it.

No damage this time. The failure mode is silent and would be very hard to attribute after the fact:
an agent's script changing under it produces a confusing wrong result, not an error.

**Fix/rule now in place:** when dispatching agents in parallel, give each one its own **namespaced
subdirectory** of the scratchpad and say so explicitly in the spec. This cost one line per dispatch
and was applied for the rest of the session with no further collisions.

**Where it lives:** logged here. Candidate addition to the orchestration doctrine's dispatch
guidance — it is a property of fanning out, so it belongs with the fan-out advice rather than being
rediscovered per session.

---

### 2026-09-02 — Instructing an agent to verify its own brief caught orchestrator errors twice (**verified**)

**What happened:** two agents were given briefs containing a factual claim the orchestrator believed
and had not checked. Both were also told to verify load-bearing facts against the primary source
rather than take the brief on trust. Both did, and both found the orchestrator's claim wrong — one
about where a set of files actually lived, one about the state of a document.

Separately, one implementation agent ran an advisor review over its own output before reporting, and
that review caught real errors in its work: instruction-violating wording, a false "verified"
claim, and a mischaracterised operation. The agent corrected them against the primary source before
reporting, and disclosed what it had changed.

**Conclusion, held as a rule:** the spec should say explicitly that the brief may be wrong and that
the agent is expected to check it, not just execute it. Without that, a wrong premise from the
orchestrator propagates into the deliverable with the agent's authority attached — and the
orchestrator has no way to detect it, because the agent is the thing that would have noticed.

The self-advisor pattern is worth making routine for anything whose output becomes a reference for
later work, rather than reserving the advisor for end-of-deliverable review.

**Where it lives:** logged here. The first half is a spec-template change ("verify the brief");
the second is a candidate doctrine change (when an implementer should run its own advisor pass
rather than waiting for the orchestrator to).

---

### 2026-09-02 — Agents killed by a rate limit leave partial state with no progress signal (**verified**)

**What happened:** two agents were terminated mid-run by a session rate limit. One had completed
nearly all of its work; the other had written nothing. The failure notification reported only that
each had failed — nothing about how far either had got.

Determining the actual state required inspecting the filesystem directly: which files existed, which
carried the expected markers, and which of a fourteen-item checklist had been applied. The nearly
complete agent's work was fully salvageable and would have been redone from scratch had the failure
been taken at face value.

**Fix/rule now in place:** on any agent failure, **assess partial state before re-dispatching**. Do
not assume a failed agent did nothing, and do not assume it finished what it last reported. Design
long agent tasks so partial completion is externally detectable — a per-item marker in the artifact
rather than a single write at the end.

**Where it lives:** logged here. The recovery worked, but only because the state happened to be
inspectable; a task that buffered its output to a single final write would have lost everything.
