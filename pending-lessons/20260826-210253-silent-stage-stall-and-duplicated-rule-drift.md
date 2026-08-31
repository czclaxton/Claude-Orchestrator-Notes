### 2026-08-25 — A research lane's chain of custody broke silently, and its relayed citations were accurate in substance but wrong in one figure

**Status: verified** (four primary sources fetched directly this session and the quotes checked
against the relayed text).

**What happened:** the user asked for research before committing to a design change, and three
lanes were dispatched in parallel. One of them returned a report that opened by referencing "my
original report" and "my earlier finding" — neither of which had ever reached the orchestrator. The
lane had delegated further, and only an addendum came back; the base report was lost somewhere in
the chain. **The lane flagged this itself**, stated plainly that it had not independently verified
the citations it was relaying, and recommended re-verification before anyone acted on them.

Spot-checking four load-bearing citations against the primary sources found the substance accurate
— the papers were real, the quoted passages present, the authors as described. But one figure was
wrong: a degradation endpoint relayed as 21% is reported as 15% in the source. Small in isolation,
and it sat in the exact sentence being used to justify a design constraint.

**Two distinct failures, worth separating.** The first is structural: a lane that delegates has a
chain of custody, and a broken link in it is *invisible from the outside* unless the lane
volunteers it. This one did volunteer it, which is the only reason it was caught. A lane that had
simply presented the addendum as its own findings would have been indistinguishable from a
complete report — the same trap as an empty `fetch` or a scoped `ls-remote`. The second is the
familiar one: relayed figures drift, and they drift in the direction of being more striking than
the source.

**Rule concluded:** when a lane's output will enter a document the user makes a decision from,
verify the load-bearing figures against primary sources before relaying them — not the whole
report, just the numbers that are actually carrying weight. And treat a lane's silence about its
own method as uninformative rather than reassuring; a lane that delegated and a lane that did the
work itself produce identically-shaped reports.

**Where it lives:** here only. This is a third occurrence of the citation-confidence theme
(2026-08-21 and 2026-08-22 entries) and the pattern is now consistent enough to justify a direct
change to the orchestration skill's Verification section rather than another log entry.

### 2026-08-25 — The orchestrator disqualified evidence that contradicted it, then nearly accepted evidence with the identical defect because it agreed

