### 2026-09-01 — A search in the wrong place reads as an empty result; two sound defaults composed into a dead feature; and two completeness claims were made before the search behind them

**Status: all three findings verified.** Finding 1 was caught and corrected inside the same
session, with the corrected search producing a non-empty result from a different path. Finding 2 is
verified from the two commits that made the decisions and from measured usage spanning both. Finding
3 is verified from the session transcript — both claims were made, both were wrong, and the
correcting evidence is quoted below.

**Finding 1 — a search that returns nothing is indistinguishable from a search of the wrong
place.** Asked to explain why one model showed near-zero usage, the orchestrator scanned the
session transcript store with a glob covering `<store>/*/*.jsonl` and found zero subagent turns. The
natural reading is "subagents never ran," and that reading is wrong: subagent transcripts live one
level deeper, in a `<session-id>/subagents/` directory the glob never touched. A field named
`isSidechain` exists on every record in the parent transcript and is never `true` there, which
independently corroborated the false conclusion — a second signal that agreed only because it was
measuring the same absence.

Nothing about the empty result announced that it was empty for the wrong reason. The corrected glob
returned 260 files and roughly 4.7 million tokens of subagent activity. The gap between "no data"
and "no data here" was two characters of path.

This is the same shape as the already-standing rule that a scoped `ls-remote` answers only about the
ref it names, and it generalises past git: any filtered query returns exactly what was asked for,
and a filter that excludes the whole population is indistinguishable from a population of zero. The
tell is that a negative result arrived cheaply and confirmed what was already suspected.

**Rule concluded:** before reporting a negative from a search, state where the positive case would
have to live and confirm the search covered it. A zero result is only evidence when the query's
coverage has been established independently of the query. Where the store's layout is not known
firsthand, enumerate one level up and look at what is actually there before globbing.

**Finding 2 — two individually sound decisions composed into a feature nobody could reach, and
nothing checked the product.** A plugin routes work across three model tiers. Two changes were made
weeks apart, each correct in isolation: the top tier was documented as a rare, deliberate escalation
rather than a default lane, and the one high-frequency agent pinned to that tier was moved down a
tier so that users on the cheaper subscription would not be billed per deliverable. Both were right.
Together they left the top tier with no realistic invocation path — measured at three invocations
across every session on record, one of which carried an explicit override to a lower tier.

The user noticed only through an external usage dashboard reading zero. Neither change's review
asked what the other one had already removed, because each was evaluated against its own rationale.
The failure is not in either decision; it is that the decisions compose, and nothing in the process
looks at the composition.

**Rule concluded:** when a change removes a path to a capability, name what other paths remain and
count them. A tier, lane, or fallback whose remaining invocation count is zero or near-zero after
the change is a decision being made silently, and it should be stated in the change rather than
discovered later from usage data.

**Finding 3 — a completeness claim needs a search behind it, and so does a causal one; "I ruled out
four alternatives" is not "I looked everywhere."** Two instances in one session.

First: asked whether a setting could be overridden locally, the orchestrator ruled out four
mechanisms — a user-level definition file, editing a cache directory, a global environment variable,
and a per-item settings key — and reported the option set as complete. It was not. A fifth mechanism
existed, purpose-built for exactly this, and had never been looked at because the search had been
scoped to one half of the system. The user pushed back with the framing "shouldn't we just be able
to configure this," which was correct, and the mechanism was found within minutes of actually
looking. Ruling things out feels like searching and is not: four negatives establish nothing about
the size of the space.

Second: having found a behavioural regression correlated with a version boundary, the orchestrator
named the version as the cause before checking that a second component had also updated inside the
same window. The confound was real — the plugin shipping the affected file had itself been updated
between the last working run and the first failure. It happened to be eliminable (the relevant
declaration was byte-identical across four consecutive plugin versions spanning both windows), but
that was luck, not method: the claim had already been made before the check was run.

Both share a shape. A claim about what does not exist, or about what caused something, is a claim
about a space rather than about an item — and the evidence for it has to cover the space, not the
items already examined.

**Rule concluded:** before asserting an option set is complete, name the axes searched and say which
were not — "I checked the harness layer, not the plugin layer" is an honest and useful answer where
"these are the options" is not. Before naming a cause from a correlation, enumerate everything that
changed inside the same window and eliminate each explicitly. Where the elimination has not been
run, state the correlation and stop short of the cause.
