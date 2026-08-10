---
title: "pulse PRD — adversarial review"
target: prd.md @ 2026-08-07 (draft), addendum.md @ 2026-08-07
reviewer: adversarial pass
date: 2026-08-08
---

# Adversarial review — pulse PRD + addendum

## Verdict

The document is well-organised and the archive layer (§4.2) is genuinely sound. But it fails in three ways that matter for a document whose stated job is to be built from:

1. **The thing the PRD calls its own contribution — the data interface contract (FR-10) — has no content.** Every other requirement leans on it and it is delegated wholesale to architecture. An agent cannot build FR-10; it can only invent it.
2. **The primary success metric (SM-1, ambient familiarity) is supported by no requirement, and SM-2 (habitual opening) by none at all.** The vision names a loop — the archive compounds only if pulse is used — and then specifies only the half that produces reports, never the half that causes them to be opened. This is not a scope decision about visual content; it is the mechanism of the primary goal, and it is absent.
3. **Roughly a third of the "Consequences (testable)" bullets are not testable**, and several are *vacuously satisfiable* — an implementation that does nothing would pass them.

Plus three hard cross-reference breaks and one live contradiction between §5 and §10 that would derail the architecture phase on day one.

Section-by-section below. Severity: **[BLOCKER]** must be fixed before architecture, **[MAJOR]** will cause rework, **[MINOR]** cleanup.

---

## A. Broken references and section integrity

These are factual, not judgements.

### A.1 [BLOCKER] `addendum §D` does not exist — cited twice

- prd.md:363 — NFR-4: "See [addendum §D](addendum.md) for the storage-quota analysis and its watch item."
- prd.md:420 — §9 Risks: "**Storage quota.** ... Analysis, decision and mitigations in [addendum §D](addendum.md)."

The addendum has sections A, B, C only. Per `.memlog.md:51`, section D was deleted (it duplicated the LFS decision rewritten into A.1), and `C.5` was repointed to A.1 — but the two PRD pointers were not. Worse, per `.memlog.md:50` the LFS **watch item was dropped entirely** at Yann's direction, so NFR-4 promises "its watch item" that was deliberately removed, and §9 keeps a risk whose analysis and mitigations no longer exist anywhere. Either restore the risk with real content or delete the §9 bullet; do not leave a dangling risk pointing at a deleted section.

### A.2 [MAJOR] `see §8` points at Success Metrics, and the thing it points to has no home

prd.md:314 — §4.7 Notes: "Recorded as a future goal; see §8."

§8 is **Success Metrics**. There is no future-goals section in this PRD. The deferred alternative (rebuild a failing source from its last passing snapshot) is therefore recorded *nowhere* — it is not in §5 Non-Goals, not in §7.4, not in §10 Open Questions. The one design decision the document explicitly says it wants to preserve for later is the one it loses. Either add a "Deferred / future" section or move it into §5 as an explicit non-goal with the rationale attached.

### A.3 [MAJOR] FR-9 cites the wrong FR

prd.md:197 — "A failing assertion does not stop the pipeline run — see FR-19."

FR-19 is *Pipeline state on the homepage*. The requirement that a failing assertion does not stop the run is **FR-21**. A downstream reader following this pointer lands on the wrong requirement.

### A.4 [MINOR] Assumption tagging discipline is stated and then broken

§0 (prd.md:14): "Inferences not confirmed by the user are tagged `[ASSUMPTION]` inline and indexed in §11." §11 lists four assumptions; only two (FR-4, FR-7) carry an inline tag. The §7.1 and §2.3 entries appear only in the index.

The relative links to `docs/initial-pitch.md` and the brainstorm synthesis both resolve correctly. Only §D and §8 are broken.

---

## B. "Consequences (testable)" that are not testable

The header makes a promise. These bullets do not keep it. Quoted verbatim.

### B.1 [BLOCKER] Vacuously satisfiable — an empty implementation passes

**FR-4 (prd.md:148):** *"Private data never leaves the local machine and never enters the public repository."*

This is the **only** consequence attached to FR-4. It tests nothing about manual ingestion. It is trivially satisfied by not implementing FR-4 at all — if the private ingestion path does not exist, private data provably never leaves the machine. A requirement whose sole test passes when the requirement is unimplemented is not a requirement. FR-4 needs a consequence about the thing it actually requires, e.g. "a file placed in the known location appears as a snapshot in the next local pipeline run and in the private build profile." (It is also a negative universal over all future time, which no finite test verifies — see B.2.)

