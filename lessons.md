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
acting on alone.

**Optional classification tag (experimental, 2026-08-19):** borrowed from Pepco's SoA-installed
review-gap ledger, which defines this taxonomy but — verified before adopting it, not assumed —
never actually recorded a real entry with it in that project. Unproven, not battle-tested; adopt
provisionally and see if it earns its keep. When an entry is about a bug/gap someone else's work
(or the architect's own) let through, tag it:
- `spec-gap` — a genuine decision the spec left missing or underspecified; a competent builder
  would have had to stop and ask. Bias against this for self-evident behavior no competent spec
  would need to enumerate ("save persists", "delete removes") — that's a shipped violation, not a
  spec-gap.
- `qa-gap` — needed experiential/manual verification outside normal review to catch.
- `review-reachable` — visible in the diff at review time; the reviewer had enough evidence and
  missed it.
- `completeness-gap` — delivered scope was valid but missed adjacent work that should have been
  shaped into the plan.
**Litmus test for the spec-gap/qa-gap split:** would a competent person, given no spec, have built
it right on instinct? If yes, it's not a spec-gap — classifying it as one launders an execution
miss into a planning miss and leaves the real gap (verification or review) unaddressed.

**Optional heavier structure for a substantial entry** (a full validation run, not a one-line
friction note) — also borrowed from Pepco/SoA, but **this one is proven**, not experimental: a
2026-08-05 retrospective written with this exact structure in that project was a real, successful
artifact, independently confirmed. Seven sections, in order: scope delivered (one paragraph, the
factual anchor); what went well (with *why* it worked, not just that it did — "TypeScript caught
it" is noise, "caught it because the type was narrow enough to make the error unambiguous at the
call site" is signal); pain points (split avoidable-waste, which could've been designed away, from
expected-cost, which is inherent — they imply different follow-ups); surprises (highest-signal
section — things not in the spec, good or bad, that won't appear anywhere else); what we'd do
differently (distinct from pain points — this is what hindsight would redesign, with the original
reasoning for why the first choice looked right); net assessment (one paragraph, no hedging — did
it work or not); follow-up (concrete next actions, not "consider X"). Use this for a run like
2026-08-19's below; a two-line friction note doesn't need all seven sections.

If this file ever grows past what plain markdown can hold usefully, the heavier structured-ledger
pattern (`docs/product/review-gaps/` in the Pepco project) is the fallback to revisit — not the
starting point.

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

### 2026-08-19 — First real validation run: Hang-Up Log MVP, sandboxed off Pepco

**What happened:** first time any of the four agents did real, non-trivial delegated work. Task: a
grouped/duplicates view for Pepco's Hang-Up Log feature, built in an isolated local sandbox (cloned
off Pepco's `develop`, origin removed, reseeded with clean data) rather than the real repo. Full
detail and setup: `first-validation-run-brief.md` in this same repo. Session ran on Max 5x (upgraded
mid-session), architect on Opus, all four lane pins confirmed correct beforehand via a fresh
single-turn smoke test.

**Model pins — closed out.** All four lanes ran on their pinned models across real multi-step
delegated builds (not just the trivial single-turn self-report from 2026-08-11): routine-implementer
→ Sonnet 5, complex-implementer → Opus 5, critical-implementer and advisor → Fable 5. The 2026-08-11
result is no longer provisional.

**Routing calls: both correct, with concrete evidence, not just "it worked."**
- The backend grouped-query endpoint went to complex-implementer (Opus) because it had a real,
  non-obvious correctness trap: computing "most recent occurrence per phone number" via independent
  `MAX(date)` / `MAX(time)` can silently stitch together a date and time from two different calls —
  plausible-looking, quietly wrong. The agent avoided it deliberately (chose `groupBy` + an
  ordered `findMany` so both values always come from one real row), explained the reasoning, then
  noticed my own verification command was weak evidence (two sort orders happened to produce
  byte-identical output on the seed data) and strengthened it unprompted — inserted a real
  divergent row via the live API, proved the sorts actually differ, proved the pitfall was really
  avoided, then cleaned up and re-verified the DB was back to its seeded state.
- The frontend (toggle + duplicates table + drill-down modal) went to routine-implementer (Sonnet)
  as a close port of two already-established patterns (an unused `Tabs` component, the existing
  modal pattern). It delivered clean, and — notably — honestly flagged in its own report that it
  had no way to drive a real browser, so it could not claim visual verification, rather than papering
  over the gap with "should work."
- critical-implementer was **not** exercised on real work this run — only the earlier trivial
  self-report smoke test. Nothing in this task was genuinely high-stakes/hard-to-reverse, so routing
  anything to it would have been gaming the test, not validating it. Honest limit of this run, not a
  finding against the lane.

**The mandatory final review earned its cost — concretely, not as a formality.** `advisor` caught a
real defect neither implementer's own verification could have: I (the architect) had made the
`HangUpLog.phoneNumber` column required via a manual raw-SQL table recreation directly against the
sandbox's live DB during setup, and never wrote an actual Prisma migration for it. Both curl-based
verifications passed because they ran against the already-drifted live DB — the gap only shows up
against a fresh `prisma migrate deploy`, which `advisor` checked by reading the migration file
against the schema. Verdict: fix-first, not ship, with two named, specific fixes (the missing
migration, and unvalidated POST/PUT now throwing a raw 500 on missing phone_number instead of a
clean 400). Both real, both confirmed independently before believing the report.

**One clean fix-loop, exactly as the doctrine describes.** Sent a corrected follow-up spec back to
the same lane (complex-implementer). One round trip, no escalation needed: it proved the drift first
(replayed migration history onto a throwaway shadow DB rather than assuming), fixed both issues,
proved the fix (fresh-environment `migrate deploy` succeeds, 400s return correctly, all 18 seeded
rows and all 4 duplicate counts still intact). I independently re-ran the same checks myself before
accepting it — all matched.

**Good restraint, worth noting on its own.** While fixing the migration, the same agent found an
unrelated pre-existing schema/migration drift on a different table (`followups`' FK constraint is
weaker in the migration than the schema implies) — and correctly declined to fix it, flagging it
precisely instead of scope-creeping into an unrelated bug it happened to notice. Matches the
"if it's architectural, stop and report" discipline in its own contract.

**A gap on the architect's side, not the plugin's doctrine.** Mid-run I reached for Claude in Chrome
for a one-off visual check by default/habit, without consulting the browser-tool decision framework
already sitting in `ideas-backlog.md` #4 — which, for exactly this case (one-off manual check, no
real auth), names Chrome DevTools MCP as the default. No functional harm, but a real "the memory
existed and wasn't consulted at the decision point" gap, distinct from anything the routing doctrine
covers. Worth remembering as its own category of failure mode, separate from routing correctness.

**Root cause of the migration gap was mine, not a lane's.** It came from a manual, out-of-band schema
edit I made directly during sandbox setup (Step 2), not from any delegated lane failing its spec.
Lesson for future sandboxed runs: any manual schema/DB edit made while setting up a sandbox needs a
real migration file written immediately, not deferred — treat sandbox setup with the same rigor as
delegated work, since gaps introduced there surface identically to real bugs later.

**Open items still not resolved by this run:**
- **Loosen the five-part spec for `critical-implementer`** — still no direct evidence either way;
  the lane wasn't exercised on real work this run (see above). Needs a task that's genuinely both
  judgment-heavy and high-stakes to test properly.
- **Fable weekly-cap consumption under real use** — thin evidence. `critical-implementer` only got
  the trivial smoke test; `advisor` made exactly one real review call this run. Can't yet say whether
  the 50% cap binds under heavier real use.
- **Opus-vs-Fable-as-architect** — no new evidence; the architect ran on Opus throughout.

**Sandbox approach itself: validated, reusable.** Clone + remove origin + copy gitignored `data`/`.env`
files + reseed with deliberately clean-but-realistic data was enough to get fully real, live,
verifiable behavior for all three pieces of the feature — including the grouped/duplicates view,
which the original real-world plan expected to ship mocked. Worth reusing this pattern for future
validation runs rather than re-deriving it.

**Where it lives:** this entry is the primary record. `project_claude_orchestrator.md`'s "open items"
section should be updated to reflect the model-pin item as closed and the remaining two items as
still open with this run's specific evidence gaps noted.

### 2026-08-19 — Mined Pepco's Son-of-Anton install for portable ideas, didn't adopt it wholesale

**What happened:** Pepco (a real client project, unrelated to Claude Orchestrator itself) has
Son-of-Anton, a third-party delivery-orchestration framework, installed. Considered whether to make
Claude Orchestrator the primary system there instead. Investigated properly before deciding
anything — read SoA's actual skill content (not just the CLAUDE.md pointer file), verified real
usage evidence in the repo (`git ls-files`/`.git/info/exclude`/`git log --grep`, not assumption),
and sent a `claude-orchestrator:advisor` (Fable) agent to independently re-verify the findings and
critique the recommendation before acting — a real commitment-boundary consult, not a rubber stamp.
The advisor caught two factual errors in my own read (the review-gap ledger was empty, not "one
entry from install day" — stronger evidence for disuse than claimed; the one retrospective was
real, successful evidence the retrospective *skill* works, not evidence of disuse as I'd framed it)
and one real content-loss risk I'd missed (Pepco's `CLAUDE.md`, despite being 100% untracked and
safe to strip structurally, held the only written "format → stage → commit" norm and a genuinely
good "don't rationalize away review findings" line — worth salvaging, not deleting outright).

**Conclusion:** SoA's autonomous, PR-centric, multi-vendor-review delivery machinery
(worktree-per-ticket, `bun run deliver` state machine, wall-clock polling on CodeRabbit/Qodo/
Greptile/SonarQube, "do not pause between tickets") is a genuine philosophy mismatch with
gate-everything-through-manual-local-review — not a maturity gap Claude Orchestrator needs to close
before "replacing" it. Confirmed by real evidence, not assumed: zero of 152/154 commits reference a
ticket/epic/SoA, the review-gap ledger was never used, `prReview` was already disabled in SoA's own
config. Three specific ideas were worth taking independent of that machinery — see the format/
classification-tag/heavier-structure additions above this entry, and the still-open question below.

**Fix/rule concluded:** (1) Pepco's `CLAUDE.md` replaced with a minimal, non-SoA note (the salvaged
format-norm + review-discipline lines), SoA's actual files (`.son-of-anton/`, skills, delivery
tooling) left untouched on disk as reference — full removal was unnecessary since it was never
git-tracked to begin with. (2) Two ideas ported into this file's own format (retrospective structure,
defect-classification tags). (3) One idea — `soa-grill-me`'s one-question-at-a-time protocol with a
stated recommendation + steelmanned opposing view + tradeoffs per question — directly contradicts
an existing doctrine rule Connor explicitly requested on 2026-08-12 ("batch questions, don't go
one-at-a-time"). Correctly **not** silently resolved either way — flagged for Connor's own call,
still open as of this entry.

**Where it lives:** `lessons.md`'s format section (this file, above) for the two adopted ideas;
`skills/orchestration/SKILL.md` is the eventual home for the third *if and when* Connor decides how
to resolve the batching conflict — not yet written there.

**Update, same day:** two things above are now stale.

1. **SoA's files were not left in place after all.** Connor pointed out `.son-of-anton/`,
   `tools/delivery/`, `.agents/`, and the `soa*` skills were genuinely redundant — a real,
   independent standalone copy already exists at `C:\Repos\Son of Anton`, and even that's just a
   convenience mirror of the real source of truth (`cesarnml/son-of-anton` on GitHub), not something
   irreplaceable. Verified directly (`find`, file comparison) before deleting, not assumed. All of it
   removed from Pepco. Nothing lost — a fresh `git subtree add` recovers it if ever wanted again.
2. **The batching conflict is resolved.** Connor stated a direct, general preference: one question
   at a time, plain-terms explanation, room to go deeper — not a Claude-Orchestrator-specific
   answer, a standing preference for how he wants to be asked things generally. `skills/orchestration/
   SKILL.md`'s planning-phase section now reflects this (version bumped to 1.0.2, reinstalled).
   The `grill-me`-derived format (recommendation + steelmanned opposing view + tradeoffs per
   question) is folded in as the concrete mechanism, not just "ask one at a time" on its own.

### 2026-08-19 — Building the testing-mode feedback loop itself (v1.0.2 → v1.0.8): infrastructure
work, not agent routing, but real friction throughout

**Status: verified** (each claim below was directly observed this session, not inferred).

**What happened:** built the actual testing-mode feedback loop this file exists to feed — the
`SessionStart` hook, `/claude-orchestrator:wrap-up`/`:catch-up`, the setup/alias mechanism, and this
toggle. Almost none of it touched the routing ladder this file otherwise tracks; it was plugin
infrastructure. Recording it anyway since `/wrap-up`'s own instructions scope this file to
"friction about the orchestrator plugin itself," not just agent routing, and there was a lot of it.

**Finding 1 — silently writing to the user's filesystem on session start is against current best
practice, not just a style preference.** First design: a `Setup` hook writing two small relay files
into `~/.claude/commands/` automatically, triggered by `claude --init-only`. Dropped after actually
searching for prevailing practice rather than trusting instinct: npm v12 and pnpm v10+ both now
default to *blocking* postinstall-style scripts specifically because "code runs automatically, no
visibility, no consent" is the exact pattern that's been abused. There's also a Claude-Code-specific
version of the same problem: a hook's raw Node `fs` writes bypass Claude Code's own Read/Write/Edit
permission-prompt system entirely — a hook is not gated by it at all, unlike the model's own Write
tool. Replaced with `/claude-orchestrator:setup`, a command whose body has the model create the
files with its own Write tool, so the write is visible and permission-prompted like any other.
**Rule concluded:** before shipping anything that writes outside the plugin's own directory (home
dir config, personal command files, etc.), default to a model-driven write via the normal tool
permission system, not a hook doing it silently — and check prevailing practice, don't just trust
what feels convenient.

