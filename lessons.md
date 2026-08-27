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

### 2026-08-25 — A research lane's chain of custody broke silently, and its relayed citations were accurate in substance but wrong in one figure

**Status: verified** (four primary sources fetched directly this session and the quotes checked
against the relayed text).

**What happened:** the user asked for research before committing to a design change, and three
lanes were dispatched in parallel. One of them returned a report that opened by referencing "my
original report" and "my earlier finding" — neither of which had ever reached the orchestrator. The
lane had delegated further, and only an addendum came back; the base report was lost somewhere in
the chain. **The lane flagged this itself**, stated plainly that it had not independently verified
the citations it was relaying, and recommended re-verification before anyone acted on them.

Spot-checking four load-bearing citations against the primary sources found the substance accurate
— the papers were real, the quoted passages present, the authors as described. But one figure was
wrong: a degradation endpoint relayed as 21% is reported as 15% in the source. Small in isolation,
and it sat in the exact sentence being used to justify a design constraint.

**Two distinct failures, worth separating.** The first is structural: a lane that delegates has a
chain of custody, and a broken link in it is *invisible from the outside* unless the lane
volunteers it. This one did volunteer it, which is the only reason it was caught. A lane that had
simply presented the addendum as its own findings would have been indistinguishable from a
complete report — the same trap as an empty `fetch` or a scoped `ls-remote`. The second is the
familiar one: relayed figures drift, and they drift in the direction of being more striking than
the source.

**Rule concluded:** when a lane's output will enter a document the user makes a decision from,
verify the load-bearing figures against primary sources before relaying them — not the whole
report, just the numbers that are actually carrying weight. And treat a lane's silence about its
own method as uninformative rather than reassuring; a lane that delegated and a lane that did the
work itself produce identically-shaped reports.

**Where it lives:** here only. This is a third occurrence of the citation-confidence theme
(2026-08-21 and 2026-08-22 entries) and the pattern is now consistent enough to justify a direct
change to the orchestration skill's Verification section rather than another log entry.

### 2026-08-25 — The orchestrator disqualified evidence that contradicted it, then nearly accepted evidence with the identical defect because it agreed

