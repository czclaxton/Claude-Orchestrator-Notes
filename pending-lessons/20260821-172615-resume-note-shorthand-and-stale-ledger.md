### 2026-08-21 (evening session) — The resume note's own shorthand became a false claim about shipped behavior, and the ledger's "where it lives" fields are stale enough to misreport the backlog

**Status: finding 1 verified** (the contradicting line was read directly from the shipped command
file, and the advisor independently cited it by line number); **finding 2 verified** (counted by
grep against the ledger, then each entry checked against the actual plugin files); **finding 3
verified** (the advisor's citations were re-checked against the source files before being accepted).

**Finding 1 — a resume note's compressed characterization was repeated as a factual claim about
shipped behavior, while the file it described was already in context.** The resume note described a
queued fix as replacing a "dead end" in one of the plugin's own commands. The orchestrator carried
that word forward and told the user the command *breaks* for a new user — that it "does nothing" and
would be the worst possible first impression, ranking it the single highest blocker to sharing the
plugin. The command file actually ends with an explicit fallback instructing exactly what to do when
the file it looks for is absent. That text had been injected verbatim into the session at startup, as
the command body itself. It was read and then contradicted within the same session.

"Dead end" was the author's own shorthand for *delivers no orientation* — accurate as shorthand,
and the queued fix is still a real improvement. But to a fresh session the phrase reads as a claim
about behavior, and nothing in the note marks it as compression.

This is the third distinct recurrence of the resume-note-trust pattern, and the first where the
contradicting evidence was already in context rather than one file read away. The existing rule
("treat live state as authoritative over the note") was followed for *git* state, which the command
explicitly calls out, and not applied to the note's characterizations of the plugin's own files —
which it doesn't.

**Rule concluded:** the note-versus-reality rule currently reads as being about git. It should
generalize: any claim in a resume note about how a *file* behaves is a pointer to that file, not a
finding. Verify against the file before repeating it, especially when the claim is load-bearing for
a recommendation. The failure skews the same direction as the volunteered-context finding logged the
day before — toward overstating severity, since a compressed negative ("dead end") decompresses worse
than it was meant.

**Finding 2 — the ledger misreports its own backlog, because "where it lives: here only" is written
once and never revisited.** The resume note claimed roughly seven findings remained diagnosed but
unbuilt, and the orchestrator relayed that number to the user unchecked. Counting directly: three
entries carry "here only," and one of those describes a fix that was subsequently built, merged, and
shipped — the entry was never updated. Actual outstanding count is two, both confirmed unbuilt by
grepping the plugin for the language each entry says should be added.

The field is written at the moment of logging, when nothing has been promoted yet, so "here only" is
correct on the day and decays silently from then on. Every backlog claim derived from the ledger
inherits the decay, and the resume note is derived from the ledger.

**Rule concluded:** promoting a finding should include updating that finding's "where it lives" line
in the same change — the promotion isn't complete until the ledger stops saying it didn't happen. A
status block at the top of the file (built / open / recurred-and-how-many-times) would make both the
staleness and the recurrence visible; the recurrence half would have surfaced finding 1's pattern
after its second logging instead of its third.

**Finding 3 — the advisor pass earned its keep, and the mechanism was file access, not model
strength.** Consulted at the user's explicit request to confirm or refute a readiness recommendation,
framed as "refute this claim." It refuted the recommendation's primary blocker by citing the exact
lines that contradicted it, found a second instance of a leak the orchestrator had only partially
identified — in the file that loads into *every* session rather than the one the orchestrator had
noticed — and correctly deflated a third concern by distinguishing tool *availability* granted in
agent frontmatter from tool *execution*, which still passes through the user's permission rules.

What made it work was not the model tier. It was that the advisor had read access and used it
against the actual files rather than reasoning from the orchestrator's summary, and that the prompt
told it to distinguish what it read from what it inferred — which it did, unprompted flagging one of
its own claims as reasoned-from-documentation rather than tested. The "refute this claim" framing
and an explicit "no findings is a valid result" both appear to have held: it confirmed two parts of
the recommendation while overturning the centerpiece, rather than splitting the difference.

**One limit worth recording:** the advisor offered a clean binary for the finding-1 discrepancy —
either a behavioral failure it couldn't see, or a stale file — and missed the actual explanation,
that the source document was compressed rather than wrong. A fresh-context reviewer has no way to
tell an author's shorthand from an author's error. Where a review turns on interpreting a working
note, that ambiguity should be named in the prompt rather than left for the reviewer to resolve.

**Where it lives:** here only. Finding 1 belongs in `commands/catch-up.md`, generalizing the existing
local-vs-remote authority language beyond git to any claim the note makes about a file's behavior.
Finding 2 belongs in this file's own format section, plus a status block at the top. Finding 3 is a
success record, not a rule change — but the advisor-prompt practice it validates (hand over read
access, demand read-versus-inferred separation, state "no findings" is acceptable) is worth writing
into the orchestration skill's advisor section if it holds up on a second use.
