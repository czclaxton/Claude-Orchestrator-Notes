# Claude Code capabilities — what the harness can actually do

Research reference, gathered 2026-08-24 via the `claude-code-guide` agent against official docs at
`code.claude.com/docs`. Kept because it was expensive to gather once and is free to reuse.

**Provenance:** `[documented]` = stated in official docs, source listed. `[inferred]` = reasoned, not
stated. `[verified-local]` = checked directly on this machine 2026-08-24. Do not upgrade an
`[inferred]` to fact without rechecking — the docs move.

---

## 1. What loads into context automatically, every session

Load order, broadest to most specific. Later entries sit closer to the model. `[documented]`

| # | Source | Scope | Conditional? |
|---|---|---|---|
| 1 | Managed policy `CLAUDE.md` (`C:\Program Files\ClaudeCode\CLAUDE.md` on Windows) | Org-wide | No — cannot be excluded |
| 2 | `~/.claude/CLAUDE.md` | User, all projects | No |
| 3 | `~/.claude/rules/*.md` | User, all projects | No — **except** path-scoped rules, which load only when a matching file is read |
| 4 | `./.claude/CLAUDE.md` or `./CLAUDE.md`, plus `./.claude/rules/*.md` | Project | No at root; **nested subdirectory `CLAUDE.md` loads on demand** when a file in that dir is read |
| 5 | `./CLAUDE.local.md` | Project, personal, gitignored | No |

Also loaded unconditionally at start: system prompt (~4,200 tokens), auto memory, environment info
(cwd, OS, git branch/status/recent commits), MCP **tool names only** (schemas deferred), **skill
descriptions only** (not skill bodies), subagent descriptions. `[documented]`

**Nothing else loads unconditionally.** `[documented]`

### Auto memory — a real harness feature, not a convention

`~/.claude/projects/<project-slug>/memory/` `[documented]`

- On by default. **The first 200 lines or 25KB of `MEMORY.md`, whichever comes first, loads at every
  session start.**
- Loads *after* CLAUDE.md files, *before* skill descriptions.
- `MEMORY.md` is an index; topic files are read on demand with file tools, not auto-loaded.
- **Per-project, and machine-local — not synced across machines.** This is the reason it cannot carry
  global preferences.
- All worktrees of one repo share a memory directory.

### `@path` imports

`[documented]` Works **in CLAUDE.md files only**. `@README` (relative) or `@~/.claude/file.md`
(absolute). Imported files are **inlined at launch** — max 4 hops deep. Skips code spans and fenced
blocks, so literal `@` in code is safe; backtick-wrap to prevent an import.

**Important:** splitting a CLAUDE.md into imports helps organization and saves **zero** context. It
all still loads.

---

## 2. Output styles

`[documented]`

- **Modify the system prompt directly** — not a user message. Stronger placement than CLAUDE.md.
- Change role, tone, and default output format.
- **Main conversation only.** Subagents run their own system prompt and do not inherit them (unless
  forked).
- User-level: `~/.claude/output-styles/`. Project-level: `.claude/output-styles/`.
- Selected via `/config` → Output style, persisted as `outputStyle` in a settings file. Takes effect
  on new session or after `/clear`.
- Docs position them for *"role, tone, or default response format every turn"* — as distinct from
  CLAUDE.md, which is for project conventions and codebase context. **Both load; they do not
  override each other.**
- **Plugins can ship them** in an `output-styles/` directory. Frontmatter `force-for-plugin: true`
  applies the style automatically whenever the plugin is enabled, overriding the user's setting. If
  two enabled plugins both force, first loaded wins.

---

## 3. Hooks

### Every hook event `[documented]`

- **Session:** `SessionStart`, `SessionEnd`, `Setup`
- **Per-turn:** `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `StopFailure`
- **Tools:** `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`,
  `PermissionRequest`, `PermissionDenied`
- **Subagents/tasks:** `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `TeammateIdle`
- **Files/config:** `FileChanged`, `CwdChanged`, `DirectoryAdded`, `ConfigChange`,
  `InstructionsLoaded`, `WorktreeCreate`, `WorktreeRemove`
- **Context/display:** `PreCompact`, `PostCompact`, `Notification`, `MessageDisplay`,
  `Elicitation`, `ElicitationResult`

### What each can do `[documented]`