**Status: verified** (caught in-session, before the reasoning reached the user's decision).

**What happened:** the orchestrator ruled out a large body of evidence on a category test — it
measured a different situation than the one under discussion, so its conclusions did not transfer.
That reasoning was sound and it was the right call.

A later lane returned evidence with **the same defect**, from the same category, measuring the same
irrelevant situation — but pointing toward the recommendation the orchestrator had already made.
The reflex was to relay it as support. It took a deliberate second pass to notice that the filter
used to dismiss the first body of evidence disqualified this one just as completely.

**Why this is worse than an ordinary reasoning slip.** A filter invoked to dismiss inconvenient
evidence and then not applied to convenient evidence is not neutral analysis — it is a mechanism
for laundering a prior conclusion through the appearance of research. The user had specifically
commissioned the research to check whether an existing decision was well-founded. Selectively
applied skepticism would have returned exactly the answer the orchestrator already held, dressed in
citations, which is worse than having done no research at all.

**Rule concluded:** a disqualifying test, once used, is standing. The moment evidence is dismissed
on a structural ground, that ground becomes a test every subsequent piece of evidence must pass —
confirming evidence first, because that is where the check will not happen on its own. When
relaying research that survives such a filter, state the filter and state that it was applied in
both directions; if it was not, the conclusion is not yet supported.

**Where it lives:** here only. Belongs in the orchestration skill's Verification section, adjacent
to the existing guidance on negatives requiring evidence — this is the same asymmetry problem
pointed at the orchestrator's own reasoning rather than at a claim about the world.

### 2026-08-25 — `/wrap-up`'s dirty-tree guard fired on an artifact the orchestrator had just written, and the narrowing fix is still sitting unmerged

**Status: verified** (observed while running the command, this session).

**What happened:** the user asked for research findings to be documented durably, so the
orchestrator wrote a reference document into the notes repo working tree and deliberately left it
uncommitted, flagging to the user that they should decide how it lands. At `/wrap-up`, the
dirty-tree guard aborted the PR step — as written, correctly, since the tree was not clean.

But the only dirt was **a single untracked file that could not have been swept into the commit**,
because the commit step stages one named file. The guard's stated rationale is avoiding sweeping
someone else's unfinished edits into a commit, and that risk was not present. The fix narrowing
this guard to a real conflict on the target file was written several sessions ago, is correct, and
has been sitting in an unmerged pull request ever since — so the running command still has the
broad version. **The command hit precisely the case its own pending fix exists to handle.**

**The second-order finding is the more useful one:** the orchestrator created the condition that
tripped the guard. A command that writes durable artifacts into the notes repo mid-session is, by
doing so, disabling the sweep step that runs at the end of that same session. Nothing in the
current design connects those two facts, so the collision is only discovered at the end, when the
options are worse.

**Rule concluded:** two separate things. The narrowing fix should ship before any further work
touches this command — an unmerged fix changes nothing, and this is now the second session where
that has had a concrete cost. And when the orchestrator writes a durable artifact into the notes
repo during a session, it should either commit it at the time or tell the user explicitly that the
end-of-session sweep will be blocked by it — the warning is nearly free and the discovery cost at
wrap-up time is not.

**Where it lives:** here only. The first half is already implemented in the open PR and needs
merging, not authoring. The second half is a new clause for the command's artifact-writing
behavior, and it generalizes: any orchestrator action that dirties the notes repo has a delayed
cost that surfaces only at wrap-up.

### 2026-08-25 (found on a user-initiated re-check) — The resume note failed twice in two sessions, in two different directions, and both failures were invisible from inside the session that caused them

**Status: verified** (failure 1 confirmed by file mtime against pull-request creation timestamps;
failure 2 confirmed by grepping the freshly-written note for the dropped material).

**What happened — two distinct failure modes of the same artifact, discovered one after the other.**

**Failure 1, discovered at `/catch-up`: the note was silently a full session stale.** Its own header
claimed a verification date, and the state it described was internally consistent. But its mtime
predated two pull requests that a later session had opened — a session that demonstrably ran the
wrap-up command, since opening those PRs is something only that command does. So the command ran,
performed its logging step, and did not rewrite the note. Nothing in the note indicated this. A
session trusting it would have believed a stale queue depth and a stale "next action," both stated
with full confidence.

**Failure 2, discovered only because the user asked for a re-check: the rewrite silently dropped a
live thread.** At the end of a long session dominated by one topic, the note was rewritten from
what was salient in that session. The previous note had carried **two** parallel threads; the
rewrite carried one. The dropped thread was not minor — it contained a decision protocol the
previous note explicitly labelled that session's most load-bearing outcome, five open questions, a
list of settled decisions marked don't-re-litigate, and the exact commands needed to reach source
files that exist nowhere on the default branch. All of it was recoverable only because the old note
had been read into context earlier in the same session; the file itself had already been
overwritten, and it is not tracked in git.

**The common cause, and it is the interesting part.** The command writes the note by composing from
session memory rather than by auditing the artifact it is replacing. That works when the session
covered everything the note contains. It fails silently whenever the session was *focused* — the
narrower and more productive the session, the more of the prior note gets dropped, because
salience is exactly the wrong selection function for a durable handoff record. Failure 1 is the
same defect one step earlier: not composing at all when the session's attention was elsewhere.

**Neither failure is detectable from inside the session that commits it.** The note that results
reads as complete and internally consistent in both cases. Only a diff against the previous version,
or an outside prompt to re-check, surfaces it. This is a third occurrence of the resume-note
reliability theme (2026-08-21 clobber-and-fabricated-state, 2026-08-21 shorthand-and-stale-ledger).

**Rule concluded:** overwriting a handoff note is a merge, not a write. Read the existing note
first, enumerate the threads it carries, and for each one either carry it forward or state in the
new note that it was deliberately closed — dropping it silently is not an available option. A
cheap mechanical check gets most of the value: after writing, grep the new note for the distinctive
identifiers in the old one and confirm every absence is intentional. And since a stale note is
indistinguishable from a current one, the note should carry a marker the reader can falsify against
live state rather than a self-reported verification date.

**Where it lives:** here only. This is a direct change to the resume-note step of the wrap-up
command — read-then-merge instead of compose-then-overwrite — and it pairs with the already-logged
finding that the same command composes from memory instead of auditing state. Both are the same
underlying defect in different steps, which strengthens the case for fixing the pattern rather than
the instances.

### 2026-08-26 — The "don't merge, batch at the version bump" rule quietly broke a citation, and a second session's wrap-up landed in the same shared file with no owner

**Status: verified** (finding 1 confirmed by `git log --all -- <file>` plus `git branch --contains`,
then reading the file out of the branch; finding 2 observed directly — `git status` on the notes
repo was clean at `/catch-up` and dirty at `/wrap-up` in the same session, with entries the
orchestrator had not written).

**Finding 1 — an instruction pointed at a file that does not exist where it said.** A project's
resume note carried a standing instruction to read a specific document in the notes repo, by
absolute path, at the start of every session, and emphasised that the document is actively
maintained so a summary of it should not be trusted. That path does not resolve. The document has
never been on the default branch: it lives only on one of the accumulated unmerged `lessons/*`
branches, alongside the entry it shipped with.

A session that follows the instruction literally gets a missing file. The failure is quiet in the
worst way — the instruction reads as satisfiable, and the natural recovery is to proceed without the
guidance and never mention it, because nothing distinguishes "this file is missing" from "this
instruction was optional." (It was recoverable here only because the search for it happened to
surface the branch.)

