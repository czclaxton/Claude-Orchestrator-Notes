# Design brief — the "chief of staff" assistant

**Status: design conversation, not a spec. Nothing here is built. One question was left unanswered
when the session ended.**

Captured 2026-08-23 from session transcript
`~/.claude/projects/C--Repos-Fable-Planner/c29c71ef-bc83-4065-aaa7-b99738472514.jsonl`
(2026-08-21 21:48 → 2026-08-22 00:54). Until this file existed, the transcript was the only copy.

**Provenance tags used throughout**, because the distinction is load-bearing and easy to lose:

- **[confirmed]** — Connor explicitly agreed, in his own words.
- **[corrected]** — Connor pushed back and the position changed. His version is what's recorded.
- **[proposed]** — put to him and never ratified. The session ended mid-thread. Do not treat these
  as decisions; several are consequential.

---

## What the thing is

Connor asked for a tool that optimizes his use of agents: documents context, eases mental workload,
lets him focus on high-level guidance, and optimizes context and model usage. Eight interview
questions in, his answer reframed it:

> "Maybe I want to build a framework for how to manage data and info **along with how to relay
> information and get my high level decision and guidance**. I find I either lose track of what is
> being done to an extent or I have to get too into the weeds and guide things more than I want to."

**The reframe that followed:** this had been treated as a *memory system*. It isn't. That quote is
two halves, and the second half is the one that solves his problem. Memory is table stakes. What's
missing is a **communication protocol** — how work comes back to him, and how his steering goes in.

### The chief-of-staff analogy [confirmed]

> Not a note-taker. Someone who holds all your ongoing matters, briefs you in two sentences, brings
> you *decisions* rather than problems, escalates only what genuinely needs you, and — critically —
> gets better at all of that the longer they work with you. The relationship matures toward you
> doing less, not more.

Connor's confirmation, verbatim: *"Chief of staff and layered zoom: confirmed."*

**The analogy's purpose is to relocate the product category** — from storage to relationship. If it
holds, the spec is organized around the relationship and its protocol, with storage as a supporting
system rather than the center. That was stated explicitly as the reason for asking him to confirm it
before anything got written up.

### Layered zoom [confirmed]

Why he swings between losing track and being in the weeds: **there's only one zoom level available.**
A wall of technical text is *a map printed at street scale* — skim it and retain nothing, or read it
properly and now you're in the weeds. No country-level view to start from.

What he wants instead: plain-language headline → mechanism carried by analogy → *he* points at what
didn't land → expand only there. Same principle as memory tiers (load what's relevant, keep the rest
retrievable), applied to conversation instead of storage.

**Not a nicety.** It's the mechanism that makes hands-off possible: you can't delegate to something
whose reports you can't parse. Full detail on this as a standing preference lives in
`communication-preferences.md` §3.

### North star and quality metric

**North star, his words:** *"the minimum amount of thinking that is still effective and lets me
remain in control."* Both halves bind — minimum thinking alone is abdication, maximum control is the
weeds. It's measurable, and it should go down over time or the system isn't working.

**Quality metric [corrected — this was his, and it reorganized the design]:**

> "A system that takes 1000 mistakes to reach a baseline level is far worse than one that can do it
> in 20 or 50."

Restated: **how much tailoring do you get per unit of your attention?** Everything else is
downstream of that. Naming it forced a reversal — see "the primary mechanism" below.

---

## The four modules

| Module | What it is | Realistic? | Standalone? |
|---|---|---|---|
| **Routing** | Model/cost selection | Already built — it's Claude Orchestrator | Yes |
| **Translation** | Layered explanations tuned to him | Very | Yes, completely |
| **Continuity** | The thread record | Yes | Yes, and everything else needs it |
| **Delegation** | Decision boundaries + precedents | Rudimentary only | No — needs Continuity |

**Proposed build order [proposed]:** Routing (done) → Translation → Continuity → Delegation.
Ordered by risk as well as dependency: each is useful alone, so stopping after any one still leaves
something usable daily.

**Translation first** because it's the smallest, has no hard dependencies, changes the very next
response, and makes the other modules legible — you can't judge whether the delegation boundary is
right if you can't parse what came back.

**On Translation being "too idiosyncratic to generalize" — his worry, answered:** he isn't
generalizing. It's built for exactly one person who is sitting right there and can correct it live.
The idiosyncrasy is what makes it *tractable*. It's a small accumulating preference record, not a
research problem.