**Finding 2 — several confident assertions about Claude Code platform behavior were wrong on first
real test, which is why Connor gave a standing instruction mid-session to verify everything
empirically going forward (see `feedback_verify_dont_assume.md` memory).** Concretely: assumed a
collision-free plugin command name would resolve bare — false, plugin commands never get a
bare-name shortcut, ever, confirmed by testing multiple names. Assumed pushing to GitHub was
required for local `plugin update` to pick up changes — false, this marketplace is a local-directory
source, not a GitHub clone; only a `plugin.json` version bump matters locally. The original command
names `/reset`/`/resume` were shipped without checking for collisions and both silently broke
(`/reset` triggered a real context-clear instead of dispatching to the command; `/resume` returned
"isn't available") — found only by testing the bare name in a scripted session, not by reading docs.
Most recently: after a real version bump + successful `plugin update`, a Claude Code session that
was *already running* did not pick up the new command — needed a full restart, matching the
CLI's own "Restart to apply changes" message, which is accurate, not boilerplate.

**Reusable technique surfaced by this, worth remembering on its own:** `claude -p
--input-format=stream-json --output-format=stream-json` lets a scripted, multi-turn session
(including sending `/clear` mid-session) be inspected via its real internal event stream
(`hook_started`/`hook_response`, `conversation_reset`, tool_use, etc.) — genuine proof of hook/command
behavior without needing a human to test manually in a second terminal, and without risking the
current session's own context. Real limitation found the same way: `-p` non-interactive mode can't
answer permission prompts, so any Write needing a fresh approval fails there specifically — don't
mistake that for a real bug when a headless test fails to complete a write.

