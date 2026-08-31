### 2026-08-21 (evening, second sweep) — The advisor lane can't research, a README-sourced claim was stated before it was verified, and the delegation practice logged earlier today got its second confirmation

**Status: finding 1 verified** (the agent roster's tool declarations were read directly, and the
routing workaround happened in this session); **finding 2 verified** (the falsifying file was fetched
and read independently after the lane reported it); **finding 3 verified** (both delegations occurred
this session and both outputs were checked against primary sources before being relayed).

**Finding 1 — the restriction that makes the advisor trustworthy also makes it unusable for
research, and the roster has no lane that fills the gap.** The user asked for a top-tier-model review
of an external open-source project alongside the architect's evaluation of it. The advisor is the
roster's designated review lane and runs the right model, but it declares `tools: Read, Grep, Glob` —
no network access. It could not read the project under review. The architect routed around it by
spawning a general-purpose agent pinned to the same model, which had fetch access and read the
external source directly.

That workaround was correct and the result was the strongest delegated output of the session — but it
means the review ran outside the roster, without the advisor's framing or its advise-only guarantee.
The tool restriction is deliberate and load-bearing: it's what makes "never implements" structural
rather than instructional, and that property should not be relaxed. The gap is that the roster
assumes every review target is already on disk. Reviews of external prior art, competing tools, or
anything requiring a fetch have no home.

**Rule concluded:** the doctrine should name this explicitly rather than leaving the architect to
improvise. Either add a distinct research lane with network access and no write tools, or state in
the advisor's section that targets not on disk require a different lane and say which. Improvising
the routing worked once; it also silently dropped the advisor's own prompt framing, which is the
part the previous entry credited for the result.

**Finding 2 — a load-bearing claim was stated from a README, flagged as unverified, and then
falsified by the source.** The architect claimed the external project did not do model-tier routing —
that this was the plugin's distinct axis — based on one pass over its README. The claim was
explicitly labelled as README-only and not proof about the code, and the delegated lane was
specifically asked to check it. It checked, and it was wrong: tier routing is a first-class,
multi-provider feature there, confirmed afterward by the architect fetching the config file directly.

The process worked — the flag was honest and the check was requested and performed. But the claim had
already been stated to the user as a positioning conclusion, and was headed into a strategic decision
about the plugin's purpose. Labelling something unverified does not make it safe to assert when the
assertion is what someone will act on.

This is the third instance today of the same skew: overstating from partial evidence, always in the
direction that makes the narrative cleaner. The volunteered-context entry, the resume-note shorthand
entry, and now this. The earlier two were about relaying an unverified claim; this one adds that
*flagging* it is not sufficient mitigation.

**Rule concluded:** the existing verification rule covers relaying. Extend it: when a claim is
load-bearing for a decision, verify before stating it, not after. An "I only read X" caveat is
appropriate for an aside and inadequate for a conclusion — the caveat travels less far than the
claim does.

**Finding 3 — the advisor-prompt practice logged earlier today reached its second confirmation, which
was the stated promotion trigger.** The previous entry recorded a practice as worth promoting "if it
holds up on a second use": give the lane real access to the primary source, frame the task as
refuting a specific claim, state that full agreement is a valid outcome, and require it to separate
what it read from what it inferred.

Second use, different lane, different target: it overturned the centrepiece of the architect's
evaluation with file-level citations, corrected a second claim the architect had gotten directionally
right but overstated, and volunteered two additions the architect had missed — while explicitly
listing what it could not verify and declining to reason around those gaps. Both uses produced a
reversal the architect would not have reached alone, and in both cases the mechanism was source
access plus the read-versus-inferred requirement, not model tier.

**Rule concluded:** promote it. The four elements belong in the orchestration skill's advisor
section as the standard framing for a review dispatch. Note the honest limit alongside it: both uses
were reviews of *claims*, not of diffs, and the practice is unproven for the advisor's actual primary
job.

**Where it lives:** here only. Finding 1 belongs in `skills/orchestration/SKILL.md`'s advisor section
and possibly `agents/`. Finding 2 extends the Verification section's existing relaying rule. Finding
3 is the promotion this file's previous entry made conditional — it is now due.
