### 2026-08-24 — The spec contract has no slot for architect-asserted environment facts, and a research lane's *primary* recommendation named a dependency that does not exist

**Status: finding 1 verified** (the installed versions were re-read directly from the resolved
dependency tree and confirmed to differ materially from the declared ranges the spec quoted);
**finding 2 verified** (the named package was queried against the public registry and returns 404);
**finding 3 asserted** (an impression about how a command's emphasis shaped report structure; not
re-tested).

**Finding 1 — the five-part spec treats everything the architect writes as known, and configuration
files are not evidence of runtime state.** Writing research specs for two parallel lanes, the
orchestrator included an "exact versions" block describing the project's stack, transcribed from the
dependency manifest's *declared ranges*. Every figure was wrong. The resolved tree had floated far
past the declared minimums — in one case by dozens of minor versions — and one of those gaps was
load-bearing: a major version of a validation library was already present on disk under a subpath,
reachable or not depending on a second package the spec had also mis-stated.

The lane caught it and reported it under a "contradicts the prompt's stated assumptions" heading it
invented itself, having not been asked for one. Had it simply complied, the research would have been
aimed at older library behaviour and its recommendations could have been quietly stale.

The doctrine's spec contract asks for Constraints and Interfaces as things the architect supplies. It
offers no way to mark a supplied fact as *asserted rather than verified*, and nothing prompts the
architect to notice the difference between "I checked this" and "I read this off a config file and
assumed it described reality." Declared configuration is the most tempting source precisely because
it is the easiest to read and looks authoritative.

**Rule concluded:** specs should distinguish verified environment facts from asserted ones, and
should explicitly invite the lane to contradict the premises it is handed. The lane that did so
volunteered the behaviour; it should not have to. Worth stating plainly in the spec contract that
declared configuration is not evidence of resolved state — resolve it before quoting it.

**Finding 2 — a research lane's headline recommendation rested on a component that has never
existed, which qualifies the previous entry's conclusion.** The 2026-08-20 entry concluded that lanes
are reliable about their assigned output and markedly looser about context they volunteer around it.
This session produced a clean counterexample in the supposedly reliable half.

A research lane recommended a particular control for a hard UI problem, and paired the
recommendation with its own risk mitigation: use a well-known primitive library's ready-made version
rather than hand-rolling the accessible behaviour, which the same document called one of the
highest-risk widgets to build correctly. That primitive does not exist. The package name returns 404
from the public registry and the library has never shipped such a component. The recommendation's
entire safety case rested on it, and the claim appeared repeatedly across the document, including in
the sentence carrying the recommendation itself.

What makes this different from the incidental-claims pattern: this *was* the deliverable. And the
catch did not come from the doctrine's verification step, which is aimed at re-running a command
against a diff. A research deliverable has no command to re-run. It came from unrelated domain
skepticism that happened to be present.

**Rule concluded:** the verification doctrine silently assumes a runnable artifact. Research and
design lanes produce documents whose load-bearing claims are *existence and capability claims about
the outside world*, and those need a different check — enumerate the claims the recommendation
actually depends on and confirm each independently. The spec contract's fifth part ("the command
that proves it works") does not apply to a research lane and should say what replaces it, rather
than leaving the slot to be filled with something unfalsifiable.

**Finding 3 — a report-producing command's internal emphasis on rigour became the report's
organising principle, against a stated user preference.** The session-start catch-up command devotes
substantial instruction to verifying remote state properly and to strict rules about which claims
may be made without server evidence. That guidance is genuinely good and is the reason an earlier
entry exists. But it governs *what may be said* and says nothing about *how much*, or in what order.

Followed faithfully, it produced an opening report organised around evidence and method — leading
with how state was verified, and with the caveats attached to each check, before reaching what any
of it meant for the next action. The user's own documented communication preference is close to the
inverse: plain headline first, methodology available on request rather than pre-expanded. The
correction came from the user within two turns.