**This is a direct consequence of the wrap-up command's own design.** The command is explicit that
its pull requests are not to be merged — they are reviewed in a batch at the next version bump. That
is a reasonable rule for *entries*, which are append-only logs nobody cites. It is not a safe rule
for *documents*, which are written to be pointed at. The same commit flow produces both, and nothing
in the command distinguishes them. The repo's own most recent commit on the default branch is a note
flagging that two cited files live only on an unmerged branch — so the symptom was already known and
recorded, while the mechanism that produces it kept running.

**Finding 2 — concurrent sessions collide in the shared log, and the loser is silently demoted.**
The notes repo was clean when checked at the start of this session and dirty at the end, carrying
two entries dated the same day that this session did not write. The dirty-tree guard fired and
correctly declined to branch. But the outcome is that this session's entry is left uncommitted, in a
file that already contains another session's uncommitted entries, with no branch, no pull request,
and no record of which session authored which block. The user is told to "sort it out themselves,"
which is exactly the situation the guard exists to avoid creating.

The guard is doing the right thing locally. The gap is that the command has no notion of *another
wrap-up* as a distinct cause of dirtiness — it treats a peer session's in-flight log entry the same
as a stray unrelated edit, when the correct handling is arguably the opposite (both entries belong
in the same commit, or in two branches cut from the same clean base).

**Rule concluded:** two things. First, separate the two kinds of artifact the sweep produces — an
appended log entry can safely sit unmerged, a document that anything else cites cannot, and a
document that is going to be referenced by path should land on the default branch at the time it is
written, not at the next version bump. Second, before a session emits an instruction naming an
absolute path into the notes repo, the path should be resolved against the default branch rather than
against whatever the working tree happens to contain — the working tree is exactly where an unmerged
file is still visible, which is what makes this failure so easy to author and so hard to notice.

**Where it lives:** here only, and both halves are command-level changes rather than doctrine. The
first is the more urgent: it is a correctness bug in how the plugin's own outputs are published, and
it has already produced one broken citation and one commit acknowledging it without fixing it.

---

## 2026-08-26 — A resume note pointed at a behavioural doc, the session read part of it, and behaved as if it had read all of it

**Status: finding 1 verified** (the truncated read and the unread section are directly observable in this
session's own tool calls — the document was fetched with a head-limit, and the remainder was read only after
the user objected). **Finding 2's observation verified, its causal claim asserted** — that output shape
degraded across the session was observed directly, and the user stopped work to correct it; that the
degradation tracked *volume of findings* rather than elapsed time or context pressure is an interpretation
of one session, not something re-run or controlled for.

**Context, project-agnostic.** A project's resume note carried a standing instruction: *read the
user's communication-preferences document at the start of every session*, with a command to fetch it.
The session ran that command, read the first ~120 lines of a ~260-line file, and summarised the rules
it had seen. It then spent the rest of the session violating the half it had not read, until the user
stopped work and asked for the format explicitly.

