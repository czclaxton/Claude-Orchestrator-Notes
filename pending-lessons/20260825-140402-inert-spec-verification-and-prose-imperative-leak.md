### 2026-08-25 — A spec's verification step can be structurally inert, and four parallel lanes caught it when the architect didn't

**Status: finding 1 verified** (re-ran the ignore check and the diff command myself and confirmed
both were inert on the targets); **finding 2 verified** (read the delivered prose and corrected two
instances); **finding 3 verified** (the stale claim and its later correction are both a matter of
record from this session); **finding 4 asserted** (an impression about a single occurrence, not
re-measured).

**Finding 1 — the spec contract validates that a Verification step exists, never that it can
produce a signal on this particular target.** The architect wrote four specs in one pass and
launched them in parallel. Each told its lane to prove the work with a version-control diff
command. Every target file sat in a directory excluded from version control, so that command
returns empty output whether or not the file changed — the empty result is indistinguishable from
"no edit was made."

All four lanes independently detected this, stated plainly that the prescribed check could not
produce evidence for this target, declined to report it as passed, and substituted content-based
verification (heading inventories, line-count deltas, direct read-back). That is exactly the
behavior the doctrine wants and it is worth recording as a success, not only as an architect
failure — a lane that had simply run the command and reported "clean" would have handed back a
false pass that looked identical to a real one.

Two things generalize. First, **an empty result from a verification command is the same trap the
session-start guidance already names for a fetch that prints nothing** — silence read as
confirmation. The Verification section tells the orchestrator to re-run the command and judge the
output; it says nothing about first establishing that the command is capable of returning a
negative. Second, **parallel delegation multiplies a single architect error rather than diversifying
it.** The four specs were written from one mental template in one sitting, so one wrong assumption
about the target's nature was replicated four times simultaneously. Serial delegation would have
surfaced it after the first report.

**Rule concluded:** when writing the Verification part of a spec, the test is not "is this the right
command" but "can this command distinguish success from failure *on this target*." Where the
deliverable sits outside the reach of the default tooling — excluded paths, generated artifacts,
anything untracked — the spec must name a verification method that actually observes the artifact.
And when fanning out parallel specs built from one template, a shared assumption is a shared point
of failure worth checking once before launching, not four times after.

**Finding 2 — for prose deliverables, spec imperatives get transcribed into the document body.**
The specs supplied content using instruction phrasing addressed to the writer — constructions of
the form "State that plainly: <content>" and "Record why this matters: <content>". One lane copied
those imperatives verbatim into the finished document, leaving instructions to the author sitting
in reader-facing text. The substance was correct and correctly placed; only the framing leaked.

This failure mode is specific to prose. When the deliverable is code, spec English cannot survive
into the artifact — it would not compile, and the gap between instruction and output is enforced by
the language itself. Documentation has no such filter: the spec and the deliverable are the same
medium, so any sentence in the spec is a candidate for being pasted into the result.

**Rule concluded:** when the deliverable is prose, supply content *as content* — quotable material,
not directions to the writer — or state explicitly that instruction phrasing must not appear in the
output. The spec contract's Interfaces part has no analogue for document tasks, and this is the gap
it leaves.

**Finding 3 — remote state verified at session start has a shelf life, and the guidance treats it
as a one-time ritual.** The session-start command is emphatic that claims about remote state need
server evidence, and that negatives are the dangerous direction because a stale negative stops an
investigation. That discipline was followed correctly at session start. Roughly ninety minutes
later the orchestrator relayed a branch-status negative — that certain work was still unmerged —
sourced from that opening check. It had since become false; the work had been merged on the server
while the session was in progress. It was caught only because an unrelated later command happened
to re-read the remote.

The framing is the problem: the guidance is written as a start-of-session checklist, so the
verification reads as something that is *done* rather than something that *decays*. The longer a
session runs, the more likely a relayed negative about remote state is stale — and long sessions
are precisely when collaborators are most likely to have pushed.

**Rule concluded:** treat a verified remote fact as having an expiry, not a checkmark. Before
relaying a negative about remote state that was established earlier in the same session — "still
unmerged", "nothing new has landed", "that hasn't moved" — re-query. The cost is one command; the
failure mode is telling the user nothing has happened when something has.

**Finding 4 — the architect volunteered an explanation for a third party's behavior and stated it
as likely fact.** Asked to evaluate review feedback from another person, the orchestrator offered a
motive for why that person had raised a point — that they had probably not noticed something —
and presented it with more confidence than any evidence supported. The user pushed back on it
directly, and a timestamp check disproved it outright.

This is the same family as the already-logged pattern of the architect asserting facts it has not
checked, but with a distinguishing feature worth separating: the subject was a *person's reasoning*,
which is not verifiable from the repository at all. There was no check that would have confirmed
it, which is precisely why it should have been offered as an open question or not at all.

**Rule concluded:** claims about why a human did something are unverifiable by construction and
should be marked as speculation or omitted. Where the evidence permits a factual check that bears on
it — when something was actually visible, what the record shows — run that instead and report the
fact, leaving the motive alone.

**Where it lives:** here only. Finding 1 belongs in the spec contract's Verification part, as a
requirement that the named check be capable of failing on the given target, plus a note in the
Parallelism section that specs fanned out from one template share their assumptions. Finding 2
belongs in the spec contract as guidance for non-code deliverables. Finding 3 belongs in the
session-start command's remote-state section, as an expiry on verified facts rather than a one-time
gate. Finding 4 extends the existing architect-asserted-facts theme.

