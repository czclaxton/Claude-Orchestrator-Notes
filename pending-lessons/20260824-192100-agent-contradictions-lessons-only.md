### 2026-08-21 — The lessons/wrap-up loop has a write path but no read path; and two-word shorthand failed inside the message arguing against shorthand

**Status: verified** (both observed directly in the session; captured retroactively on 2026-08-23
from the transcript — the session ended without a `/wrap-up`, which is itself the third finding).

**What happened:** a long design conversation about a separate tool surfaced a structural critique of
Claude Orchestrator's own feedback loop, plus a live instance of an explanation failure the plugin
has now logged three times in different forms.

**Finding 1 — the feedback loop is a diary, not a learning system.** `lessons.md` gets written at the
end of a session by `/wrap-up`. Nothing reads it *at the moment a decision is being made*. The only
read path is a human opening the file, or `/catch-up` surfacing the resume note — both at session
boundaries, neither at decision time. A record that is never consulted when it would change an
outcome makes the author feel organized and changes nothing.

Three properties separate a diary from a loop that actually learns, and the current design has none
of them reliably:

1. **Consulted at the moment of the decision**, not just at session start.
2. **Entries specific enough to bind behavior.** "Be more careful with migrations" changes nothing;
   "migrations always come to me before running, even in dev" can actually fire. The enforceable
   shape is always *when X, do Y* — observable trigger, specific action. The stranger test: could
   someone follow it without exercising judgment?
3. **Entries carry verdicts** — whether the intervention was the system's failure or the user's
   prerogative.

This is loop design, not model capability. Note the plugin already has the deterministic layer for
the strongest version of #1 — hooks can inject context or block a tool call — and currently spends
its entire hook surface on one `SessionStart` context injector. That connects directly to the
enforcement-vs-guidance question in `RESUME-PROMPT.md` and to notes PR #9.

Related structural point: the `where it lives` field in this file is written once and never
revisited, which has already caused a stale ledger to misreport the backlog (2026-08-21 entry). A
file whose own status fields go stale is more evidence for the same finding — writing is cheap here,
re-reading is not designed for at all.

**Finding 2 — shorthand failed in the exact message arguing for plain explanation.** The phrase
"case law" was used as two words carrying a paragraph of meaning, in a message making the case that
explanations should be layered and plain. The user reported it didn't land. This is the third
recurrence of one pattern in different costumes: the 2026-08-21 resume-note entry ("dead end" read as
a behavioral claim), the compressed-negatives rule in `RESUME-PROMPT.md`, and now this. The common
shape is **a compressed reference standing in for a concept the reader hasn't been given**, and it
recurs because the author always knows what the shorthand means.

**Finding 3 — the session produced substantial durable material and none of it was swept.** The
conversation defined a communication protocol, a four-module design, and an explicit statement of the
user's own explanation preferences. It ended without a `/wrap-up`, so none of it reached this file,
the backlog, or the resume note; the transcript was the only copy for two days until recovered by
grepping `~/.claude/projects/**/*.jsonl` on 2026-08-23. Testing mode was on the whole time. Testing
mode captures nothing on its own — it injects a reminder, and the sweep is still a manual command the
user has to remember at the end of a long session, which is precisely when it is least likely to
happen.

**Fix/rule concluded:** none promoted yet — logged. The candidate rules, in order of confidence:
(1) an entry in this file should be written in the *when X, do Y* form wherever the finding supports
it, so it can bind behavior rather than describe a mood; (2) entries need the verdict distinction
(system failure vs. user prerogative vs. elective involvement) or the log optimizes toward never
bothering the user; (3) the read path is the real gap and is a hooks question, not a prose question.

**Where it lives:** here only, deliberately — all three are candidates, none has been decided. Finding
1 overlaps notes PR #9's enforcement research and should be resolved with it, not separately. The
design conversation this came from is captured in full in `assistant-design-brief.md`; the user's
explanation preferences are in `communication-preferences.md`.

### 2026-08-23 — `/catch-up` has no read path into session transcripts, and a "not documented" negative was asserted without searching them

**Status: verified** (both observed directly this session; the transcript search was re-run and its
results are what corrected the error).

**What happened:** `/catch-up` ran and produced an accurate reconciliation of the resume note against
local git, remote git, and the notes repo. It was still substantially incomplete, and neither the
command nor the orchestrator noticed. A prior session had produced a large body of durable material —
a confirmed design framing, an explicit statement of the user's own working preferences, several
reversals the user argued for — and none of it had been swept. It existed only in the harness's own
session transcript. `/catch-up` reads the resume note, git state, and the companion notes repo. It
does not read transcripts, so that material was invisible to it. It surfaced only because the user
asked a question that happened to require it, two sessions later.