**Finding 1 — "read this doc" is an unverifiable instruction, and partial compliance is invisible.**
A resume note can direct a session to a document, but it cannot tell whether the session read the
whole thing. Worse, a *partial* read is more dangerous than a skipped one: the session comes away
with genuine, correct rules and therefore with confidence, while the governing section sits below the
cut. Here the unread portion contained the part that determined the *shape* of every reply, so the
rules that were read could not compensate for the one that was missed. The session had no signal that
anything was wrong — the instruction had been followed, by its own reckoning.

**Rule concluded:** a resume note that cites a document for *behaviour* should state its length and
say explicitly that partial reads do not count. Cheaper and more robust: inline the two or three rules
that most change output, so the pointer is an enrichment rather than a dependency. A pointer alone
makes correct behaviour contingent on a read the note cannot verify.

**Finding 2 — behavioural compliance decays across a session in proportion to how much the session finds.**
This is the more generalizable half. Early replies followed the format. They degraded steadily as
research accumulated — and the degradation tracked *volume of findings*, not elapsed time or context
pressure. The more a session discovers, the stronger the pull to present all of it, and the further it
drifts from any instruction about restraint, compression, or shape.

That inverts the usual assumption. Sessions that do a lot of successful work are treated as the good
case; for output-shape instructions they are the **high-risk** case, because the instruction competes
with a growing pile of things that genuinely seem worth saying.

**Rule concluded:** instructions that govern *how output is shaped* cannot be discharged by a
session-start read, because they are re-decided on every message while the pressure against them grows
monotonically. They need a per-message check cheap enough to actually run — here, *"does this reply
open with plain language, or with a table, a header stack, or a field name?"* A one-line mechanical
test caught every bad message in the session retrospectively and would have caught them prospectively.

**Where it lives:** here, plus the affected project's own lessons file and resume note. The
generalizable piece is for `/catch-up` and `/wrap-up`: both write and consume resume notes that lean on
"read X" pointers, and neither distinguishes a *reference* pointer (fine to cite) from a *behavioural*
one (must be inlined or length-stamped, because unverifiable compliance is the whole failure).

## 2026-08-26 — A spec that demands gated verification forces the agent to break a rule or fail the task

**Status: verified.** Re-read both agents' final reports and the two specs I had sent them, and
confirmed the difference in behaviour tracked the difference in wording, not the agents.

**Context, project-agnostic.** The project had a standing user rule: ask before invoking any
browser-automation tool. I delegated a batch of UI fixes and wrote, for two of the items, that the
agent should "reason about real rendered layout — neither is catchable by type-checking." Both items
were only verifiable by measuring a live page. The spec never said whether a browser was permitted.
The agent launched a headless browser to measure, and disclosed it unprompted, explicitly flagging
that it was unsure about the convention.

**Finding — the contrast is the diagnostic, and it exonerates the agent.** An earlier lane on the
same work, under the same standing rule, had *declined* to pick a browser driver and reported the
verification as still owed. Same repo, same rule, opposite behaviour. The variable was the spec: the
earlier brief had not demanded runtime proof, so honouring the rule cost it nothing. The later brief
demanded evidence that only the gated capability could produce.

A subagent has no user to ask. "Ask before doing X" is therefore unenforceable at the lane level: when
a spec requires X, the lane's only options are to violate the gate or fail the task. Both outcomes
belong to whoever wrote the spec. The rule silently converts into "the lane decides."

**Rule concluded:** before dispatching, check whether any verification the spec demands requires a
capability that is gated for the orchestrator. If so, resolve it *in the spec* — either grant it
explicitly with its constraints stated, or instruct the lane to stop and report what it could not
verify, so the orchestrator performs that step itself under the approval it already holds. Never
leave it to inference. This is capability-agnostic; the same omission around a higher-blast-radius
tool or a destructive command would be an incident rather than a harmless disclosure.

**Where it lives:** here, plus the affected project's own lessons file and the relevant memory. The
generalizable piece is for the orchestration doctrine's spec template: a "verification" section that
demands evidence should be checked against the orchestrator's own permission surface before it is
sent.

## 2026-08-26 — Naming a planning document "authoritative" makes its stale sections a false-positive generator

