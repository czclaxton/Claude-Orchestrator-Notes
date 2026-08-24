# Behavioral steering — what the research and Anthropic's guidance actually say

Research reference, gathered 2026-08-24. Question that prompted it: *is a checklist of imperative
`when X → do Y` rules the right format for a persistent communication-style profile?* Answer: partly,
and three specifics of the draft were wrong.

**Provenance:** `[documented]` = official Anthropic docs or a cited paper, source given.
`[study]` = published research, with its limits stated. `[inferred]` = reasoning, not stated anywhere.
`[not found]` = searched and found nothing; not the same as "does not exist."

Companion file: `claude-code-capabilities.md` (what the harness can do). This file is about what to
*put* in those mechanisms.

---

## 1. The headline findings, ranked by how much they change a design

### 1.1 Conditional phrasing measurably underperforms plain statements ⭐

`[study]` AgentIF ([arXiv 2505.16944](https://arxiv.org/html/2505.16944v1)) is the only benchmark
found that isolates conditional constraints. Best model (o1-mini):

| How the constraint is presented | Success rate |
|---|---|
| Plain, unconditional | **80.8%** |
| Conditional (fires only when a stated trigger is met) | 66.1% |
| Implied by example only | 59.1% |

**~15-point penalty for conditional framing.** Diagnosis in the paper: two independent failure modes —
misjudging *whether the condition fired*, then failing to follow. Over 30% of failures were the first
kind. Conditional constraints are ~42.6% of real-world agentic instructions and are "usually
overlooked in existing instruction-following research."

**Consequence:** `when X → do Y` adds a step that can fail on its own. Use unconditional statements
for anything wanted nearly always. Reserve triggers for genuinely occasional behavior, and make the
trigger a property of **the input** (topic, question type, what was asked) — never of the output not
yet written. A trigger like *"when a reply would run past three paragraphs"* requires the model to
predict its own unwritten output, which has nothing observable to check.

### 1.2 Instruction count has no cliff — cost is smooth ⭐

`[study]` *Curse of Instructions* / ManyIFEval ([OpenReview](https://openreview.net/forum?id=R6q67CDBCH),
published as [When Instructions Multiply, EMNLP Findings 2025](https://aclanthology.org/2025.findings-emnlp.896/)):
joint adherence follows **P(all) ≈ p^N** — per-instruction rate raised to instruction count. A
logistic regression on count predicts performance within ~10% error, even for unseen combinations.

`[study]` IFScale ([arXiv 2507.11538](https://arxiv.org/abs/2507.11538), 20 models, 7 providers, up to
500 instructions): per-instruction accuracy holds ~98–100% through 100 instructions, degrades sharply
around 150–250, and the best models reach only ~69% at 500. Three decay shapes observed: threshold,
linear, exponential (model-dependent).

**Reconciling:** IFScale measures the *fraction* satisfied on easy orthogonal constraints. ManyIFEval
measures *all-or-nothing* joint satisfaction on harder ones. Both true.

**Consequence:** at ~12 rules, per-rule adherence is high but joint adherence is p^12 — at p=0.97
that's 69%; at p=0.99, 89%. Expect most rules honored most turns and **roughly one slipping on a
meaningful fraction of turns.** That is the realistic expectation; "all twelve, always" is not.

**Therefore a rule budget is not an adherence device.** There is no number to hit. Every added rule
costs a little; cutting rules you would not notice being broken buys more than any phrasing change.
A cap can still be justified as a *process* device — forcing the compile step to make real choices —
but it should not be sold as improving adherence.

`[documented]` Position bias is near zero at this scale. IFScale found primacy effects "start low at
minimal instruction densities…, peak around 150–200 instructions, then level off."

### 1.3 Anthropic's stated guidance, and where it cuts against a terse checklist ⭐

All `[documented]`:

- **Positive over negative.** "Tell Claude what to do instead of what not to do." Instead of *"Do not
  use markdown"* → *"Your response should be composed of smoothly flowing prose paragraphs."*
  ([Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices))
- **Rationale beats bare imperative.** *"NEVER use ellipses"* is weaker than *"Your response will be
  read aloud by a text-to-speech engine, so never use ellipses since it will not know how to
  pronounce them."* With the note: "Claude is smart enough to generalize from the explanation." Same
  source.
- **⭐ Prompt register leaks into output register.** "The formatting style used in your prompt may
  influence Claude's response style… try matching your prompt style to your desired output style as
  closely as possible. For example, removing markdown from your prompt can reduce the volume of
  markdown in the output." Same source. **A dense telegraphic bolded checklist asking for flowing
  plain prose is fighting itself.**
- **Specific and concrete.** "Use 2-space indentation" not "Format code properly"; "Run `npm test`
  before committing" not "Test your changes." Criterion: "concrete enough to verify."
  ([Memory docs](https://code.claude.com/docs/en/memory))
- **Length hurts.** "Target under 200 lines… Longer files consume more context and reduce adherence."
  Same source.
- **Conflict is resolved arbitrarily.** "If two rules contradict each other, Claude may pick one
  arbitrarily." Same source. This is the argument against having two mechanisms both carrying
  behavioral rules.
- **Structure helps scanning.** "Use markdown headers and bullets to group related instructions.
  Claude scans structure the same way readers do." Same source.
- **Dial back emphatic imperatives.** For Opus 4.5+: "CRITICAL: You MUST use this tool when…" now
  causes *over*triggering; plain "Use this tool when…" is better. Also: remove
  verification/self-check instructions carried from older prompts — they cause over-verification.
- **Tail reminder for long prompts.** Opus 5 guidance gives this pattern verbatim: "In a long system
  prompt, pair the instruction with a short reminder near the end:
  `<tone_preference>Keep outputs reasonably concise.</tone_preference>`" ~15 tokens.
  ([Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5))
- **Smallest high-signal set.** "The smallest possible set of high-signal tokens that maximize the
  likelihood of some desired outcome." ([Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))

⚠️ **Two Anthropic sources are in tension and nothing reconciles them.** The memory doc says bullets
and headers beat dense paragraphs for adherence; the best-practices doc says match your prompt's
register to the output you want. `[inferred]` Read: the first is about *scannability and grouping*,
the second about *voice and texture*. Both are satisfiable — headers and one rule per line, but each
rule written as a full plain sentence in the target voice, not a compressed fragment.

### 1.4 Rules vs. examples: combined wins; the split framing is wrong

`[documented]` Anthropic leads with examples for style: "Examples are one of the most reliable ways
to steer Claude's output format, tone, and structure… Include 3–5 examples for best results." Should
be relevant, diverse, and wrapped in `<example>` tags.

`[documented]` Opus 5 page, on communication style specifically: "explicitly describe what updates
should look like **and provide examples**. Positive examples of the communication style you want tend
to be more effective than instructions about what not to do."

⚠️ **Read that comparison precisely.** It says positive examples beat *negative instructions*. It does
**not** say examples beat positive instructions. Nothing found says that.

`[study, low–medium confidence]` *Show and Tell* ([arXiv 2511.13972](https://arxiv.org/html/2511.13972v1))
is the only controlled head-to-head located. Four conditions × 40 observations, Gemini 2.5 Pro,
two-turn code generation:

| Condition | Turn-1 token reduction | Turn-2 expansion |
|---|---|---|
| Instructions | −56% | +175 |
| Examples | −20% | +268 |
| **Combined** | **−70%** | **+126** |
| Control | — | +266 |

**Two findings.** Effects were **additive, not redundant** — combined beat both singles. And
**examples did not persist**: by turn 2 the Examples condition expanded essentially identically to
no-prompt Control. Instructions held discipline across a new request.

Limits, stated: one model (not Claude), code generation not prose, n=40 per cell, single-author
preprint, not peer-reviewed. Directly designed for this question, which is why it is recorded, but it
is one study and should not outweigh vendor guidance for the model actually in use.

`[study]` Examples have a ceiling for personal style. *Catch Me If You Can? Not Yet*
([arXiv 2509.14543](https://arxiv.org/html/2509.14543v1), 6 models, 400+ authors, ~40k generations per
model): few-shot beat zero-shot on every metric, but *more* examples produced "limited gains in
stylistic alignment," and models struggled specifically with "nuanced, informal expression."

**Consequence:** the honest reading is **hybrid**, and it is not a compromise — both the vendor
guidance and the one controlled study point at it. Rules carry persistence; examples carry
calibration. Examples are expensive (a worked example is 100–300 tokens), so spend them only on the
two or three behaviors that resist description. A compact contrast pair (three lines wrong, three
lines right) captures most of the calibration signal far cheaper than a full worked example.

### 1.5 Placement: the system prompt, not a memory file

`[documented]` **CLAUDE.md is not a system prompt.** Verbatim: "CLAUDE.md content is delivered as a
user message after the system prompt, not as part of the system prompt itself. Claude reads it and
tries to follow it, but **there's no guarantee of strict compliance**, especially for vague or
conflicting instructions." ([Memory docs](https://code.claude.com/docs/en/memory))

`[documented]` **Output styles modify the system prompt** and are documented for exactly this use:
"Output styles change how Claude responds, not what Claude knows… Use one when you keep re-prompting
for the same voice or format every turn." And: "For instructions about your project, conventions, or
codebase, use CLAUDE.md instead." ([Output styles](https://code.claude.com/docs/en/output-styles))

Mechanics that matter:
- Custom instructions are appended **to the end of the system prompt** — the recency-favored position.
- **Output styles self-reinforce:** "All output styles trigger reminders for Claude to adhere to the
  output style instructions during the conversation." **Nothing equivalent exists for CLAUDE.md.**
  This is the strongest single argument for the placement.
- Set `keep-coding-instructions: true` or you lose Claude Code's built-in software-engineering
  instructions. Default is `false`.
- Read once at session start; edits apply after `/clear` or a new session.
- ⚠️ **Do not reach subagents.** "Output styles apply to the main conversation only: a subagent runs
  its own system prompt." A fork is the exception — it inherits the parent's system prompt.
- A built-in **Concise** style exists (v2.1.237+) and already covers lead-with-the-result / skip
  preamble, at zero authoring cost. Worth trying first to see what remains to be written.

`[documented]` Survival through compaction: system prompt and output style are "unchanged; not part of
message history." Project CLAUDE.md and auto memory are "re-injected from disk." Instructions given
only in conversation do not survive. ([Context window](https://code.claude.com/docs/en/context-window))

### 1.6 Token cost is lower than it looks

`[documented]` Claude Code orders requests so the system prompt sits first, then project context, then
conversation, and `cache_read_input_tokens` bill at roughly 10% of the standard input rate.
([Prompt caching](https://code.claude.com/docs/en/prompt-caching))

`[inferred]` A stable ~600-token block is paid at full rate once per session and ~60 token-equivalents
per turn after. **But only if it is stable** — a block that varies per session or is edited
mid-session forfeits the cache.

### 1.7 Drift over a long session

`[study]` *Lost in the Middle* ([TACL 2024](https://aclanthology.org/2024.tacl-1.9/)): U-shaped curve —
best at the beginning and end of context, "significantly degrades" in the middle.

`[study]` *Instruction (In)Stability* ([arXiv 2402.10962](https://arxiv.org/abs/2402.10962)) measured
"significant instruction drift within eight rounds," attributed to attention decay. Its mitigation
(split-softmax) is an inference-time change and **is not available here**. Models tested were
LLaMA2-70B and GPT-3.5 — dated.

`[study, medium confidence]` Chroma's *Context Rot* found all 18 frontier models tested degrade as
input grows, even on simple tasks. Industry lab, not peer-reviewed.

`[documented, vendor claim, unaudited]` Anthropic states Opus 5's "instruction following, tool calling,
and reasoning stay consistent throughout" its 1M-token window. No independent evaluation found.

**Available mitigations:** the tail reminder (§1.3), and output styles' built-in adherence reminders
(§1.5). Both cheap.

---

## 2. What this changes about a draft

1. Make most rules **unconditional**. Keep triggers only for occasional behaviors, and only on
   observable properties of the input.
2. **Positive framing throughout**; add a rationale clause where the reason is not self-evident.
3. **Write it in the voice being asked for** — the profile's own register is a style signal.
4. **Hybrid:** rules for persistence + two or three compact contrast pairs for calibration.
5. **Tail reminder**, ~15 tokens, naming the one or two behaviors that matter most.
6. **Place it in an output style**, not a memory file. Set `keep-coding-instructions: true`.
7. **Cut rules you would not notice being broken.** This buys more than any phrasing change.
8. Avoid emphatic imperatives and self-verification instructions — both now backfire.

## 3. Open gaps — do not fill these from plausibility

- `[not found]` No head-to-head study of imperative trigger-action phrasing vs. descriptive prose for
  behavioral steering. Persona-prompting literature answers a different question (does role injection
  improve task performance) with mixed results.
- `[not found]` No benchmark of adherence decay for **style/tone** instruction sets. Every
  instruction-count benchmark uses mechanically verifiable constraints, because style is hard to
  auto-grade. IFScale's authors explicitly disclaim generalization beyond their task type.
- `[not found]` No evidence on whether numbering rules affects adherence versus unnumbered bullets.
- `[not found]` No published number for "how many style rules is too many." The p^N model implies
  there is no threshold, only smooth cost.
- Could not independently verify Opus 5's consistency-across-context-window claim.
- Curse-of-Instructions absolute figures come from a third-party summary (OpenReview PDF was behind a
  browser check). The p^N relationship is the durable finding; treat the percentages as indicative.

## 4. The thing no document can settle

There is **no eval harness for this**. All of the above is guidance and adjacent evidence, none of it
a measurement of *this* profile on *this* model. Behavioral steering is precisely the domain where
subjective impressions of improvement are unreliable.

The cheap remedy, if it ever matters enough: ten fixed prompts, responses saved before and after,
judged blind. An afternoon's work, and it would settle what no amount of documentation can.

---

## Sources

[Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ·
[Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) ·
[Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) ·
[Claude Code memory](https://code.claude.com/docs/en/memory) ·
[Output styles](https://code.claude.com/docs/en/output-styles) ·
[Context window](https://code.claude.com/docs/en/context-window) ·
[Prompt caching](https://code.claude.com/docs/en/prompt-caching) ·
[IFScale 2507.11538](https://arxiv.org/abs/2507.11538) ·
[Curse of Instructions](https://openreview.net/forum?id=R6q67CDBCH) ·
[When Instructions Multiply, EMNLP 2025](https://aclanthology.org/2025.findings-emnlp.896/) ·
[AgentIF 2505.16944](https://arxiv.org/html/2505.16944v1) ·
[Show and Tell 2511.13972](https://arxiv.org/html/2511.13972v1) ·
[Catch Me If You Can 2509.14543](https://arxiv.org/html/2509.14543v1) ·
[Lost in the Middle, TACL 2024](https://aclanthology.org/2024.tacl-1.9/) ·
[Instruction (In)Stability 2402.10962](https://arxiv.org/abs/2402.10962) ·
[Context Rot, Chroma](https://www.trychroma.com/research/context-rot)