**FR-13 (prd.md:232):** *"Adding or changing a convention does not require touching every existing visual."*

"Every" is the wrong quantifier. Touching all-but-one visual satisfies this. The intent (from `.memlog.md:43`) is clearly "does not require touching existing visuals". As written it is nearly free to pass.

**FR-1 (prd.md:127):** *"Adding a source requires no change to shared ingestion code."*

Satisfiable by having no shared ingestion code — put everything in per-source files and this passes forever. The bullet only bites against a definition of "shared ingestion code" the PRD never gives, and it is in direct tension with FR-23's *"Adding a source does not duplicate ingestion logic"* (prd.md:334). One bullet rewards moving code out of the shared layer, the other forbids duplication, and nothing says where the line sits.

### B.2 [MAJOR] Unfalsifiable universals and unobservable process claims

- **FR-1 (prd.md:127):** *"If several sources require the same specific code, then this should become part of the shared ingestion code."* — "several" is unquantified and "should" is a refactoring norm. No one can fail this.
- **FR-5 (prd.md:160):** *"A snapshot is never modified or deleted after being written."* — no finite test. Testable form: "the pipeline contains no code path that writes to or deletes an existing snapshot path", plus a CI check that no commit deletes files under the archive path.
- **FR-12 (prd.md:225):** *"No visual is produced by selecting a chart type."* — a statement about process, not about the artifact. You cannot inspect a repository and determine this. Delete it or replace it with an artifact-level test.
- **FR-13 (prd.md:231):** *"The store of conventions starts close to empty and grows only as conventions emerge from use."* — "close to empty" and "emerge from use" are unobservable. No reviewer can distinguish a convention that emerged from one that was predefined.
- **FR-13 (prd.md:233):** *"A convention can be revised or withdrawn."* — vacuous; text files can always be edited.
- **FR-15 (prd.md:254):** *"A report renders with visuals of arbitrary design."* — "arbitrary" cannot be enumerated, so this can never be verified, only failed.
- **FR-10 (prd.md:208):** *"Two visuals of entirely different design receive their rows identically."* — "identically" is undefined and unverifiable while the contract itself is unspecified (see D.1).
- **FR-10 (prd.md:211) / FR-22 (prd.md:326):** *"...without reading other visuals' rendering code"* / *"...without reading unrelated system code"* — claims about what a reader consulted. Not observable. Reframe as a property of the workflow document (e.g. "the workflow is self-contained: every file it requires the agent to open is named in it").
- **FR-11 (prd.md:218):** *"Moving from authoring to wired is a substitution, not a rewrite."* — needs a bound to be testable (e.g. "the diff touches only the data-acquisition block"). See C.4 for why this one is also probably false.
- **FR-23 (prd.md:333):** *"Two sources added at different times have the same shape."* — "shape" is undefined, and this is untestable in v1 because §7.1 scopes exactly **one** source.
- **NFR-1 (prd.md:360):** *"...a description that fits in markdown."* — everything fits in markdown. This phrase carries no constraint.
- **NFR-4 (prd.md:363):** *"...under normal operation."* — undefined, and its supporting analysis is the deleted §D.

### B.3 [MAJOR] Undefined thresholds masquerading as tests

Each of these reads as a test but cannot be run until someone invents a number the PRD declines to state:

- **FR-9 (prd.md:196):** *"A source whose row count collapses, whose expected columns are missing, or whose latest period failed to advance is flagged."* — "collapses" has no threshold. Per-source, presumably. Nothing requires the threshold to be declared alongside the source.
- **FR-19 (prd.md:296):** *"A source that has gone stale is visibly marked."* — the Glossary defines Freshness "relative to that source's cadence" but never says what multiple of cadence constitutes stale. Nothing requires a source to declare its staleness tolerance.
- **FR-6 (prd.md:168):** *"Deleting the entire warehouse and rebuilding produces an equivalent result."* — "equivalent" is doing all the work. Byte-identical parquet? Row-identical? Parquet writes are not byte-stable across versions and compression settings, so the strictest reading fails for reasons that have nothing to do with pulse.

### B.4 [MAJOR] FR-9's "latest period advanced" assertion is wrong for the MVP's own source

Annual and quarterly World Bank indicators do not advance for most of the year — an annual indicator's latest period advances once every twelve monthly runs. As specified, the assertion fires eleven times out of twelve on the flagship v1 source, and FR-21 marks it on the homepage each time. The result is a homepage that is permanently marked, which destroys exactly the property §4.7 is built on: *"Nothing is marked"* (UJ-2, prd.md:85) as the signal that all is well. The assertion must be cadence-relative ("the latest period advanced, or the expected publication date has not yet passed"), and the source must therefore declare its *publication* calendar, not only its *fetch* cadence. Neither is in the document.

