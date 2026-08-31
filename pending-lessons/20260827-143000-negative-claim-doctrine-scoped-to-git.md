### 2026-08-27 — `/catch-up`'s negative-claim doctrine is scoped to git remotes, and the identical trap fired three times on non-git claims in the same session that read it

**Status: verified** (each of the three wrong claims was re-checked directly against its primary
source in-session, and each was contradicted; the correction sequence is a matter of record from
this session's transcript).

**The doctrine is right, well-argued, and never fired.** `/catch-up` carries an explicit section on
negative claims — that a negative about a remote requires server evidence before you make it,
because "a stale positive prompts someone to go look, a stale negative stops the investigation
entirely." It was read at session start and its git-remote instructions were followed correctly.

Then, across the same session, the orchestrator made three confident negative claims about non-git
subject matter — the contents of a large third-party web document, a constraint on how a piece of
third-party software can be used, and a compatibility conclusion drawn from a binary file-format
mismatch. All three were wrong. All three were caught by the user pushing back and asking for
verification, at which point checking the primary source took under a minute and contradicted the
claim each time.

**Why the doctrine didn't transfer.** It is written as a *procedure attached to one command* —
"run `git ls-remote`, compare the hash" — rather than as a standing epistemic rule. Running that
command reads as discharging the obligation. Nothing in the framing signals that the reasoning
behind it ("negatives stop investigation") applies to every claim in the session, so the discipline
terminates when the git check completes, roughly turn one.

**The sharpest instance is one the doctrine already names, on a different tool.** `/catch-up`
explicitly warns that a *filtered* remote query "that returns exactly what you asked for is
indistinguishable from a complete picture," and prescribes the unscoped form for any repo-wide
claim. The same trap fired on a summarizing fetch tool: a very large document was retrieved, the
summarizer answered the question posed, and the section of that document which directly
contradicted the answer was never surfaced. The orchestrator relayed the summary as a finding about
the document. Re-fetching the raw document and searching it directly took one command and produced
the contradicting text immediately. Identical failure shape — a filtered view mistaken for a
complete one — and the doctrine names it precisely, then confines it to one git subcommand.

A secondary instance is worth recording because it inverts the usual failure: a *file-format*
mismatch (an artifact built in one container format, the host's own files in another) was correctly
observed, then over-extended into "therefore the host cannot read it." The evidence supported
"different," not "rejected," and a source cited in the same breath actually said the legacy format
may still load. The user's pushback was the only thing that surfaced the gap. Notably the underlying
conclusion later turned out to be right for an unrelated reason — which is its own hazard, since a
lucky negative is indistinguishable from a sound one at the moment it is stated.

**Rule concluded:** the negative-claim discipline should be lifted out of `/catch-up`'s git section
and stated once as a session-wide rule, with the git procedure as one *instance* of it rather than
its definition. Two specific generalizations are worth stating explicitly, because both fired here:
(1) any tool that returns a *summary* or a *filtered subset* — a summarizing fetch, a scoped query,
a subagent report — cannot support a negative claim about the whole; go to the unfiltered source
before asserting absence. (2) An observed difference is not an observed incompatibility; state the
difference, and mark the causal step as inference unless it was tested.

Worth flagging for the wrap-up side too: the failure was not forgetting the doctrine. It was read,
understood, and correctly applied within its stated scope. Doctrine delivered once at session start
and framed as a per-command procedure decays into a checklist item; the parts meant to be standing
rules need to be labelled as such.

**Where it lives:** here, and in `/catch-up`'s command text — the negative-claim section should say
it governs every claim in the session, not only the remote-state ones, and should name the
summary-and-filter generalization directly.