**Finding 3 — the advisor consult earned its cost on infrastructure/design work, not just code
review.** Used twice this session (the `/claude-orchestrator:setup` design, the testing-mode toggle
command), both returned "ship" but each caught a real, non-obvious gap on the first pass: the
`/claude-orchestrator:setup` command's original wording would have claimed an existing file was
"already set up" without checking it was actually this plugin's relay file rather than an unrelated
user command with the same name. Small fixes both times, but real ones a same-session self-review
plausibly would have missed.

**Where it lives:** `feedback_verify_dont_assume.md` memory (finding 2, the standing instruction);
`CONTRIBUTING.md` in the main repo (the version-bump and naming-collision rules, already promoted
there as concrete process docs, not just logged here); this entry is the fuller record of *why*
those rules exist. Finding 1's "model-driven write over silent hook write" rule is not yet written
into any doctrine file — currently only lives here and in the `commands/setup.md` /
`commands/testing-mode.md` implementations themselves.

### 2026-08-19 — `/catch-up` and `/wrap-up` hardcode one resume-note path, and silently lose to a
project's own convention

**Status: verified** (observed directly this session — the lookup failed, the alternate note existed
and was current).

**Finding 1 — `/catch-up` looks for `RESUME-PROMPT.md` at the project root and nowhere else, so a
project that already maintains its own resume note under a different path reads as having none.**
In the project used this session, a project-local skill (unrelated to this plugin) had for several
sessions been writing a detailed, accurate, current resume note to a different path under the docs
directory. `/catch-up`'s existence check missed it entirely. Per its own instructions the correct
behavior at that point is "say so plainly and ask the user what they'd like to work on, rather than
guessing" — i.e. the command would have reported *no context available* while a better resume note
than the one it was looking for sat two directories away. The catch only happened because the
existence check was run against both paths on a hunch, not because the command suggested it.

The failure is silent and it is exactly inverted from what you want: the more mature the project's
own note-keeping, the more likely `/catch-up` is to report nothing. And `/wrap-up` completes the
problem from the other end — it writes its note to the same hardcoded root path, so on a project
with an existing convention it *creates a second competing resume note* rather than updating the
one already in use. Two notes, both claiming to be current, guaranteed to drift.

**Rule concluded:** both commands should discover an existing resume note before assuming their own
filename — glob a few conventional names/locations, and if one is found that isn't the plugin's own,
`/wrap-up` should update that file (or at minimum say out loud that it is creating a second one)
rather than writing a duplicate. Hardcoding a single filename is fine for a project the plugin
owns; it is wrong for a plugin that drops into arbitrary existing repos.

