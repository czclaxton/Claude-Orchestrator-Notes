### 2026-08-30 — A long-running lane reported its *last pass* as an addendum to a report the orchestrator never received

**What happened** [verified]: two research lanes were dispatched in parallel, each with an explicit
five-section report format. One ran long (~100 tool calls, two internal research passes). Its
completion notification contained only the second pass, written throughout as revisions to the
first — "three revisions to my recommended change set", "I proposed it, then withdrew it", "worse
than my §3 table implies". The §3 table had never left the lane. The spec was not the problem; the
lane had a correct format to fill and departed from it. What it appears to have done is treat its
own internal reasoning as shared context with the orchestrator, and report the delta.

The recovery worked cleanly: a follow-up message asking for a standalone consolidation, explicitly
prefaced with "assume I have seen nothing" and enumerating what to restate, returned a complete and
well-formed report. But that was a second full pass on an expensive lane, and the failure was
silent — the addendum was internally coherent and read like a finished deliverable. Nothing about
it announced that a prior report was missing except the dangling cross-references, which are easy
to skim past as ordinary section numbering.

**Rule concluded:** for any lane expected to run long or iterate internally, the spec should state
that the final message is the *only* thing the orchestrator will ever see, and that it must stand
alone with no reference to the lane's earlier reasoning. Correspondingly, an orchestrator receiving
a report that references its own unshown sections should treat that as a truncated deliverable and
re-request, not reconstruct the missing half by inference — the temptation is real, because the
delta usually contains the most interesting conclusions and reads as sufficient.

**Finding 2 — when a lane's finding contradicts the project's own maintained documentation, the
documentation is a live suspect, and the spec should say so out loud** [verified]. Both lanes were
specced with the project's existing written claims quoted inline and labelled "verify, don't
trust". Both came back contradicting those claims, and on independent re-reading of the underlying
source files the lanes were right in every instance — the documentation had recorded values
measured under a configuration that no longer applied, and presented them as the baseline. One of
the contradicted claims was the project's own headline recommendation.

This inverts the usual posture. The standing verification doctrine, and the 2026-08-20 entry above,
are both about a lane's claims being *less* reliable than they look. The reverse case has its own
failure mode: the orchestrator has been reading the project's documentation all session, trusts it
as settled, and is primed to treat a contradicting lane as the thing that got it wrong. Here that
instinct would have been wrong three times, and would have shipped stale numbers into a build.

**Rule concluded:** quoting existing project claims into a spec is worth doing — it gave both lanes
a concrete target and is what surfaced the staleness — but the spec should name the documentation
as a *candidate* wrong answer rather than as context, and the verification step should resolve the
conflict against primary sources rather than against whichever side the orchestrator already
believes. Cheap in practice: the contradicted values were re-read directly in a single command
each.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's spec-writing section,
as a standing clause for long-running lanes. Finding 2 belongs in the Verification section, as the
counterpart to the 2026-08-20 entry — that one covers a lane overstating; this one covers an
orchestrator under-crediting one.
