### 2026-08-31 — A third-party config file's own comment was reported as behavior, the user agreed to a change based on it, and the implementation said otherwise

**What happened** [verified]: the orchestrator was diagnosing why a workflow felt tedious for the
user. It grepped a vendored third-party module's configuration file, found a setting that was
enabled, and read the setting's inline documentation comment — written by that module's authors —
which described the feature it suppressed in broad terms. The orchestrator reported that broad
description to the user as the cause of the problem, and additionally asserted a second, related
effect that the comment did not state at all but which the orchestrator inferred from the setting's
name and its own prior knowledge of the domain.

The user agreed to the change on that basis. Only while preparing to make the edit did the
orchestrator open the module's source. The setting's implementation set two flags, and tracing those
flags into the core showed the affected code path was branch-guarded to one narrow object class —
the broad case the user actually cared about was handled in a sibling branch with no config check
and had never been affected. The second, inferred effect was wrong too: the data backing it was
fully populated, confirmed by counting it directly, and had been working the entire time.

So the diagnosis was wrong, the user had already assented to a remedy premised on it, and the real
answer was that the user did not know a long-standing feature existed. The correction had to be
issued after agreement rather than before it.

Two things made this worse than a plain mistake. The setting's name matched the user's complaint
closely enough to feel like a hit, which is exactly when a claim stops getting checked. And the
config comment was a *primary-looking* source — it shipped inside the artifact being discussed,
which reads very differently from a note someone wrote about it, even though it is the same kind of
object.

This is the second consecutive sweep logging this shape, and the prior one was one day earlier. That
entry concluded a rule about a **script's own header comments** being unverified claims about the
code. The rule generalizes and did not get applied, partly because it was stated about executable
artifacts the project itself had written, and this was documentation prose inside a third party's
config file.

**Rule concluded:** documentation that ships *inside* an artifact — a config file's comments, a
header block, a docstring — is a claim about behavior written by someone who was not observing the
behavior at the time. It is a pointer, not a finding, and being co-located with the code buys it
nothing. When a claim sourced that way is about to become the premise of a change the user is asked
to approve, read the implementation before asking, not after. The cost asymmetry is the whole
argument: checking first is one grep, and checking after means retracting something the user has
already agreed to.

**Second half of the rule, which is the part that failed here:** confidence should drop, not rise,
when a candidate cause matches the symptom by name. A name match is the cheapest possible evidence
and it is the point at which verification gets skipped.

**Finding 2 — a context-reload command's "reconcile, don't restate" instruction produced real value
in a project with no version control, where its git-specific machinery was inapplicable** [verified].
The command's body is largely about establishing local versus remote state through version control,
and the project had none, so most of its procedure was inert. The instruction that survived was the
general one: reconcile the resume note against what is actually observable rather than restating the
file.

Applied to filesystem evidence instead of commits, it worked. The resume note asserted that a piece
of work had never been exercised. Modification timestamps on a state file, cross-referenced against
a tool's own log showing a start and exit time, showed it had been exercised for roughly half an
hour, and a diff against the pre-change backup showed the written state had survived the round trip
intact. That materially changed the next action the user was given — from a debugging session to a
short confirmation.

Worth recording because the project has already logged twice that these commands assume a git repo.
That framing treats the non-git case as a deficiency. This session is evidence the commands' *core
doctrine* — treat the note as stale until checked, prefer observed state over asserted state — is
substrate-independent, and only the mechanism is git-shaped. A future revision should separate the
doctrine from the git procedure rather than adding a non-git branch to the procedure.
