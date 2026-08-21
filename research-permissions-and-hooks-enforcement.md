# Research topic: enforcing guardrails with permissions and hooks

**Date:** 2026-08-21 · **Status:** findings recorded, not yet promoted to doctrine
**Question that prompted it:** an outside developer's `CLAUDE.md` was shared for critique. It wrote
its hard guardrails ("NEVER commit to main", "NEVER merge PRs") as prose instructions. Checking
whether that actually enforces anything led to checking this project's own posture, which has the
same gap.
**Method:** direct doc fetches by the orchestrator session (not delegated) against
`code.claude.com/docs/en/hooks` and `/settings`, plus inspection of this repo's own agent
frontmatter, `hooks/hooks.json`, and live GitHub branch-protection settings. Claims below are
marked verified or inferred.

## Why this was worth checking

The plugin's core doctrine — "nothing is closed without the user" — is currently written only as
prose in `CONTRIBUTING.md`. Prose binds the model by cooperation, not by construction. That held
through PRs #7, #8 and #9, but it held because the model complied, which is not the same as it
being unable to do otherwise. The question is which of this project's rules are actually
constructed and which are merely stated.

## The layer model

The only question that separates a guardrail from a wish: **does this rule hold because the model
chose to cooperate, or because it had no choice?**

| Layer | Mechanism | Cooperation required? |
|---|---|---|
| Capability | agent `tools:` allowlist; not granting a tool at all | No — the tool does not exist to call |
| Managed policy | `managed-settings.json` + `allowManagedPermissionRulesOnly` | No — user/project cannot override |
| External | branch protection, CI, filesystem ACLs | No — outside Claude entirely |
| Permission rules | `permissions.deny` in `settings.json` | No — the harness refuses the call |
| Hooks | `PreToolUse` and other blocking events | No, but see finding 3 |
| Instructions | `CLAUDE.md`, skills, agent prompts | **Yes** |

### Finding 1 — blocklists leak; allowlists enforce. This is the central one.

`permissions.deny` and `PreToolUse` are both *blocklists*: they enumerate forbidden actions. Every
blocklist is only as good as its enumeration, and the enumeration is never complete.

Capability removal is the opposite move and is categorically stronger. This project already does it
correctly in one place: `agents/advisor.md` declares `tools: Read, Grep, Glob`. The advisor cannot
implement — not because it is instructed not to, but because no write tool exists in its context.
Nothing to circumvent, no pattern to slip past.

**Implication:** the strongest available guardrail is subtractive. Prefer "this lane never had that
capability" over "this lane is forbidden from using it."

### Finding 2 — Bash is an arbitrary-code escape hatch, and this is structural

`permissions.deny` matches command *text*. `Bash(git push *)` does not stop `cd sub && git push`, a
shell script that pushes, or `gh`. This is not a weak pattern that a better regex fixes — deciding
what a program will do by reading it is undecidable in general. Any text-matching rule over shell is
advisory.

A `PreToolUse` hook improves on this only partially, and the distinction matters: the hook still has
to pattern-match to recognise *what kind* of command it is looking at. What it adds is the ability
to then evaluate **state** — run `git rev-parse --abbrev-ref HEAD`, check a file, call an API.
It solves "am I on master," not "is this a push."

**Implication:** for shell, the real control is whether the lane has `Bash` at all.

### Finding 3 — hooks fail open (verified)

Per the exit-code table: a hook that crashes, times out, or emits malformed JSON results in the
action **proceeding**. Timeout explicitly means "no decision rendered, action continues."

This makes hooks materially less reliable than `permissions.deny`, which is declarative and cannot
crash. Reliability order: capability removal > deny rules > hooks.

**Implication:** an untested hook is a placebo, and "no errors observed" is not evidence one works.
Any hook adopted here must be deliberately triggered once to confirm it denies.

### Finding 4 — deny beats allow across every scope (verified)

Permission rules **merge** across managed/local/project/user settings rather than following strict
precedence, and a `deny` from any source wins over an `allow` from any other. A deny placed in user
settings cannot be accidentally undone by a project-level allow.

### Finding 5 — `SessionStart` cannot block. Detection is not prevention.

Verified: `SessionStart` is context-only. Valid matchers are `startup | resume | clear | compact |
fork`.

This project's own hook is the clean illustration. `hooks/session-start.js` checks the wrap-up
sentinel and warns that findings were lost — but it fires *after* the clear, when they already are.
It is a detector, and it cannot become a preventer at that event.