**Status: verified** (caught in-session, before the reasoning reached the user's decision).

**What happened:** the orchestrator ruled out a large body of evidence on a category test — it
measured a different situation than the one under discussion, so its conclusions did not transfer.
That reasoning was sound and it was the right call.

A later lane returned evidence with **the same defect**, from the same category, measuring the same
irrelevant situation — but pointing toward the recommendation the orchestrator had already made.
The reflex was to relay it as support. It took a deliberate second pass to notice that the filter
used to dismiss the first body of evidence disqualified this one just as completely.

**Why this is worse than an ordinary reasoning slip.** A filter invoked to dismiss inconvenient
evidence and then not applied to convenient evidence is not neutral analysis — it is a mechanism
for laundering a prior conclusion through the appearance of research. The user had specifically
commissioned the research to check whether an existing decision was well-founded. Selectively
applied skepticism would have returned exactly the answer the orchestrator already held, dressed in
citations, which is worse than having done no research at all.

**Rule concluded:** a disqualifying test, once used, is standing. The moment evidence is dismissed
on a structural ground, that ground becomes a test every subsequent piece of evidence must pass —
confirming evidence first, because that is where the check will not happen on its own. When
relaying research that survives such a filter, state the filter and state that it was applied in
both directions; if it was not, the conclusion is not yet supported.

**Where it lives:** here only. Belongs in the orchestration skill's Verification section, adjacent
to the existing guidance on negatives requiring evidence — this is the same asymmetry problem
pointed at the orchestrator's own reasoning rather than at a claim about the world.

### 2026-08-25 — `/wrap-up`'s dirty-tree guard fired on an artifact the orchestrator had just written, and the narrowing fix is still sitting unmerged

**Status: verified** (observed while running the command, this session).

**What happened:** the user asked for research findings to be documented durably, so the
orchestrator wrote a reference document into the notes repo working tree and deliberately left it
uncommitted, flagging to the user that they should decide how it lands. At `/wrap-up`, the
dirty-tree guard aborted the PR step — as written, correctly, since the tree was not clean.

But the only dirt was **a single untracked file that could not have been swept into the commit**,
because the commit step stages one named file. The guard's stated rationale is avoiding sweeping
someone else's unfinished edits into a commit, and that risk was not present. The fix narrowing
this guard to a real conflict on the target file was written several sessions ago, is correct, and
has been sitting in an unmerged pull request ever since — so the running command still has the
broad version. **The command hit precisely the case its own pending fix exists to handle.**

**The second-order finding is the more useful one:** the orchestrator created the condition that
tripped the guard. A command that writes durable artifacts into the notes repo mid-session is, by
doing so, disabling the sweep step that runs at the end of that same session. Nothing in the
current design connects those two facts, so the collision is only discovered at the end, when the
options are worse.

**Rule concluded:** two separate things. The narrowing fix should ship before any further work
touches this command — an unmerged fix changes nothing, and this is now the second session where
that has had a concrete cost. And when the orchestrator writes a durable artifact into the notes
repo during a session, it should either commit it at the time or tell the user explicitly that the
end-of-session sweep will be blocked by it — the warning is nearly free and the discovery cost at
wrap-up time is not.

**Where it lives:** here only. The first half is already implemented in the open PR and needs
merging, not authoring. The second half is a new clause for the command's artifact-writing
behavior, and it generalizes: any orchestrator action that dirties the notes repo has a delayed
cost that surfaces only at wrap-up.

### 2026-08-25 (found on a user-initiated re-check) — The resume note failed twice in two sessions, in two different directions, and both failures were invisible from inside the session that caused them

**Status: verified** (failure 1 confirmed by file mtime against pull-request creation timestamps;
failure 2 confirmed by grepping the freshly-written note for the dropped material).

**What happened — two distinct failure modes of the same artifact, discovered one after the other.**

**Failure 1, discovered at `/catch-up`: the note was silently a full session stale.** Its own header
claimed a verification date, and the state it described was internally consistent. But its mtime
predated two pull requests that a later session had opened — a session that demonstrably ran the
wrap-up command, since opening those PRs is something only that command does. So the command ran,
performed its logging step, and did not rewrite the note. Nothing in the note indicated this. A
session trusting it would have believed a stale queue depth and a stale "next action," both stated
with full confidence.

**Failure 2, discovered only because the user asked for a re-check: the rewrite silently dropped a
live thread.** At the end of a long session dominated by one topic, the note was rewritten from
what was salient in that session. The previous note had carried **two** parallel threads; the
rewrite carried one. The dropped thread was not minor — it contained a decision protocol the
previous note explicitly labelled that session's most load-bearing outcome, five open questions, a
list of settled decisions marked don't-re-litigate, and the exact commands needed to reach source
files that exist nowhere on the default branch. All of it was recoverable only because the old note
had been read into context earlier in the same session; the file itself had already been
overwritten, and it is not tracked in git.

**The common cause, and it is the interesting part.** The command writes the note by composing from
session memory rather than by auditing the artifact it is replacing. That works when the session
covered everything the note contains. It fails silently whenever the session was *focused* — the
narrower and more productive the session, the more of the prior note gets dropped, because
salience is exactly the wrong selection function for a durable handoff record. Failure 1 is the
same defect one step earlier: not composing at all when the session's attention was elsewhere.

**Neither failure is detectable from inside the session that commits it.** The note that results
reads as complete and internally consistent in both cases. Only a diff against the previous version,
or an outside prompt to re-check, surfaces it. This is a third occurrence of the resume-note
reliability theme (2026-08-21 clobber-and-fabricated-state, 2026-08-21 shorthand-and-stale-ledger).

**Rule concluded:** overwriting a handoff note is a merge, not a write. Read the existing note
first, enumerate the threads it carries, and for each one either carry it forward or state in the
new note that it was deliberately closed — dropping it silently is not an available option. A
cheap mechanical check gets most of the value: after writing, grep the new note for the distinctive
identifiers in the old one and confirm every absence is intentional. And since a stale note is
indistinguishable from a current one, the note should carry a marker the reader can falsify against
live state rather than a self-reported verification date.

**Where it lives:** here only. This is a direct change to the resume-note step of the wrap-up
command — read-then-merge instead of compose-then-overwrite — and it pairs with the already-logged
finding that the same command composes from memory instead of auditing state. Both are the same
underlying defect in different steps, which strengthens the case for fixing the pattern rather than
the instances.

### 2026-08-26 — The "don't merge, batch at the version bump" rule quietly broke a citation, and a second session's wrap-up landed in the same shared file with no owner

**Status: verified** (finding 1 confirmed by `git log --all -- <file>` plus `git branch --contains`,
then reading the file out of the branch; finding 2 observed directly — `git status` on the notes
repo was clean at `/catch-up` and dirty at `/wrap-up` in the same session, with entries the
orchestrator had not written).

**Finding 1 — an instruction pointed at a file that does not exist where it said.** A project's
resume note carried a standing instruction to read a specific document in the notes repo, by
absolute path, at the start of every session, and emphasised that the document is actively
maintained so a summary of it should not be trusted. That path does not resolve. The document has
never been on the default branch: it lives only on one of the accumulated unmerged `lessons/*`
branches, alongside the entry it shipped with.

A session that follows the instruction literally gets a missing file. The failure is quiet in the
worst way — the instruction reads as satisfiable, and the natural recovery is to proceed without the
guidance and never mention it, because nothing distinguishes "this file is missing" from "this
instruction was optional." (It was recoverable here only because the search for it happened to
surface the branch.)

