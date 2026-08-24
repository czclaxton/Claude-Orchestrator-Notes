# PLAN — making communication preferences load automatically, and closing a learning loop

**Written by the orchestrator, 2026-08-24, for review. Nothing here is installed.**

⚠️ **Read this warning first.** This document captures decisions made in conversation. Some of it
supersedes the other files in this directory, and one thing in it supersedes an earlier statement of
my own. Where this document and another file disagree, sections 1 and 6 below say which wins. Do not
treat the other files as current without checking here.

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

## 3. What is PROPOSED and NOT approved

- **The 12 rules in `DRAFT-rules.md`.** Drafted this session. Connor has not reviewed them line by
  line. Open to cuts.
- **The 12-rule budget.** A number I picked. The design brief proposes ~30 for the general ruleset;
  12 is my compression of the communication subset. Unjustified beyond "small."
- **R6 is explicitly blocked** — see §5.

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

1. **Mechanism: `~/.claude/rules/` or an output style?** Asked twice, not yet answered. My
   recommendation is `rules/` (global, plain markdown, clean profile seam, independent of the plugin
   scope question). The case against is that output styles edit the system prompt, which is stronger
   placement, and the docs describe them as being for exactly this. Risk of splitting: two homes for
   one profile, which the design brief warns against.
2. **Is a checklist of imperative rules even the right format?** Connor raised this and he is right
   to. A separate research agent is investigating rules vs. worked examples vs. both. **The 12 rules
   may need reformatting depending on what comes back.** Do not treat `DRAFT-rules.md`'s format as
   settled.
3. **From the design brief, still unanswered and it gates R6:** *when is being asked worth it, and
   when is it an imposition?* Asking spends the attention the whole system exists to conserve. Until
   answered, R6 ("when you notice you are guessing → ask") has no volume control and could produce
   constant interviewing — the exact load he is escaping.
4. **Does this extend Claude Orchestrator or become its own tool?** Open in the design brief, open in
   `RESUME-PROMPT.md`. Bears on where the generator command lives.
5. **What triggers a compile session?** See Problem 1 in §7.

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
