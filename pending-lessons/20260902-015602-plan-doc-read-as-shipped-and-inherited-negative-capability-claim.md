### 2026-09-02 — A subagent reported a design document as shipped behavior, inverting a build-critical conclusion

**What happened** [verified]: a research subagent was dispatched to establish what a fast-moving
third-party component could actually do today. Its spec told it to read the component's repository
documentation. It did, thoroughly, and reported that a capability the architect had flagged as
decision-critical was **not implemented**. That conclusion was relayed to the user and written into
a decision register.

The document it had read was a **build plan** — a dated list of primitives the maintainer intended
to add. Every item on it had shipped in the weeks since. A second dispatch, instructed to read the
implementation rather than the documentation, found the capability present and the original
conclusion inverted.

The cost compounded twice. The architect relayed the finding to the user as established, and a
second file in the project was written on top of it before the error surfaced. Retracting it meant
correcting a user-facing claim, a decision register row, and a research file — and the retracted
claim had been described to the user as "the single most consequential question for the build."

**Why the spec did not prevent it.** The spec said "read the repository docs, they are the richest
source." That is true and was good routing. What it did not say is that in a repository under
active development, `Docs/` describes *intent* at the time of writing while the source describes
*state*, and the two diverge at exactly the rate the project is moving. The component in question
was shipping multiple releases per day; its documentation was weeks stale in a codebase where
weeks are a long time.

**The general shape.** Delegated research is asked to establish "what does X do." Documentation
answers "what did the author intend X to do, as of some date." For a mature, slow-moving component
those converge. For a young, fast-moving one they do not, and nothing in the document announces
which kind it is. A subagent given a whole `Docs/` directory will read it as authoritative because
that is what a docs directory usually is.

**Rule concluded:** when dispatching research at a fast-moving dependency, the spec must state the
precedence explicitly — *implementation outranks documentation; cite source, and treat any doc
claim contradicting it as stale.* A cheap heuristic worth putting in the spec: check the
repository's commit cadence first, and if it is high, treat every doc file as a dated artifact
rather than a description. Worth considering whether the orchestration skill should carry this as
a standing clause for research lanes, since it is not specific to any one dependency.

Related but distinct from the existing "a summary is a pointer, not a finding" rule: that rule
governs the architect re-reading *its own* prior notes. This one governs a subagent reading a
*third party's* notes, where the failure is invisible because the document looks authoritative.

---

### 2026-09-02 — A negative capability claim propagated through nine agent specs and was false

**What happened** [verified]: an early session established that a particular external resource
returned HTTP 403 to automated fetching. That finding was recorded in the project's shared context
file and, from then on, copied verbatim into the spec of every research agent dispatched — roughly
nine of them across two sessions. Each spec instructed the agent not to attempt that resource and
to use a metadata API instead.

The claim was **tool-specific, not resource-specific**. The resource 403s to one fetch tool and
returns HTTP 200 to a plain request carrying an ordinary browser User-Agent. The final agent in the
sequence tried it and reported the correction.

The cost was not hypothetical. The blocked resource was the only place a particular class of
decision-relevant information was published; a metadata API the specs had been steering agents
toward does not expose that field at all. So a question the architect had been treating as
unanswerable for two sessions had been answerable the whole time, and it turned out to bear
directly on a live decision.

**The general shape, and why this class is worse than it looks.** The project already carried an
explicit rule that negative claims require evidence. That rule was followed *when the claim was
first made* — someone did try, and did get a 403. The failure is in what happened next: the claim
was written into shared context as a property of the **resource**, when the evidence only supported
it as a property of the **resource plus one tool**. Once in shared context it was inherited by every
downstream spec, and an instruction not to attempt something guarantees no agent will ever generate
counter-evidence. The claim became unfalsifiable by construction, and stayed that way until an
agent ignored it.

**Rule concluded:** a negative capability claim entering shared context must record *how* it was
established, not just *what* it concluded — "403 via tool T" rather than "403". The distinction is
cheap to write and decides whether a later session can re-test it. And a standing instruction not
to attempt something should carry an expiry or a re-test note, because it removes the only
mechanism by which it could ever be corrected. Access-limitation tables in shared context are
prime candidates for periodic re-verification for exactly this reason.

---

### 2026-09-02 — Two agents on adjacent framings of one decision produced stronger evidence than either alone

**What happened** [verified]: a high-stakes, expensive-to-reverse configuration decision was
approached twice. One agent was dispatched to read the implementation and establish the mechanism.
Separately, and after the user asked for a comparison rather than a mechanism, a second agent was
dispatched to evaluate two candidate components against stated criteria, with an explicit scope
boundary telling it the mechanism question belonged to the other agent and not to re-derive it.

The two arrived at the same recommended configuration from different directions — one from reading
the allocation logic, one from comparing licence terms, maintenance cadence and extension surface.
Neither cited the other. The convergence was materially more convincing than either report was on
its own, and it was what made the recommendation safe to put to the user as a decision rather than
an option.

**Why this is not just redundancy.** The two agents were not given the same question. Their briefs
overlapped in *subject* and differed in *method* — mechanism versus comparative evaluation — and
the scope boundary in the second brief actively prevented duplicated work. The cost was one extra
dispatch; the return was an independent check on a conclusion that would have been expensive to get
wrong and that the architect could not verify directly.

**Rule concluded, tentatively:** for a decision that is expensive to reverse and that the architect
cannot verify itself, consider dispatching two agents on *adjacent framings* rather than one agent
on the whole question — and give the second an explicit scope boundary naming what the first owns.
This is not blanket duplication and should not become the default; it costs a full dispatch and
only pays when the decision is genuinely load-bearing. Logged as a technique that worked once, not
yet a doctrine change. Worth watching for whether it repeats before promoting it.
