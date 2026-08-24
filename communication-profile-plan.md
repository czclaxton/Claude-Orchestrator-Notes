# PLAN — making communication preferences load automatically, and closing a learning loop

**Written by the orchestrator 2026-08-24; brought current at end of session the same day. Nothing
here is installed — no file exists under `~/.claude/`.**

⚠️ **Read this warning first.** This document was written mid-session and then had to be corrected
twice, because decisions kept being made in conversation after it was written. That is a real hazard
logged in `lessons.md`, not a hypothetical — a review dispatched against an earlier version of this
file produced a confident false finding.

**Reading order that avoids the trap:** §2 and **§2b** are current. §3, §5 and §7 were written
earlier and have been annotated where §2b overrides them — strikethrough means dead, not merely
downgraded. §6 corrects a statement of my own. If §2b and anything below disagree, §2b wins.

⚠️ **Two files this document cites are NOT on `master`.** `communication-preferences.md` and
`assistant-design-brief.md` exist only on PR #14's branch,
`lessons/20260824-192100-agent-contradictions-and-stale-artifacts`. They are not in the working tree.
To read one without merging:

```
git fetch origin lessons/20260824-192100-agent-contradictions-and-stale-artifacts
git cat-file -p FETCH_HEAD:communication-preferences.md
```

That PR bundles them with three lessons entries — 1078 lines across 5 files — which is why it has
not been triaged. Splitting it is pending the user's decision.

---

## 1. The problem

The user (Connor) has a well-developed set of communication preferences, captured in
`communication-preferences.md` (317 lines, 11 sections). It is accurate and well-sourced.

**It is loaded by nothing.** Verified on his machine 2026-08-24: no `~/.claude/CLAUDE.md`, no
`~/.claude/rules/`, no `~/.claude/output-styles/`, no `outputStyle` setting. The project memory
directory holds 5 files, newest modified 2026-08-19 — the 2026-08-21 session that produced the most
important preference (layered explanation, the chief-of-staff confirmation) never wrote one.

Three of eleven sections appear in the auto-loaded memory index. The section the file itself calls
most important appears in zero runtime-loaded surfaces.

**Second problem, same shape:** the plugin's own feedback log (`lessons.md`) has 13 entries and ~40
findings. Every entry from 2026-08-20 onward closes with "Where it lives: here only." 13 open PRs in
the notes repo, zero merged. Promotion of a finding into plugin text currently happens only when
Connor personally asks for it. There is no other trigger.

---

## 2. What is DECIDED (do not re-litigate)

- **Compile the preferences into a small always-loaded rules block; keep the long file as a reference
  read only during revision.** This is the checklist / accident-report split from
  `assistant-design-brief.md`. Connor's words this session: *"I agree."*
