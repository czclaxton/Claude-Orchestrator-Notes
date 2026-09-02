### 2026-09-01 — `catch-up` assumes the project is a git repository and offers no fallback when it isn't

**What happened** [verified]: a session ran `/catch-up` in a project that is not a git
repository and never will be — the work lives as configuration files, application logs, a
database, and a hand-maintained wiki, with no version control anywhere in it. The first git
command the skill directs returned `fatal: not a git repository`.

That single failure invalidated most of the command. Of the skill's four numbered steps, step 2
(local state via `git status` / `git log`) and step 3 (companion-repo state) are entirely git,
and step 4's reconciliation instruction is phrased as "reconcile the resume note against what
git actually shows." Roughly two thirds of the command's text, including the whole of its
carefully-argued section on remote-state verification and negative claims, had nothing to
operate on. The skill's only stated contingency is for a missing resume note; it says nothing
about a missing repository.

The session improvised a substitute and it worked well: the running process list, file
modification times, and the timestamp on the application's own log file together established
what was true *right now* and, crucially, proved a claim the resume note made about work being
staged-but-not-yet-applied. That is the same job `git status` and `git log` do — establish
present state independently of what a note claims — using different evidence.

**The general shape.** The command's underlying discipline is sound and not actually
git-specific: distinguish state you can read directly and now from state you are quoting out of
a cache, and never let a cached value support a negative claim. Git is one instantiation of
that. A non-versioned project has the same two categories — process and filesystem state you
can query directly, versus a resume note that may be days stale — and the same trap, since a
resume note's "nothing is running" reads exactly like a fresh observation. But the skill
encodes the discipline only in git vocabulary, so a session in a non-git project must
re-derive the principle from the examples rather than being handed it.

Worth noting the asymmetry in effort this creates: the git path is spelled out to the level of
exact commands and two explicit reporting rules, while the non-git path is unwritten. The
quality of a catch-up in a non-versioned project therefore rests entirely on the session
noticing the analogy on its own.

**Rule concluded:** not yet promoted — logged. `catch-up` would be more robust if its opening
step established *what kind of state this project has* before assuming version control, with
git as the common case rather than the only one. The generalization to write down: identify
the authoritative present-state sources available in this project (git for versioned work;
process list, file mtimes, and application logs otherwise), read them directly, and treat the
resume note as a cache that loses to them on any disagreement. The negative-claim rule carries
over unchanged and is the part most worth stating outside git terms.
