### 2026-08-30 — The always-on `CLAUDE.md` line loses silently to a harness-level ban on subagents, and nothing surfaces that routing is off

**Status: verified** (the absence of every implementation and advisor lane across the whole session
is a matter of record; the `CLAUDE.md` directive was written during this session's own setup step
and re-read afterward).

**Finding — the plugin's recommended opt-in is advisory text in a user message, and a harness
instruction forbidding subagent use overrides it without producing any signal.** The README tells
you to make the doctrine always-on by adding the architect paragraph to a project's `CLAUDE.md`:
delegate all implementation, never type code yourself, get an advisor review before reporting any
deliverable done. That line was added at the start of the session, exactly as documented.

The session then ran for several hours — research, iterative binary inspection, staging a file
tree, authoring a setup script, publishing a document deliverable, and a multi-round debugging
loop where two successive diagnoses were wrong before the third held. On the plugin's own routing
table that is a mix of routine and complex lane work with at least one clear advisor checkpoint at
the end. **Zero implementation lanes were invoked. The advisor was never consulted.** One
general-purpose agent ran, and only because the user had asked for one by name in their opening
message.

The cause was a harness-level system instruction — "do not call the Agent tool unless the user
requested it" — which sits above `CLAUDE.md` in precedence. That resolution is correct. The problem
is that it is completely silent. Nothing in the session indicated the routing table was
unreachable; the orchestration skill was loaded and available the entire time, and its doctrine
simply never applied. Cost discipline, lane selection and the mandatory end-of-deliverable review
all quietly evaporated, and the session read as a normal productive session throughout.

This is the same failure shape the README already warns about for model pins — "the pattern
degrades quietly rather than erroring" — but one layer up, and with no equivalent warning. The
model-pin case at least still delegates; this one removes delegation entirely while leaving every
outward sign of the plugin working.

**Rule concluded:** the doctrine should not assume its own reachability. The orchestration skill
wants a cheap precondition check — at minimum, notice when a deliverable completes without any
lane having been invoked and say so once, rather than reporting done as though the advisor review
had happened and simply found nothing. The README's "Requirements" section should also name this
alongside the model-pin caveat: a harness or policy restriction on subagents disables the entire
pattern, and the `CLAUDE.md` line will not tell you.

Worth flagging separately that the setup advice actively contributes to the blind spot. Adding the
paragraph feels like configuration — like a switch that is now on — when it is a request with no
guarantee of compliance. Anyone who adds it and then works normally has no way to tell whether
routing ever engaged.

**Where it lives:** here only. The precondition check belongs in the orchestration skill; the
caveat belongs in the README's "Requirements" section, next to the existing note about pinned
models falling back silently.