**Finding 2 — `/wrap-up`'s "never sweep in other uncommitted work" guard is unenforceable as
written, because the collision it needs to guard against is in `lessons.md` itself.** The
instruction says stage and commit only `lessons.md`, never `git add -A`, so unrelated uncommitted
work in that repo isn't swept in. But the notes repo was found this session with a large
uncommitted entry *already sitting in `lessons.md`* from a previous session whose `/wrap-up` had
appended but never committed. Staging "only `lessons.md`" therefore commits that prior session's
work too, under this session's commit message, and the guard as phrased gives no signal that
anything unusual is happening. **Rule concluded:** the step should have the model check whether
`lessons.md` is already dirty *before* appending, and if it is, say so and let the user decide
whether the pre-existing content goes in the same commit — the file being the plugin's own is
precisely why it's the likeliest file to already hold someone else's uncommitted work.

**Where it lives:** here only. Neither finding is written into `commands/catch-up.md` or
`commands/wrap-up.md` yet; both are concrete enough to be fixed directly in those command bodies.

### 2026-08-20 — `/catch-up`'s git check trusts local tracking refs, and confidently reported a stale remote

**Status: finding 1 verified** (re-checked against the server and the ref reflog after the user
challenged the claim); **finding 2 asserted** (a doctrine-level observation, not a re-run
experiment).

**Finding 1 — "run `git status` and treat it as source of truth" is wrong for any claim about what
a *remote* branch contains.** `/catch-up` directs the model to establish real current state from
local git. This session that produced a confidently-wrong report on the first substantive question
asked: a `git fetch` was run at session start, returned no output, and the per-branch tips read
from local remote-tracking refs were reported to the user as "nothing new has been pushed." Four
branches had in fact moved. The pushes landed 20–40 seconds *before* the fetch completed, and the
tracking refs updated moments after the branch listing was read — so every individual command was
correct and the conclusion drawn from them was not. The error surfaced only because the user
pushed back citing a third party's contrary claim; `git ls-remote` (which queries the server rather
than reading local refs) then showed the true tips immediately, and the ref reflog showed exactly
when they had moved.

The failure mode is nasty because it is silent and self-confirming: a fetch that returns nothing
looks identical to a fetch that had nothing to return, and the model has no signal distinguishing
them. It is also worst at session start, which is precisely when `/catch-up` runs and when a
concurrent push from a collaborator is most likely to have just landed.

**Rule concluded:** the command should distinguish *local* state (working tree, staged changes,
local branch position — `git status` is authoritative) from *remote* state (what a branch actually
contains right now — only `git ls-remote` is authoritative). Any assertion of the form "nothing new
has been pushed" or "branch X is unchanged" must come from the server, and a negative claim about
remote state should be treated as requiring stronger evidence than a positive one. Reporting a
stale negative is far more damaging than reporting a stale positive, because it terminates
investigation rather than prompting it.

**Finding 2 — the doctrine's blanket "delegate all implementation, never type code yourself" has no
carve-out for investigative verification, and collides with harness rules against unprompted agent
spawning.** This session was almost entirely review work: reproducing reported defects, building
throwaway harnesses to test whether a claimed fix actually worked, and disproving one of the
orchestrator's own hypotheses empirically. Under a strict reading of the routing doctrine, every
one of those harnesses was implementation that should have been delegated. Two reasons that reading
is wrong in practice, neither of which the doctrine addresses:

1. Verification output is load-bearing evidence the orchestrator is separately required to re-check
   before relaying. Delegating it and then re-verifying it is strictly more expensive than doing it
   directly — the usual delegation economics invert.
2. The harness-level instruction in effect forbade spawning agents unless the user asked, while the
   project-level doctrine said to delegate everything. Both were active simultaneously and the
   doctrine offers no precedence rule, leaving the conflict to be resolved ad hoc each time.

**Rule concluded:** the routing table should name investigative/verification work as a distinct
category that the orchestrator does directly by default — not because it is cheap, but because its
product is evidence the orchestrator must personally stand behind. And the doctrine should state
explicitly that a harness-level prohibition on spawning agents wins over "always delegate," rather
than leaving the model to arbitrate between two instructions that both present as mandatory.

**Where it lives:** here only. Finding 1 is concrete enough to write directly into
`commands/catch-up.md` as a local-vs-remote distinction in the git-state step. Finding 2 belongs in
the orchestration skill's routing table, and is closely related to the 2026-08-19 entry's theme of
the plugin's own instructions being under-specified where they meet an existing environment.

### 2026-08-20 (second session) — Lanes' *incidental* claims are markedly less reliable than their primary output, and the verification doctrine only covers the latter

**Status: finding 1 verified** (both inaccuracies re-checked directly against the source of truth,
in both cases contradicting the lane); **finding 2 verified** (the sequence of proposals and the
final chosen design are a matter of record from this session).

**Finding 1 — the doctrine tells the orchestrator to verify the deliverable, and says nothing about
the asides bundled alongside it.** "Reports are claims, not evidence… read the diff, and re-run the
verification command" is aimed squarely at the work the lane was asked to do. It worked: every
substantive claim about the delivered change held up when re-run. What did *not* hold up, twice,
was material the lane volunteered outside its assigned scope.

An implementation lane appended a "pre-existing problem, unrelated to this task" note to its report,
naming four-plus areas of the codebase as affected. Re-running the diagnostic showed exactly one.
The same lane repeated the same overstated claim in a later report on the same branch, unprompted
and unchanged, after the first had already been checked and found wrong. Separately, the review lane
supported an accessibility finding with a prevalence claim — that the pattern in question already
existed elsewhere in the codebase — which a single search disproved: it was the only instance. That
one mattered more than it looks, because prevalence was the entire difference between "consistent
with existing convention" and "a lone deviation from it," and the orchestrator was about to relay
the former to the user.

The shape is consistent: lanes are reliable about what they were asked to do and noticeably
looser about context they offer around it — counts, prevalence, "this also affects…", "same as
elsewhere." These land in reports as supporting detail, sail past a verification step aimed at the
diff, and are exactly the sentences an orchestrator is most tempted to relay verbatim because they
sound like helpful extra diligence.