**Status: verified.** Re-read the two flagged items against the current source and confirmed both were
deliberate, current decisions rather than deviations; also confirmed the reviewer's four other findings
were real by reproducing the failing interaction in a browser.

**Context, project-agnostic.** A large implementation was produced in one bulk pass, deliberately
bypassing incremental human review, on the agreement that a thorough adversarial review would stand in
for the skipped gates. I dispatched a review lane and told it that a specific planning document was
"the authoritative spec — do not re-litigate its decisions; implement them." The review returned six
findings.

**Finding 1 — two of the six were false, and both traced to the document rather than the code.** They
were flagged as silent deviations from the plan's stated decisions. Both were in fact deliberate
choices made *after* that document was written, superseding it, based on evidence the document
predated. The reviewer had no way to know: I had told it the document was authoritative, and it was
diligent in comparing against it.

This is the third logged instance of a review lane being misled by superseded source material. The
recurring shape is not the lane trusting a bad source — it is the *orchestrator* conferring authority
on a source without checking whether it is current.

**Rule concluded:** when a spec cites a document as authoritative, state its as-of date and name any
sections known to be superseded — or, better, say which decisions the *dispatch itself* supersedes.
An authority grant is an instruction to stop thinking about whether the source is right, so it must
only be given for the parts that still are.

**Finding 2 — the same review comfortably earned its keep, and this half should not be lost.** Its
top finding was a genuine, user-visible defect in the exact controls that were about to be
demonstrated: a validation-timing configuration that raised errors on fields *while they were being
answered correctly*, and would not clear them. Confirmed by reproducing it. It was invisible to type
checks, lint and server-rendered markup inspection, all of which passed clean, and it existed in work
that had skipped incremental review by agreement.

**Rule concluded:** an adversarial end review is an adequate substitute for step-by-step gates when
the work is well-specified and the review is genuinely adversarial — but only if the orchestrator then
triages the output rather than relaying it. Four of six findings were real; two were artefacts of the
brief. Relaying all six as defects would have sent an implementer to "fix" two deliberate decisions.

**Where it lives:** here. Relevant to the orchestration doctrine's guidance on when the advisor pass
can replace intermediate review, and to how findings are relayed to the user.

## 2026-08-26 — "Verified" is a claim about one path, and volatile facts get the least rigour

**Status: verified.** Both findings were reproduced first-hand after a context-free agent reported
them — the data-loss path by driving the real UI and reading the database before and after, the stale
file by reading it directly.

**Context, project-agnostic.** At the end of a long build session, with a high-stakes handoff the next
morning, I wrote a runbook and a resume note, then pointed a fresh agent with no inherited context at
them and asked where the documentation failed it. It found two errors that would have caused visible
failure in front of a client. Both were mine, and both had been written down as verified facts.

**Finding 1 — I verified one code path and documented it as the behaviour.** A form field rendered
blank because the stored value was in a format the browser control rejects. I tested whether saving
destroyed the value: I opened the dialog, saved without touching the field, and the value survived.
I wrote "it does not save wrong — verified" into the runbook.

The fresh agent traced a *different* path through the same component and predicted data loss. I
reproduced it: clicking into the blank field and tabbing away — no typing — caused the save to
overwrite the stored value with empty, with a success confirmation shown. The form library reads the
control's value on blur, and the browser had already sanitised the unparseable value to empty.

The distinction I collapsed: my test established *"saving without touching this field is safe."* I
wrote *"saving is safe."* The word "verified" then made the broader claim unfalsifiable to anyone
downstream, because it signalled the question was closed.

**Rule concluded:** a verification claim must name the path exercised, not the conclusion drawn from
it. "Verified: saving without focusing the field preserves the value" is honest and invites the
obvious next question. "Verified: it does not save wrong" forecloses it. When writing a safety claim
into a document others will act on, state the interaction tested — if that reads as awkwardly narrow,
that narrowness is the actual state of the evidence.

**Finding 2 — the volatile operational facts got the least rigour, and they were the ones that
mattered.** The same documents contained analytical claims — counts derived from a large source
dataset — that the fresh agent recomputed from the raw source and found exact to the row. The claims
that were *wrong* were all operational: which directory a service should run from, which of two
copies of a data file was current, whether a dependency was actually pinned.

