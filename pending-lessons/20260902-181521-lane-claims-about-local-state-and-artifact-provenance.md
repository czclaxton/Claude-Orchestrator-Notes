### 2026-09-02 — A research lane asserted facts about local state it had no access to, and durable artifacts carried no provenance line between user decisions and agent recommendations

**Status: finding 1 verified** (the lane's claim was contradicted by re-checking the artifact
directly, twice, using an independent method); **finding 2 verified** (the user's intervention and
the resulting restructure are a matter of record from this session).

Context: a long single-session investigation, mostly direct orchestrator work, with exactly one
read-only research subagent dispatched — web access only, no filesystem tools.

---

**Finding 1 — a lane with no access to the local environment will still make confident claims
about it, and those claims are structurally unfalsifiable from inside the lane.**

The research lane was given a factual background paragraph describing an artifact the orchestrator
held locally, including a version string it reported. The lane's job was purely external research.

Its report was strong overall — and it did something genuinely commendable, covered below. But it
also closed a section with an inference about the orchestrator's own machine: that the version
string implied the local artifact was the *wrong variant* for the hardware, phrased as "a strong
signal you may currently have" the wrong build. The orchestrator had already hashed that artifact
and confirmed the opposite. The lane's reasoning was sound given what it knew; the premise was
simply wrong, because both variants report identical version strings — a fact the lane had no way
to discover and no way to test.

This is a distinct axis from the existing 2026-08-27 entry about volunteered context. That entry
covers claims a lane makes about the *codebase it can see* — counts, prevalence, "this also
affects…" — which the orchestrator can check by re-running a command. This is a lane reasoning
about state it **cannot** see at all, from a description the orchestrator itself supplied. Three
things make it worse than the earlier pattern:

- It is unfalsifiable in-lane. There is no command the lane could have run to catch itself.
- The premise came *from the orchestrator's own prompt*, so the error round-trips: the orchestrator
  supplies a partial description, the lane reasons over it correctly, and the conclusion comes back
  wearing the authority of an independent finding.
- It is the most tempting kind of sentence to relay, because it reads as the lane connecting
  external research to the user's specific situation — exactly the synthesis you dispatched it for.

The only thing that caught it was the orchestrator having independently verified the artifact
*before* dispatching. Had the order been reversed, the claim would have been relayed.

**The commendable part, worth logging as a success:** the lane opened its report with an unprompted
methodology note stating that two of its own tool-call summaries had returned fabricated facts —
naming them specifically — and that it had therefore switched to fetching raw source and verifying
directly. It then labelled every claim as either independently read or summary-only. That is
precisely the behavior the doctrine wants and it was not asked for. The lane's self-reported
reliability was better calibrated than the doctrine's default assumption about lane output.

**Rule concluded:** a lane's claims sort into three tiers, not two — assigned output (verify by
re-running), volunteered context about what it can see (verify independently), and **claims about
state the lane has no access to** (never relay; these are restatements of the orchestrator's own
prompt with added confidence). The third tier should be named explicitly, because it is the one
where verification inside the lane is impossible by construction. A practical corollary: verify
local state *before* dispatching a lane you will hand a description of it, not after.

---

**Finding 2 — the doctrine has nothing about provenance in durable artifacts, and the gap surfaced
as a direct user complaint.**

The session produced a set of written reference documents intended to outlive it. They mixed, with
no marking, three different kinds of content: requirements the user had actually stated, factual
observations the orchestrator had verified, and recommendations the orchestrator had reasoned its
way to.

Mid-session the user interrupted to ask, unprompted, for a hard separation — explicitly so that a
future session would not read an earlier session's recommendation and cite it back as an
established decision. His framing: he did not want a later agent using something a previous agent
generated as *evidence* for a position he had never agreed to.

That is a real failure mode and the doctrine does not address it. The orchestration skill treats
verification as a within-session concern: the lane reports, the orchestrator checks, the orchestrator
relays. It says nothing about what happens when the relayed material is written down and read later
by a session with no memory of who originated it. At that point every sentence in the artifact has
equal apparent authority, and the reasonable-sounding recommendation is indistinguishable from the
requirement — with the recommendation often *better argued*, because it was written to persuade.

Two details worth keeping:

- The default was wrong in the accumulating direction. Nothing marked anything, so everything read
  as settled. Silence defaulted to authority rather than to "unconfirmed."
- The user had to ask. Left alone, the session would have kept producing well-written documents
  that quietly laundered agent reasoning into apparent consensus, and the problem would only have
  been visible several sessions later when something got built on it.

The fix adopted was cheap: one file holding only user-stated items, each sourced and dated, with an
explicit rule at the top that agent output never enters it and that silence is not agreement; a
banner on every other document saying it is not authoritative and that the canonical file wins on
conflict. Roughly fifteen minutes, and it noticeably changed how the rest of the session was
written — claims that would have been stated flatly got marked as inference instead.

**Rule concluded:** where a session produces durable artifacts, the orchestrator should separate
user-stated decisions from agent-generated recommendations *by construction* — a distinct location
for the former, provenance marking on the latter — rather than relying on a future session to
re-derive who said what. This is the multi-session analogue of "reports are claims, not evidence":
the doctrine already refuses to let a lane's report become fact within a session, and should refuse
to let a session's own conclusions become fact across sessions. Related in spirit to the 2026-09-01
entry on state that exists but is unrecorded, though this is the inverse: the information is all
present, and what's missing is its origin.

---

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's Verification section,
extending the assigned-output / volunteered-context distinction with a third tier for claims about
inaccessible state, plus the ordering corollary. The lane self-reporting its own tool-summary
fabrications is worth noting as behavior to encourage in agent definitions. Finding 2 has no
existing home — the doctrine covers within-session verification and says nothing about artifact
provenance across sessions; it may warrant its own section.