---

## C. Internal contradictions

### C.1 [BLOCKER] §5 forbids what §10 carries forward as a live candidate

- **§5 Non-Goals (prd.md:353):** *"**A chart library, or a consumer of one.** At any layer, ever."*
- **FR-12 consequence (prd.md:224):** *"No chart library appears in the dependency tree."*
- **§10 Open Question 1 (prd.md:428):** *"Architecture candidate. Evidence.dev, Observable Framework, or a general build tool..."*
- **addendum §B.1:** Evidence.dev is *"Opinionated component library over ECharts."*

Selecting Evidence.dev puts ECharts in the dependency tree and makes pulse a consumer of a chart library. §5 and FR-12 eliminate a candidate that §10 and addendum §C.4 carry forward as live, and neither document notices. §1.3 tries to permit this — *"Everything below that line is open... provided the adopted tool does not constrain visual authoring"* — but §1.3's carve-out is about **authoring**, while FR-12's test is about the **dependency tree**, and §5's phrase "or a consumer of one" forecloses adoption entirely. The architecture phase cannot resolve this; it is a product decision and it must be made here. Concretely: either FR-12's test becomes "no chart library is used to author a visual" (permitting an adopted framework that happens to bundle one), or Evidence.dev is struck from §10 and §C.4.

### C.2 [BLOCKER] The archive must survive; private raw is explicitly allowed not to

- **§1.2 (prd.md:40):** *"**Archive-first.** ... Never losing history is a core requirement."*
- **Glossary (prd.md:99):** *"**Raw archive** — the complete set of snapshots **across all sources**. Durable; the only thing that must survive."*
- **§5 (prd.md:350):** *"**Durable storage for private data.** Private raw is local-only on one machine in v1."*

The glossary defines the raw archive to include private snapshots and declares it the one durable thing; §5 says a third of it lives on one unbacked machine. One disk failure and "never losing history" fails for every private source. Either scope the durability claim ("the *public* raw archive is the only thing that must survive"), or accept that FR-6's *"A rebuild at any time reproduces the current warehouse from committed snapshots alone"* (prd.md:170) is false for any dataset with a private source — which it currently is, unstated.

### C.3 [MAJOR] FR-5 requires snapshots "before any transformation", then requires a transformation

- **FR-5 body (prd.md:157):** *"Every fetch writes a snapshot of the source's data **as received, before any transformation**."*
- **FR-5 consequence (prd.md:161):** *"File-based sources are kept in their original format. **Other sources write into parquet files.**"*

Writing an API's JSON response into parquet requires parsing, flattening and typing it — transformation, performed before the snapshot is written. That is precisely the class of operation the archive-first principle exists to keep out of the raw layer, and it has a concrete consequence: a source that adds or renames a column can now fail at *snapshot-write* time, meaning the anomaly is never archived and FR-9's "expected columns are missing" assertion never gets to see it. The failure the archive is supposed to make recoverable becomes the failure that prevents archiving.

Also: "file-based source" is undefined. Is a CSV downloaded over HTTP file-based, or is it "other"? An agent will guess.

### C.4 [MAJOR] "Idempotent" contradicts "every fetch writes a snapshot"

- **FR-3 (prd.md:142):** *"the pipeline is idempotent and a missed run is not data loss."*
- **FR-5 (prd.md:157):** *"**Every** fetch writes a snapshot."*

Running the pipeline twice in one day produces two snapshots. That is not idempotent under any usual definition, and "idempotent" is not in the Glossary despite §3's *"Used verbatim throughout"* rule. What FR-3 presumably means is "re-runnable without corruption" — say that instead.

Separately, *"a missed run is not data loss"* is **only true for sources whose API serves history**. For a source that serves current-values-only, a dropped run permanently loses that period — and the raw archive, the thing that must never lose history, is where the loss lands. Nothing in FR-1 requires a source to declare whether it is backfillable, and §9's scheduler-decay risk claims *"FR-3's idempotence requirement covers the second"* — it does not, for that class of source. This is a risk the document believes it has mitigated and has not.

### C.5 [MAJOR] NFR-3 claims structural privacy; FR-17 implements it as a flag you must remember