**Rule concluded:** the verification section should separate a lane's *assigned output* (verify by
re-running the stated command) from its *volunteered context* (verify independently before
relaying, or relay explicitly as unverified). A quantified or comparative claim outside the task's
scope — "N places are affected", "this already exists elsewhere", "unrelated pre-existing issue" —
should be treated as a lead to check, never as a finding to pass on. Worth noting the failure is
not random: it skews toward overstating breadth, which inflates apparent severity.

**Finding 2 — consulting the advisor at a commitment boundary works, but the architect should carry
the *fork* back to the human, not the advisor's answer.** Faced with a design problem where two of
the user's own stated constraints turned out to collide, the orchestrator consulted the advisor as
the doctrine prescribes. The advisor corrected a piece of the orchestrator's reasoning, confirmed
another, and proposed a fourth option better than the three on the table. All useful. But it
optimized strictly *within* the constraints it was handed — as it should, having no access to the
user — and its recommendation carried a real cost: a data-model change made to satisfy a
requirement.

Taking the fork to the user instead produced a materially better outcome. The user pointed out that
one of the constraints the advisor had treated as fixed was not actually a requirement at all,
which dissolved the problem and removed the need for any data-model change. The advisor could not
have known that; the constraint had been stated to it as given, and it had been stated to the
advisor because the orchestrator believed it.

**Rule concluded:** the advisor's verdict is an input to the human's decision at a commitment
boundary, not a substitute for it. When a boundary arises because the user's own stated constraints
conflict, the orchestrator should present the tradeoff and let the user relax a constraint — they
are the only party who can. The doctrine's "act on the verdict or surface the disagreement" implies
a binary that misses this third case: agreeing with the advisor's reasoning entirely while still
needing to check whether the premises it reasoned from are actually binding.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's Verification section,
as a distinction between assigned output and volunteered context. Finding 2 belongs in the
commitment-boundaries section, and extends the 2026-08-20 entry's theme of the doctrine being
under-specified where two of its own instructions meet.

### 2026-08-21 — The lessons/wrap-up loop has a write path but no read path; and two-word shorthand failed inside the message arguing against shorthand

**Status: verified** (both observed directly in the session; captured retroactively on 2026-08-23
from the transcript — the session ended without a `/wrap-up`, which is itself the third finding).

**What happened:** a long design conversation about a separate tool surfaced a structural critique of
Claude Orchestrator's own feedback loop, plus a live instance of an explanation failure the plugin
has now logged three times in different forms.

**Finding 1 — the feedback loop is a diary, not a learning system.** `lessons.md` gets written at the
end of a session by `/wrap-up`. Nothing reads it *at the moment a decision is being made*. The only
read path is a human opening the file, or `/catch-up` surfacing the resume note — both at session
boundaries, neither at decision time. A record that is never consulted when it would change an
outcome makes the author feel organized and changes nothing.

Three properties separate a diary from a loop that actually learns, and the current design has none
of them reliably:

1. **Consulted at the moment of the decision**, not just at session start.
2. **Entries specific enough to bind behavior.** "Be more careful with migrations" changes nothing;
   "migrations always come to me before running, even in dev" can actually fire. The enforceable
   shape is always *when X, do Y* — observable trigger, specific action. The stranger test: could
   someone follow it without exercising judgment?
3. **Entries carry verdicts** — whether the intervention was the system's failure or the user's
   prerogative.

This is loop design, not model capability. Note the plugin already has the deterministic layer for
the strongest version of #1 — hooks can inject context or block a tool call — and currently spends
its entire hook surface on one `SessionStart` context injector. That connects directly to the
enforcement-vs-guidance question in `RESUME-PROMPT.md` and to notes PR #9.

Related structural point: the `where it lives` field in this file is written once and never
revisited, which has already caused a stale ledger to misreport the backlog (2026-08-21 entry). A
file whose own status fields go stale is more evidence for the same finding — writing is cheap here,
re-reading is not designed for at all.

**Finding 2 — shorthand failed in the exact message arguing for plain explanation.** The phrase
"case law" was used as two words carrying a paragraph of meaning, in a message making the case that
explanations should be layered and plain. The user reported it didn't land. This is the third
recurrence of one pattern in different costumes: the 2026-08-21 resume-note entry ("dead end" read as
a behavioral claim), the compressed-negatives rule in `RESUME-PROMPT.md`, and now this. The common
shape is **a compressed reference standing in for a concept the reader hasn't been given**, and it
recurs because the author always knows what the shorthand means.

**Finding 3 — the session produced substantial durable material and none of it was swept.** The
conversation defined a communication protocol, a four-module design, and an explicit statement of the
user's own explanation preferences. It ended without a `/wrap-up`, so none of it reached this file,
the backlog, or the resume note; the transcript was the only copy for two days until recovered by
grepping `~/.claude/projects/**/*.jsonl` on 2026-08-23. Testing mode was on the whole time. Testing
mode captures nothing on its own — it injects a reminder, and the sweep is still a manual command the
user has to remember at the end of a long session, which is precisely when it is least likely to
happen.

**Fix/rule concluded:** none promoted yet — logged. The candidate rules, in order of confidence:
(1) an entry in this file should be written in the *when X, do Y* form wherever the finding supports
it, so it can bind behavior rather than describe a mood; (2) entries need the verdict distinction
(system failure vs. user prerogative vs. elective involvement) or the log optimizes toward never
bothering the user; (3) the read path is the real gap and is a hooks question, not a prose question.

**Where it lives:** here only, deliberately — all three are candidates, none has been decided. Finding
1 overlaps notes PR #9's enforcement research and should be resolved with it, not separately. The
design conversation this came from is captured in full in `assistant-design-brief.md`; the user's
explanation preferences are in `communication-preferences.md`.

### 2026-08-23 — `/catch-up` has no read path into session transcripts, and a "not documented" negative was asserted without searching them

**Status: verified** (both observed directly this session; the transcript search was re-run and its
results are what corrected the error).