**This is a direct consequence of the wrap-up command's own design.** The command is explicit that
its pull requests are not to be merged — they are reviewed in a batch at the next version bump. That
is a reasonable rule for *entries*, which are append-only logs nobody cites. It is not a safe rule
for *documents*, which are written to be pointed at. The same commit flow produces both, and nothing
in the command distinguishes them. The repo's own most recent commit on the default branch is a note
flagging that two cited files live only on an unmerged branch — so the symptom was already known and
recorded, while the mechanism that produces it kept running.

**Finding 2 — concurrent sessions collide in the shared log, and the loser is silently demoted.**
The notes repo was clean when checked at the start of this session and dirty at the end, carrying
two entries dated the same day that this session did not write. The dirty-tree guard fired and
correctly declined to branch. But the outcome is that this session's entry is left uncommitted, in a
file that already contains another session's uncommitted entries, with no branch, no pull request,
and no record of which session authored which block. The user is told to "sort it out themselves,"
which is exactly the situation the guard exists to avoid creating.

The guard is doing the right thing locally. The gap is that the command has no notion of *another
wrap-up* as a distinct cause of dirtiness — it treats a peer session's in-flight log entry the same
as a stray unrelated edit, when the correct handling is arguably the opposite (both entries belong
in the same commit, or in two branches cut from the same clean base).

**Rule concluded:** two things. First, separate the two kinds of artifact the sweep produces — an
appended log entry can safely sit unmerged, a document that anything else cites cannot, and a
document that is going to be referenced by path should land on the default branch at the time it is
written, not at the next version bump. Second, before a session emits an instruction naming an
absolute path into the notes repo, the path should be resolved against the default branch rather than
against whatever the working tree happens to contain — the working tree is exactly where an unmerged
file is still visible, which is what makes this failure so easy to author and so hard to notice.

**Where it lives:** here only, and both halves are command-level changes rather than doctrine. The
first is the more urgent: it is a correctness bug in how the plugin's own outputs are published, and
it has already produced one broken citation and one commit acknowledging it without fixing it.

---

## 2026-08-26 — A resume note pointed at a behavioural doc, the session read part of it, and behaved as if it had read all of it

**Status: finding 1 verified** (the truncated read and the unread section are directly observable in this
session's own tool calls — the document was fetched with a head-limit, and the remainder was read only after
the user objected). **Finding 2's observation verified, its causal claim asserted** — that output shape
degraded across the session was observed directly, and the user stopped work to correct it; that the
degradation tracked *volume of findings* rather than elapsed time or context pressure is an interpretation
of one session, not something re-run or controlled for.

**Context, project-agnostic.** A project's resume note carried a standing instruction: *read the
user's communication-preferences document at the start of every session*, with a command to fetch it.
The session ran that command, read the first ~120 lines of a ~260-line file, and summarised the rules
it had seen. It then spent the rest of the session violating the half it had not read, until the user
stopped work and asked for the format explicitly.