- **NFR-3 (prd.md:362):** *"The separation is **structural, not procedural** — it does not depend on remembering to exclude anything."*
- **FR-17 (prd.md:264):** *"A report **declares** whether it is private."*

A declaration is the definition of procedural. The load-bearing question — **what is the default for a report that declares nothing?** — is not answered anywhere. If the default is public, forgetting the flag leaks; NFR-3's promise is then simply false. Structural separation means private reports and private data live somewhere the public build cannot reach (separate directory tree, separate build root), so that inclusion rather than exclusion is the deliberate act. The PRD should state the default explicitly, and NFR-3 should be given a testable consequence — it currently has none at all.

### C.6 [MAJOR] Glossary self-violation: "Visual language" is defined and then never used

§3 states: *"Introducing a synonym anywhere in this document is a discipline violation."* It then defines **Visual language** (prd.md:108) — and the term appears nowhere else in the PRD. FR-13 (prd.md:228) and §7.2 (prd.md:388) both use *"store of conventions"* / *"a store for cross-visual conventions"* instead. The document violates its own rule against its own glossary entry.

### C.7 [MAJOR] "Indicator" is undefined but has a mandated workflow

FR-22 (prd.md:326) requires a workflow for *"add an indicator"*. **Indicator** is not in the Glossary. It is used in §1.1 goal 1, UJ-4 and FR-22, and it is not obviously any existing term: is an indicator a source, a column in a dataset, a row-filtered series, or a report element? "Add an indicator" is one of four mandated workflows and nobody can write it without inventing the concept first. Either add it to §3 or drop it from FR-22.

### C.8 [MAJOR] Glossary defines a five-stage pipeline; FR-19 shows three

- **Glossary (prd.md:111):** *"**Pipeline run** — one execution of the chain: fetch → snapshot → warehouse build → report generation → publish."*
- **FR-19 (prd.md:293):** *"...the state of the last pipeline run across **ingestion, warehouse build and report generation**"*

"Ingestion" collapses fetch+snapshot (fine, but then it should be the glossary term), and **publish is dropped from visibility**. That omission is not cosmetic — see F.1: publish failure is the one that hides all the others.

### C.9 [MAJOR] A Must depends on a Should

- **§7.1 In Scope (Must, prd.md:380):** *"Homepage with navigation and pipeline state"* → FR-19, whose testable consequence is *"A source that failed its assertions is visibly marked"* (prd.md:297).
- **§7.2 Should (prd.md:386):** *"Per-source assertions beyond the transformation layer's defaults"* → FR-9.

If the Should does not ship, a Must's consequence is untestable. More pointedly: §4.7's entire rationale — the extended passage about *"the dangerous failure is the one that succeeds"* (prd.md:290) — is defended by requirements sitting in optional scope. The document's most emphatically argued failure mode is mitigated only if there is time left over. FR-21 (suspect data published and marked) has **no MoSCoW placement at all**.

### C.10 [MAJOR] FR-16 annotations vs the multi-source non-goal

- **FR-16 (prd.md:257):** *"Annotations are stored as data and joined to visuals."*
- **§5 (prd.md:351):** *"**Multi-source datasets.** A dataset is built from one source in v1."*

An annotation table is a second origin of data joined to an indicator series. If annotations live in a dataset, FR-16 violates the non-goal. If they are joined client-side in duckdb-wasm, the PRD does not say so. And nothing anywhere declares annotations as a **source** — they are not covered by FR-1, are not in §7.1, and have no ingestion path, storage location, or join key. FR-16 is a requirement with no supply chain.

### C.11 [MINOR] "The data interface is the stable contract" vs "might require migration of the contracts"

§1.2 (prd.md:45) names the data interface *"the stable contract"*; §1.2 (prd.md:41) says schema changes *"might require migration of the contracts and reports"*. Two adjacent principles, one calling the contract stable and the other admitting it migrates. Pick one, or distinguish which part is stable (the *shape* of delivery) from which part is not (the *columns* delivered).

### C.12 [MINOR] Addendum B.4 still recommends a superseded decision

addendum §B.4: *"**The build-time / client-side split** (Evidence): build-time queries serve the narrative; client-side SQL is reserved for interaction. **This is exactly pulse's 'wasm optional per report' decision**, arrived at independently."*

addendum §C.5 lists *"baked JSON at build time by default, wasm opt-in per report"* as **superseded** by duckdb-wasm on all pages. B.4 continues to endorse the superseded model as a convention worth adopting.

### C.13 [MINOR] Addendum C.4 marks layer D "Open" while fixing its decision