**What happened:** `/catch-up` ran and produced an accurate reconciliation of the resume note against
local git, remote git, and the notes repo. It was still substantially incomplete, and neither the
command nor the orchestrator noticed. A prior session had produced a large body of durable material —
a confirmed design framing, an explicit statement of the user's own working preferences, several
reversals the user argued for — and none of it had been swept. It existed only in the harness's own
session transcript. `/catch-up` reads the resume note, git state, and the companion notes repo. It
does not read transcripts, so that material was invisible to it. It surfaced only because the user
asked a question that happened to require it, two sessions later.

**Finding 1 — the resume note is treated as the only memory of a prior session, and it is a lossy
one written at the end of a session that may never have run `/wrap-up`.** When a sweep is skipped,
the loss is total and silent: `/catch-up` reports confidently on what it can see and has no way to
signal that a session's worth of material is missing. It correctly flagged the note as stale by two
sessions — it could see the *dates* had moved on via new branches — but "stale" was reported as a
freshness problem, not as "there are two unswept sessions whose content is unrecoverable from
anything I read." The harness stores transcripts locally in a known, greppable location; the command
never looks there. This is the same write-path/no-read-path shape logged in the 2026-08-21 entry,
now confirmed at a second layer: the session-boundary commands have the same gap the lessons log has.

**Finding 2 — a negative claim was asserted from absence of evidence, in a deliverable, without
searching the evidence that was available.** A written summary of the user's preferences included an
explicit "not documented — never stated, no evidence either way" list. One item on that list had in
fact been stated directly and at length by the user, in a transcript that a single grep would have
surfaced. The negative was written first and the search was run only after the user asked a follow-up
question.

This is the third recurrence of one pattern and the second in the *negative* direction specifically.
The plugin already documents the rule — `/catch-up`'s own body says a negative claim requires real
evidence, and that a stale negative stops the investigation entirely while a stale positive merely
prompts someone to go look. The rule is scoped in the command text to git remote state. The failure
mode generalizes and the text does not: "I found no documentation of X" is exactly the same claim
shape, with the same asymmetry, and nothing in the doctrine says so. A documented "what's missing"
section is more dangerous than an ordinary claim, because it reads as the product of a search and
tells the reader to stop looking.

**Fix/rule concluded:** none promoted — logged, and the two findings want different treatments.
Finding 1 is a real feature gap in `/catch-up` (and correspondingly in `/wrap-up`, which is what
should have captured the material in the first place); the candidate is a transcript-aware discovery
step, which is genuine work and should be scoped rather than bolted on. Finding 2 is a doctrine
wording change: generalize the existing negative-claim rule beyond remote git state to any asserted
absence, and treat an enumerated "not documented" list as a claim requiring the same evidence bar as
any other conclusion.

**Where it lives:** here only, both. Finding 1 belongs with the enforcement/read-path thread — it is
the same structural gap as the 2026-08-21 entry and notes PR #9, and should be resolved with them
rather than as an isolated command tweak. Finding 2 belongs in the orchestration skill's Verification
section, adjacent to the existing incidental-vs-assigned-claims item already outstanding there.

### 2026-08-24 — Fan-out over an already-covered corpus cross-checks itself for free; and a `⛔ do-not-edit` constraint has no paired rule for work that belongs to another lane

**Status: findings 1, 2 and 4 verified** (each correction was re-checked directly against the source
of truth before being accepted, and in every case the lane was right); **finding 3 verified**
(observed directly — the handover block was read and the relocation performed by the orchestrator).

Context: a large documentation-and-analysis build. Roughly fifteen lanes across four fan-out batches,
each lane assigned an adjacent slice of one shared corpus, then a second wave of lanes that consumed
the first wave's output as input.

**Finding 1 — lanes fanned out over a corpus that earlier passes already covered will correct those
earlier passes, repeatedly, and this is the highest-value output of the pattern.** Four separate
times a lane working on its own slice found a factual error in material produced by an *earlier*
lane, or by the orchestrator itself. One found three wrong values in a chapter written two batches
prior, including the headline figure. Another corrected two "unchanged" claims that were wrong in the
user's favour. A third found that a value the orchestrator had personally recommended acting on meant
the opposite of what the orchestrator said it meant. None of these were asked for; each lane found
them incidentally while reading adjacent material with no investment in the earlier conclusion.

This is the useful inverse of the 2026-08-20 *incidental claims* entry. There, incidental output was
less reliable than assigned output and the doctrine only covered the latter. Here the same asymmetry
runs the other way: a lane's incidental *corrections of prior work* were consistently right — four for
four on re-check — precisely because they were incidental. The lane had no stake in the earlier claim,
read the primary source directly, and reported the discrepancy as an aside. The doctrine has no
instruction to expect this, so an orchestrator that does not read reports carefully will discard it.

**Finding 2 — a long fan-out's premise changed mid-flight, and only luck separated the durable half
of each lane's output from the perishable half.** Four lanes were launched under one framing; the user
reversed that framing while they ran. Their output had two separable parts: a factual index (what
exists, where it lives, what the baseline is) and a verdict column (what to do about it under the old
framing). The index survived the reversal intact and remained the input the next phase needed; the
verdicts became actively misleading and had to be explicitly demoted in writing so a later session
would not apply them. That separation was not designed — the spec happened to ask for both in
distinct columns. Had the spec asked only for recommendations, all four lanes' work would have been
unusable.

**Finding 3 — `⛔ do-not-edit` constraints tell a lane what not to touch and nothing about what to do
with work that belongs to a file it cannot touch.** A lane was correctly forbidden from editing
sibling lanes' artifacts, and correctly discovered that two of its assigned changes belonged in files
two *other* lanes owned — putting them in its own artifact would have created a silent, build-breaking
duplicate-ownership bug. It handled this well: it wrote both changes out in full inside a clearly
marked handover block in its own file, excluded them from the active output, and stated plainly in its
report that someone had to move them. That worked. But it worked entirely on the lane's own judgment.
The constraint as written admits an equally compliant response — silently dropping both changes as
out-of-scope — and two user-approved changes were one unread report away from vanishing with no error
anywhere. A prohibition without a disposal route is an invitation to drop work quietly.