The pattern is that analytical claims *feel* like claims and get checked, while operational facts feel
like context and get typed from memory at the end of a session, when attention is lowest. They are
also the ones with same-morning consequences: an incorrect derived statistic starts an argument, an
incorrect directory starts a service against stale data in front of a client.

A related instance in the same document: a deliberate dependency-version decision was recorded in the
resume note, which is overwritten every session, while the file it described kept a range that would
silently undo it. Intent recorded in a disposable artefact is not recorded.

**Rule concluded:** treat operational facts as the highest-risk category in any handoff document, not
the lowest. Anything of the form *"run X from Y"*, *"Z is the current copy"*, or *"W is pinned/locked/
disabled"* should be re-checked against the machine at write time, not recalled — and where a decision
must outlive the document, encode it in the artefact it governs rather than describing it in prose.

**Where it lives:** here, and the affected project's runbook and resume note were corrected. For
`/wrap-up` specifically: the resume-note step should push operational facts toward verification-at-
write-time, and should discourage recording durable intent in a file whose own header says it is
overwritten each session.

## 2026-08-26 (later session) — A pipeline stage with no completion check stalls silently, and the operator's belief that it ran is not evidence

**Status: verified.** The version gap was read directly off the installed copy on disk, and the
merge state was read from the server rather than from local tracking refs.

**Context, project-agnostic.** Shipping a change from a repository into the sessions that execute it
is a four-hop chain: merge, publish, refresh the package index, install. Three of those hops report
completion. The fourth — whether the running process is now executing the new code — reports nothing
at all, and no hop verifies its predecessor.

The result compounded invisibly. The plugin the session was actually running was **three releases
behind** the default branch. Every session in between had inherited old behaviour while the
repository looked healthy, and nothing anywhere would have surfaced it. Separately, the operator
stated that a pull request had been merged; it had not. A one-line server query disproved it. The
belief was sincere and the interface had given no contradicting signal.

**Finding 1 — the failure is silence, not error.** Every command in the chain exited zero. A stage
that cannot fail loudly will fail quietly for as long as nobody thinks to look, and "nobody thinks to
look" is the default state of a working system.

**Finding 2 — there are three states here, not two, and the third is the one a naive check misses.**
An obvious drift check compares the installed version against the published one. That comparison is
blind to the window between installing and restarting, where the files on disk are current and the
running process is not. A two-way check reports "up to date" while old code keeps executing. That is
the original bug reproduced one level down, inside the fix for it.

**Finding 3 — the operator's report of a completed step is a claim, not evidence, and it is cheap to
check.** Asking the server directly cost one command and overturned the premise of the next three
actions. The temptation to accept it is that disputing a human's account of their own action feels
like a discourtesy; it is not, because the interface they acted through may have failed silently and
they would not know.

**Rule concluded:** a stage that changes what a session executes must end with a check that reads the
executing artifact, not the command's exit code. Where a chain of stages exists, the last one needs
that check most, because it is the only one whose failure is invisible from every other stage. When
someone reports having completed an out-of-band step, verify it from the authoritative source before
building on it — silently, without ceremony, and say plainly when it disagrees.

**Where it lives:** a session-start drift check was built and shipped, comparing running / installed /
available and reporting the restart case first. That covers this instance. The general rule — verify
the executing artifact, not the exit code — belongs wherever multi-stage delivery is described.

## 2026-08-26 (later session) — A harness command was recommended without being verified, and this is the same failure that already has a standing rule against it

**Status: verified.** The command's absence was confirmed by inspecting the installed binary, which
also revealed the correct mechanism and the correct settings key.

**Context, project-agnostic.** After installing a configuration file for the user, the orchestrator
told them to run a specific slash command to activate it. The command did not exist in their version.
The recommendation came from trained-in knowledge about the tool, stated as fact, with no check.

What makes this worth logging rather than shrugging off: **there is already a standing instruction to
verify harness behaviour empirically rather than relying on trained knowledge**, and it did not fire.
The failure mode is not ignorance of the rule. It is that a command name feels like a fact rather than
a claim, so it never gets routed through the verification path at all.

