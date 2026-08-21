# Research topic: AI systems that review their own work

**Date:** 2026-08-20 · **Status:** findings recorded, not yet promoted to doctrine
**Question that prompted it:** the plugin now edits itself, reviewed only by models from the same
family. Connor asked how others have built self-reviewing AI designs and what pitfalls to avoid.
**Method:** direct web search + source fetches by the orchestrator session (not delegated). Sources
listed at the bottom with reliability labels — some are vendor marketing and are marked as such.

## Why this was worth checking

An advisor pass earlier the same day recommended killing a proposed "two models debate each change"
tier and replacing it with behavioral replay. That recommendation was reached from first principles
by a model in the same family as the one it was reviewing. Checking it against outside literature is
exactly the independence that internal review cannot supply.

**Headline: the literature supports the advisor's verdict, and sharpens it in one important way it
missed.**

## Findings

### 1. Self-correction without external feedback doesn't work — it degrades performance

The most direct result: LLMs asked to correct their own reasoning *without external feedback* often
make things worse, not better. Feedback quality is bounded by the same limitations that produced the
original error, so the loop has no new information to work with.

**Implication:** any "review" step that is just the model re-reading its own prose and thinking
harder is not a control. It is the failure mode with a process name attached. The control has to
inject something the generation step didn't have — execution, a test, an outside document, a fresh
context that never saw the reasoning.

### 2. Self-preference bias is real and mechanically explained

LLM evaluators score their own outputs higher than others' where humans rate them equal. Models can
recognize their own generations at non-trivial accuracy, and there is a measured **linear correlation
between self-recognition ability and strength of self-preference**.

**Implication — the single most actionable finding here:** never let the model that produced an
artifact be its reviewer. Cross-model review (Opus authors, Fable reviews) is materially better than
self-review for this specific reason. It does *not* make the review independent — same family,
heavily overlapping training — but the bias literature is specifically about reviewing *one's own
generations*, and that part is genuinely avoided. This is a real argument for keeping the advisor
lane distinct from the implementer lanes, beyond raw capability.

### 3. Multi-agent debate underperforms simpler baselines

Evaluations across nine benchmarks and five debate frameworks found current multi-agent debate (MAD)
methods **fail to consistently beat single-agent Chain-of-Thought or Self-Consistency**, despite
costing far more. Reported failure modes:

- **Over-aggressiveness** — debate converts *correct* answers into incorrect ones at a higher rate
  than the simpler baselines.
- Rigid adversarial role assignment (angel vs devil) performed *worst*, because the structure
  prevents productive continuation once the agents disagree.
- On many tasks MAD degrades to an inefficient resampling method.
- More rounds and more agents did not reliably help; extra inference compute was largely wasted.
- Separately: majority voting alone accounts for most of the gain usually attributed to debate.
- Adversarial or persuasive agents can *lower* system accuracy by 10-40% while *increasing* consensus
  on wrong answers by more than 30% — and more rounds did not mitigate this.

**Implication:** the proposal to have Opus and Fable argue each design decision was close to the exact
structure the literature finds worst — assigned opposing roles, open-ended disagreement, no
termination condition. Killing it was correct, and for stronger reasons than "it would feel like
theater."

### 4. But refutation is not debate — and refutation does work

The one source cutting against the above is a 2026 adversarial *stage-gated* method for defect
discovery: agent one flags a specific defect, agent two attempts to **refute it as a false
positive**, a third gates promotion, with progressively stricter criteria per stage. The authors
report it significantly reduces false positives while preserving detection. (Caveat: the specific
precision/recall numbers were not extractable from the PDF and are **unverified** here.)

The distinction that reconciles this with finding 3 is the most useful idea in this document:

| Structure | What is contested | Result |
|---|---|---|
| **Debate** | An open question with no ground truth ("which design is better?") | Over-aggressive, flips correct answers, expensive, inconclusive |
| **Refutation** | One specific falsifiable claim ("this finding is real" / "this fix prevents failure X") | Improves precision, terminates cleanly |

