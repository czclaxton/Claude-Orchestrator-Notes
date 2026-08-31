### 2026-08-23 — Both commands assume a git repo; in a non-VCS project most of `/catch-up`'s body is inert

**Status: verified** (observed directly this session — the repository check failed and the majority
of the command's instructions had no applicable target).

**What happened:** ran `/catch-up` in a project that is not a git repository at all — a
documentation/knowledge-base project maintained in place, no VCS of any kind. The command's body
devotes most of its length to establishing local versus remote state, including a long and
genuinely well-reasoned passage on not trusting local tracking refs, scoping `ls-remote` queries to
match the claim being made, and never asserting a negative about a remote without server evidence.
None of it had a target. There is no branch in the command for "not a repository," so the correct
behavior had to be improvised: report the absence plainly, then substitute direct filesystem
inspection as the source of truth to reconcile the resume note against.

Nothing was misreported and the command still produced a useful result, so this is a gap rather
than a failure. Two things make it worth logging anyway. First, it is the second distinct
*project-shape* assumption to surface (the first being the hardcoded resume-note path, logged
2026-08-19) — the underlying pattern is that the commands assume a code repo under git, and degrade
in undefined ways when dropped into a project that is neither. Second, it had already surfaced
silently in this same project at least once before: the existing resume note carried a line
recording that there was nothing to exclude from version control, i.e. `/wrap-up`'s git-exclude
step had no-opped there in an earlier session and nobody logged it.

**Rule concluded (not yet promoted):** both commands should establish whether the project is a git
repository as an early, explicit step, and skip their VCS sections outright when it is not, rather
than leaving the model to decide what to do with a large block of inapplicable instructions.
`/wrap-up`'s `.git/info/exclude` step already degrades gracefully on its own. `/catch-up`'s
state-reconciliation section is the one that needs an explicit non-VCS fallback — naming filesystem
inspection as the substitute source of truth, so the "treat observed state as authoritative over
the resume note" principle survives the absence of git.

**Where it lives:** here only. Not yet written into `commands/catch-up.md`.

**Also worth noting, asserted:** this session used zero implementation lanes and made no routing
decisions — it was research and Q&A end to end. The three-rung ladder was never exercised, so this
entry says nothing about it either way.