**Rule concluded:** commands whose output is a user-facing report should either say something about
output altitude, or explicitly defer to the user's own communication preferences where those exist.
Rigour instructions shape form as well as content, and a command that is silent on form will have
its form set by whatever it talks about most.

**Where it lives:** here only. Finding 1 belongs in the spec contract section, as a distinction
between verified and asserted supplied facts. Finding 2 belongs in the Verification section, as the
research/design-deliverable case the current wording does not cover — and it partially qualifies the
2026-08-20 entry, which should not be read as "primary output is trustworthy." Finding 3 belongs
with the catch-up command.

### 2026-08-24 — `/wrap-up`'s dirty-tree abort guards against a risk its own commit step already eliminates, and has now silently deferred two sessions' findings

**Status: verified** (the abort fired this session and was traced to its cause; the command text was
read directly from `origin/master`, not from the local working copy, which is two commits behind).

**What happened:** a session's `/wrap-up` logged three findings to `lessons.md` and then reported:
*"PR skipped — the notes repo already had two untracked files, so I didn't branch from a dirty
tree."* The two untracked files were unrelated research references written earlier in the same
session by the orchestrator. The findings were left as an uncommitted working-tree change, visible
to nobody but whoever next opens that repo on that machine.

**Finding 1 — the abort condition is broader than the risk it names, and the command already
contains the narrower protection.** `commands/wrap-up.md:36-41` says: *"If step 1 found the repo
already dirty, or not on its default branch, before you touched anything: don't create a branch or
open a PR — branching from a dirty tree risks sweeping someone else's unfinished edits into the
commit."* But the normal path, thirteen lines later at `:49`, already says: *"Stage and commit **only
`lessons.md`** — never `git add -A` or sweep in other uncommitted work that might be sitting in that
repo for unrelated reasons."*

The sweeping risk is eliminated by the commit step regardless of tree state. Untracked files in
particular are never staged by a path-scoped `git add`, and they survive a branch switch untouched —
there is no mechanism by which they could have been swept. The guard aborts a whole deliverable to
prevent something that cannot happen on the path it is guarding.

The correct condition is narrow: **abort only when `lessons.md` itself already carries a modification
that is not yours** — a genuine conflict on the one file the command commits. Anything else in the
tree is irrelevant to a path-scoped commit.

**Finding 2 — the failure is silent in the direction that matters, and it compounds.** The command
does tell the user the PR was skipped, so it is not silent to the person present. But the entry then
sits as an uncommitted change, which *is* the dirty-tree condition — so the **next** wrap-up aborts
too, for the same reason, now caused by the previous abort. Each skipped sweep guarantees the next
one skips. Two sessions' findings were deferred this way before anyone noticed (2026-08-23 per
`RESUME-PROMPT.md`, and again today), and the mechanism gets monotonically worse, not self-correcting.

This is the second recorded instance of the same abort with the same cause. **It clears the
recurrence bar this file sets, and is a promotion candidate rather than a log entry.**

**Finding 3 — the abort's stated fallback puts manual git work on the user, which is the opposite of
what the command is for.** The text says the user *"will need to sort out committing it themselves
alongside whatever else is there."* A session-hygiene command exists to spend the user's attention
less, and its failure mode spends more of it, on exactly the mechanical git work it was written to
absorb.

**Fix/rule concluded:** promote. Replace the tree-state check at `commands/wrap-up.md:36-41` with a
file-scoped one — proceed unless `lessons.md` itself is already modified by someone else, and say so
in that case only. The "not on its default branch" half of the condition is a separate question and
is better defended, since the command branches off the default branch; it should be considered on its
own merits rather than removed alongside the dirty-tree half.

**Where it lives:** here, and this one should not stay here — it is the second instance and the fix is
a small edit to one command body. Related to notes PR #13 (commands assume a git repo) only in that
both are `/wrap-up` and `/catch-up` making assumptions about repository shape; they are separate
fixes.