**Finding 1 — the resume note is treated as the only memory of a prior session, and it is a lossy
one written at the end of a session that may never have run `/wrap-up`.** When a sweep is skipped,
the loss is total and silent: `/catch-up` reports confidently on what it can see and has no way to
signal that a session's worth of material is missing. It correctly flagged the note as stale by two
sessions — it could see the *dates* had moved on via new branches — but "stale" was reported as a
freshness problem, not as "there are two unswept sessions whose content is unrecoverable from
anything I read." The harness stores transcripts locally in a known, greppable location; the command
never looks there. This is the same write-path/no-read-path shape logged in the 2026-08-21 entry,
now confirmed at a second layer: the session-boundary commands have the same gap the lessons log has.

**Finding 2 — a negative claim was asserted from absence of evidence, in a deliverable, without
searching the evidence that was available.** A written summary of the user's preferences included an
explicit "not documented — never stated, no evidence either way" list. One item on that list had in
fact been stated directly and at length by the user, in a transcript that a single grep would have
surfaced. The negative was written first and the search was run only after the user asked a follow-up
question.

This is the third recurrence of one pattern and the second in the *negative* direction specifically.
The plugin already documents the rule — `/catch-up`'s own body says a negative claim requires real
evidence, and that a stale negative stops the investigation entirely while a stale positive merely
prompts someone to go look. The rule is scoped in the command text to git remote state. The failure
mode generalizes and the text does not: "I found no documentation of X" is exactly the same claim
shape, with the same asymmetry, and nothing in the doctrine says so. A documented "what's missing"
section is more dangerous than an ordinary claim, because it reads as the product of a search and
tells the reader to stop looking.

**Fix/rule concluded:** none promoted — logged, and the two findings want different treatments.
Finding 1 is a real feature gap in `/catch-up` (and correspondingly in `/wrap-up`, which is what
should have captured the material in the first place); the candidate is a transcript-aware discovery
step, which is genuine work and should be scoped rather than bolted on. Finding 2 is a doctrine
wording change: generalize the existing negative-claim rule beyond remote git state to any asserted
absence, and treat an enumerated "not documented" list as a claim requiring the same evidence bar as
any other conclusion.

**Where it lives:** here only, both. Finding 1 belongs with the enforcement/read-path thread — it is
the same structural gap as the 2026-08-21 entry and notes PR #9, and should be resolved with them
rather than as an isolated command tweak. Finding 2 belongs in the orchestration skill's Verification
section, adjacent to the existing incidental-vs-assigned-claims item already outstanding there.

### 2026-08-24 — Fan-out over an already-covered corpus cross-checks itself for free; and a `⛔ do-not-edit` constraint has no paired rule for work that belongs to another lane

**Status: findings 1, 2 and 4 verified** (each correction was re-checked directly against the source
of truth before being accepted, and in every case the lane was right); **finding 3 verified**
(observed directly — the handover block was read and the relocation performed by the orchestrator).

Context: a large documentation-and-analysis build. Roughly fifteen lanes across four fan-out batches,
each lane assigned an adjacent slice of one shared corpus, then a second wave of lanes that consumed
the first wave's output as input.

**Finding 1 — lanes fanned out over a corpus that earlier passes already covered will correct those
earlier passes, repeatedly, and this is the highest-value output of the pattern.** Four separate
times a lane working on its own slice found a factual error in material produced by an *earlier*
lane, or by the orchestrator itself. One found three wrong values in a chapter written two batches
prior, including the headline figure. Another corrected two "unchanged" claims that were wrong in the
user's favour. A third found that a value the orchestrator had personally recommended acting on meant
the opposite of what the orchestrator said it meant. None of these were asked for; each lane found
them incidentally while reading adjacent material with no investment in the earlier conclusion.

This is the useful inverse of the 2026-08-20 *incidental claims* entry. There, incidental output was
less reliable than assigned output and the doctrine only covered the latter. Here the same asymmetry
runs the other way: a lane's incidental *corrections of prior work* were consistently right — four for
four on re-check — precisely because they were incidental. The lane had no stake in the earlier claim,
read the primary source directly, and reported the discrepancy as an aside. The doctrine has no
instruction to expect this, so an orchestrator that does not read reports carefully will discard it.