**Finding 1 — "read this doc" is an unverifiable instruction, and partial compliance is invisible.**
A resume note can direct a session to a document, but it cannot tell whether the session read the
whole thing. Worse, a *partial* read is more dangerous than a skipped one: the session comes away
with genuine, correct rules and therefore with confidence, while the governing section sits below the
cut. Here the unread portion contained the part that determined the *shape* of every reply, so the
rules that were read could not compensate for the one that was missed. The session had no signal that
anything was wrong — the instruction had been followed, by its own reckoning.

**Rule concluded:** a resume note that cites a document for *behaviour* should state its length and
say explicitly that partial reads do not count. Cheaper and more robust: inline the two or three rules
that most change output, so the pointer is an enrichment rather than a dependency. A pointer alone
makes correct behaviour contingent on a read the note cannot verify.

**Finding 2 — behavioural compliance decays across a session in proportion to how much the session finds.**
This is the more generalizable half. Early replies followed the format. They degraded steadily as
research accumulated — and the degradation tracked *volume of findings*, not elapsed time or context
pressure. The more a session discovers, the stronger the pull to present all of it, and the further it
drifts from any instruction about restraint, compression, or shape.

That inverts the usual assumption. Sessions that do a lot of successful work are treated as the good
case; for output-shape instructions they are the **high-risk** case, because the instruction competes
with a growing pile of things that genuinely seem worth saying.

**Rule concluded:** instructions that govern *how output is shaped* cannot be discharged by a
session-start read, because they are re-decided on every message while the pressure against them grows
monotonically. They need a per-message check cheap enough to actually run — here, *"does this reply
open with plain language, or with a table, a header stack, or a field name?"* A one-line mechanical
test caught every bad message in the session retrospectively and would have caught them prospectively.

**Where it lives:** here, plus the affected project's own lessons file and resume note. The
generalizable piece is for `/catch-up` and `/wrap-up`: both write and consume resume notes that lean on
"read X" pointers, and neither distinguishes a *reference* pointer (fine to cite) from a *behavioural*
one (must be inlined or length-stamped, because unverifiable compliance is the whole failure).

## 2026-08-26 — A spec that demands gated verification forces the agent to break a rule or fail the task

**Status: verified.** Re-read both agents' final reports and the two specs I had sent them, and
confirmed the difference in behaviour tracked the difference in wording, not the agents.

**Context, project-agnostic.** The project had a standing user rule: ask before invoking any
browser-automation tool. I delegated a batch of UI fixes and wrote, for two of the items, that the
agent should "reason about real rendered layout — neither is catchable by type-checking." Both items
were only verifiable by measuring a live page. The spec never said whether a browser was permitted.
The agent launched a headless browser to measure, and disclosed it unprompted, explicitly flagging
that it was unsure about the convention.

**Finding — the contrast is the diagnostic, and it exonerates the agent.** An earlier lane on the
same work, under the same standing rule, had *declined* to pick a browser driver and reported the
verification as still owed. Same repo, same rule, opposite behaviour. The variable was the spec: the
earlier brief had not demanded runtime proof, so honouring the rule cost it nothing. The later brief
demanded evidence that only the gated capability could produce.

A subagent has no user to ask. "Ask before doing X" is therefore unenforceable at the lane level: when
a spec requires X, the lane's only options are to violate the gate or fail the task. Both outcomes
belong to whoever wrote the spec. The rule silently converts into "the lane decides."

**Rule concluded:** before dispatching, check whether any verification the spec demands requires a
capability that is gated for the orchestrator. If so, resolve it *in the spec* — either grant it
explicitly with its constraints stated, or instruct the lane to stop and report what it could not
verify, so the orchestrator performs that step itself under the approval it already holds. Never
leave it to inference. This is capability-agnostic; the same omission around a higher-blast-radius
tool or a destructive command would be an incident rather than a harmless disclosure.

**Where it lives:** here, plus the affected project's own lessons file and the relevant memory. The
generalizable piece is for the orchestration doctrine's spec template: a "verification" section that
demands evidence should be checked against the orchestrator's own permission surface before it is
sent.

## 2026-08-26 — Naming a planning document "authoritative" makes its stale sections a false-positive generator

