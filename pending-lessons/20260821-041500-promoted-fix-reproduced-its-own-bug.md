### 2026-08-21 — A fix promoted from this log reproduced the bug it was written to prevent; and the human's out-of-band edit survived only because a push was rejected

**Status: findings 1, 3, 4, 5 verified** (each re-checked directly against the artifact, the server,
or git history this session); **finding 2 verified against published sources** (read, not merely
recalled).

**Finding 1 — the first finding ever promoted from this log into a command body contained the exact
defect it was written to prevent, and no amount of prose review would have caught it.** A logged
failure was that a command trusted a locally cached view of remote state. The promoted fix correctly
told the model to query the authoritative source — and then illustrated it with a single *narrowly
scoped* query, while a rule three lines later demanded a *broad* claim about the whole remote. The
narrow query returns exactly what it was asked for, which is indistinguishable from a complete
answer. Following the new rule reproduced the original failure, and did so in the same session that
shipped it: the orchestrator ran the narrow query, got a correct answer, and reported it as complete.

This was found by an advisor consultation, but not by the advisor "reviewing the diff." It was found
by reading the edited text *against the concrete failure scenario* and asking whether it actually
prevents it. Two models discussing the wording would both have anchored on the wording that exists.

**Rule concluded:** promoting a logged finding into a command body must include a step that runs the
original failure scenario against the edited text — a replay or a fresh-context probe — not a
read-through. For documentation that programs a future model, prose review is a proxy and behavioral
evidence is the actual validation. Corollary: treat a clean probe as weak evidence and a failed probe
as strong, since one lucky pass proves little.

**Finding 2 — model-vs-model debate was proposed as a quality mechanism; both internal review and
external literature rejected it, and the literature then corrected the correction.** The orchestrator
proposed having two model tiers argue each decision, on the theory that competition sparks deeper
questioning. The advisor rejected it, citing the doctrine's own statement that it is a fresh-eyes
check and never an independent one. External sources went further: multi-agent debate frameworks fail
to consistently beat single-agent baselines, are *over-aggressive* (converting correct answers to
incorrect), and perform worst under exactly the rigid assigned-opposing-roles structure that was
proposed.

But the same search surfaced a method that does work, and the distinction is the useful part:
**debate contests an open question with no ground truth; refutation contests one falsifiable claim.**
Refutation of a specific finding improves precision; open debate degrades it. What the advisor
actually did well in this session was refutation — it falsified the claim "this edit prevents failure
X" — which neither party had identified as the operative mechanism beforehand.

**Rule concluded:** frame an advisor consultation as *"Claim: this change prevents failure X.
Attempt to falsify it"* rather than "review this." Same cost, sharper target, defined stopping
condition. Also: make "no findings" an explicitly valid result, since a model asked to review will
otherwise nearly always produce something, and review noise is what causes a human to stop reading
review output at all.

**Finding 3 — the human edited a branch out-of-band, and that edit survived only because a routine
push happened to be rejected.** The user amended a file directly through the forge's web UI while
the session was preparing another commit to the same branch. The session's push was refused as
non-fast-forward, which is the only reason it looked. Had the push succeeded, or had the session
reached for a force-push on seeing the rejection, the human's edit would have been destroyed with no
signal to either party. The recovery was correct — inspect the commit, rebase onto it, preserve it in
history — but it was triggered by an error message, not by policy.

**Rule concluded:** before pushing to a branch that has an open PR, query the server for that
branch's tip and compare it to the local tip. Never force-push a branch under review. The human's
direct edits are the highest-value signal in the system and the channel carrying them is currently
protected only by a default git safety check.

**Finding 4 — user-sourced: an edit was read as feedback and generalized wrongly; asking produced a
different and better rule.** The human's edit replaced two personal-name references with a generic
term. The orchestrator inferred the reason was *public exposure of a name* and extended the change
accordingly. Asking revealed the actual reason was **register** — the document should read as product
documentation rather than as a working note between two people. The two readings generalize
differently: the inferred rule would have stripped names and stopped; the actual rule also caught a
public contributor document citing a *private* companion repo as its evidence, and an anecdotal
parenthetical about how a rule had been discovered. Neither would have been touched under the
inferred rule.

**Rule concluded:** when a human edit is treated as feedback, state the inferred intent and ask,
rather than silently generalizing it — the inference is cheap to make and was wrong in a way that
would have produced a narrower rule that looked correct. Log the answer as **user-sourced**, distinct
from model-observed findings, under the existing promotion discipline. First user-sourced entry in
this file.

**Finding 5 — the session designed the process for draining this backlog instead of draining it.**
Verified against git: the default branch of the plugin repo ended the session byte-identical to how
it started. Of the findings logged in this file, exactly one was converted into a fix, and that fix
is unmerged. Meanwhile the session produced a review process, a research document, a backlog entry,
and a triage system. The meta-work was genuine and partly user-directed, and it did catch a real
defect — but a self-improving system has a structural bias toward improving its own improvement
machinery, because that work is always available, always feels principled, and never requires
finishing anything.

**Rule concluded:** `/wrap-up` should report the concrete delta — findings converted to fixes, PRs
merged — alongside whatever process work happened, so that a session which produced only meta-work
is visible as such at the moment it ends rather than only when someone asks directly.

**Finding 6 — six consecutive version bumps silently did nothing, because the edit was keyed on an
assumed current value rather than the actual one.** Verified against the merged history. The plugin's
own contributor doc requires a version bump on every change, since the install cache keys on it and a
content change without one stays stale forever. The session performed that bump six times on one
branch using a find-and-replace that searched for a specific old version string — a string carried
over from a *different* branch rather than read from the file being edited. Every replacement matched
nothing. The tool exited successfully each time. The branch merged carrying no version change at all,
and the failure surfaced only when an unrelated check compared the published version against the
merged content.

Two properties make this worse than an ordinary slip. It is **silent by construction** — a
find-and-replace that matches nothing is indistinguishable from one that matched, at the exit-code
level. And it is **self-concealing across a chain** — once the first bump no-ops, every later bump
searches for a value that now never existed, so a sequence of them fails identically and the file
looks untouched rather than half-edited.

**Rule concluded:** never edit a value by searching for its assumed current contents. Read the value,
then write the new one — or assert the match and fail loudly when it is absent. This is the same
class as the session's other findings: an operation that returns exactly what it was asked for,
where what it was asked for was wrong, and the correct-looking result suppresses the check. The
version-bump instruction in the contributor doc should say to read-then-write rather than simply
"bump the version," since the natural mechanical reading of that instruction is the one that fails.

**Where it lives:** here only. Finding 1's replay requirement and finding 2's refutation framing
belong in the orchestration skill's Verification and advisor-consultation sections. Finding 3 belongs
wherever push behavior is described. Finding 4 is already written into the contributor doc's
reasoning-capture section, pending review. Finding 5 belongs in `/wrap-up` itself. Finding 6 belongs in the contributor doc's version-bump
section.