**Finding 4 — the mandatory end-of-deliverable advisor review earned its cost on the one artifact the
orchestrator wrote itself.** Across the whole build the advisor returned *ship, after one fix*, and
the fix was in the orchestrator's own chapter — numbers that were derived through a model and
conditional on an unproven assumption, but carried the same "verified" label as directly-read values.
Every lane-written artifact passed. That is the expected distribution: lane output had been
spot-checked by the orchestrator, and the orchestrator's own output had been checked by nobody. The
doctrine already mandates this review; what this session adds is *where its value concentrated*, which
argues for the review being pointed at orchestrator-authored artifacts first rather than treated as a
uniform pass over the diff.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. Finding 3 is the one worth fixing soon and is a small wording change: pair every `⛔ do-not-edit`
   constraint with a required disposal route — *if work you were assigned belongs in a file you may
   not touch, write it out in full, exclude it from your active output, and name it in your report as
   requiring relocation.* This session's lane invented exactly that protocol unprompted; it should not
   have had to.
2. Finding 2 argues for a spec-shape rule for any fan-out expected to outlive its own premise:
   **ask for the durable artifact and the perishable judgment as separately identifiable outputs.**
   The index survives a reversal; the recommendation does not.
3. Finding 1 argues for a verification-doctrine addition symmetric to the 2026-08-20 incidental-claims
   entry: when lanes fan out over a corpus earlier passes already covered, **treat their unsolicited
   corrections of prior work as high-value and verify them first** — they are the cheapest real
   cross-check the pattern produces, and they arrive buried in reports as asides.
4. Finding 4 is a small emphasis change to the final-review instruction rather than a new rule.

**Where it lives:** here only, all four. Findings 1 and 4 belong in the orchestration skill's
Verification section — finding 1 sitting directly alongside the 2026-08-20 incidental-claims item it
inverts, since the two together say something more useful than either alone. Finding 3 belongs in the
spec contract, under Constraints. Finding 2 belongs in the spec contract as well, but is the least
proven of the four and should wait for a second instance before it earns doctrine text.

### 2026-08-24 — `/catch-up` inherits written state claims as fact, and a verification that reads the artifact instead of the system cannot fail

**Context:** a long-running documentation project (no fan-out this session; findings are about
`/catch-up` and the verification doctrine, not about lanes). The project maintained a "live state"
document asserting that a piece of tooling was installed and running. It was not, and had never been.
The false claim had stood for roughly two months across many sessions.

**Finding 1 — `/catch-up` has no mechanism for distinguishing a *documented* state claim from a
*verified* one, and this is the third entry in this file describing the same failure shape.**
`[verified]` The session-start routine reads a resume note and a state document, then reports what
they say. This session opened by restating a state claim from a project document as established fact.
It was false. The evidence contradicting it was not hidden or subtle — it sat in the same state
document's own machine-state table, where a prior session had accurately recorded the observable
condition that *proved* the tool was inert, labelled it as a benign configuration variant, and written
"installed" in the adjacent row. Every subsequent session, including this one's opening summary,
inherited the conclusion and never re-derived it from the recorded evidence.

⭐ This is the same shape as **2026-08-20** (`/catch-up` trusted local tracking refs and confidently
reported stale remote state) and **2026-08-23** (`/catch-up` asserted a "not documented" negative
without a read path to check). Three instances now. The common root: **`/catch-up` treats every
readable source as equally authoritative, and has no notion that some claims describe a live system
whose current truth must be *queried*, not *read*.** The 2026-08-20 entry already fixed this for one
specific case by mandating a server query for remote git state. That fix was correct but was scoped as
a git rule rather than as the general principle it actually is.

**Finding 2 — "verify with evidence" does not distinguish evidence produced by the artifact from
evidence produced by the system, and a check of the first kind passes unconditionally.**
`[verified]` The project had a documented verification procedure that was run repeatedly across many
sessions and reported success every time. The procedure inspected the build artifact's own output
files and confirmed they contained the intended values. They always did — the build was genuinely
correct. But the artifact's contents are identical whether or not anything downstream ever consumes
it, so the check could not fail, and it was labelled as confirmation that the change was live. A
single different check — *does the consuming system actually have the artifact* — was one command,
available from the start, and would have caught the failure immediately.