**Status: verified.** Re-read the two flagged items against the current source and confirmed both were
deliberate, current decisions rather than deviations; also confirmed the reviewer's four other findings
were real by reproducing the failing interaction in a browser.

**Context, project-agnostic.** A large implementation was produced in one bulk pass, deliberately
bypassing incremental human review, on the agreement that a thorough adversarial review would stand in
for the skipped gates. I dispatched a review lane and told it that a specific planning document was
"the authoritative spec — do not re-litigate its decisions; implement them." The review returned six
findings.

**Finding 1 — two of the six were false, and both traced to the document rather than the code.** They
were flagged as silent deviations from the plan's stated decisions. Both were in fact deliberate
choices made *after* that document was written, superseding it, based on evidence the document
predated. The reviewer had no way to know: I had told it the document was authoritative, and it was
diligent in comparing against it.

This is the third logged instance of a review lane being misled by superseded source material. The
recurring shape is not the lane trusting a bad source — it is the *orchestrator* conferring authority
on a source without checking whether it is current.

**Rule concluded:** when a spec cites a document as authoritative, state its as-of date and name any
sections known to be superseded — or, better, say which decisions the *dispatch itself* supersedes.
An authority grant is an instruction to stop thinking about whether the source is right, so it must
only be given for the parts that still are.

**Finding 2 — the same review comfortably earned its keep, and this half should not be lost.** Its
top finding was a genuine, user-visible defect in the exact controls that were about to be
demonstrated: a validation-timing configuration that raised errors on fields *while they were being
answered correctly*, and would not clear them. Confirmed by reproducing it. It was invisible to type
checks, lint and server-rendered markup inspection, all of which passed clean, and it existed in work
that had skipped incremental review by agreement.

**Rule concluded:** an adversarial end review is an adequate substitute for step-by-step gates when
the work is well-specified and the review is genuinely adversarial — but only if the orchestrator then
triages the output rather than relaying it. Four of six findings were real; two were artefacts of the
brief. Relaying all six as defects would have sent an implementer to "fix" two deliberate decisions.

**Where it lives:** here. Relevant to the orchestration doctrine's guidance on when the advisor pass
can replace intermediate review, and to how findings are relayed to the user.

## 2026-08-26 — "Verified" is a claim about one path, and volatile facts get the least rigour

**Status: verified.** Both findings were reproduced first-hand after a context-free agent reported
them — the data-loss path by driving the real UI and reading the database before and after, the stale
file by reading it directly.

**Context, project-agnostic.** At the end of a long build session, with a high-stakes handoff the next
morning, I wrote a runbook and a resume note, then pointed a fresh agent with no inherited context at
them and asked where the documentation failed it. It found two errors that would have caused visible
failure in front of a client. Both were mine, and both had been written down as verified facts.

**Finding 1 — I verified one code path and documented it as the behaviour.** A form field rendered
blank because the stored value was in a format the browser control rejects. I tested whether saving
destroyed the value: I opened the dialog, saved without touching the field, and the value survived.
I wrote "it does not save wrong — verified" into the runbook.

The fresh agent traced a *different* path through the same component and predicted data loss. I
reproduced it: clicking into the blank field and tabbing away — no typing — caused the save to
overwrite the stored value with empty, with a success confirmation shown. The form library reads the
control's value on blur, and the browser had already sanitised the unparseable value to empty.

The distinction I collapsed: my test established *"saving without touching this field is safe."* I
wrote *"saving is safe."* The word "verified" then made the broader claim unfalsifiable to anyone
downstream, because it signalled the question was closed.

**Rule concluded:** a verification claim must name the path exercised, not the conclusion drawn from
it. "Verified: saving without focusing the field preserves the value" is honest and invites the
obvious next question. "Verified: it does not save wrong" forecloses it. When writing a safety claim
into a document others will act on, state the interaction tested — if that reads as awkwardly narrow,
that narrowness is the actual state of the evidence.

**Finding 2 — the volatile operational facts got the least rigour, and they were the ones that
mattered.** The same documents contained analytical claims — counts derived from a large source
dataset — that the fresh agent recomputed from the raw source and found exact to the row. The claims
that were *wrong* were all operational: which directory a service should run from, which of two
copies of a data file was current, whether a dependency was actually pinned.