**On Delegation's ceiling, stated bluntly:** it will not autonomously get smarter. It accumulates
precedents he periodically reviews and ratifies. Deliberate tuning with his hand on it.

**Soft dependencies of Translation, initially overstated as "none":** Continuity would keep a
preference learned today true next month; the intervention log sharpens it fastest. It works alone
but improves fastest last.

**Continuity, in plain terms:** the part that remembers *the work*, not the conversation. Like a
medical chart — not a transcript of every appointment, but what's being treated, what's been tried
and failed, what to check next time, maintained deliberately at the end of each visit.
**[corrected] Connor's own reframe was sharper:** it's state management with a curated hydration
step at session open. Two things distinguish it from ordinary hydration — the snapshot is *lossy on
purpose* (curation is the feature), and some state is *computed, not stored* (anything readable from
the live artifact gets re-derived at hydration, which keeps stored state small and stops it going
stale).

---

## The self-improvement engine

### Interview → Capture → Compile → Apply [proposed]

Connor's closing definition, which became the module names:

> "ask the questions correctly, document properly, translate that into actionable changes or ideas,
> then actually implement them."

**The primary mechanism is interviewing, not observation — this was a reversal.** The earlier
position made interventions the main input. That's correction-driven learning, and correction-driven
learning is exactly the 1000-mistake shape: it only learns from what already went wrong, one
instance at a time.

**Asking is enormously cheaper than observing.** One good question extracts in ten seconds what
fifty observations would establish statistically — and it comes with the *reason*, which generalizes
to cases not yet hit. Observation only ever tells you what happened.

**The concrete design consequence:** when the system notices it's guessing, it should **ask rather
than guess and wait to be corrected.** That single behavior is most of the difference between 20
mistakes and 1000.

So: **interviewing is primary; interventions are a secondary safety net** that catches what the
interview missed; **compile** turns both into rules.

### Capture: track interventions, not decisions [confirmed direction]

Connor asked how you'd ever know what "the decision" was in the moment. The answer: you don't track
decisions at all — you track **the moments he had to speak up.**

*You don't need a camera in every room to find the leak. You just need to notice where the floor is
wet.*

Decisions are invisible and there are thousands per session. Interventions are **self-marking**,
because he generates them by typing: he corrected it, answered a question it asked, stopped it, or
re-explained something it should have known. The capture criterion isn't "which decisions matter"
(unanswerable) but **"did the human have to push?"** (trivially detectable).

One question at session close catches the opposite direction: *"did I ask you anything I shouldn't
have needed to?"* — that's how it learns to bother him less, not just to be corrected more.

### Verdicts [corrected — Connor's catch]

"Log every intervention and drive it to zero" is wrong, because some interventions are healthy — he
*wanted* in. Each one needs a verdict:

- **Failure** — it should have handled this, or should have asked and didn't. *Change something.*
- **Acceptable** — reasonable that he was involved. *No change.*
- **Elective** — he engaged though he didn't need to. Curiosity, interest, or he felt like it.
  *No change, and specifically don't learn from this.*

**Separating Elective out is load-bearing.** Without it the system reads his curiosity as failure and
starts hiding things to avoid triggering it — optimizing toward exactly the opacity he's escaping.

**Why this is nearly free:** he already does this review. He accepts or reverts regardless. The only
thing missing is capturing the outcome, which costs one line.

---

## Rules, and how they're stored

### The case-law analogy, and why it was replaced [corrected — Connor broke it]

"Case law" was originally used as shorthand for how the decision boundary should form: not a rule
book written in advance, but precedents accumulated from actual moments. *Mar 3 — stopped it before
it refactored auth. Verdict: correct, I want eyes on auth.*

**Connor's objection, which held:**

> "that analogy doesn't cover the side of how case-law is actually used and applied. How often does
> it set automatic precedent versus the need of lawyers to be clever or smart enough to dig it up and
> apply it? To me it doesn't seem context efficient or enforceable without having it be looked up
> every time."

Correct. Precedent in real law needs a lawyer to *find* it and argue it — retrieval cost on every
decision, dependent on someone being clever enough to dig up the right case. **The analogy fails at
exactly the operational moment.**

**The replacement is a pilot's checklist**, and the fix was a missing step, not a better metaphor.
Checklists are *compiled from* accident reports. No pilot reads accident reports in the cockpit.
Precedents are raw material, not what gets consulted.

