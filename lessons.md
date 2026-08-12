# Lessons (developer-only — never shipped with the plugin)

Tracks real friction and failure points from actually using Claude Orchestrator's agents in live
work — not speculative features (see `ideas-backlog.md` for those). This file is about what
*actually happened* when `routine-implementer`, `complex-implementer`, `critical-implementer`, or
`advisor` got used for real, and what it implies about whether the routing doctrine or the agents
themselves need to change.

**Why this matters more than usual for this project:** the three-rung ladder was designed from
first principles and pressure-tested only against secondhand research — treated as reasoning fuel,
not settled fact (see the `grain-of-salt-ai-behavior-research` memory). It hasn't been validated
against real use yet. This file is where that validation actually happens.

**Not shipped, developer-only for now.** Gitignored — a marketplace install from the published
GitHub repo will never see this file, since only committed content ships that way. One caveat worth
remembering: installing directly from a local path (the way the plugin was smoke-tested earlier)
copies the whole directory, gitignored files included — not a concern while it's only Connor
testing his own tool locally, but worth remembering before ever handing the plugin to someone else
via that install path instead of the GitHub one.

**Future possibility, not being built now:** once other people are testing this, an opt-in path for
them to contribute their own equivalent friction log for review. One real design problem to solve
before that ships, flagged now so it isn't an afterthought: unlike this file — Connor's own
project, Connor controls what's in it — a contributed log from someone else's project could
inadvertently capture snippets of their own private code or task context. Needs an actual answer
(redaction? a template that captures the pattern, not the raw transcript?) before this becomes a
real feature.

**When to add an entry:** a real, specific incident — a routing call that was wrong, an agent that
drifted from its spec, a report that didn't capture what was actually needed, a case where the
five-part spec contract was awkward or insufficient. Not every minor annoyance — a repeated
pattern, or one incident concrete enough to actually learn from.

**Format per entry:** date, what happened, the fix or rule concluded (if any), where it lives (a
specific agent file, `SKILL.md`, `ideas-backlog.md`, or "not yet promoted — logged for now").
**A logged entry does not automatically become a rule change** — same promotion bar as everywhere
else in this project: a recurring pattern, or one incident specific and severe enough to justify
acting on alone. If this file ever grows past what plain markdown can hold usefully, the heavier
structured-ledger pattern (`docs/product/review-gaps/` in the Pepco project) is the fallback to
revisit — not the starting point.

---

## Log (append-only)

### 2026-08-11 — Model-pin frontmatter smoke test: passed on this install

**What happened:** invoked all four agents (`advisor`, `routine-implementer`,
`complex-implementer`, `critical-implementer`) with a no-tool-use self-report prompt ("which model
are you?"), mirroring the exact technique used in the GitHub bug reports (#18346, #44385, #58450)
that documented subagents silently inheriting the parent/session model regardless of frontmatter.
All four came back correctly matched to their pin: advisor → Fable 5, routine-implementer →
Sonnet 5, complex-implementer → Opus 5, critical-implementer → Fable 5. Environment: this session
itself (Claude Code, plugin installed locally, CLI v2.1.228) — the same surface Connor's real usage
runs on, not a different one.

**Conclusion, held provisionally:** the frontmatter-ignored bug does not appear to affect this
install. This is one trial per agent, not exhaustive — worth a second confirmation once a real
multi-step delegated task runs (this test used a trivial single-turn prompt, and some bug reports
specifically implicated longer/nested delegation), but there's no reason yet to build the "avoid
one tier" workaround discussed earlier. Not promoted to a standing rule — logged as the evidence
for why the earlier "proceed assuming it's probably fine, verify empirically" decision held up.

**Where it lives:** `project_claude_orchestrator.md` memory, "open items" section — updated to
reflect this result instead of "untested."

### 2026-08-11 — Local plugin install doesn't refresh on source edits, and `plugin update` didn't fix it

**What happened:** edited `skills/orchestration/SKILL.md` (added the planning-phase section), then
checked the installed plugin's cached copy — still had the old content. `claude plugin marketplace
update` + `claude plugin update claude-orchestrator@claude-orchestrator` reported "already at the
latest version (1.0.0)" and did not refresh the cache; the update mechanism appears to gate on the
version string in `plugin.json`, not actual file contents. A full `uninstall` + `install` cycle did
force a fresh copy.

**Fix/rule now in place:** during active local development, don't trust `plugin update` to pick up
source edits without a version bump. Either bump `plugin.json`'s version on meaningful edits, or
uninstall/reinstall to force a refresh, before testing anything that depends on a doctrine or agent
file changed in the same session.

**Where it lives:** not yet promoted to a doc — logged here. Worth turning into an actual dev-loop
habit (bump version per edit) once real testing is more than occasional, rather than re-discovering
this each time.