The pattern is that analytical claims *feel* like claims and get checked, while operational facts feel
like context and get typed from memory at the end of a session, when attention is lowest. They are
also the ones with same-morning consequences: an incorrect derived statistic starts an argument, an
incorrect directory starts a service against stale data in front of a client.

A related instance in the same document: a deliberate dependency-version decision was recorded in the
resume note, which is overwritten every session, while the file it described kept a range that would
silently undo it. Intent recorded in a disposable artefact is not recorded.

**Rule concluded:** treat operational facts as the highest-risk category in any handoff document, not
the lowest. Anything of the form *"run X from Y"*, *"Z is the current copy"*, or *"W is pinned/locked/
disabled"* should be re-checked against the machine at write time, not recalled — and where a decision
must outlive the document, encode it in the artefact it governs rather than describing it in prose.

**Where it lives:** here, and the affected project's runbook and resume note were corrected. For
`/wrap-up` specifically: the resume-note step should push operational facts toward verification-at-
write-time, and should discourage recording durable intent in a file whose own header says it is
overwritten each session.

## 2026-08-26 (later session) — A pipeline stage with no completion check stalls silently, and the operator's belief that it ran is not evidence

**Status: verified.** The version gap was read directly off the installed copy on disk, and the
merge state was read from the server rather than from local tracking refs.

**Context, project-agnostic.** Shipping a change from a repository into the sessions that execute it
is a four-hop chain: merge, publish, refresh the package index, install. Three of those hops report
completion. The fourth — whether the running process is now executing the new code — reports nothing
at all, and no hop verifies its predecessor.

The result compounded invisibly. The plugin the session was actually running was **three releases
behind** the default branch. Every session in between had inherited old behaviour while the
repository looked healthy, and nothing anywhere would have surfaced it. Separately, the operator
stated that a pull request had been merged; it had not. A one-line server query disproved it. The
belief was sincere and the interface had given no contradicting signal.

**Finding 1 — the failure is silence, not error.** Every command in the chain exited zero. A stage
that cannot fail loudly will fail quietly for as long as nobody thinks to look, and "nobody thinks to
look" is the default state of a working system.

**Finding 2 — there are three states here, not two, and the third is the one a naive check misses.**
An obvious drift check compares the installed version against the published one. That comparison is
blind to the window between installing and restarting, where the files on disk are current and the
running process is not. A two-way check reports "up to date" while old code keeps executing. That is
the original bug reproduced one level down, inside the fix for it.

**Finding 3 — the operator's report of a completed step is a claim, not evidence, and it is cheap to
check.** Asking the server directly cost one command and overturned the premise of the next three
actions. The temptation to accept it is that disputing a human's account of their own action feels
like a discourtesy; it is not, because the interface they acted through may have failed silently and
they would not know.

**Rule concluded:** a stage that changes what a session executes must end with a check that reads the
executing artifact, not the command's exit code. Where a chain of stages exists, the last one needs
that check most, because it is the only one whose failure is invisible from every other stage. When
someone reports having completed an out-of-band step, verify it from the authoritative source before
building on it — silently, without ceremony, and say plainly when it disagrees.

**Where it lives:** a session-start drift check was built and shipped, comparing running / installed /
available and reporting the restart case first. That covers this instance. The general rule — verify
the executing artifact, not the exit code — belongs wherever multi-stage delivery is described.

## 2026-08-26 (later session) — A harness command was recommended without being verified, and this is the same failure that already has a standing rule against it

**Status: verified.** The command's absence was confirmed by inspecting the installed binary, which
also revealed the correct mechanism and the correct settings key.

**Context, project-agnostic.** After installing a configuration file for the user, the orchestrator
told them to run a specific slash command to activate it. The command did not exist in their version.
The recommendation came from trained-in knowledge about the tool, stated as fact, with no check.

What makes this worth logging rather than shrugging off: **there is already a standing instruction to
verify harness behaviour empirically rather than relying on trained knowledge**, and it did not fire.
The failure mode is not ignorance of the rule. It is that a command name feels like a fact rather than
a claim, so it never gets routed through the verification path at all.