| | The rules | The case log |
|---|---|---|
| **Size** | Small, capped | Grows forever |
| **When read** | Every session, always loaded | Only during revision |
| **Cost** | Fixed and tiny | Zero at decision time |
| **Purpose** | Bind behavior | Justify and revise the rules |

That kills the retrieval problem entirely — nothing is ever looked up mid-task — and it introduced
the **compile** step the earlier design was missing.

### What makes a rule a rule [proposed]

**The test: could a stranger follow it without exercising judgment?**

- *"Be careful with migrations"* — fails. "Careful" means nothing operationally.
- *"Never run a migration without showing me the SQL first"* — passes.

The enforceable form is always **when X, do Y** — observable trigger, specific action. If it can't be
written that way it isn't a rule yet; it's a feeling, and it belongs in a separate notes pile.

**Two enforcement tiers, and knowing which one a rule is in matters:**

- **Soft rules** — in the file, consulted at decision time. Followed reliably, not guaranteed.
  Reading isn't enforcement.
- **Hard rules** — enforced by Claude Code hooks, which can block a tool call before it runs.
  Guaranteed, but only available for things expressible as "don't run that command."

### Conflicting feedback is the most valuable signal [proposed]

Connor asked what happens when feedback conflicts due to context nuance. This is what made compile
non-optional.

> *"Don't ask me about renames."* vs. *"You should have asked before renaming that public API."*

Not a contradiction — **a rule with a missing condition**, and the conflict is what exposed the
boundary: *don't ask about internal renames; do ask about anything with external callers.* That rule
couldn't have been written up front, because the line wasn't known until two cases straddled it.
**Conflict is how the boundary reveals itself.** It needs a step where someone notices and resolves
it — a human with judgment, not an append-only file.

The other resolution is simpler: he changed his mind. Newer wins, older is marked superseded rather
than deleted, so the reversal stays visible.

### Preventing the mess structurally [proposed]

He asked whether this gets messy at scale. Answer: prevent it structurally, not with discipline.

- **The ruleset gets a hard budget** — say thirty rules. At the cap, compile *must* consolidate or
  retire something before adding. Forces the quality bar up as the system matures.
- **Rules are retirable on evidence** — log when each rule actually fires. One that hasn't fired in
  months is visibly dead weight rather than a guess.

### Layers [corrected, then sharpened]

Connor proposed user → category → task → one-off, then said he wasn't sure what "layering" even
meant because he didn't know what data was in a layer. That was a fair hit — it had been hand-waved.

**A layer contains rules and nothing else, all of one shape: `when X, do Y`.** Not prose, not
general preferences, not context. Once that's true, layering is just **resolution order**: two rules
fire on the same trigger, the more specific layer wins, and the system says which one it used.

**The spaghetti risk isn't layering — it's layers holding different kinds of content**, which then
need different merge logic per kind. Homogeneous layers can't produce that.

**The discipline that keeps it debuggable:** every rule that fires must be able to say where it came
from. *"I'm asking because of the finance-thread rule, not your default."* With provenance, four
layers stay debuggable; without it, two layers become mud.

**On the category layer** — initially proposed as "cut it, let categories emerge from evidence."
Connor pushed back that unbounded emergence has no design for overlap. **The reconciled version:**
emergence is *capped and deliberate* — depth capped at three, and a rule only moves up a layer when
he approves it during compile. Categories can't quietly proliferate; every one is a decision he made.

---

## Build tailored, not generic [proposed, with a stated seam]

Connor distrusted "build it generic, you're just the first user." The steel-manned case against
generic:

- **Every preference becomes a setting.** Tailored, a preference is a line in a file. Generic, it
  needs a default, docs, validation, and a migration path.
- **Cold start becomes a required feature** he would never once use.
- **You lose your assumptions** — technical user, Claude Code, terminal, these domains.
- **"Done" stops existing.** A personal tool is done when it works for him. A generic tool is never
  done, and the daily-use tool he wanted stays at 70%.

**Verdict: build tailored, keep exactly one seam clean — engine vs. profile.** The engine is prompts,
commands, the loop. The profile is preferences, rules, precedents, threads — kept as data in its own
files, never baked into the engine's prompts.

**Be clear-eyed about what the seam preserves: his data, not his code.** Generalizing later means
rewriting the prompts anyway.

