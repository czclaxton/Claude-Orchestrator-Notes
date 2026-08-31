### 2026-08-27 — `/wrap-up` froze a hard *negative* into a resume note, and it went stale in a day

**Status: verified** (the claim was re-checked directly against the referenced repository at the
start of the next session; the file was exactly where the note asserted it was not, and the commit
that put it there was identified).

**What happened.** A prior session's `/wrap-up` wrote a resume note that pointed at a reference
document living in a companion repository. Rather than recording where the document *was*, the note
recorded where it *was not* — an emphatic "it is not at `<path>` — that path does not resolve",
carrying two ⚠ markers, followed by a workaround command to retrieve the file's contents from an
unmerged branch instead.

One day later, at `/catch-up`, the file was sitting at that exact path on the companion repo's
default branch. It had been merged in the interim. The note's negative was simply false, and the
warning markers made it read as the most reliable line in the file.

**Why it is worse than an ordinary stale fact.** The failure is silent and self-perpetuating. The
recorded workaround still *works*, so a session that follows the note gets a correct document and no
signal that anything was wrong — it just takes a longer route and inherits a false belief. And
because `/wrap-up` refreshes a note largely by carrying forward what the previous note said, a
negative recorded once tends to survive every subsequent sweep. Nothing in the loop is positioned to
notice it, because noticing requires attempting the very thing the note warns you off.

**The asymmetry between the two commands is the actual gap.** `/catch-up`'s own instructions already
articulate this exact principle, and articulate it well: a negative claim needs real evidence,
because "a stale positive prompts someone to go look; a stale negative stops the investigation
entirely." But that discipline is scoped to what `/catch-up` *reports* about remote state at read
time. `/wrap-up` — the command that actually authors the durable artifact where negatives get
frozen for future sessions — carries no equivalent instruction. One command is careful about
asserting negatives; the other is free to record them permanently.

**Rule concluded:** `/wrap-up`'s note-writing step should prefer positive, dated location claims
("found at `<path>`, checked `<date>`") over negative ones. Where a negative genuinely earns a place
in the note — a path that was tried and failed, a branch that did not exist — it should be stamped
as a dated observation rather than written as a standing fact, so the next session reads it as a
snapshot to re-check rather than a rule to obey. Emphasis markers on a negative are actively
harmful: they raise the confidence of the one class of claim most likely to have decayed.

**Recurrence.** Third entry in this family. The 2026-08-20 entry covered `/catch-up` trusting local
tracking refs and confidently reporting a stale remote; the 2026-08-24 session covered stale
artifacts contradicting live state more generally. Same underlying shape each time — a claim
recorded once and later re-read as current — but this is the first instance where the stale claim
was *authored by the wrap-up command itself* rather than inherited from an agent or a tool's cached
output. That makes it the most fixable of the three: the write path is ours.

**Where it lives:** here only, for now. If promoted, it belongs in `/wrap-up`'s resume-note
instructions (step 3), as a constraint on how location and availability claims are phrased — a
sibling to the negative-claim discipline `/catch-up` already carries in its step 2.