`| **D** | Data delivery to page | Open. duckdb-wasm on all pages |` — a layer cannot be open and decided. State it as "Mechanism open; duckdb-wasm on all pages is a requirement."

### C.14 [MINOR] Open Question 3 references a concept that exists nowhere

prd.md:430: *"...only **cross-filtering** would force the choice early."* Cross-filtering appears in no other line of the PRD — not in §3, §4, §5 or §7. Per `.memlog.md:53`, it was removed from Non-Goals and the reference in OQ3 was knowingly left behind. As it stands, OQ3's deferral rests on a term the reader cannot resolve.

### C.15 [MINOR] "Two runtimes are a permanent property" of a feature scoped as a Could

§4.1 (prd.md:121): *"Two runtimes are therefore unavoidable and are a permanent property of the system, not a v1 compromise."* §7.3 puts *"Manual ingestion path for private data"* in **Could**. The permanent property may not exist in v1.

---

## D. Load-bearing claims with nothing behind them

### D.1 [BLOCKER] FR-10, the data interface contract, is empty

addendum §B.2 says it plainly: *"the data-interface contract is not a chore but **the actual contribution**, and no candidate supplies it because no candidate has pulse's problem."* §1.2 calls it *"what makes an unusual visual cheap to add rather than a research project."* `.memlog.md:29` flagged it as owed so it *"cannot fall through."*

It fell through. FR-10 says a visual receives data and registers *"through a single predictable contract"* and never says anything about the contract. Not its shape (array of row objects? a typed table handle? a promise? a SQL string the framework resolves?), not who owns the query, not how registration works, not what happens on error or empty. Its three consequences are all statements *about* a contract that does not exist yet, which is why all three are untestable (B.2).

This is the single largest gap in the document. Everything the PRD claims is genuinely novel routes through FR-10, no success metric validates it, and it is the one deliverable no candidate framework supplies. An agent handed this PRD invents the contract, and whatever it invents becomes pulse's actual contribution by default.

Minimum the PRD owes: whether the contract is synchronous or asynchronous, whether the visual or the report owns the query, and what the visual is handed. That is three sentences and it is product-level, not architecture-level, because §1.2 elevated it to a principle.

### D.2 [BLOCKER] SM-2 (habitual opening) has no requirement behind it, and SM-1 depends on an unanswered question

§1 states the engine: *"the raw archive only compounds if pulse keeps being used, and pulse is only used if the reports are worth opening."* §8 makes SM-1 primary — *"This is the actual goal; everything else is instrumental"* — and its mechanism (§1.1) is *"retention through frequent incidental exposure."*

Nothing in §4 produces frequency or exposure. There is no notification, no digest, no reminder, no scheduled prompt, no browser-start surface, no email on new data. §5 rules out an always-running service, and NFR-5 says pulse may break without notice. The only candidate mechanism is FR-19's *"Reaching a report from the homepage passes the reader through this state"* — which is about health, not numbers, and which `.memlog.md:40` records as an open question ("*Does the index page promote from Should/nav-aid to the primary retention surface (core numbers seen on every visit)?*") that was **never answered**. It is not in §10 either. The primary goal's mechanism is an open question that was dropped rather than resolved.

To be clear about the scope boundary: I am not asking what the homepage *shows*. I am asking whether "the surface a reader passes through on every visit carries the tracked numbers" is a requirement or is not. That is an engine question, and it is the difference between SM-1 being achievable and being a wish.

### D.3 [MAJOR] Thirteen of twenty-four FRs are validated by no success metric

Union of FRs named across SM-1…SM-7: {3, 5, 6, 13, 14, 15, 18, 19, 20, 22, 23}.

Unvalidated: **FR-1, FR-2, FR-4, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-16, FR-17, FR-21, FR-24.** That set includes the contribution (FR-10), the non-negotiable (FR-12), privacy (FR-17), and the entire data-quality chain (FR-9, FR-21). NFR-1, NFR-3, NFR-4 and NFR-5 have no metric either.

Two of the stated links are also wrong: SM-1 (recall) claims to validate **FR-20** (a report states its data date) — a date on a page does not move whether Yann remembers a number. SM-5 (archive integrity) claims to validate **FR-19** (homepage state), which detects rather than preserves.

### D.4 [MAJOR] UJ-6 and NFR-2 (reproducibility) have no requirement realizing them

