# Claude Orchestrator — Notes

Private companion repo to [Claude-Orchestrator](https://github.com/czclaxton/Claude-Orchestrator).
This is development and testing material, not the plugin itself — nothing here ships, nothing here
is referenced by the plugin at install time.

- `lessons.md` — friction/incident log from real use of the shipped agents. What actually happened,
  not speculation. Empty entries mean nothing's been logged yet, not that nothing's been tested.
- `ideas-backlog.md` — speculative features and ideas, not yet built. Promoted to the real plugin
  only once proven out in real use — see the log at the bottom of that file for what's actually
  been tested.
- `model-routing-research-prompt.md` — the research brief used to pressure-test the routing doctrine
  against practitioner reports (via Grok, for its X/Reddit access). Kept for reference and reuse.
- `communication-preferences.md` — how Connor wants things explained, questioned, and pushed back on.
  **Portable and self-contained: point any agent at this file directly.** Every item is tagged
  `[stated]` or `[observed]` so an agent can tell a confirmed preference from an inference.
- `assistant-design-brief.md` — the "chief of staff" design conversation (2026-08-21/22). A
  communication-protocol design in four modules, with what was confirmed, corrected, and merely
  proposed kept distinct. Not built, not specced; four questions still open.

`first-validation-run-brief.md` records the first real end-to-end run of the agent ladder.

Both `lessons.md` and `ideas-backlog.md` follow the same promotion discipline: an entry here becomes
an actual edit to the plugin's `agents/*.md` or `skills/orchestration/SKILL.md` only once it recurs
or is severe/certain enough to justify acting on alone — never automatically.