Related, and already learned here the hard way: commit `415dc3e` removed a `SessionStart` hook that
*acted* (wrote alias files) because a hook's raw `fs` writes bypass Claude Code's permission prompt
entirely. Hooks that inject context are nudges; hooks that act are unreviewed writes. Keep them on
the nudge side unless the event can block.

## Blocking vs context-only events (verified, relevant subset)

**Can block:** `PreToolUse`, `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `SubagentStop`,
`PreCompact`, `PostToolBatch`, `TaskCreated`, `TaskCompleted`, `ConfigChange`.

**Context-only:** `SessionStart`, `SessionEnd`, `PostToolUse`, `SubagentStart`, `PermissionRequest`,
`PermissionDenied`, `InstructionsLoaded`, `PostCompact`, `Notification`.

`PreToolUse` decision schema:

```json
{"hookSpecificOutput": {
  "hookEventName": "PreToolUse",
  "permissionDecision": "allow | deny | ask",
  "permissionDecisionReason": "..."
}}
```

`ask` is the under-used middle setting — it escalates to the user rather than hard-blocking, which
suits a project whose whole doctrine is human-in-the-loop.

## What this implies for this project — none of it promoted, all of it needs the usual bar

1. **All three implementer lanes hold unrestricted `Bash`, and no agent file mentions git at all.**
   Verified: `routine`, `complex`, and `critical` each declare `Bash, Read, Write, Edit, Grep, Glob`;
   a grep for git/push/merge/commit across `agents/` returns nothing but incidental prose in
   `advisor.md`. So a lane can commit, push, or `gh pr merge` today, and the only thing preventing it
   is that nothing has told it to. This is the largest gap between stated doctrine and constructed
   guarantee in the plugin.

2. **`SubagentStop` can block — and this project has a logged failure it maps onto directly.**
   Notes PR #8 records *fabricated state in a lane report*. A `SubagentStop` hook could refuse a
   lane report that lacks verification evidence, turning "returns a structured report with
   verification evidence" from a prompt instruction into a checked precondition. Strongest candidate
   here because it responds to a real incident rather than a hypothetical.

3. **`Stop` can block.** The doctrine says every deliverable gets an advisor pass before the
   orchestrator reports done. That is currently prose. A `Stop` hook could check it. Flagged as
   interesting, not recommended — the false-positive cost on ordinary turns is likely high, and a
   `Stop` hook that misfires is exactly the over-blocking that gets ripped out mid-task.

4. **`SessionStart` could inject live state rather than leaving it to be derived.** Current branch,
   local-vs-remote divergence, open PR count. `/catch-up` derives these every time, and Notes PR #8
   is about fabricated state — injecting facts is cheaper than trusting they get re-derived
   correctly. Small, safe, directly aligned with the resume-note discovery work already queued.

5. **`enforce_admins` is false on the plugin's `master`.** Verified via the GitHub API. Branch
   protection requires a PR and one approving review, but admins bypass — and the agent acts with
   admin credentials. The external layer is present but not binding.

6. **Open question, needs an empirical test before anyone asserts either way:** whether `/clear` can
   be intercepted at all. `PreCompact` blocks, so the compaction path is coverable; `/clear` is a CLI
   action and may never reach a blocking event. Unknown. Do not claim the wrap-up sentinel can be
   made preventative until this is actually tested.

## Verified vs. inferred

**Verified** (docs fetched this session, or read from live state): the hook event list and their
blocking status; `PreToolUse` schema; `SessionStart` matcher values; hook fail-open on timeout and
error; deny-beats-allow merge semantics; this repo's agent tool lists; the absence of git guidance in
`agents/`; `enforce_admins: false` on `master`; the absence of any `permissions` block in
`~/.claude/settings.json` and of any project `.claude/settings.json`.

**Inferred, not tested:** that a `SubagentStop` hook can practically inspect a lane report's content;
that `Stop`-hook false positives would be costly; that shell blocklists are defeatable *in this
harness specifically* (the undecidability argument is general, the specific bypasses were not
attempted).

**Not attempted:** no hook was written, no setting was changed, no protection was altered. This
document records findings only.

## Sources

- `https://code.claude.com/docs/en/hooks` — official Claude Code documentation. Hook event
  inventory, blocking status, `PreToolUse` schema, `SessionStart` matchers, exit-code conventions.
- `https://code.claude.com/docs/en/settings` — official. Permissions schema, rule syntax, settings
  precedence and merge semantics, capability-restricting settings.
- This repo's own `agents/*.md`, `hooks/hooks.json`, `hooks/session-start.js`, and commit `415dc3e`
  — primary evidence for current posture.
- GitHub branch-protection API for `czclaxton/Claude-Orchestrator` — live state, read this session.