| Event | Inspect | Block | Modify | Inject context |
|---|---|---|---|---|
| `SessionStart` | — | — | — | Yes (stdout) |
| `UserPromptSubmit` | Yes (prompt text) | — | Yes (`updatedInput`) | Yes (`additionalContext`) |
| `PreToolUse` | Yes (tool + input) | **Yes** (`permissionDecision`) | Yes (`updatedInput`) | Yes |
| `PostToolUse` | Yes (tool output) | — | — | Yes |
| `Stop` | — | — | — | Yes (`additionalContext`, `systemMessage`) |
| `MessageDisplay` | Limited — text as it streams | — | — | — |

### ⭐ The load-bearing limitation

**No hook receives the assistant's completed message.** `[documented — by absence across the full
event list]` `MessageDisplay` sees text streaming; `Stop` fires after the turn but is not given the
content.

**Consequence:** the harness cannot observe or grade assistant output. Any check on whether the
assistant followed a soft behavioral rule must be done by a human reading it, or by a separate
after-the-fact pass over the on-disk transcript. There is no live, automatic detection. This is the
single most important finding for any self-improving-loop design.

`SessionStart` receives `cwd`, so a hook **can** inject conditionally per project. `[inferred]`

---

## 4. Skills

`[documented]`

- **Automatic invocation is a model decision**, driven by description-matching — not deterministic.
  The exact matching mechanism is undocumented.
- Only the **description** loads at startup (~50–100 tokens; truncated at 1,536 chars). Body loads on
  invocation.
- Once invoked, the body **stays in context for the session**. On compaction, invoked skills are
  re-attached at first 5,000 tokens each, 25,000 total budget.
- `disable-model-invocation: true` → user-only. `user-invocable: false` → model-only, hidden from `/`.
- Skills can register hooks via a `hooks:` frontmatter field.

**Not suitable for behavioral rules** that must apply every message — invocation depends on model
judgment, so it is unreliable for anything that must always be on. `[inferred, high confidence]`

---

## 5. What a plugin can and cannot contribute

**Can ship** `[documented]`: skills, agents, hooks, output styles, MCP servers, LSP servers,
monitors, themes (experimental).

**Cannot ship** `[documented]`: a `CLAUDE.md`. Quoted from the plugin reference — *"A `CLAUDE.md` file
at the plugin root is not loaded as project context. Plugins contribute context through skills,
agents, and hooks rather than CLAUDE.md."*

**The workaround, and it is the clean one:** a plugin cannot *carry* always-on context, but it can
ship a **command that writes** a `CLAUDE.md`, a `~/.claude/rules/*.md`, or an output style into the
user's own space. The plugin becomes a generator, and the generated file is user data. This maps onto
the engine/profile seam in `assistant-design-brief.md` — engine ships, profile is generated and
owned by the user. `/claude-orchestrator:setup` already works this way. `[inferred from documented
constraints]`

---

## 6. Local state on this machine

`[verified-local] 2026-08-24` — none of these exist yet:

- `~/.claude/CLAUDE.md` — absent
- `~/.claude/rules/` — absent
- `~/.claude/output-styles/` — absent
- `outputStyle` in settings — not set

Present: `~/.claude/skills/`, `commands/`, `plugins/`, `projects/`, `settings.json`,
`orchestrator-testing.md`.

Memory dir for this project holds 5 files, newest modified 2026-08-19.

---

## 7. Gaps — asked and not established

Do not fill these in from plausibility.

1. Per-file token cost of CLAUDE.md — docs say "costs tokens," give no estimate.
2. Precedence when a plugin's `force-for-plugin` output style meets a user `outputStyle`. Docs say
   plugin wins; load order unstated.
3. Whether a nested subdirectory `CLAUDE.md` can override a parent's conflicting instruction.
4. The mechanism behind skill description-matching (embeddings? rules? unstated).
5. Whether hook `additionalContext` is treated as a system message or a user message.
6. Exact ordering of `SessionStart` hook output vs. CLAUDE.md load. Reported as hook-first, but
   adjacent details were undocumented — treat as probable, not certain.

## 8. Mechanisms not originally asked about, flagged as relevant

1. `claudeMd` field in managed-settings.json — inject CLAUDE.md content via settings, no file needed.
2. Settings precedence: managed policy > user > project > local.
3. Skills can register hooks in frontmatter — conditional behavior without a separate plugin hook.
4. Subagents do **not** inherit output styles unless forked. Relevant to any lane-facing rules.
5. `/context` — shows everything currently loaded, categorized. Directly useful for verifying any of
   the above empirically rather than trusting this file.

---

## Sources

`code.claude.com/docs/en/` — `memory`, `context-window`, `how-claude-code-works`, `output-styles`,
`hooks`, `skills`, `plugins-reference`.
