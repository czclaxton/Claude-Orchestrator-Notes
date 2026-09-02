### 2026-09-01 — `/wrap-up` writes a handoff note that nothing ever validates; cold-reading it found ten defects

**What happened** [verified]: at the end of a long session, `/wrap-up` produced a resume note in
the usual way — decided vs. open, the single next action, what a fresh session would otherwise
re-derive. It read well. The user then asked for a second pass, and an agent with no session
context was handed the note and asked the standard cold-read questions: *what is the state, what
is the next action, can you start it, what does it assume you know, where would you do the wrong
thing.*

**Ten defects, of which three would have cost real time:**

- **The next action was not actionable.** The note said "then open the PR" and never gave the
  base branch, the host, or the body format. The reviewer noted the repository's remote was not
  the host the obvious CLI reflex assumes, so the first command a fresh session would reach for
  would simply fail.
- **A verified-against-the-server claim was true but incomplete.** The note asserted branch state
  had been confirmed against the remote, and omitted that the branch was nineteen commits behind
  its integration branch. The reviewer established the divergence was harmless; the orchestrator
  had not checked at all. **A partial truth under a "verified" label is worse than no claim** —
  it stops the reader looking.
- **An open question had been flattened into a closed one.** The note listed several things that
  were "supposed to look wrong, do not file these" — a genuinely useful section. But one item was
  cited against the wrong identifier: the underlying behaviour was settled, while the *remedy* for
  it was still an explicitly open question in the register, marked undecided with a correction
  note attached. The next session would have read a live decision as answered. **This is the
  failure running in the dangerous direction:** the note's own safety mechanism, mis-cited.

Also found: a second in-flight workstream the note never mentioned; a maintenance rule ("keep
these two branches from drifting") shipped with no command to detect drift; no pass criterion for
what "verified" would even mean; and a stated claim that everything referenced lived in a durable
document, when the referenced directory was git-excluded and had never been pushed — meaning a
large reasoning corpus existed on exactly one machine.

**Why this is a `/wrap-up` finding and not a one-off.** The command's entire purpose is that the
*next* session can act on the note. The author is the worst-placed reader for exactly the reason
cold reads exist everywhere else in this project: they cannot see their own assumed context.
Every one of the ten defects is invisible from inside the session and obvious from outside, and
the cost of finding them is one short-lived agent.

**Rule concluded:** `/wrap-up` should cold-read its own resume note before finishing — hand a
fresh agent the note and nothing else, ask whether it could execute the stated next action, and
fix what comes back. The prompt is already standardised elsewhere in this project's protocol
documents; it just needs pointing at the note. Candidate home: a step in the `/wrap-up` command,
between writing the note and marking findings swept.

**One specific check worth naming in that step:** have the reader verify the note's own factual
claims against the server, not just against local state. Two of this note's defects were claims
that were true locally and misleading about the remote.

---

### 2026-09-01 — The verification discipline covers implementation lanes and lets review output through unchecked

**What happened** [verified]: across a long multi-agent run, the orchestrator held the doctrine's
verification line rigorously against every *implementation* lane — re-ran all mechanical gates
itself after each step rather than trusting a report, read every diff, re-checked cited library
claims in the installed dependency, and caught real errors doing it.

Then a *review* agent returned a ten-item report, and the orchestrator verified two items and
wrote three of the remaining eight into a durable handoff artifact as fact — including one
phrased as "byte-identical", on which a future session was told it could safely delete a backup.

When the user asked directly how the review agent had been evaluated, the honest answer was: it
had not been. The three claims were then checked and all held — but the verification came after
the artifact was written, and only because someone asked.

**The structural cause, which is the actually useful part.** The doctrine's verification section
is written entirely about implementation: *reports are claims, not evidence; read the diff,
re-run the command; "should work" means not done.* Nothing in it addresses output from an
`advisor` or a review-shaped agent. And review output **feels like verification rather than
something requiring verification** — it arrives in the same shape as a finished check, framed as
findings about someone else's work, which is precisely the framing that disarms scrutiny. An
implementer's claim invites a re-run; a reviewer's claim invites agreement.

**A second, cheaper miss in the same interaction** [verified]: the review agent was never asked a
follow-up. It returned ten findings in its own severity order and that order was accepted whole.
The list mixed a stylistic redundancy note and a mis-cited open decision at the same level. One
question — *which three would you keep if you could only keep three?* — would have produced a
better fix list at trivial cost. Multi-turn continuation with a subagent is available and went
unused; the default posture treated the report as final rather than as the opening of a
conversation.

**Rule concluded:**
- **Extend the verification rule explicitly to review and advisor output.** A reviewer's factual
  claims get re-checked before they are acted on or written into anything durable — the same bar
  as an implementer's. The asymmetry is not justified by anything; it is an artifact of the
  doctrine only ever discussing lanes that produce code.
- **Verify before propagating, not after being challenged.** The specific failure was writing an
  unverified claim into a durable artifact, which converts a soft assertion into something a
  future reader will act on.
- **Treat a review report as the start of an exchange.** At minimum, ask a returning reviewer to
  rank or prune its own findings before acting on the list.

Candidate home: the verification section of the orchestration doctrine, and the advisor guidance
alongside it.