The recovery was cheap and should have been the first move: the tool's own binary was searchable, and
one pass over it produced the real answer (the command had been folded into a general configuration
surface), the settings key, and enough confidence to write the setting directly instead of sending
the user hunting through a menu.

**Rule concluded:** the name, existence, and syntax of any harness affordance — a command, a flag, a
settings key, a file path the tool reads — is a claim about a specific installed version, not general
knowledge. Check the installed artifact before telling a user to type something. When a recommendation
does turn out to be wrong, the corrective move is to search the shipped binary or package rather than
to guess a second time; a second guess after a failed first is where confidence outruns evidence
fastest.

**Where it lives:** here. The standing rule already exists and was not enough on its own, which
suggests the gap is in recognising *what counts as a claim*, not in the willingness to verify one.

## 2026-08-26 (later session) — A rule written in two places disagreed with itself, and the copy that executes won silently

**Status: verified.** The contradiction was read from both files directly, and adherence was counted
across thirty pull requests in two repositories.

**Context, project-agnostic.** A contributor-facing document specified an output format in detail. A
command that the agent actually executes contained its own, shorter paraphrase of the same rule — and
the paraphrase said the opposite: *keep it to a heading and one sentence.* Adherence to the documented
format was 3 of 11 in one repository and 4 of 19 in another, and every compliant instance had been
written by hand rather than produced by the command.

**Finding 1 — the executing copy wins, and nothing announces the conflict.** The agent following the
command was not disregarding the doctrine. It never saw it. Neither command file contained a single
reference to the document, the format, or its vocabulary. A rule that lives only in a file nothing
loads is not a soft rule; it is an absent one.

**Finding 2 — the fix is not better prose in the document.** Restating the rule more carefully in the
place already being ignored is doing the failing thing harder. The fix is to collapse to one
definition and put it where the executing path loads it, then have the human-facing document point at
that rather than restate it.

**Finding 3 — a pointer that summarises is still a second copy.** Reducing a document's section to
"see X" plus a convenient outline reintroduces the drift in miniature, because the outline can go
stale. Where an outline genuinely earns its place for human readers, state the precedence explicitly:
name which copy is authoritative and say the other is stale if they differ. The previous arrangement
did not fail because it had two copies; it failed because it had two copies and no rule about which
one won.

**Rule concluded:** state a behavioural rule exactly once, in the artifact the executing path loads.
Everything else points at it. When two copies are genuinely warranted, precedence must be written
down inside both of them — an undeclared duplicate resolves itself arbitrarily and silently.

## 2026-08-26 (later session) — Two delegated claims outran their evidence in the same session, in opposite directions, and both were caught by opening the cited file

**Status: verified.** Both were checked against the primary sources the lane itself cited.

**Context, project-agnostic.** A read-only research lane produced a long, well-structured audit. Its
load-bearing claims were spot-checked before being relayed. Most held. Two did not, and they failed
in different ways worth separating.

**Finding 1 — severity inflation on a claim the source itself refutes.** The lane flagged two
passages in one document as a *direct tension* and named it the document's highest-severity defect.
Opening the first passage showed it explicitly cross-referencing the second and warning the reader
about exactly the tradeoff. The underlying observation was real but much weaker: the caveat sits far
below the requirement it qualifies. **Severity inflation has now recurred across many sessions and
several lanes; it is a standing property of delegated review, not an occasional slip.**

**Finding 2 — purpose inferred from file properties and reported as fact.** The same lane
recommended deleting a file as stray, having inferred that from its being empty at scan time and
excluded from version control. It was neither stray nor accidental: it was a deliberate input channel
the user writes into, excluded on purpose. Every observable property was correct; the conclusion drawn
from them was not.

The shared shape: **"unused," "stray," "contradictory," and "high-severity" are all conclusions, not
observations.** A lane can establish that a file is empty and excluded, or that two passages exist. It
cannot establish what a file is *for*, or that two passages *conflict*, without reading further than
the properties.

**Rule concluded:** before relaying a delegated finding, re-open the primary source for any claim that
carries a severity judgment or attributes purpose. Verifying a lane's *observations* is often
unnecessary; verifying its *characterisations* is where the errors live. A lane's confidence is
uncorrelated with which of the two it is doing.

## 2026-08-26 (later session) — Destructive version-control work was fully built before the environment refused to run it