**One correction on the reasoning [corrected]:** an earlier framing said "the nuance is the product."
Connor disagreed — he viewed the framework and structure as the product, with the data as what it
produces. He was right. **The framework is the product; the profile is its output.** The narrower
defensible claim is only that a framework built for one known person can afford to assume and infer
more aggressively than one built for strangers.

**Privacy consequence:** the profile — preferences, decision precedents, intervention history — is
collectively a detailed map of how he thinks and what he works on. It should be private regardless
of what happens to the rest.

---

## What Connor rejected or corrected

Recorded so none of it gets rebuilt by a later session:

1. **The "custodial thread" second architecture.** His dog's vet records and medications were turned
   into a second thread type with its own session loop. He said this over-thought an example. The
   point was the opposite: *the same* process handles software and vet records, because the process
   is about managing context and communicating well, not about the subject. **Dropped.**
2. **"Drive interventions to zero."** Some are healthy. Produced the three-verdict scheme above.
3. **The case-law analogy.** Broken on retrieval cost. Replaced by the checklist/case-log split.
4. **"The nuance is the product."** Framework is the product, profile is its output.
5. **Unbounded category emergence.** Capped at depth three, promotion only on his approval.
6. **A PR-and-summary-doc workflow for this repo.** Named a side quest — workshop tidying, not the
   product. The kernel worth keeping: a maintained summary of open work *is* Continuity applied to
   this repo, so build Continuity and point it here rather than building a separate PR system.

---

## Analogy glossary

Kept because the analogies are how this design is meant to be explained, and because their
*breaking points* are as important as their mappings.

| Analogy | Explains | Where it breaks |
|---|---|---|
| **Chief of staff** | The whole product — a relationship with a protocol, not storage | Untested; the chief of staff has real autonomy and judgment this won't have |
| **Map at street scale** | Why a wall of text fails him | — |
| **Pilot's checklist compiled from accident reports** | Rules vs. case log; the compile step | — |
| **Two doctors with the same lab results** | Translation — one hands over the printout, one says "your iron's low, that's why you're tired, want to see the numbers?" Same information, same accuracy; one is actionable | — |
| **Medical chart** | Continuity — curated, maintained deliberately, not a transcript | Superseded by his own sharper framing: lossy-on-purpose state hydration |
| **House rules vs. room rules** | Layered profile scope | — |
| **Wet floor, not a camera in every room** | Capture interventions, not decisions | Silent wrong actions leave no wet floor — this is the open question below |
| **Case law** | *Retired.* Was: precedent accumulating from real moments | Retrieval cost — needs someone clever enough to dig up the right case |

---

## Open questions — genuinely unanswered

**1. The one the session ended on, never answered.** Asking beats observing — but asking spends the
exact thing this product exists to conserve, his attention. A system that interviews him constantly
is the mental load he's running from, wearing a helpful face.

> **When is being asked worth it, and when is it an imposition?**

The instinct offered was that a dedicated compile session — batched, occasional, deliberate — is
welcome, while questions interrupting live work are not, even good ones. **Not confirmed.** It sets
how aggressively the whole system may interview him, so it gates the Interview module.

**2. The invisible-failure gap.** If the signal is "did Connor have to push," an assistant that
quietly does the wrong thing in a way he doesn't notice generates no intervention, and the system
never learns it. Put to him as *"does that gap worry you, or is 'if I didn't notice, it didn't
matter' good enough?"* — he engaged the surrounding material but never answered this directly.

**3. Whether this extends Claude Orchestrator or is its own thing.** His words: *"It could be an
extension of claude orchestrator if it works out that way or its own thing, nothing is off the
table."* Still open, and it bears directly on the scope discussion in `RESUME-PROMPT.md`.

**4. Model/cost routing's home.** Whether routing lives in this tool or stays in the orchestrator was
called a wiring question, not a hard problem. Undecided.

---

## Bearing on Claude Orchestrator as it exists today

One structural critique of the shipped plugin came out of this conversation and is logged separately
in `lessons.md` (2026-08-21 entry): **the wrap-up notes have a write path but no read path.** Notes
are written at session end; nothing consults them at the moment a decision is being made. A record
that isn't read at decision time is a diary.

Three things separate a diary from a system that learns: it's consulted at the moment of the
decision, its entries are specific enough to bind behavior, and its entries carry verdicts. None of
that needs a smarter model — it's loop design.
