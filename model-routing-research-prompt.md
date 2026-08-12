# Research brief: practitioner experience routing coding work across Claude's Sonnet, Opus, and Fable tiers

## Context

I'm designing a system that routes software-engineering tasks across three Claude model tiers
based on how much judgment a task needs and how expensive a mistake would be:

- **Sonnet** — default lane. Used when a fully-specified task (boilerplate, wiring, CRUD, mechanical
  edits) just needs correct execution — the spec fully determines the outcome.
- **Opus** — used when the outcome depends on judgment a spec can't fully capture (non-trivial
  algorithms, hard debugging, real design tradeoffs), but a wrong call is cheap to catch and fix.
- **Fable** — the most capable/expensive tier. Reserved for tasks that are *both* judgment-heavy
  *and* high-stakes: concurrency bugs, security-sensitive code, data migrations, wide-blast-radius
  refactors. Also does a mandatory read-only "second opinion" review of finished work before it
  ships.

This routing table was designed from first principles, not from field experience. I want it
pressure-tested against how people who actually use these models day-to-day for coding report
they behave — including where they diverge from what the tier positioning implies.

## Objective

Find **nuanced, hands-on user experience** — not benchmark leaderboards, not marketing copy, not
release-note feature lists — describing how Claude's Sonnet, Opus, and Fable tiers actually differ
in practice for software engineering / coding-agent work. I want the kind of detail that only
shows up after someone has used a model for real work and hit its edges: complaints, surprises,
"here's the thing nobody tells you," workflow write-ups, before/after comparisons.

**Scope constraint:** focus on the *current* Claude model generation (the "5" family — Sonnet 5,
Opus 5, Fable 5) wherever possible. Model behavior changes significantly between generations, and
a lot of existing discussion will be about older models (Sonnet 3.5, Sonnet 4, Opus 4, etc.) whose
quirks may no longer apply. You can include older-generation findings, but **label them clearly as
such** and note if the discussion suggests the behavior persisted into the current generation.

## Where to look

Lean fully into the access you have on X and Reddit that a generic web search doesn't — don't
just skim search-result snippets or top posts, pull actual thread and comment context. That depth
is the reason this brief is going to you first.

**X/Twitter — use your live search:**
- Prioritize the last ~60–90 days. Recency matters more here than in typical research: X is where
  day-one reactions to a model's real behavior show up, well before anyone writes it up elsewhere.
- Follow reply threads, not just top-level posts. The real nuance is usually in a reply — someone
  disagreeing, adding a caveat, or saying "actually in my experience it's the opposite."
- Weight by specificity and apparent track record where you can tell: a detailed, specific report
  from someone who's visibly building agentic coding workflows day-to-day is worth more than a
  one-line hype or hate post from an account with no visible history on the topic.
- Deliberately search for both enthusiastic and critical takes. Don't let positive/hype content
  dominate just because it's louder or more reposted — actively look for the skeptical replies
  underneath it.

**Reddit — go past the top post:**
- r/ClaudeAI, r/ClaudeCode, r/singularity (coding-agent threads only), r/LocalLLaMA if relevant.
- Search terms: "Opus vs Sonnet", "Claude Code subagent", "Fable model", "model routing", "which
  Claude model for coding", "Claude Code orchestrator/architect pattern".
- Pull comment threads, not just original posts — the most useful nuance is usually a reply that
  disagrees with or qualifies the top comment, not the top comment itself.
- Use upvotes and comment-reply patterns as a trust signal: a heavily-upvoted comment, or a claim
  multiple commenters independently converge on, is stronger evidence than a single buried reply
  with no engagement. Note this explicitly in the trust-level line for each finding.

**Secondary, if useful:** Hacker News threads/comments, GitHub issues/discussions on Claude Code
or similar agentic coding tools, engineering blog posts describing internal use of Claude for
coding.

Within that material, pull out anything addressing:

1. **Failure modes and quirks per model** — the type of thing I mean (not claims, just
   illustrating the shape): does a tier over-engineer or gold-plate simple tasks? deviate silently
   from a spec's stated interfaces/constraints? stub things out or claim completion prematurely?
   follow strict instructions reliably under a long/complex system prompt? call tools reliably and
   recover from tool errors, or get stuck in loops? degrade over long context? handle ambiguity by
   asking, guessing, or silently picking an interpretation?

2. **Where a higher tier genuinely earned its cost** — specific reported cases where stepping up a
   tier caught something a cheaper tier missed (or didn't, and the extra cost/latency wasn't worth
   it). Especially interested in concurrency bugs, security review, data-migration safety, and
   large refactors.

3. **Multi-model / multi-agent routing patterns other people use** — has anyone described an
   "architect delegates to cheaper implementer, escalates on judgment/stakes" pattern for Claude
   Code or similar tools? What heuristics do they use to decide when to escalate?

4. **Cost and latency tradeoffs in practice** — real reports of what it costs in time, money, or
   usage limits to lean on the top tier for coding work, and whether people felt it was worth it.

5. **Disagreements** — if different people report contradictory experiences with the same model,
   note both sides rather than picking one. That disagreement is itself signal.

## How to report back

Don't force everything into a polished summary or a tidy comparison table yet — prioritize
**fidelity over neatness**. I'll do the synthesis myself once I have the raw material, and a
premature summary risks smoothing over exactly the nuance I'm after. For each finding, give me:

- The claim, in the reporter's own words or a close paraphrase
- Which model/tier it's about, and which generation if stated or inferable
- Source: subreddit/account/site + a link if you have one + the date (or your best estimate)
- One line on how much to trust it — use the platform-native signals you gathered: upvotes or
  engagement counts if notable, whether multiple independent people converge on the same claim, or
  whether it's a single unconfirmed anecdote

Group loosely by the five topic areas above, but don't discard something interesting just because
it doesn't fit neatly into one — a raw, slightly messy list of well-sourced findings is more
useful to me than a clean summary I can't trace back to evidence.

## Ground rules

- Every finding should be traceable to a real source — no "it's generally understood that..."
  framing without something backing it.
- Prefer recency. If a source is more than ~4 months old, flag it as possibly describing outdated
  model behavior.
- If you can't find good material on one of the five topic areas, say so explicitly rather than
  padding with generic model-capability commentary you already know.
- Include weakly-sourced or single-anecdote findings if they're specific and interesting — just
  label them as weak. I'd rather see and discard a shaky claim myself than have it silently
  filtered out.