**Status: verified.** The permission boundary was established by attempting progressively narrower
operations until one succeeded.

**Context, project-agnostic.** A migration was designed, scripted, and dry-run successfully across
fourteen branches — every branch verified as a clean append, extraction confirmed lossless, backups
planned. The environment then refused to execute it, because the migration required force-pushing
branches and the sandbox blocks destructive version-control operations.

Nothing was wasted permanently, but the ordering was wrong: the work that could not proceed was
completed in full before the blocker was discovered, and the user then had to be handed a decision
they could have been given much earlier.

The boundary itself was not obvious and had to be probed: a dry run passed, a single ordinary tag
passed, a forced tag across many refs failed, and running the script failed. Read operations and
ordinary writes were fine; anything that rewrote or force-updated refs was not.

**Rule concluded:** when a planned task's final step is destructive — force-push, history rewrite,
bulk deletion — establish that the environment permits it before building the rest. One cheap probe
against a single representative target costs seconds and reorders the whole task. And when a sandbox
does refuse, stop and hand the user the decision rather than searching for a narrower phrasing that
slips through; the refusal is the mechanism working, and routing around it is a different act from
completing the task.

**Where it lives:** here. A command that plans destructive version-control work should probe the
permission boundary during planning, not at execution.

## 2026-08-26 (later session, found on a user-requested re-review) — A file's warning about how it fails was read, understood, and then violated in the same act of rewriting it; and the sweep produced the exact artifact its pending fix exists to prevent

**Status: both findings verified.** The dropped content was established by diffing the rewritten note
against the version it replaced. The duplicate-artifact count was produced by re-running the
migration's own dry run after the sweep completed.

**Context, project-agnostic.** The session-end command writes a resume note for the next session. The
note it overwrites carried, in its own header, a warning that this had gone wrong twice before, plus
an explicit instruction: *diff against the old version before overwriting, because rewriting from
session memory drops live threads.* The orchestrator read that header at session start — it was the
first thing the session-start command surfaced — and then rewrote the note from memory anyway.

The user asked for a review of the session's documentation. That review found the failure. Nothing in
the sweep itself would have.

**Finding 1 — the warning was in the right place, addressed to the right reader, and still did not
fire.** The first draft dropped seven standing constraints, two published artifact URLs, four open
questions, and one undone decision. Several were not stale: one dropped open question had been
*activated* earlier in the same session (a risk that installing something would make live, which
installing it duly did), and the dropped artifacts were deliverables the user had been reading hours
before.

What makes this worth logging rather than filing as carelessness: the instruction did not fail
through inattention. **The act of writing a comprehensive note feels like remembering, so the check
that would reveal what was forgotten never gets triggered** — the same shape as a partial read
producing confidence. A rewrite composed from a rich session context is *more* susceptible, not less,
because the author has so much genuine material that the absence of the rest is unnoticeable.

**Rule concluded:** an instruction to compare against a prior version cannot live only in the artifact
being replaced, because the replacement is authored by someone who believes they already know the
contents. The comparison has to be a step in the procedure that produces the artifact, with the old
version actually retrieved and read — not a caution the author is trusted to remember. Where a
document is overwritten rather than appended to, the writing step and the diffing step are two
separate obligations, and only the second one catches loss.

**Finding 2 — the sweep created a new instance of the problem its own pending fix addresses.** A fix
was written, reviewed and merged this session that changes the sweep to write a fresh file per
session instead of appending to a shared one, precisely because appending had produced fourteen
mutually-conflicting branches that never merged. The fix sat unreleased. The sweep then ran the old
behaviour and produced a fifteenth.

This was foreseen — the orchestrator had told the user in as many words that every sweep before the
fix ships adds another. It still happened, because the sweep executes the *installed* command, and
merging is not installing.

**Rule concluded:** between merging a behavioural fix and installing it, the old behaviour keeps
running and keeps generating the problem. When a fix changes what a routine command produces, the
backlog it addresses is still growing during that window, and any count taken before the window
closes is already stale. Either install before running the routine again, or record that the count
moved and by how much.

**Where it lives:** here. The resume-note step in the session-end command should require retrieving
the previous version and diffing before writing the new one, rather than describing that requirement
inside the note it is about to destroy.