The recovery was cheap and should have been the first move: the tool's own binary was searchable, and
one pass over it produced the real answer (the command had been folded into a general configuration
surface), the settings key, and enough confidence to write the setting directly instead of sending
the user hunting through a menu.

**Rule concluded:** the name, existence, and syntax of any harness affordance — a command, a flag, a
settings key, a file path the tool reads — is a claim about a specific installed version, not general
knowledge. Check the installed artifact before telling a user to type something. When a recommendation
does turn out to be wrong, the corrective move is to search the shipped binary or package rather than
to guess a second time; a second guess after a failed first is where confidence outruns evidence
fastest.

**Where it lives:** here. The standing rule already exists and was not enough on its own, which
suggests the gap is in recognising *what counts as a claim*, not in the willingness to verify one.

## 2026-08-26 (later session) — A rule written in two places disagreed with itself, and the copy that executes won silently

**Status: verified.** The contradiction was read from both files directly, and adherence was counted
across thirty pull requests in two repositories.

**Context, project-agnostic.** A contributor-facing document specified an output format in detail. A
command that the agent actually executes contained its own, shorter paraphrase of the same rule — and
the paraphrase said the opposite: *keep it to a heading and one sentence.* Adherence to the documented
format was 3 of 11 in one repository and 4 of 19 in another, and every compliant instance had been
written by hand rather than produced by the command.

**Finding 1 — the executing copy wins, and nothing announces the conflict.** The agent following the
command was not disregarding the doctrine. It never saw it. Neither command file contained a single
reference to the document, the format, or its vocabulary. A rule that lives only in a file nothing
loads is not a soft rule; it is an absent one.

**Finding 2 — the fix is not better prose in the document.** Restating the rule more carefully in the
place already being ignored is doing the failing thing harder. The fix is to collapse to one
definition and put it where the executing path loads it, then have the human-facing document point at
that rather than restate it.

**Finding 3 — a pointer that summarises is still a second copy.** Reducing a document's section to
"see X" plus a convenient outline reintroduces the drift in miniature, because the outline can go
stale. Where an outline genuinely earns its place for human readers, state the precedence explicitly:
name which copy is authoritative and say the other is stale if they differ. The previous arrangement
did not fail because it had two copies; it failed because it had two copies and no rule about which
one won.

**Rule concluded:** state a behavioural rule exactly once, in the artifact the executing path loads.
Everything else points at it. When two copies are genuinely warranted, precedence must be written
down inside both of them — an undeclared duplicate resolves itself arbitrarily and silently.

## 2026-08-26 (later session) — Two delegated claims outran their evidence in the same session, in opposite directions, and both were caught by opening the cited file

**Status: verified.** Both were checked against the primary sources the lane itself cited.

**Context, project-agnostic.** A read-only research lane produced a long, well-structured audit. Its
load-bearing claims were spot-checked before being relayed. Most held. Two did not, and they failed
in different ways worth separating.

**Finding 1 — severity inflation on a claim the source itself refutes.** The lane flagged two
passages in one document as a *direct tension* and named it the document's highest-severity defect.
Opening the first passage showed it explicitly cross-referencing the second and warning the reader
about exactly the tradeoff. The underlying observation was real but much weaker: the caveat sits far
below the requirement it qualifies. **Severity inflation has now recurred across many sessions and
several lanes; it is a standing property of delegated review, not an occasional slip.**

**Finding 2 — purpose inferred from file properties and reported as fact.** The same lane
recommended deleting a file as stray, having inferred that from its being empty at scan time and
excluded from version control. It was neither stray nor accidental: it was a deliberate input channel
the user writes into, excluded on purpose. Every observable property was correct; the conclusion drawn
from them was not.

The shared shape: **"unused," "stray," "contradictory," and "high-severity" are all conclusions, not
observations.** A lane can establish that a file is empty and excluded, or that two passages exist. It
cannot establish what a file is *for*, or that two passages *conflict*, without reading further than
the properties.

**Rule concluded:** before relaying a delegated finding, re-open the primary source for any claim that
carries a severity judgment or attributes purpose. Verifying a lane's *observations* is often
unnecessary; verifying its *characterisations* is where the errors live. A lane's confidence is
uncorrelated with which of the two it is doing.

## 2026-08-26 (later session) — Destructive version-control work was fully built before the environment refused to run it

