---

## 2026-08-28 — Two findings on the resume-note lifecycle (`/wrap-up`, `/catch-up`)

Both surfaced in a long-running documentation project. Neither is specific to it.

**Finding 1 — `/wrap-up` writes to a hardcoded `RESUME-PROMPT.md` at the project root, and projects
that keep their notes elsewhere are forced into a workaround that wrap-up itself can destroy.**

The project in question maintains its real resume note inside its own documentation tree, where its
other session files live and where its index links to it. Because wrap-up always writes to the root
path, the project's solution was to make the root file a *pointer* — a short page that says "the real
note is over there," plus a summary — and to leave a warning inside it telling future sessions not to
replace it with a full note.

That workaround is load-bearing and undefended. Any wrap-up run that does the obvious thing — write
the note to the path it was given — silently produces two divergent notes, and the next `/catch-up`
reads whichever it happens to open first. The project noticed the risk and wrote the warning by hand;
a project that did not would drift without knowing.

**Rule concluded:** the resume-note destination should be discoverable rather than fixed — honour an
existing note wherever the project's own entry-point file (`CLAUDE.md` or equivalent) points, and only
fall back to the root path when there is no such pointer. Failing that, wrap-up should at minimum read
the existing root file before overwriting it, since a pointer file is trivially recognisable and a
short one is cheap to read.

**Finding 2 — a resume note records open decisions but has no way to escalate one that keeps being
deferred.** ⭐ This is the more interesting of the two.

The project carried a single blocking decision — a rebuild the user had to authorise — that was
offered in three consecutive sessions and answered in none of them. Each session it was faithfully
written back into the note as "offered, not answered." Each session the conversation opened on it,
moved to something more immediate, and wrapped up without it. The note recorded the fact perfectly and
did nothing about it.

The failure is that "offered and not answered" reads identically on its first appearance and its
third. Nothing in the format distinguishes *new proposal* from *stalled for three sessions on the item
that blocks everything else*. A human chief of staff would raise their voice by the third pass; the
note has one volume setting.

**Rule concluded:** open decisions in a resume note should carry the count of sessions they have
survived, and wrap-up should increment it rather than restating the item fresh. Past some small
threshold the item stops being an entry in a list and becomes the thing the next session leads with —
or is explicitly withdrawn as "not actually wanted," which is also a resolution. What should not
happen is a blocking decision rolling forward indefinitely at the same visual weight as a nice-to-have.

⚠ Worth separating from a genuine user preference: the project's user has an explicit standing rule
that nothing gets closed without him, and deferring is a legitimate answer. The problem is not that he
deferred; it is that the note could not tell that he had deferred *repeatedly*.

**Where it lives:** here only. Finding 1 belongs in the wrap-up command's note-writing step. Finding 2
belongs in the resume-note format itself, and is a format change rather than a doctrine change.

---

## 2026-08-28 — Two findings on doctrine under constraint, and on verification harnesses

Both surfaced during a multi-round code review of an external developer's pull request. Neither is
specific to that project.

**Finding 1 — the orchestration doctrine has no defined behaviour when delegation is unavailable.**
⭐ verified

The project's entry-point file states the architect pattern in absolute terms: never write code
yourself, delegate broad exploration to cheap read-only agents, and obtain an advisor review before
reporting any deliverable done. The session also ran under a standing harness-level instruction not
to invoke the subagent tool unless the user explicitly asked for it. The user never asked.

Every one of the doctrine's three mandates was therefore unsatisfiable for the whole session. I
resolved the conflict by working directly and continuing — which was the right outcome, but I made
that call silently and it never surfaced to the user. Several substantial review deliverables were
reported as complete without the advisor pass the doctrine describes as mandatory.

The gap is that the doctrine is written as though delegation is always available. It has no fallback
tier, no instruction to announce that it is operating degraded, and no guidance on which of its three
mandates matters most when only one can be honoured. "Get an advisor review before reporting done" is
stated as an invariant, so a session that cannot obtain one is either silently violating an invariant
or silently downgrading it, and nothing in the doctrine says which.

*Possible prior instance:* the unmerged branch
`lessons/20260821-130440-recurring-resume-path-and-non-git-catch-up` carries a commit message
mentioning that the advisor mandate has no fallback. I could not locate that finding's body in the
branch's own file, so treat this as a possible recurrence for the reviewer to dedupe rather than a
confirmed one.

**Rule concluded:** the doctrine should state what the architect does when the delegation tier is
closed to it — at minimum, that the session says so once, in plain terms, rather than absorbing the
conflict. A mandate that cannot be met should degrade loudly. Worth noting the constraint is not
exotic: any harness setting, permission mode, or user preference that discourages subagents produces
it, and those are common.

**Finding 2 — a verification harness that writes test state through a different path than the code
under test can manufacture a finding that does not exist.** ⭐ verified

Standard practice when checking a lane's or a developer's claim is to build a scratch copy of the
data store, seed known values, and execute the real code against it. I did this, seeding rows with
direct statements against the store rather than through the object-mapping layer the application
uses. The code under test then matched none of them, and the obvious reading was a defect in the
code's matching logic.

It was an artifact. The mapping layer serialises that column type differently from the literal form I
had inserted, so the comparison was between two encodings of the same value. Re-seeding through the
same mapping layer the application uses produced a match, and the "defect" evaporated — though a
narrower, real issue remained underneath it.

I caught it only because the result was too broad to be plausible: a search feature that matched
nothing at all would not have shipped. A subtler false negative would have been reported as verified,
carrying the extra authority that empirical testing confers over code reading.

**Rule concluded:** the verification guidance should say that a harness must construct state through
the same layer the code under test reads it through, and that a finding produced by a harness which
does not is unverified regardless of how empirical it looks. The failure mode is specifically
dangerous because the doctrine correctly ranks empirical evidence above inference — so a false
empirical result outranks the code reading that would have contradicted it.

**Where it lives:** here only. Finding 1 belongs in the orchestration skill, as a new section on
operating when a tier is unavailable. Finding 2 belongs in its Verification section, as a constraint
on how evidence is produced rather than on how it is judged.