**This matches what actually happened in the session that prompted the research.** The advisor's real
win was not arguing about a design — it was refuting a specific claim ("PR #7 prevents the logged
failure") by reading the artifact against the concrete failure scenario and showing the fix
reproduced the bug it was written to prevent. That is refutation with a termination condition, and it
worked on the first pass.

**Implication:** don't ask the advisor to "review this diff." Ask it to **refute a stated claim**:
*"Claim: this edit prevents failure X described in lessons entry Y. Attempt to falsify it."* Same
cost, sharper target, defined stopping point.

### 5. Critics catch real bugs — and hallucinate bugs that mislead people

Trained code critics caught inserted bugs in a majority of cases, their critiques were preferred over
human ones in most comparisons, and they found hundreds of real errors in data previously rated
flawless. But critics also **hallucinate bugs that can mislead a human into breaking working code**,
and the measured best configuration was **human + critic**, which caught comparably many bugs while
hallucinating less than the critic alone.

**Implication:** Connor keeping merge authority is not conservatism or a temporary measure — it is
the empirically strongest configuration on record. Worth stating explicitly in the doctrine, because
the natural drift of a project like this is toward removing the human from the loop for speed.

### 6. Verification is easier than generation — and the gap widens with tools

There is a consistent measured asymmetry: models spot errors in existing content more reliably than
they avoid making those errors while generating. Crucially, **the gap grows when the verifier has
access to tooling the generator lacked** — for example a judge that can execute code versus a
generator that had to reason it out.

**Implication:** the advisor is currently read-only (Read/Grep/Glob, no execution). That is a
deliberate safety choice and should not change casually, but it caps how much of this asymmetry is
available. The cheap way to capture most of it without granting execution: **hand the advisor the
replay or probe transcript as evidence**, so it verifies against a behavioral artifact rather than
against prose. Worth an explicit experiment before deciding whether the advisor ever needs Bash.

### 7. False-positive economics decide whether the review survives contact with a human

From practitioner sources (vendor-authored, treat numbers as directional): the cost of one false
positive is seconds of attention; the cost of a thousand is that the reviewer learns to skip every
comment the tool produces. Untuned tools are reported at 15-30% low-value or incorrect comments. The
stated cause is that models are optimized for plausibility and helpfulness — asked to review, a model
will nearly always find *something* and phrase it with conviction, because saying "this is fine"
feels unhelpful.

**Implication:** the advisor prompt must make **"no findings" a first-class, expected outcome**, and
the review must stay narrow. A review lane that always returns something is one that will be ignored
within a month — and it would be ignored by the single reader whose attention the entire process
exists to protect.

## What this changes about the proposed process

Confirmed, no change needed: replay/behavioral evidence over prose reflection (1); advisor lane kept
separate from the authoring lane (2); no debate tier (3); Connor holds final say (5).

**Changed by this research:**

- **a.** Frame the advisor pass as **refutation of a specific stated claim**, not open review (4).
- **b.** State explicitly that **"no findings" is a valid and expected result** (7).
- **c.** Feed the advisor the **replay transcript as evidence**, and treat that — not more argument —
  as the way to widen the verification advantage (6).
- **d.** Record in the doctrine that **human-in-the-loop is the measured-best configuration** (5), so
  it is not optimized away later for speed.

## Not checked / open

- Whether *same-family* cross-model review (Opus to Fable) measurably reduces self-preference bias
  versus true self-review. The literature covers self-review and cross-vendor review; the same-family
  middle case is the one this plugin actually occupies, and no source addressing it was found.
- Whether an advisor with execution access outperforms one given a replay transcript, and at what cost.
- Refute-or-Promote's actual measured numbers (PDF extraction failed).
- Whether stage-gating (progressively stricter criteria) is worth it at single-developer scale, or is
  overhead designed for security-team throughput.

## Sources

Peer-reviewed / preprint:

- Huang et al., *Large Language Models Cannot Self-Correct Reasoning Yet* — https://arxiv.org/abs/2310.01798
- Panickssery et al., *LLM Evaluators Recognize and Favor Their Own Generations* (NeurIPS 2024) — https://arxiv.org/pdf/2404.13076
- *Quantifying and Mitigating Self-Preference Bias of LLM Judges* (2026) — https://arxiv.org/pdf/2604.22891
- McAleese et al., *LLM Critics Help Catch LLM Bugs* (CriticGPT) — https://arxiv.org/html/2407.00215v1
- *Multi-LLM-Agents Debate — Performance, Efficiency, and Scaling Challenges* (ICLR 2025 blogpost) — https://d2jud02ci9yv69.cloudfront.net/2025-04-28-mad-159/blog/mad/
- *Debate or Vote: Which Yields Better Decisions in Multi-Agent LLM Systems* — https://www.emergentmind.com/papers/2508.17536
- *When collaboration fails: persuasion driven adversarial influence in multi-agent LLM debate* — https://www.nature.com/articles/s41598-026-42705-7
- *Refute-or-Promote: Adversarial Stage-Gated Multi-Agent Review for High-Precision LLM-Assisted Defect Discovery* (2026) — https://arxiv.org/pdf/2604.19049
- *Shrinking the Generation-Verification Gap with Weak Verifiers* — https://arxiv.org/html/2506.18203v1

Practitioner / vendor (directional only — these have a product to sell):

- Sonar, *Best AI Code Review Tools* (2026) — https://www.sonarsource.com/resources/library/best-ai-code-review-tools/
- Sourcegraph, *AI Code Review in 2026* — https://sourcegraph.com/blog/ai-code-review

**Reliability note:** findings 1-6 rest on primary literature. Finding 7's *numbers* come from
vendor-authored posts and should be treated as directional; its *mechanism* (plausibility-optimized
models always find something to say) is independently supported by the critic-hallucination result in
finding 5.