- ~~**A rule must be `when X → do Y`**~~ — **DOWNGRADED 2026-08-24, this was my error.** The design
  brief tags "What makes a rule a rule" `[proposed]` at line 209, not confirmed. Connor agreed this
  session to the *checklist shape* ("I agree") while explicitly questioning the *format* ("is a
  checklist the optimal approach?"). Treating the `when X → do Y` form as decided would suppress the
  research in §5 open question 2. **Open, not decided.**
- **The plugin ships a generator, not the file.** A plugin cannot ship a `CLAUDE.md` (documented).
  It can ship a command that *writes* a rules file into the user's own space. Connor accepted this:
  *"As long as it results in the intended functionality then I have no issue."* This also enforces
  the engine/profile seam the design brief already decided on.
- ~~**Interviewing is primary; observation is a secondary safety net.**~~ — **DOWNGRADED 2026-08-24,
  this was my error.** The design brief tags "Interview → Capture → Compile → Apply" `[proposed]` at
  line 119. Only "Capture: track interventions, not decisions" carries `[confirmed direction]`
  (line 142). The interviewing-primary reversal was never ratified by Connor, and it is entangled
  with §5 open question 3, which is also unanswered. **Open, not decided.**

### 2b. Decided LATER the same session — these supersede parts of §3 and §5 below

Everything here was settled after §3–§5 were written. **Where they disagree, this wins.**

- **Placement: a custom output style at `~/.claude/output-styles/`, `keep-coding-instructions: true`.
  This reverses my `~/.claude/rules/` recommendation in §5.1, which is now dead.** Documented
  grounds: CLAUDE.md-class files are delivered as a user message *after* the system prompt with "no
  guarantee of strict compliance"; output styles modify the system prompt itself, sit at the
  recency-favoured end, self-reinforce via built-in adherence reminders, and survive compaction.
- **Format: hybrid — rules plus a few contrast pairs. This answers §5.2.** Unconditional statements
  rather than `when X → do Y` (~15 points worse on the one benchmark isolating it). Positive framing.
  Rationale clause where the reason isn't obvious. Written in the target voice, because prompt
  register leaks into output register. A ~15-token tail reminder. Evidence in
  `prompt-steering-findings.md`.
- **No rule budget. This retires the 12-rule cap.** Adherence cost is smooth (p^N), so a cap buys no
  adherence. It survives only as a process device to force consolidation, and must not be sold as
  more than that.
- **The decision protocol — the session's most load-bearing outcome, and mostly Connor's design.**
  Default to making reversible calls and reporting them; stop and pitch when undoing would cost him
  more than a minute, when it locks in a direction, or when only he can answer. A pitch carries the
  decision in plain terms, a recommendation, and the strongest case against it, so "go" is usually
  the whole reply. **Bring decisions, not problems.** Nothing genuinely open gets *quietly* closed —
  announced calls are fine, silent ones are not.
- **This answers §5.3 and unblocks R6.** The volume control is the pitch shape itself: cost per ask
  drops far enough that asking stops being the load he is escaping.
- **Bluntness.** No softening, no hedging, never a yes-man. Closes a gap
  `communication-preferences.md` lists under "do not invent answers to these."
- **A dichotomy is a negative claim.** "It's A or B" asserts "there is no C," so the
  negatives-need-a-search rule applies to option spaces. Name what you considered and discarded, and
  say whether you searched or only thought. Connor's point, and it is the preventive fix for what old
  rule 10 only patched after the fact.
- **Old rule 10 is cut to its residual.** "Offer a third framing when you disagree" only fires after
  the failure has happened; his objection was that the bug is in the first evaluation. Only "engage
  directly when he says he is confused about framing" survives.

## 3. What is PROPOSED and NOT approved

- **The rules themselves**, now in `draft-communication-output-style.md` — not `DRAFT-rules.md`,
  which was session scratch and no longer exists. Advisor-reviewed once, three findings applied.
  **Connor has not reviewed them line by line.** Open to cuts.
- **His gut-feel pick of which matter most — 1, 4, 6, 7.** Stated as feeling, not severity, and
  explicitly a hypothesis to test, **not a cut list.** Ship all of them and cut on evidence.
- ~~**R6 is explicitly blocked**~~ — **unblocked, see §2b.**

## 4. What was ESTABLISHED this session (research, verified)

Full detail in `claude-code-capabilities.md`. The load-bearing facts:

- `~/.claude/rules/*.md` exists, loads unconditionally, user-global. Purpose-named for this.
- Output styles modify the **system prompt** (stronger placement than CLAUDE.md), are documented for
  *"role, tone, or default response format every turn"*, and **can be shipped by a plugin** with
  `force-for-plugin: true`.
- Auto memory is a real harness feature — first 200 lines of `MEMORY.md` load every session — but is
  **per-project and machine-local**, so it cannot carry global preferences.
- **No hook receives the assistant's completed message.** `MessageDisplay` sees streaming text;
  `Stop` fires but is not given content. The harness cannot grade assistant output.
- **`UserPromptSubmit` can read the user's prompt text in full.**

## 5. OPEN QUESTIONS — not answered, do not assume

1. ~~**Mechanism: `~/.claude/rules/` or an output style?**~~ **ANSWERED — output style. See §2b.**
2. ~~**Is a checklist of imperative rules the right format?**~~ **ANSWERED — hybrid. See §2b.**
3. ~~**When is being asked worth it?**~~ **ANSWERED by the decision protocol in §2b.** The nuance
   that resolved it: the variable is not volume but *category*. Don't ask what you could find out;
   don't ask what's already decided; do ask what only he knows. And "ask or guess" was itself a false
   dichotomy — the third move is proceed under a stated assumption.
4. **Does this extend Claude Orchestrator or become its own tool?** Still open. Bears on where the
   generator command lives.
5. **What triggers a compile session?** Still open, and it is the deciding risk — see §7 Problem 1.
   The advisor's proposed fix: `UserPromptSubmit` can inject `additionalContext`, so the hook that
   counts escalations can also say "compile is overdue." **Do not ship the generator without it.**
6. **Does `~/.claude/rules/*.md` reach subagents?** Unestablished anywhere. Output styles are
   documented as *not* reaching them. Matters because lane output reaches the user through the
   orchestrator's relay.
7. **Which mechanism owns behavioural rules?** Project auto-memory already holds some
   (`feedback_one_question_at_a_time`, `feedback_verify_dont_assume`). An output style makes a third
   mechanism, and contradictory rules are documented as resolved arbitrarily. Decide deliberately.
8. **`communication-preferences.md` §1 is stale and nobody has fixed it.** It says "never a numbered
   batch"; Connor superseded that on 2026-08-24 — his driver was the cost of composing a structured
   reply, not topic count, so several items in one message are fine when answering is cheap, framed
   as a trial. **A review has already produced a false finding from the stale text.** Updating it is
   the first real instance of the missing compile step.

## 6. ⚠️ SUPERSEDED — a correction to my own earlier statement

Earlier this session I told Connor "no hook can read what you say back to me" and framed detection as
broadly unavailable. **That was an overstatement and he should not act on the earlier version.**

Corrected: the *assistant's* output cannot be read by any hook. The *user's* prompts can be read in
full by `UserPromptSubmit`. Since the design brief's capture signal is **interventions — the moments
Connor had to push back — and those arrive by his keyboard**, the primary capture mechanism is in
fact available.

What survives / what breaks:

| Wanted | Status |
|---|---|
| Count interventions; trend "minimum thinking" down | Works — reads his prompts |
| Feed the compile step real input | Works |
| Check whether a specific rule was followed on a given message | Broken live; needs a human or a later transcript pass |
| Retire rules on evidence of never firing | Broken — nothing can observe a rule firing |

**Fallback for the broken half:** transcripts are on disk as `.jsonl`. A batch agent can grade a
finished session. That is a job to build, not a harness feature. Nothing blocks it.

---

## 7. Known problems with this plan — my own list, for you to extend or dispute

**Problem 1 — no compile trigger.** Aviation checklists work because someone periodically compiles
accident reports into them. We have the checklist and the reports and nothing in between. If we
install the rules with no trigger for revision, the block goes stale exactly as `RESUME-PROMPT.md`
went stale and as the "where it lives" ledger went stale — and that ledger is provably wrong today
(the 2026-08-20 git rule says "here only"; it was in fact shipped as plugin PR #7).

**Problem 2 — R6 has no volume control.** Gated on open question 3.

**Problem 3 — rules cannot announce which one fired.** The design brief requires provenance
(*"I'm asking because of the finance-thread rule, not your default"*) and requires retiring rules on
evidence. An always-on text block provides neither. Without it we cannot distinguish a rule working
from the model behaving that way anyway.

**Problem 4 — layer/scope undecided.** The rules are global; the plugin is project-shaped and cannot
ship a CLAUDE.md at all. Where the profile lives depends on open question 4.

---

## 8. What I am asking you to review

Not the prose. The plan's structure. Specifically:

- Is the checklist/case-log split the right architecture given §4's constraints, or is there a better
  one available that we have not considered?
- Are the four problems in §7 the real risks, or am I missing a larger one?
- Is anything in §2 ("decided") actually unsafe to treat as decided?
- Is the 12-rule budget defensible, or arbitrary in a way that will hurt?
- **Most important:** the loop we are building is meant to improve itself. Given that assistant
  output cannot be observed automatically, is there a design here that actually closes, or are we
  building a better diary? Say so plainly if it is the latter.