**Status: verified.** The permission boundary was established by attempting progressively narrower
operations until one succeeded.

**Context, project-agnostic.** A migration was designed, scripted, and dry-run successfully across
fourteen branches — every branch verified as a clean append, extraction confirmed lossless, backups
planned. The environment then refused to execute it, because the migration required force-pushing
branches and the sandbox blocks destructive version-control operations.

Nothing was wasted permanently, but the ordering was wrong: the work that could not proceed was
completed in full before the blocker was discovered, and the user then had to be handed a decision
they could have been given much earlier.

The boundary itself was not obvious and had to be probed: a dry run passed, a single ordinary tag
passed, a forced tag across many refs failed, and running the script failed. Read operations and
ordinary writes were fine; anything that rewrote or force-updated refs was not.

**Rule concluded:** when a planned task's final step is destructive — force-push, history rewrite,
bulk deletion — establish that the environment permits it before building the rest. One cheap probe
against a single representative target costs seconds and reorders the whole task. And when a sandbox
does refuse, stop and hand the user the decision rather than searching for a narrower phrasing that
slips through; the refusal is the mechanism working, and routing around it is a different act from
completing the task.

**Where it lives:** here. A command that plans destructive version-control work should probe the
permission boundary during planning, not at execution.

## 2026-08-26 (later session, found on a user-requested re-review) — A file's warning about how it fails was read, understood, and then violated in the same act of rewriting it; and the sweep produced the exact artifact its pending fix exists to prevent

**Status: both findings verified.** The dropped content was established by diffing the rewritten note
against the version it replaced. The duplicate-artifact count was produced by re-running the
migration's own dry run after the sweep completed.

**Context, project-agnostic.** The session-end command writes a resume note for the next session. The
note it overwrites carried, in its own header, a warning that this had gone wrong twice before, plus
an explicit instruction: *diff against the old version before overwriting, because rewriting from
session memory drops live threads.* The orchestrator read that header at session start — it was the
first thing the session-start command surfaced — and then rewrote the note from memory anyway.

The user asked for a review of the session's documentation. That review found the failure. Nothing in
the sweep itself would have.

**Finding 1 — the warning was in the right place, addressed to the right reader, and still did not
fire.** The first draft dropped seven standing constraints, two published artifact URLs, four open
questions, and one undone decision. Several were not stale: one dropped open question had been
*activated* earlier in the same session (a risk that installing something would make live, which
installing it duly did), and the dropped artifacts were deliverables the user had been reading hours
before.

What makes this worth logging rather than filing as carelessness: the instruction did not fail
through inattention. **The act of writing a comprehensive note feels like remembering, so the check
that would reveal what was forgotten never gets triggered** — the same shape as a partial read
producing confidence. A rewrite composed from a rich session context is *more* susceptible, not less,
because the author has so much genuine material that the absence of the rest is unnoticeable.

**Rule concluded:** an instruction to compare against a prior version cannot live only in the artifact
being replaced, because the replacement is authored by someone who believes they already know the
contents. The comparison has to be a step in the procedure that produces the artifact, with the old
version actually retrieved and read — not a caution the author is trusted to remember. Where a
document is overwritten rather than appended to, the writing step and the diffing step are two
separate obligations, and only the second one catches loss.

**Finding 2 — the sweep created a new instance of the problem its own pending fix addresses.** A fix
was written, reviewed and merged this session that changes the sweep to write a fresh file per
session instead of appending to a shared one, precisely because appending had produced fourteen
mutually-conflicting branches that never merged. The fix sat unreleased. The sweep then ran the old
behaviour and produced a fifteenth.

This was foreseen — the orchestrator had told the user in as many words that every sweep before the
fix ships adds another. It still happened, because the sweep executes the *installed* command, and
merging is not installing.

**Rule concluded:** between merging a behavioural fix and installing it, the old behaviour keeps
running and keeps generating the problem. When a fix changes what a routine command produces, the
backlog it addresses is still growing during that window, and any count taken before the window
closes is already stale. Either install before running the routine again, or record that the count
moved and by how much.

**Where it lives:** here. The resume-note step in the session-end command should require retrieving
the previous version and diffing before writing the new one, rather than describing that requirement
inside the note it is about to destroy.
