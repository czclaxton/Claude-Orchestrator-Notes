### 2026-08-28 — Anti-inflation instruction in the subagent prompt worked; non-git `/catch-up` recurs a third time

**Status: finding 1 partially verified** (the agent's self-limiting behavior is directly observable
in its returned report; two of its substantive claims were independently cross-checked against
local state and matched — the rest were not re-checked); **finding 2 verified** (the command's
behavior on a non-repo project was observed directly this session, and the two prior log entries
covering it are a matter of record).

**Finding 1 — telling the subagent up front that negative claims require a named search produced
exactly that behavior, unprompted, in a research task.** The recurring theme in this log has been
lanes being reliable about assigned output and loose about volunteered context — counts,
prevalence, breadth claims — with the concluded mitigation sitting on the orchestrator's side
(verify volunteered context before relaying). This session tested the other lever: putting the
constraint in the delegation prompt itself.

The dispatch included, in substance: say "not found" explicitly rather than inferring; a negative
claim must be backed by an actual search you ran, and name which search; distinguish what you
verified from what you inferred.

The returned report complied in ways that would not have happened by default. It carried a
dedicated section on what it *could not* search, naming the specific domains that refused its
crawler and the exact error, and stating plainly that it was not claiming absence of reports there
— only that it could not read those sources. Several negative claims were each followed by the
literal search string that produced no hits. Inferences were labelled inline as inference rather
than folded into the findings. Where two sources disagreed about authorship, it flagged the
discrepancy instead of picking one.

Two of its substantive claims happened to be independently checkable against state the orchestrator
had gathered in parallel; both matched, including one that corrected an earlier orchestrator
assumption. That is a small sample and not a general accuracy result — the value here is the
*shape* of the report, not a verified accuracy rate.

**Caveat that limits how far this generalizes:** this was a general-purpose research agent, not one
of the four implementation/advisory lanes, and the task was external research rather than a diff.
So this is a datapoint about prompt-level mitigation transferring, not a lane result.

**Rule concluded (candidate, not promoted):** the orchestrator-side verification rule from the
prior entry is worth pairing with a delegation-side one — the standard spec could carry an explicit
epistemic clause (name the search behind any negative claim; say "not found" rather than infer;
mark verified vs. inferred). Cheap to add, and this run suggests it is actually honored rather than
ignored. Wants at least one more observation, ideally on a real lane, before promotion.

**Finding 2 — `/catch-up` on a non-git project, third logged occurrence.** The command ran against
a project with no repository. Its instruction body is dominated by local-vs-remote state
discipline, and the two rules it flags as mattering "more than remembering to run the command" are
both about not overstating knowledge of a remote. All of it was inapplicable. The command still
produced a useful result — the fallback of reconciling the resume note against on-disk reality
worked fine — so this is a proportionality complaint, not a failure.

Logged only as a frequency datapoint: this is already covered by the 2026-08-21 and 2026-08-23
entries, and adds no new detail beyond happening again. Noting it because recurrence count is the
thing those entries were waiting on.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill's spec contract if a
second observation supports it. Finding 2 needs no new analysis — it is a third tally mark against
the two existing entries.
