### 2026-09-01 — The cost discipline governs a twentieth of spend; a workaround was documented as the only option without searching the extension layer; and the ladder went inert without anything noticing

**Status: all three findings verified.** Finding 1 is measured from a week of real transcript data
rather than estimated. Finding 2 is verified from the shipped documentation text and from finding
the mechanism it said did not exist. Finding 3 is verified by deliberate reproduction — a lane
pinned to a cheaper model was dispatched with no override and ran on the session model.

**Finding 1 — the routing doctrine's cost discipline optimises the smallest slice of spend.** The
orchestration skill's cost guidance is almost entirely about which lane receives which model, and
about keeping expensive-model token volume minimal by routing well. Measured over seven days of
ordinary use, counting input plus output tokens:

- the orchestrator session itself — the one the user types into — was **76.8%** of consumption;
- all four plugin lanes together were **5.95%**;
- a generic, non-plugin research agent was **16.7%**, across 113 dispatches.

So the routing table governs roughly one twentieth of what is spent, while the largest single
consumer is the thing the doctrine never mentions: the orchestrator's own context size and turn
count. The second largest is delegation that lands outside the ladder entirely and is therefore
invisible to the routing rules.

This was not discoverable from inside the doctrine. It surfaced only because the user asked an
unrelated question about model usage and the answer required measuring rather than reasoning.

**Rule concluded:** the cost section should state where tokens actually go before prescribing lane
routing, and should carry guidance for the orchestrator's own spend — context hygiene, turn count,
effort level — which it currently has none of. It should also acknowledge that delegation to a
generic agent bypasses the ladder completely, since that is where a large share of real delegation
lands.

**Finding 2 — a workaround was documented as the only durable option without the extension layer
having been searched.** The README told users whose subscription differs from the shipped default
that editing the relevant agent file would silently revert on the next update, and that forking the
marketplace was the only way to make such a change stick. It also stated plainly that no
per-project override was documented for plugin-shipped agents.

A supported mechanism exists: the plugin manifest can declare user-configurable options, the host
prompts for them, stores them outside the plugin directory, and substitutes them into plugin
content. It survives updates by design. It was found only after the user pushed back on the premise
— "shouldn't we just be able to configure this locally" — at which point it took minutes to locate.
The earlier search had covered the host's settings surface and the agent-definition surface, and had
stopped there.

"Fork it" is a strong claim about a platform's extension surface, and it shipped in user-facing
documentation.

**Rule concluded:** before documenting a workaround as the only option, search the host-config layer
and the plugin/extension layer separately, and say in the document which ones were checked. A
negative about extensibility is a negative claim like any other and needs the search behind it —
and unlike most, this one gets published to users.

**Finding 3 — the ladder went inert and nothing in the plugin noticed.** The plugin's central claim
is routing work across model tiers, expressed as a `model:` pin in each agent definition. The host
stopped honouring those pins; dispatched lanes silently inherited the session's model instead.

The failure is invisible from behaviour. Every lane still received its correct prompt, its correct
tool set, and its correct task, and returned a correctly formatted report. Only the model
underneath was wrong. Nothing errored, nothing warned, and no output looked different. It was found
by reading the per-turn model field in subagent transcript logs — a place nothing in the normal
workflow looks — and confirmed by deliberately dispatching a cheap-tier lane and observing it run
on the session model.

The plugin already carries a README warning that an unavailable pinned model degrades silently to
the session model. That is the same failure surface, it was anticipated in writing, and no
mechanism was ever built to observe it. Meanwhile the dispatch-time model parameter continued to
resolve correctly throughout, so a working alternative existed the whole time.

**Rule concluded:** a plugin whose value depends on a host feature should verify the effect, not the
declaration. The cheapest form costs nothing structural: have the orchestrator state the model it
expects when it dispatches, and have each lane report the model it is actually running as in its
report header. A mismatch then appears in the deliverable rather than only in logs nobody reads.
Second, the doctrine should stop assuming a declared pin binds — passing the model explicitly at
dispatch is independently resolved and is the more robust instruction regardless of whether the
pin is currently working.
