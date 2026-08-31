### 2026-08-21 — The hardcoded resume-note path recurred in a second project, and `/catch-up` has no non-git branch

**Status: verified** for all three findings (each re-checked against the command bodies and the
observed project state this session).

**Finding 1 — the 2026-08-19 hardcoded-path finding recurred in a different project, and the user
has since built a manual workaround for it.** The rule concluded that day (both commands should
*discover* an existing resume note before assuming their own filename) is still unimplemented. In
this session's project — documentation-only, maintaining its own resume note in a subdirectory —
someone had already worked around the gap from the other side: the root `RESUME-PROMPT.md` had been
turned into a **deliberate pointer file** whose entire content is "this project maintains its own
resume note, read that one instead, this file exists only because `/wrap-up` writes to this
hardcoded root path." The file even states the reason explicitly — a pointer rather than a second
copy, *so the two can't drift apart*, which is precisely the drift the earlier entry predicted.

This is worth recording as a recurrence rather than a duplicate for two reasons. First, it is the
second independent project to hit it, which moves this from "one incident" to "a repeated pattern"
under this file's own promotion bar. Second, the workaround is informative: given no discovery
mechanism, users don't file a bug, they neutralize the command by feeding it a decoy file. That
means the friction stops being *visible* while continuing to cost something — `/catch-up` step 1
("read `RESUME-PROMPT.md` in full") now returns a signpost rather than context, and the session only
gets oriented because the model follows the pointer on its own initiative. Same failure shape as the
original finding: the command's own instructions do not carry the recovery step.

**Rule concluded:** unchanged from 2026-08-19, but the case for actually promoting it is now
materially stronger. Add a discovery step to both commands rather than leaving users to build decoy
files.

**Finding 2 — `/catch-up` has no branch for a project that isn't a git repository, and `/wrap-up`
does.** Roughly three-quarters of `/catch-up`'s instruction body is git work: separating local from
remote state, `git status` vs `git ls-remote`, scoping `refs/heads/<branch>` against `--heads`, and
two reporting rules about never asserting a negative about a remote without server evidence. All of
it is good guidance and none of it applies when there is no repository. The command's only stated
fallback covers a *missing resume note*, not a missing repo, so the model is left to improvise the
framing for most of what it was just told to do.

The asymmetry is the useful part: `/wrap-up` step 3 handles this correctly in a single clause ("If
this project is a git repository, make sure the file is excluded via `.git/info/exclude`"), so the
plugin already knows non-git projects exist — the handling just isn't consistent across its two
commands. Cheap to fix: one early conditional in `/catch-up` saying that if the project isn't a git
repo, skip steps 2–3, say so plainly, and go straight to the resume note.

**Finding 3 — the orchestration doctrine's "ALWAYS consult the advisor before reporting done" has
no precedence rule for a host environment that forbids spawning subagents.** This session ran under
a standing harness-level instruction not to invoke the agent tool unless the user asked. That
instruction and the skill's advisor mandate are in direct conflict, and the doctrine does not say
which wins or what to do instead. Deliverables were produced and no advisor review happened; the
harness instruction won by default, silently, because it was the more specific and more local of the
two.

Worth being precise about scope: this is not the plugin misbehaving — the conflicting instruction
came from the host environment, not from the plugin. But a mandate phrased as ALWAYS should say what
happens when it cannot be honored. The useful answer is probably not "spawn anyway" but "say out
loud that the review step was skipped and why," so an unreviewed deliverable is never silently
reported as done. That also matches the theme of the 2026-08-20 entry — the doctrine being
under-specified exactly where two of its own instructions meet, except here the second instruction
comes from outside the plugin.

**Where it lives:** here only. Finding 1 reinforces the still-unpromoted 2026-08-19 rule and belongs
in `commands/catch-up.md` and `commands/wrap-up.md`. Finding 2 belongs in `commands/catch-up.md` as
an early conditional. Finding 3 belongs in the orchestration skill's advisor section as a fallback
clause.
