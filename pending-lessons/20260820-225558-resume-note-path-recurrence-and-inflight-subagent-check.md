### 2026-08-20 (third session) — The hardcoded resume-note path recurred, masked by a project pointer; and resume notes predict the fate of in-flight subagent output instead of recording the check

**Status: finding 1 verified** (both candidate paths checked directly this session; the competing-note
outcome was produced, not predicted); **finding 2 verified** (the artifact in question was re-checked
directly on disk this session — it existed and was empty).

**Finding 1 — recurrence of 2026-08-19's finding 1, with a new wrinkle: the failure is *masked* on any
project whose agent-instructions file names the note's path.** `/catch-up` ran on a project that
maintains its own resume note at a nested path under its docs tree, not at the plugin's hardcoded root
filename. The command's lookup would have missed it, exactly as the earlier entry predicted. It did not
miss it — but only because the project's own agent-instructions file (auto-loaded into context, nothing
to do with this plugin) opened with an explicit "read this file first" pointer to that path. The catch-up
succeeded on the strength of project-local instructions; the command's own discovery logic was never
exercised.

That masking is worth logging separately, because it means the bug is least visible exactly where the
plugin is most likely to be tested. A mature project tends to have *both* a custom resume note and a
pointer to it in its agent-instructions file — the pointer rescues `/catch-up` every time and the
hardcoded lookup never gets tested. The naked failure only appears on a project that has the custom note
but no pointer, where the command reports no context available while a current note sits a directory or
two away.

`/wrap-up` then completed the other half of the problem precisely as the earlier entry predicted: its
step 3 instructs writing the plugin's own resume filename at the project root, which on this project
means a second resume note competing with the one the project's own instructions point every reader at.
Worked around this session by writing the project's own note as the substantive one and the plugin's root
file as a pointer to it — a resolution the command body does not suggest and that a less suspicious
session would not have reached.

**Rule concluded:** reaffirms the 2026-08-19 rule and sharpens the discovery mechanism. Before assuming
its own filename, the discovery step should *read the project's agent-instructions file for an explicit
resume-note pointer* — that is the highest-signal source available, it is already in context at zero
cost, and it beats globbing conventional names. Two sessions of evidence now; this should stop being a
logged observation and become an actual edit to both command bodies.

**Finding 2 — a resume note predicted whether in-flight delegated work would be recoverable, instead of
recording the one-line check that settles it.** The prior session ended with a research subagent launched
and never reported. Its resume note handled that responsibly in one respect: it said the status was
unknown and told the next session to re-launch from scratch rather than trust any recovered findings. But
it also asserted the agent's output artifact "likely won't be reachable from a fresh session." That
assertion was wrong. The artifact was still on disk days later, readable in a single command, and
answered the question definitively — zero bytes, the agent had produced nothing at all. The correct
instruction (re-run) was arrived at through a guess that happened to point the same direction as the
facts.

The general shape: a resume note is written at the moment of *least* information about delegated work
still in flight, and the temptation at that moment is to write a prediction about recoverability. A
prediction is unfalsifiable to the next session, and it displaces the cheap check that would settle the
matter. It costs in both directions — here the guess was harmless because the agent had produced nothing,
but had the artifact held real findings, "assume it's unreachable, re-run from scratch" would have
silently discarded completed research and paid for it twice. Caveat for accuracy: the note in question
was authored by the project's own convention, not by `/wrap-up`. The gap applies to `/wrap-up` regardless,
because its step 3 offers no guidance on in-flight delegated work at all, and delegation is the plugin's
entire subject matter.

**Rule concluded:** `/wrap-up` step 3 should carry an explicit clause for subagent work still in flight at
wrap time — record the agent's identifier and the concrete location of its output, instruct the next
session to check that location first, and state what each outcome means (empty means it produced nothing,
re-run; non-empty means read it before deciding). Never predict whether the check will succeed. "Here is
where to look and what each result implies" is strictly better than "this probably won't be there."

**Where it lives:** here only. Finding 1 is a second occurrence of 2026-08-19's finding 1 and should raise
its priority for a direct fix in `commands/catch-up.md` and `commands/wrap-up.md`. Finding 2 is a new
clause for `commands/wrap-up.md` step 3.