§4.6's description claims *"Realizes UJ-1, UJ-6"*, but FR-18's three consequences are about static hosting, a single local command, and duckdb-wasm — none about a stranger rebuilding anything. NFR-2 calls reproducibility *"a requirement rather than a nicety"* and it is supported by no FR: nothing requires pinned dependency versions, a lockfile, a documented bootstrap, a README, or a CI job that performs a clean-clone rebuild. SM-6 tests it manually, once, by a stranger who may never appear. For a document whose second stated goal is Portfolio, the portfolio-critical property is the least specified thing in it. The cheapest fix is one FR: *a clean-clone rebuild runs in CI on a schedule and fails loudly.* That converts SM-6 from an anecdote into a signal.

### D.5 [MAJOR] Risks with no mitigation and no trigger

- **"The v1 bet"** (§9, prd.md:418) and §1.3's falsification condition — *"if purpose-built visuals prove slow to produce even with agent authoring"* — have **no threshold, no measurement, and no review point**. §1.3 says stating the boundary precisely *"is what keeps it from being relitigated"*, but a falsification condition nobody can evaluate never fires; it just gets argued about. Give it a number and a checkpoint (e.g. "after the third visual, if median author-to-wired time exceeds X, revisit"). SM-4 gestures at this and is itself unmeasurable — *"does not get slower as the system grows"* with no defined cost unit and no recorded baseline.
- **"Portfolio versus personal tool"** (§9, prd.md:419) is stated and then abandoned: no mitigation, no tiebreak rule, no owner. When a personally revealing indicator is useful and embarrassing, what happens? The risk exists only to be acknowledged.
- **"Storage quota"** (§9, prd.md:420) points at the deleted §D (A.1).
- **Goal 3's guard** (prd.md:34): *"may **explicitly not be optimised away** ... unless absolutely necessary and after discussion"* — the escape clause reopens precisely what the sentence exists to close, and "absolutely necessary" is unfalsifiable.

### D.6 [MAJOR] duckdb-wasm on every page is required by nothing in §4

FR-18's consequence *"Data is queried in the browser via duckdb-wasm on all pages"* (prd.md:282) is a mechanism asserted as a requirement. No functional requirement in §4 needs client-side querying: reports are standing analyses, nothing in the document requires interactivity, filtering, or parameterisation (the one term that would — cross-filtering — is undefined, see C.14). §7.1 then justifies a second Must with it: parquet materialisation is promoted *"duckdb-wasm reads parquet directly"*. A mechanism justifying a mechanism.

The honest justification exists — it is goal 3, playground, stated in §1 as *"duckdb-wasm are wanted for their own sake"*. Say that at FR-18 rather than presenting it as a consequence, because a reader who does not see the justification will optimise it away, which §1.1 goal 3 explicitly forbids. This is the exact failure mode the document was written to prevent.

### D.7 [MAJOR] An unverified technical assumption sits under two Must items

addendum §A.5: *"Whether an external model also registers a catalog view for tests and `ref()` was **not confirmed** — smoke-test one external model with a `not_null` test before building the pipeline on this pattern."*

§7.1 makes *"Datasets materialised to parquet"* a Must, and FR-9 requires datasets to carry data tests. If external materialisation does not register for tests, those two Musts conflict, and the gate exists only as a recommendation buried in the addendum. It is not in §10 Open Questions, not in §9 Risks, and not in §11 Assumptions. Promote it.

---

## E. Where a coding agent must invent a decision the PRD should have made

Ordered by blast radius. Each of these is a decision that shapes every later one.