**Finding 2 — a long fan-out's premise changed mid-flight, and only luck separated the durable half
of each lane's output from the perishable half.** Four lanes were launched under one framing; the user
reversed that framing while they ran. Their output had two separable parts: a factual index (what
exists, where it lives, what the baseline is) and a verdict column (what to do about it under the old
framing). The index survived the reversal intact and remained the input the next phase needed; the
verdicts became actively misleading and had to be explicitly demoted in writing so a later session
would not apply them. That separation was not designed — the spec happened to ask for both in
distinct columns. Had the spec asked only for recommendations, all four lanes' work would have been
unusable.

**Finding 3 — `⛔ do-not-edit` constraints tell a lane what not to touch and nothing about what to do
with work that belongs to a file it cannot touch.** A lane was correctly forbidden from editing
sibling lanes' artifacts, and correctly discovered that two of its assigned changes belonged in files
two *other* lanes owned — putting them in its own artifact would have created a silent, build-breaking
duplicate-ownership bug. It handled this well: it wrote both changes out in full inside a clearly
marked handover block in its own file, excluded them from the active output, and stated plainly in its
report that someone had to move them. That worked. But it worked entirely on the lane's own judgment.
The constraint as written admits an equally compliant response — silently dropping both changes as
out-of-scope — and two user-approved changes were one unread report away from vanishing with no error
anywhere. A prohibition without a disposal route is an invitation to drop work quietly.

**Finding 4 — the mandatory end-of-deliverable advisor review earned its cost on the one artifact the
orchestrator wrote itself.** Across the whole build the advisor returned *ship, after one fix*, and
the fix was in the orchestrator's own chapter — numbers that were derived through a model and
conditional on an unproven assumption, but carried the same "verified" label as directly-read values.
Every lane-written artifact passed. That is the expected distribution: lane output had been
spot-checked by the orchestrator, and the orchestrator's own output had been checked by nobody. The
doctrine already mandates this review; what this session adds is *where its value concentrated*, which
argues for the review being pointed at orchestrator-authored artifacts first rather than treated as a
uniform pass over the diff.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. Finding 3 is the one worth fixing soon and is a small wording change: pair every `⛔ do-not-edit`
   constraint with a required disposal route — *if work you were assigned belongs in a file you may
   not touch, write it out in full, exclude it from your active output, and name it in your report as
   requiring relocation.* This session's lane invented exactly that protocol unprompted; it should not
   have had to.
2. Finding 2 argues for a spec-shape rule for any fan-out expected to outlive its own premise:
   **ask for the durable artifact and the perishable judgment as separately identifiable outputs.**
   The index survives a reversal; the recommendation does not.
3. Finding 1 argues for a verification-doctrine addition symmetric to the 2026-08-20 incidental-claims
   entry: when lanes fan out over a corpus earlier passes already covered, **treat their unsolicited
   corrections of prior work as high-value and verify them first** — they are the cheapest real
   cross-check the pattern produces, and they arrive buried in reports as asides.
4. Finding 4 is a small emphasis change to the final-review instruction rather than a new rule.

**Where it lives:** here only, all four. Findings 1 and 4 belong in the orchestration skill's
Verification section — finding 1 sitting directly alongside the 2026-08-20 incidental-claims item it
inverts, since the two together say something more useful than either alone. Finding 3 belongs in the
spec contract, under Constraints. Finding 2 belongs in the spec contract as well, but is the least
proven of the four and should wait for a second instance before it earns doctrine text.

### 2026-08-24 — `/catch-up` inherits written state claims as fact, and a verification that reads the artifact instead of the system cannot fail

**Context:** a long-running documentation project (no fan-out this session; findings are about
`/catch-up` and the verification doctrine, not about lanes). The project maintained a "live state"
document asserting that a piece of tooling was installed and running. It was not, and had never been.
The false claim had stood for roughly two months across many sessions.

**Finding 1 — `/catch-up` has no mechanism for distinguishing a *documented* state claim from a
*verified* one, and this is the third entry in this file describing the same failure shape.**
`[verified]` The session-start routine reads a resume note and a state document, then reports what
they say. This session opened by restating a state claim from a project document as established fact.
It was false. The evidence contradicting it was not hidden or subtle — it sat in the same state
document's own machine-state table, where a prior session had accurately recorded the observable
condition that *proved* the tool was inert, labelled it as a benign configuration variant, and written
"installed" in the adjacent row. Every subsequent session, including this one's opening summary,
inherited the conclusion and never re-derived it from the recorded evidence.

