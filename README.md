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
- `research-*.md` — **research topics.** One file per question we actually went and researched,
  recording findings, what they change, what was left unchecked, and sources with reliability labels
  (primary literature vs. vendor marketing). Distinct from a `*-research-prompt.md`, which is a brief
  written *for* an outside researcher; a research topic holds the answers. Convention: whenever a
  question is worth researching, it gets one of these — otherwise the findings live only in a
  conversation and are gone at the next `/clear`.

Both `lessons.md` and `ideas-backlog.md` follow the same promotion discipline: an entry here becomes
an actual edit to the plugin's `agents/*.md` or `skills/orchestration/SKILL.md` only once it recurs
or is severe/certain enough to justify acting on alone — never automatically.