1. **The data interface contract's actual shape** (FR-10). See D.1.
2. **The default privacy of a report** (FR-17/NFR-3). Public-by-default plus a forgettable flag is a leak; the PRD does not say. See C.5.
3. **Which sources are due in a given run, and where last-fetch state lives** (FR-2, FR-3). Cadence is per-source and declared; scheduling is a single cron (addendum A.4). Does every run fetch everything, or does the run evaluate due-ness? If the latter, the state cannot live in the warehouse (disposable, FR-6) — it must be derived from the archive, which requires a snapshot naming convention the PRD never states.
4. **Snapshot identity and "latest"** (FR-5, FR-7). FR-7 builds *"from the latest snapshot of its sources"* with no definition of how latest is determined — path date, commit time, content hash? addendum B.4 mentions a "snapshot hash" as a convention. Not required anywhere.
5. **Whether an unchanged fetch writes a duplicate snapshot.** FR-5 says every fetch writes one. Over years of monthly runs on slow-moving annual indicators, most snapshots are byte-identical duplicates in Git LFS forever. Dedupe-by-content-hash is the obvious answer and it changes what "the archive" means. Not addressed.
6. **Staleness and assertion thresholds** (FR-9, FR-19). See B.3, B.4.
7. **"Indicator"** (FR-22). See C.7.
8. **Which date a report shows** (FR-20: *"Each report states the date of the data it displays"*). A report composes visuals over multiple datasets and sources; there is no single date. Max? Min? Per-visual? addendum B.4 recommends per-visual (*"as of `<date>`, source `<X>`, snapshot `<hash>`"*), which contradicts FR-20's report-level singular.
9. **Loading, empty and error states for browser-side queries.** duckdb-wasm is asynchronous and can fail. No requirement mentions any of the three, while FR-11 demands wiring be *"a substitution, not a rewrite"* from a synchronous mock array. Substituting a sync array with an async, failable query **is** a rewrite of the render path unless the contract handles it — which returns to D.1.
10. **Annotation storage, ingestion and join key** (FR-16). See C.10.
11. **Where pipeline-state data comes from.** FR-19 requires the homepage to display per-source freshness, per-source assertion results, and per-stage run outcomes. That is a dataset, produced by the pipeline, that must survive stages failing. Nothing declares it. See F.1.
12. **Whether a failed build may overwrite a good deployment.** No atomicity or rollback requirement anywhere.

---

## F. Failure modes the document does not consider at all

### F.1 [BLOCKER] The status surface is generated by the pipeline whose failure it must report

FR-19 requires the homepage to mark *"A failed stage of the last pipeline run"*. The homepage is produced by report generation and made visible by publish — two of the stages it is supposed to report on. Three concrete cases:

- **Report generation fails.** No new homepage exists. The deployed site keeps yesterday's homepage, which says everything is fine. The one failure mode §4.7 was written to defend against — *"The run goes green, the site deploys, the report renders, and a wrong or stale figure sits on a page"* — is reproduced exactly, by the defence itself.
- **Publish fails.** Same outcome, and publish is not even in FR-19's list of stages (C.8).
- **Warehouse build fails.** Whether the homepage can be generated at all now depends on whether pipeline-state data is a warehouse dataset or a separate artifact. The PRD does not say, so the agent picks — and if it picks "a dataset", the status page dies with the thing it monitors.

This needs an explicit requirement: pipeline state is written by each stage to an artifact that does not depend on later stages succeeding, and a stale homepage is itself detectable (the homepage states when it was generated, and going stale is a marked condition).

### F.2 [BLOCKER] No performance requirement exists, and the primary goal is a latency goal

SM-1 and SM-2 depend on brief, frequent, low-friction opening — *"opened as part of a rhythm"*, checking a number mid-conversation (UJ-1). The chosen delivery is a multi-megabyte WebAssembly runtime instantiated in the browser on **every page**, single-threaded by decision (addendum A.2: *"loses intra-query parallelism (roughly linear slowdown on scan/aggregate-heavy work)"*), reading parquet over HTTP where *"remote reads are effectively sequential/blocking XHR"* (addendum A.3).

There is no NFR for time-to-first-number, no budget for page weight, no requirement that anything renders before the runtime loads, and no fallback if wasm fails or is blocked. If opening the macro report takes four seconds before a figure appears, SM-2 does not happen, and no requirement in the document would have been violated. A single NFR — *a tracked figure is legible within N seconds on a cold load* — would make the tension visible and testable, and would be the honest place to record that goal 3 is being paid for in latency.

### F.3 [BLOCKER] Source licensing and redistribution are never mentioned

FR-5 requires raw snapshots of third-party data to be **committed to a public repository** (prd.md:162). The document never asks whether the sources permit redistribution, never requires attribution, and never requires a source to declare its licence. World Bank data is CC BY 4.0 and requires attribution; other public sources restrict redistribution or bulk republication outright. For a repository whose second goal is to be shown to people as evidence of how the author works, an unattributed public mirror of someone else's data is both a legal exposure and a credibility problem. FR-1 should require a source declaration to carry its licence and attribution string, and the report or site should surface it.

### F.4 [MAJOR] Sources that break, disappear, or need credentials

Nothing in the document contemplates a source whose URL changes, whose API is retired, whose schema is redesigned, whose rate limit is hit, or that requires an API key. That last one is concrete and immediate: pulse runs in **GitHub Actions against a public repository**. Where do secrets live? How does a stranger cloning the repo (UJ-6, NFR-2) rebuild a source that needs a key they do not have — does the rebuild fail, skip, or fall back to committed snapshots? The answer is probably "snapshots make it work", which is a genuinely good answer that the document should state, because it is one of the strongest things about the archive-first design and it is currently invisible.