Re-verified this session by running both: the artifact check passed (values correct), the system check
failed (artifact absent from the consumer's load path). Also worth recording as a method note: when a
verification tool errored on the artifact, running it against a **known-good control and a known-bad
control** was what separated "tool doesn't support this input" from "artifact is broken" — it was the
former, and without controls that error would have been misreported as a defect.

**Finding 3 — when a pipeline's final delivery step lies outside the tool's own scope, "done"
silently truncates to the tool's definition, and the orchestrator reports success at a boundary the
user never agreed to.** `[verified]` The third-party build tool defines "build" as *produce an
artifact*. The user's definition — the operative one — was *the change is live in the running
system*. The project's own written procedure encoded the narrow definition, defining the deployment
step as a file copy that in fact deployed nothing. Across the session the orchestrator repeatedly
reported "the build succeeded," which was true under the tool's definition and false under the user's.
When the gap surfaced, the orchestrator's first framing was *"the builds were fine, the install was
the problem"* — a distinction the user had never accepted, which functioned as an excuse rather than
an explanation. The user pushed back on the terminology directly and was right to.

**Finding 4 — a user asking materially the same question a third time is a signal to go look, not to
restate.** `[asserted]` The user asked the same scoping question three times in succession. The first
two answers restated a design principle already established earlier in the session; both were
defensible and neither involved inspecting anything. The third time the orchestrator actually examined
the artifacts and immediately found several real defects — stale trackers claiming work was unapplied
when it was live, an internal contradiction between a file's header comment and its own contents, and
an off-by-one count — none of which the principle-based answers would ever have surfaced. Repetition
was the user correctly detecting that the answers were being generated from doctrine rather than from
evidence.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. **Finding 1 is ready for doctrine and should generalize the 2026-08-20 git-specific rule.** Proposed
   wording for `/catch-up`: *a claim about the current state of a live system is not established by
   reading it in a document, however authoritative the document. Documents record what was believed
   true when written. Where a state claim is load-bearing for the session's next action, either query
   the system directly or report it explicitly as "documented, not verified."* The existing git rule
   becomes one instance of this rather than a standalone item. Three occurrences now; this is the
   highest-confidence item in this entry.
2. **Finding 2 belongs in the Verification section** and is close to a one-line addition: *prefer
   evidence from the consuming system over evidence from the produced artifact. An artifact check
   proves what a thing would do; only a system check proves what is happening. If a check cannot
   distinguish success from failure, it is not a check.* The control-testing note is a worthwhile
   sub-point: **before reporting a tool error as a defect, run a known-good and known-bad control.**
3. **Finding 3 argues for an explicit definition-of-done rule** wherever a pipeline's last step is
   outside the tool's scope: *state the completion boundary in the user's terms, not the tool's, and
   when reporting partial completion name what remains rather than reporting success against the
   narrower definition.* Lower confidence only because one instance.
4. **Finding 4 is a behavioral note, not doctrine text** — but it pairs naturally with finding 2: both
   describe answering from a model of the thing instead of from the thing. Worth a second instance
   before it earns wording.

**Where it lives:** here only, all four. Finding 1 belongs in the `/catch-up` command text, replacing
the git-scoped rule with the general one and keeping git as its worked example. Finding 2 belongs in
the orchestration skill's Verification section, adjacent to the 2026-08-20 incidental-claims entry.
Findings 3 and 4 stay here pending a second instance.

### 2026-08-24 — Parallel read-only agents: two contradicted each other on a checkable fact, and one inherited a stale premise from an orchestrator-written doc

**Context:** three read-only exploration agents were dispatched in parallel to survey different
areas of an unfamiliar codebase ahead of a design task. The orchestrator had already established
most of the domain facts itself and deliberately narrowed the fan-out to only genuinely unknown
areas, telling the user explicitly what it was *not* delegating and why. That scoping worked —
none of the three re-derived held context, and their areas did not overlap.

**Finding 1 — two agents surveying adjacent areas reported contradictory facts about the same
interface, and only a direct check resolved it.** `[verified]` One agent reported that a server
interface exposed a full set of verbs including a delete operation. A second agent, covering a
neighbouring area, reported the same interface exposed no delete operation. Both stated it flatly,
neither hedged. The orchestrator re-read the source directly: the second agent was right. The first
had asserted a complete verb set without enumerating it.

The same agent's report also placed a real defect in the wrong file — the finding itself was
accurate and the line number happened to match, but the file was a different one in the same call
chain. Two accuracy failures in one report, both caught only because the claims were load-bearing
enough to re-check before being written into a planning document.

Worth noting what made this recoverable: the contradiction was *visible* because two agents happened
to cover overlapping ground. Where a single agent owns an area alone, a confidently-stated wrong
fact has nothing to collide with.

**Finding 2 — an agent discovered a document the orchestrator had written earlier in the same
session and treated it as authoritative spec, including a premise the user had already overridden in
conversation.** `[verified]` Mid-session the orchestrator wrote a reference document capturing a
transcribed external source. It recorded, accurately, that the source specified a particular field as
unstructured free text. Some turns later the user made an explicit decision to override that and use
a properly typed input instead. That decision lived only in the conversation.

A subsequently-dispatched agent found the document, correctly identified it as the spec for the work,
and mapped its own findings against it — reporting that field as "already covered, no work needed."
Under the document it was right. Under the actual decision it was wrong, and it had no way to know.

The failure is not the agent's. It is that **orchestrator-written artifacts become authority for
agents while remaining stale relative to decisions made in conversation after they were written.**
The orchestrator is the only party that can see both. This is a general hazard of the
write-a-doc-then-delegate pattern, and it gets worse the longer a session runs and the more
decisions accumulate verbally.

**Finding 3 — the orchestrator announced a verification in prose and ended the turn without
performing it.** `[asserted]` Having identified the contradiction in Finding 1, the orchestrator
wrote that it was checking, and then produced no tool call at all. The turn simply ended. The user
had to prompt for recovery, and the recovery prompt reasonably assumed some error had occurred —
there was no error, only an announced action that never happened. Recorded as `asserted` because the
cause was not diagnosed, only the effect. Related in shape to the standing rule about claiming a tool
action completed without confirming it landed, but distinct: here the claim was prospective rather
than retrospective, which is if anything easier to make and equally misleading.

**Fix/rule concluded:** none promoted — logged. Candidates, in descending confidence:

1. **Finding 2 is the most generalizable and is close to ready.** Proposed wording for the
   orchestration skill's delegation section: *when handing an agent a document you wrote earlier in
   the session, state in the spec which parts have been superseded by later decisions, or re-read the
   document yourself before dispatching. An agent treats your artifacts as authority and cannot see
   the conversation that has since moved past them.* First instance, but the mechanism is clear and
   the blast radius is large — a stale premise propagates silently into every downstream report.
2. **Finding 1 reinforces the existing verify-agent-claims discipline** rather than adding to it, with
   one new sub-point worth wording: *a claim about the completeness of a set — all the verbs an
   interface exposes, all the places a symbol appears — should be reported as an enumeration, not a
   summary. "Full CRUD" is a conclusion; the list of verbs is the evidence.* Also worth noting for
   fan-out design: deliberate overlap between agent areas is what surfaced the contradiction, so
   perfectly disjoint scoping trades away a real error-detection mechanism.
3. **Finding 3 is a behavioral note, not doctrine text.** One instance, cause undiagnosed. Worth a
   second occurrence before it earns wording. If it recurs, the rule is roughly *do not narrate an
   action you have not taken; either take it in the same turn or say you will take it next.*

**Where it lives:** here only, all three. Finding 2 belongs in the orchestration skill's delegation
guidance if it recurs or if the reasoning is accepted on its merits. Finding 1's enumeration
sub-point belongs alongside the existing verification material. Finding 3 stays here pending a second
instance.
