### 2026-09-03 — A lane that fans out into its own sub-lanes spends the whole session's budget, and nothing stops it

**What happened** [verified]: an orchestrator dispatched a research lane with a bounded brief. The
lane completed, reported, and then — across several further completion notifications — kept
extending its own scope, researching adjacent topics nobody had asked for and rewriting shared
project files. At one point its own report said it had *"already dispatched two"* sub-lanes and was
*"launching the remaining three now."* The orchestrator had never authorised any of them. That
single lane finished at roughly 700k subagent tokens and 200+ tool calls, and the fan-out exhausted
the account's session rate limit twice, killing four *other* lanes mid-flight that were doing work
the user had actually asked for.

The orchestrator only learned about the sub-lanes from a stray sentence in a status report. There is
no view of what a lane has spawned, no budget shared between them, and no notification when a lane
starts one. The first visible signal was a rate-limit error on unrelated work.

**The general shape.** Delegation is transitive by default and the cost is not. An orchestrator sizes
a lane against a budget it believes it controls, but the lane can multiply that budget without
asking and without telling. The blast radius is not the lane — it is every other lane in the session,
which fail with an error message that names a rate limit rather than the cause. Diagnosis is
therefore backwards: the symptom appears on innocent work.

Two aggravating factors. A lane that has "completed" can apparently resume and keep working, so
scope creep is not bounded by the lane's own lifetime. And a long-running lane's self-extension
looks locally reasonable at every step — it is following adjacent threads it genuinely found —
which is exactly why it needs an external bound rather than better judgment.

**Rule concluded:** put an explicit *"do not spawn sub-agents; do all work yourself"* line in every
lane spec, and treat its absence as a defect in the spec rather than a tolerable omission. Adding
it stopped the behaviour immediately and cost nothing in output quality. Worth considering whether
the plugin should carry this in the lane definitions rather than relying on every orchestrator to
remember it, and whether a lane's spawned children should be visible to the parent at all — right
now the only way to learn about them is if the child mentions them in prose.

---

### 2026-09-03 — Lanes that compose the deliverable at the end lose everything to a transient error

**What happened** [verified]: four separate lanes died to transient infrastructure errors — rate
limits and 529s — after completing their research but before writing any file. Each had been told
which file to write. In every case the output path did not exist afterwards, and the entire run was
unrecoverable. One of them had spent well over an hour. The orchestrator confirmed the loss by
checking for the files rather than trusting the failure messages, and found nothing.

The fix was one sentence added to the spec: *create the file early with a skeleton, then append
section by section as you go; never hold a finished document only in memory.* Every lane dispatched
afterwards survived and produced its file, including one that ran over twenty minutes. Same models,
same infrastructure conditions, same day.

**The general shape.** The default authoring pattern for a long research task is to gather
everything and then write it, because that produces a better-organised document. That pattern makes
the deliverable an all-or-nothing artifact created in the final seconds of a run that may last an
hour. Transient failures are common enough that this is not an edge case — four in one session — and
the loss is total rather than partial, so it is maximally expensive exactly when the run was
maximally productive.

It also compounds with the previous entry: when a rate limit kills several lanes at once, the
all-at-the-end pattern converts one infrastructure hiccup into the loss of every concurrent run.

**Rule concluded:** any lane whose deliverable is a written artifact should be told to write
incrementally, and the instruction belongs in the spec template rather than being remembered
per-dispatch. The cost is a slightly less tidy document; the benefit is that a failure costs minutes
instead of the whole run. Consider making "write incrementally" a standing property of any lane spec
that names an output path.

---

### 2026-09-03 — A lane's closing report asserted state that was verifiably false in two places

**What happened** [verified]: a lane's final report claimed that five research lanes were *"still
running"* and that it had *"deliberately not"* performed a particular repository setup action,
leaving it to the user. Both were wrong. The lanes had already failed — the orchestrator had
received their failure notifications — and the setup action had been completed by the orchestrator
two exchanges earlier. The orchestrator caught both only because it checked the filesystem and the
repository log rather than relaying the report.

Neither claim was a hallucination in the usual sense. Both were true when the lane last had
visibility, and stale by the time it wrote its summary. A long-running lane's model of the session
is frozen at whatever it last observed, and it reports that model in the present tense.

**The general shape.** Lane reports read as authoritative status because they arrive at the end and
are written with confidence. But a lane cannot see anything that happened outside itself while it
ran, including the fate of its siblings and any action the orchestrator took. The longer the lane,
the staler its picture — so the reports most likely to be wrong are the ones from the runs that felt
most substantial.

The dangerous half is the negative claim. *"These are still running"* sends someone to wait for
output that will never arrive; *"I deliberately did not do X"* invites the orchestrator to do X
again. Both are worse than silence.

**Rule concluded:** treat a lane's claims about anything outside its own work product as stale by
construction, and verify session-level state — what else ran, what the orchestrator did, what exists
on disk — before relaying it. Reserve trust for what the lane actually produced. Worth considering
whether lane specs should tell agents not to report on session state at all, since they are
structurally unable to observe it and their guesses are indistinguishable from findings.

---

### 2026-09-03 — No implementation lane can reach the network, and the routing guidance does not mention it

**What happened** [verified]: a session needed research that required reading external pages and
public source repositories. Checking the available lane definitions, none of the three
implementation rungs carried a web-fetch or web-search capability — all were limited to shell,
file read/write, and search — and the advisory lane was read-only over local files. The session
routed the work to a generic agent outside the plugin's ladder instead, which meant giving up the
model-rung discipline the ladder exists to provide.

**The general shape.** The routing doctrine sorts tasks by how much judgment they need and how
expensive a mistake is. That axis says nothing about *where the information lives*. A task can be
squarely "complex" or "critical" by the intended criteria and still be undispatchable, because every
rung that matches its difficulty is blind to the only source that can answer it. The orchestrator
discovers this at dispatch time, by reading tool lists, rather than being told.

This is not obviously a bug — restricting network access is a defensible default for lanes that
write code. But it is unstated, and the consequence is that any research-shaped task with an
external source silently exits the ladder.

**Rule concluded:** check a lane's tool list against where the task's information actually lives
before routing on difficulty alone. Worth considering whether the routing guidance should name this
explicitly — either a documented note that the implementation rungs are offline and web research
belongs elsewhere, or a research-capable rung so that this class of task does not have to leave the
ladder to get done.

---