### F.5 [MAJOR] A permanently bad source has no exit

Combine FR-7 (*build from the latest snapshot*), FR-21 (*build with suspect data and mark it*) and the §4.7 deferral of last-good-snapshot rebuild: a source that starts returning garbage and then stops publishing leaves the report permanently wrong-and-marked, with no in-scope remedy. Deferring the machinery is a defensible call (`.memlog.md:34`); not noticing that the failure is unbounded in duration is not. At minimum, §9's *"Silent success with bad data"* should say so, and there should be a manual escape hatch — even "delete the bad snapshot" is unavailable, because FR-5 forbids deletion (C.2/B.2).

### F.6 [MAJOR] Archive growth versus the reproducibility goal

Every full clone of the repository must pull the whole raw archive to rebuild (NFR-2, UJ-6, SM-6), and the archive only grows — by design, forever, with duplicate snapshots (E.5). The LFS **bandwidth** question was deliberately judged overblown and dropped, which is fine; the question left standing is different: at what archive size does *"anyone cloning the public repository can rebuild end to end"* stop being a pleasant experience for a stranger evaluating a portfolio? Nothing bounds it, nothing measures it, and the mitigation (partitioning, pruning, a "shallow rebuild" path) is much cheaper to design now than to retrofit into an immutable archive.

### F.7 [MINOR] Base-path breakage is a known silent failure with no requirement

addendum §C.4: *"GitHub project pages serve from `/reponame/`, so base-path configuration is required or asset URLs break silently on first deploy."* Known, documented, silent — and no FR covers it. FR-18 would be the natural home.

### F.8 [MINOR] Bookmarks bypass the homepage

FR-19's consequence *"Reaching a report from the homepage passes the reader through this state"* assumes the homepage is the entry point. Reports are static URLs; a bookmark, a browser suggestion or a shared link goes straight to the report — and FR-20 only requires the report to show a **date**, not source health. UJ-3 promises *"He knows which figures on which reports not to trust"*, but marking lives on the homepage and is **per-source**, while the reader needs it **per-report**. Nothing requires the mapping from failed source to affected report to exist. addendum B.4 proposes dbt exposures for exactly this and it is filed as a convention worth adopting rather than a requirement. UJ-3 is currently unrealized.

---

## G. What is sound

Short, because it is.

- **§4.2 (FR-5/FR-6) is the strongest part of the document.** The archive-first / disposable-warehouse split is coherent, its consequences mostly are testable, and it earns the design decisions elsewhere. The defects above are at its edges (parquet-at-snapshot-time, private durability), not at its centre.
- **§1.3's layer boundary** does what it claims: it states precisely what is closed and why, and addendum §C.4 operationalises it. The §5 contradiction (C.1) is a wording failure in §5, not a failure of the boundary itself.
- **§3's glossary discipline** is real and mostly held — two violations out of fifteen terms is a good record for a document this size.
- **FR-16's annotations-as-data** is the right requirement, correctly stated, with a correctly identified consequence. It just has no supply chain.
- **addendum §A.2/A.3/A.4** are concrete, sourced and directly actionable. That is what an addendum is for.
- **The scope discipline in §0 holds throughout.** The document never once drifts into specifying what a visual shows. Given `.memlog.md` records four attempts, that is a real achievement.

---

## H. Recommended minimum before architecture

Nine items, in order:

1. Fix the three broken references: §D ×2 (A.1), §8 (A.2), FR-19→FR-21 (A.3).
2. Resolve §5 vs §10: does "no chart library" mean *not used to author* or *not in the dependency tree*? Evidence.dev's viability turns on the answer (C.1).
3. Give FR-10 three sentences of actual content: sync/async, who owns the query, what the visual receives (D.1).
4. State the default privacy of an undeclared report, and give NFR-3 a testable consequence (C.5).
5. Answer the dropped question: does the habitual surface carry the tracked numbers, yes or no (D.2)? Then give SM-1 and SM-2 a requirement each, or downgrade them from primary.
6. Add one performance NFR (F.2) and one pipeline-state-independence requirement (F.1).
7. Require sources to declare licence/attribution and backfillability (F.3, C.4).
8. Move FR-9 and FR-21 into Must, or move FR-19's assertion consequence out of Must (C.9).
9. Rewrite the eleven consequences named in B.1–B.3, or drop the word "testable" from the header. The header is a promise the section should keep.
