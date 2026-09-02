### 2026-09-02 — The mandatory advisor gate lives in a skill the plugin's own entry points never load

**What happened** [verified]: a session opened with `/catch-up` and ran end to end — three
substantial deliverables produced, each reported done. The advisor review that the doctrine
describes as mandatory ("ALWAYS once at the end of a deliverable, to review the accumulated
changes before the orchestrator reports done") never happened, and was never even considered.

The mechanism is worth stating precisely, because it is not the same as the previously logged
no-subagent conflict. In that earlier case the doctrine was in context — carried by the
project's `CLAUDE.md` — and collided with a session-level instruction not to spawn subagents.
The conflict was visible, was flagged, and was resolved by improvisation.

Here the doctrine was never in context at all. `/catch-up` does not load the orchestration
skill. The session-level instruction said not to use the Agent tool "unless the user, a
`CLAUDE.md` file, or a skill asks for it" — and since no skill asked, nothing did. The gate did
not lapse after being noticed; it never entered the picture. From inside the session there was
no observable signal that a mandatory step had been skipped.

**The general shape.** A rule described as unconditional is only unconditional within the
context that carries it. Putting the always-do-this gate in a skill that loads on demand makes
it conditional on the skill being loaded, which is precisely the thing an "always" rule is
supposed to be robust against. The plugin's own commands are the most likely entry points into
a session, and they are exactly the paths that do not load it.

**Secondary note:** this is a **second occurrence** of the doctrine-vs-no-subagents conflict
logged on 2026-09-01, which said it was worth watching for a repeat before acting. This is that
repeat, with a different mechanism — worth raising that entry's confidence rather than treating
this as an independent finding.

**Rule concluded:** not yet promoted — logged. Two candidate fixes, and they are not the same
shape. Either the entry-point commands load the orchestration doctrine themselves, so the gate
is always present; or the gate is restated in whatever context is guaranteed to be loaded and
stops depending on skill invocation. The failure mode to design against is the one seen here:
not a violated rule, but an unobserved one — a session that has no way to notice the obligation
exists.

---

### 2026-09-02 — `wrap-up`'s resume-note exclusion is a silent no-op once the file is tracked

**What happened** [verified — re-tested in-session]: `wrap-up` instructs that the resume note
is a process artifact and, in a git repository, should be excluded via `.git/info/exclude`
rather than `.gitignore`. The reasoning is sound and the file choice is right.

The mechanism does not work if the file is already tracked, which it was. Git applies exclude
rules only to untracked paths. Appending the filename to `.git/info/exclude` was re-tested
directly: the file remained in `git ls-files` afterwards and stays committed on every future
`git add -A`. No error, no warning — the instruction reports success and changes nothing.

Getting into that state is the normal path, not an unusual one. `catch-up` reads the resume
note at the project root and treats it as a first-class artifact of the project. A session that
creates one early — reasonably, so the next `catch-up` has something to read — will commit it
alongside real work, because nothing before `wrap-up` says it should be untracked. By the time
`wrap-up` gives the instruction, it is already too late for that instruction to do anything.

**The general shape.** An instruction whose effect depends on prior state, given at the point
where that state is usually already wrong, and which fails silently rather than reporting it.

**Rule concluded:** not yet promoted — logged, with a concrete fix available. `wrap-up` should
check tracked status first and `git rm --cached` the note before adding the exclude rule,
rather than assuming untracked. A cheaper partial fix is for `catch-up` to say the note is a
process artifact at the moment it first reads or creates one, so it never gets committed in the
first place — though that leaves existing repositories in the broken state.

---

### 2026-09-02 — No project-initialization path, and the user expected one

**What happened** [verified]: a session was opened with `/catch-up` against an empty directory
— no git repository, no files at all — with arguments asking to "initialize this project with
this plugin" and then start a research task.

`catch-up`'s defined fallback for a missing resume note is to say so plainly and ask the user
what to work on. That is the right behaviour for a *stale or unfamiliar* project. It is the
wrong behaviour for a *new* one, where the user has already said what they want and the answer
to "what should I work on" is in the arguments.

Nothing in the plugin defines what an initialized project looks like. `setup` creates command
aliases and is user-level, not project-level; `catch-up` and `wrap-up` both assume a project
already exists. So initialization was improvised: initialize the repository, create a notes
directory, and hand-write a resume note — which is `wrap-up`'s artifact, produced by a session
that had not run `wrap-up`.

The improvisation was fine and the session proceeded normally. What makes it worth logging is
that the user asked for initialization by name, unprompted. That is a signal the capability is
discoverable-by-expectation: someone reading the plugin's surface assumes a bootstrap exists
because `catch-up`/`wrap-up` imply a project shape, and there is no command that establishes
it.

**Rule concluded:** not yet promoted — logged, one occurrence, and the mildest of the three
entries here. Worth noting that a bootstrap command has an obvious second job: it is the
natural place to mark the resume note as a process artifact before it can ever be committed,
which is the cheap fix for the entry above. If both gaps want the same command, that is an
argument for building it rather than patching each separately.