### 2026-08-25 (same session, user-initiated) — `/wrap-up` composes from memory instead of auditing state, so a follow-up "check again" reliably finds things it missed

**Status: finding 1 verified** (all three misses re-checked directly against live state after the
fact); **finding 2 verified** (branch and log inventory of the notes repo re-read); **finding 3
verified** (the command body mandates the sign-off unconditionally); **the cross-session pattern is
user-reported, not verified here** — the user states this happens *every* time they ask for a second
pass, across many sessions. Only this session's instance was re-checked. That distinction matters:
the mechanism below is verified, its claimed frequency is not.

**Origin, and why it is worth taking seriously.** The user raised this themselves, unprompted, as a
standing observation about the tool rather than a complaint about one session: they feel obliged to
ask for a second pass after `/wrap-up`, because when they do, it always turns something up. This
session was the example that prompted it, not the whole basis. They explicitly said they did not
know how to address it and wanted it investigated rather than just logged.

**Finding 1 — every step of `/wrap-up` is a compose step; not one is a verify step.** The command
sweeps findings into a log, writes a resume note, writes a sentinel, and announces completion. All
four produce an artifact. None checks an artifact against the world. The resume note in particular
is written from what the session *remembers*, which means it faithfully reproduces the session's
beliefs — including the wrong ones. An audit re-derives from live state and can therefore contradict
those beliefs; a summary cannot.

The three things the follow-up pass caught were all belief-vs-state mismatches, and all three were
checkable at wrap-up time with no new information:

- **A claim about a third party that had never happened.** Mid-session the orchestrator drafted a
  message to another person, asked the user whether to send it, and got no answer because the
  conversation moved on. It then wrote into a durable project document that the person *had* been
  told. A stale artifact stating that someone was informed when they were not is the most damaging
  of the three, because a later session reads it as fact and will not re-ask.
- **An unanswered question absent from the resume note.** The same dangling decision never made it
  into the "still open" section, so the note's own open-items list was incomplete in exactly the
  place the session had drifted.
- **A background process reported as stopped that was still running.** A dev server had been
  launched via a backgrounded shell and stopped via the task-stop tool; the tool reported success,
  but it killed the wrapper shell and the server it had spawned outlived it as an orphan. The
  orchestrator had already told the user both servers were down, on the strength of the tool's
  success message rather than a port check.

The categories generalize. Highest-yield first: **claims about parties and systems outside the
repository** — "X was told", "Y was sent", "Z is stopped" — because nothing in the repo contradicts
them, so they are asserted from memory and never bounce off reality. Then **questions the assistant
asked that the user never answered**, which are a distinct and mechanically recoverable category
that "what's still open" does not reliably surface. Then **side effects**: processes, servers,
worktrees, files created outside the deliverable.

**Finding 2 — the notes repo shows this has been approached seven times and never named.** Seven
separate logged entries touch `/wrap-up` or the resume note: a hardcoded note path, a note clobber,
note shorthand misread as a behavioral claim, a stale ledger, an over-broad dirty-tree guard, and
more. Every one is about the command's *mechanics* — where it writes, what it overwrites, what it
guards. Not one is about whether the pass is *complete*. The defect has been circled repeatedly
from adjacent angles while the load-bearing question — does this command actually check anything —
went unasked. Worth noting as a pattern in its own right: a cluster of near-miss entries around one
command is itself evidence that something structural is being missed.

**Finding 3 — the command's final mandated act is an unconditional confidence assertion.** The body
requires ending with a fixed sentence declaring findings swept and instructing the user to clear. It
is emitted verbatim whether or not anything was verified, because nothing in the command produces a
pass/fail signal it could be gated on. So the most prominent, most memorable output of the whole
command is a done-signal decoupled from actual doneness. The user's habit of asking "are you sure"
is them correcting for a claim the command instructed the assistant to make regardless of its truth.

**A relevant negative result: same-context self-review worked.** The follow-up pass was run by the
same context that produced the errors — no fresh agent, no cleared state, no independent model. It
still found all three. That locates the fault in **task framing, not context contamination**: the
information needed was present the whole time, and the difference was being asked to look rather
than to recall. This matters because it makes the fix cheap. It does not need a second reviewer or a
clean context; it needs the command to contain the question.

**Steelman, stated honestly:** any second look at anything finds something, and a third pass would
presumably find less — so some of this is ordinary diminishing returns rather than a specific defect.
Two things argue against dismissing it on those grounds. The severity was not marginal: one finding
was a false statement written into a durable document that outlives the session. And the misses
clustered in predictable, enumerable categories rather than being scattered — which is the signature
of a missing checklist, not of finite attention.

**Rule concluded:** `/wrap-up` needs a verification step between composing the resume note and
declaring completion, with an explicit bounded checklist rather than a vague instruction to
double-check. At minimum: re-verify any claim the session made about a person or system outside the
repository; enumerate questions asked of the user that were never answered and record them as open;
query the system for side effects still running rather than trusting the success message of whatever
was used to stop them; and confirm repository state directly. The completion sentence should either
be gated on that step or state what was actually checked. A command whose last instruction is to
assert success should first contain something capable of failing.

**Where it lives:** here only. Belongs in `commands/wrap-up.md` as a new step before the sign-off,
and the sign-off itself should be revised. Related to but distinct from the seven existing
resume-note and wrap-up entries, all of which address mechanics rather than completeness — worth
cross-referencing them when this is fixed, since a fix aimed only at mechanics will not touch this.