⭐ This is the same shape as **2026-08-20** (`/catch-up` trusted local tracking refs and confidently
reported stale remote state) and **2026-08-23** (`/catch-up` asserted a "not documented" negative
without a read path to check). Three instances now. The common root: **`/catch-up` treats every
readable source as equally authoritative, and has no notion that some claims describe a live system
whose current truth must be *queried*, not *read*.** The 2026-08-20 entry already fixed this for one
specific case by mandating a server query for remote git state. That fix was correct but was scoped as
a git rule rather than as the general principle it actually is.

**Finding 2 — "verify with evidence" does not distinguish evidence produced by the artifact from
evidence produced by the system, and a check of the first kind passes unconditionally.**
`[verified]` The project had a documented verification procedure that was run repeatedly across many
sessions and reported success every time. The procedure inspected the build artifact's own output
files and confirmed they contained the intended values. They always did — the build was genuinely
correct. But the artifact's contents are identical whether or not anything downstream ever consumes
it, so the check could not fail, and it was labelled as confirmation that the change was live. A
single different check — *does the consuming system actually have the artifact* — was one command,
available from the start, and would have caught the failure immediately.

Re-verified this session by running both: the artifact check passed (values correct), the system check
failed (artifact absent from the consumer's load path). Also worth recording as a method note: when a
verification tool errored on the artifact, running it against a **known-good control and a known-bad
control** was what separated "tool doesn't support this input" from "artifact is broken" — it was the
former, and without controls that error would have been misreported as a defect.

**Finding 3 — when a pipeline's final delivery step lies outside the tool's own scope, "done"
silently truncates to the tool's definition, and the orchestrator reports success at a boundary the
user never agreed to.** `[verified]` The third-party build tool defines "build" as *produce an
artifact*. The user's definition — the operative one — was *the change is live in the running
system*. The project's own written procedure encoded the narrow definition, defining the deployment
step as a file copy that in fact deployed nothing. Across the session the orchestrator repeatedly
reported "the build succeeded," which was true under the tool's definition and false under the user's.
When the gap surfaced, the orchestrator's first framing was *"the builds were fine, the install was
the problem"* — a distinction the user had never accepted, which functioned as an excuse rather than
an explanation. The user pushed back on the terminology directly and was right to.

**Finding 4 — a user asking materially the same question a third time is a signal to go look, not to
restate.** `[asserted]` The user asked the same scoping question three times in succession. The first
two answers restated a design principle already established earlier in the session; both were
defensible and neither involved inspecting anything. The third time the orchestrator actually examined
the artifacts and immediately found several real defects — stale trackers claiming work was unapplied
when it was live, an internal contradiction between a file's header comment and its own contents, and
an off-by-one count — none of which the principle-based answers would ever have surfaced. Repetition
was the user correctly detecting that the answers were being generated from doctrine rather than from
evidence.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. **Finding 1 is ready for doctrine and should generalize the 2026-08-20 git-specific rule.** Proposed
   wording for `/catch-up`: *a claim about the current state of a live system is not established by
   reading it in a document, however authoritative the document. Documents record what was believed
   true when written. Where a state claim is load-bearing for the session's next action, either query
   the system directly or report it explicitly as "documented, not verified."* The existing git rule
   becomes one instance of this rather than a standalone item. Three occurrences now; this is the
   highest-confidence item in this entry.
2. **Finding 2 belongs in the Verification section** and is close to a one-line addition: *prefer
   evidence from the consuming system over evidence from the produced artifact. An artifact check
   proves what a thing would do; only a system check proves what is happening. If a check cannot
   distinguish success from failure, it is not a check.* The control-testing note is a worthwhile
   sub-point: **before reporting a tool error as a defect, run a known-good and known-bad control.**
3. **Finding 3 argues for an explicit definition-of-done rule** wherever a pipeline's last step is
   outside the tool's scope: *state the completion boundary in the user's terms, not the tool's, and
   when reporting partial completion name what remains rather than reporting success against the
   narrower definition.* Lower confidence only because one instance.
4. **Finding 4 is a behavioral note, not doctrine text** — but it pairs naturally with finding 2: both
   describe answering from a model of the thing instead of from the thing. Worth a second instance
   before it earns wording.

**Where it lives:** here only, all four. Finding 1 belongs in the `/catch-up` command text, replacing
the git-scoped rule with the general one and keeping git as its worked example. Finding 2 belongs in
the orchestration skill's Verification section, adjacent to the 2026-08-20 incidental-claims entry.
Findings 3 and 4 stay here pending a second instance.

### 2026-08-24 — Parallel read-only agents: two contradicted each other on a checkable fact, and one inherited a stale premise from an orchestrator-written doc

**Context:** three read-only exploration agents were dispatched in parallel to survey different
areas of an unfamiliar codebase ahead of a design task. The orchestrator had already established
most of the domain facts itself and deliberately narrowed the fan-out to only genuinely unknown
areas, telling the user explicitly what it was *not* delegating and why. That scoping worked —
none of the three re-derived held context, and their areas did not overlap.

**Finding 1 — two agents surveying adjacent areas reported contradictory facts about the same
interface, and only a direct check resolved it.** `[verified]` One agent reported that a server
interface exposed a full set of verbs including a delete operation. A second agent, covering a
neighbouring area, reported the same interface exposed no delete operation. Both stated it flatly,
neither hedged. The orchestrator re-read the source directly: the second agent was right. The first
had asserted a complete verb set without enumerating it.

The same agent's report also placed a real defect in the wrong file — the finding itself was
accurate and the line number happened to match, but the file was a different one in the same call
chain. Two accuracy failures in one report, both caught only because the claims were load-bearing
enough to re-check before being written into a planning document.

Worth noting what made this recoverable: the contradiction was *visible* because two agents happened
to cover overlapping ground. Where a single agent owns an area alone, a confidently-stated wrong
fact has nothing to collide with.

**Finding 2 — an agent discovered a document the orchestrator had written earlier in the same
session and treated it as authoritative spec, including a premise the user had already overridden in
conversation.** `[verified]` Mid-session the orchestrator wrote a reference document capturing a
transcribed external source. It recorded, accurately, that the source specified a particular field as
unstructured free text. Some turns later the user made an explicit decision to override that and use
a properly typed input instead. That decision lived only in the conversation.

A subsequently-dispatched agent found the document, correctly identified it as the spec for the work,
and mapped its own findings against it — reporting that field as "already covered, no work needed."
Under the document it was right. Under the actual decision it was wrong, and it had no way to know.

The failure is not the agent's. It is that **orchestrator-written artifacts become authority for
agents while remaining stale relative to decisions made in conversation after they were written.**
The orchestrator is the only party that can see both. This is a general hazard of the
write-a-doc-then-delegate pattern, and it gets worse the longer a session runs and the more
decisions accumulate verbally.

**Finding 3 — the orchestrator announced a verification in prose and ended the turn without
performing it.** `[asserted]` Having identified the contradiction in Finding 1, the orchestrator
wrote that it was checking, and then produced no tool call at all. The turn simply ended. The user
had to prompt for recovery, and the recovery prompt reasonably assumed some error had occurred —
there was no error, only an announced action that never happened. Recorded as `asserted` because the
cause was not diagnosed, only the effect. Related in shape to the standing rule about claiming a tool
action completed without confirming it landed, but distinct: here the claim was prospective rather
than retrospective, which is if anything easier to make and equally misleading.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. **Finding 2 is the most generalizable and is close to ready.** Proposed wording for the
   orchestration skill's delegation section: *when handing an agent a document you wrote earlier in
   the session, state in the spec which parts have been superseded by later decisions, or re-read the
   document yourself before dispatching. An agent treats your artifacts as authority and cannot see
   the conversation that has since moved past them.* First instance, but the mechanism is clear and
   the blast radius is large — a stale premise propagates silently into every downstream report.
2. **Finding 1 reinforces the existing verify-agent-claims discipline** rather than adding to it, with
   one new sub-point worth wording: *a claim about the completeness of a set — all the verbs an
   interface exposes, all the places a symbol appears — should be reported as an enumeration, not a
   summary. "Full CRUD" is a conclusion; the list of verbs is the evidence.* Also worth noting for
   fan-out design: deliberate overlap between agent areas is what surfaced the contradiction, so
   perfectly disjoint scoping trades away a real error-detection mechanism.
3. **Finding 3 is a behavioral note, not doctrine text.** One instance, cause undiagnosed. Worth a
   second occurrence before it earns wording. If it recurs, the rule is roughly *do not narrate an
   action you have not taken; either take it in the same turn or say you will take it next.*

**Where it lives:** here only, all three. Finding 2 belongs in the orchestration skill's delegation
guidance if it recurs or if the reasoning is accepted on its merits. Finding 1's enumeration
sub-point belongs alongside the existing verification material. Finding 3 stays here pending a second
instance.
