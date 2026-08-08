---
title: "pulse — Product Requirements Document"
status: draft
created: 2026-08-07
updated: 2026-08-07
---

# PRD: pulse

## 0. Document Purpose

This PRD specifies **pulse**, a personal data hub and reporting tool. It is written for the downstream BMad workflows that consume it — architecture, UX, and epic breakdown — and for the coding agents that will build from those. It builds on two prior artifacts and does not duplicate them: [`docs/initial-pitch.md`](../../../../docs/initial-pitch.md) and the [brainstorming synthesis](../../../brainstorming/brainstorm-pulse-data-hub-2026-08-06/brainstorm-intent.md) of 2026-08-06.

Vocabulary is anchored in §3 Glossary and used verbatim throughout. Features are grouped in §4 with functional requirements nested and numbered globally (FR-1…FR-N) so downstream artifacts have stable references. Inferences not confirmed by the user are tagged `[ASSUMPTION]` inline and indexed in §11.

**Scope discipline.** pulse is an engine that produces reports; it is not the reports. This document specifies the engine and the workflows that operate it. What any individual visual shows — which figures, at what precision, with what comparisons — is decided per visual, in session, against real data. Those decisions are deliberately absent here and belong nowhere in this document.

Technical depth, competitive research, superseded framings and the decision record live in [`addendum.md`](addendum.md). The running decision log is [`.memlog.md`](.memlog.md).

---

## 1. Vision

pulse answers **standing questions** — the ones watched over months and years, not the ones closed in an afternoon. It fetches public data automatically, accepts private data manually, archives every fetch immutably so the time series compounds, rebuilds a curated warehouse from that archive on every run, and publishes dense, purpose-built reports.

Its engine is a loop: the raw archive only compounds if pulse keeps being used, and pulse is only used if the reports are worth opening. **Visualization is therefore co-equal with ingestion, not downstream of it.** This ordering is not a preference — every comparable personal-data-warehouse project surveyed (Dogsheep, HPI, QS Ledger, datadex) converges on the same ingestion stack and then dies at the presentation layer. See [addendum §B.3](addendum.md).

The repository is public and reproducible end to end by anyone who clones it. The DAG, the documented datasets and the agent-authorable workflows are themselves the showcase. And pulse is deliberately a playground for tooling that is enjoyable to build: purpose-built visuals and duckdb-wasm are wanted for their own sake, not because they are the efficient choice.

### 1.1 Goals

1. **Ambient familiarity.** Carry key numbers in mind — their approximate magnitude and direction — without looking them up. The reference experience is what an economics researcher acquires by constantly reading prose that uses those numbers: retention through frequent incidental exposure, not efficient retrieval. Economic indicators are the worked example; climate and emissions indicators are equally in view.
2. **Portfolio.** A public repository anyone can clone and rebuild end to end.
3. **Playground.** Purpose-built visuals and duckdb-wasm are wanted for their own sake. This goal was unstated in the original pitch and surfaced only under challenge. It reconciles the choices that look inefficient under goal 1, and it is **explicitly not to be optimised away** by a future efficiency-minded pass, human or agent.

### 1.2 Core Principles

These constrain every requirement below and survive into architecture.

- **Archive-first.** Value compounds with time-series depth. Never losing history is the core requirement; reports are second.
- **Warehouse disposable, raw durable.** Every run is a full rebuild from the raw archive. Only the raw archive must survive. Consequence: schema changes never require migrations.
- **At-a-glance means zero assembly, not low density.** The competition is fifteen minutes of finding, downloading and plotting. pulse wins by being pre-composed, however dense.
- **Reports, not tiles.** A report is a standing analysis: a curated, multi-angle page on one topic. Multi-page and scrolling are acceptable.
- **Expressiveness-first visuals.** Each visual is purpose-built for its data and its message. There is no fixed chart vocabulary. Harder reuse is an accepted cost.
- **The data interface is the stable contract.** Maximum predictability in how a visual receives rows and registers on a report; maximum freedom above that line.
- **Predictable structure, shared implementation.** What repeats is the *shape* — file layout, naming, wiring steps, conventions. What must **not** repeat is the *code*. Shared render components, macros and libraries carry the implementation. Abstractions stay shallow and domain-named, extracted after the second or third visual rather than designed up front, because over-abstraction is as hostile to an agent as duplication.
- **Structure follows content.** No premature universal abstraction, ever imposed up front. Conventions emerge from use and stay revisable.

### 1.3 What pulse builds, and what it may adopt

pulse's stack — static site, DuckDB, code-authored charts, scheduled rebuild — is not novel. Evidence.dev ships it; Observable Framework ships it with a stronger ingest story. What pulse declines is narrower than the whole stack, and stating the boundary precisely is what keeps it from being relitigated.

**The non-negotiable is one layer: visual authoring.** No chart or visualization library, at any level. Every visual is purpose-built. This is goal 3 expressed as a constraint — a tool that supplies the expressive visual layer removes the part of the project that makes it worth doing.

**Everything below that line is open.** Fetch mechanism, transformation, site build, data delivery and dev server may all be adopted rather than built, provided the adopted tool does not constrain visual authoring. "Sources as folders" is a structuring convention and carries no implication that ingestion must be bespoke.

The data interface is the contribution: maximum predictability in how a visual receives rows and registers on a report, maximum freedom above that line. That contract is what makes an unusual visual cheap to add rather than a research project.

**The condition under which this is wrong:** if purpose-built visuals prove slow to produce *even with agent authoring*, the premise fails and this is the first decision to revisit. Tracked as a risk in §9.

---

## 2. Target User

pulse has one user and one audience. They want different things and the tension between them is real (§9).

### 2.1 Jobs To Be Done

**As the user:**
- Know the current approximate value and direction of the indicators I care about, without looking them up
- Check whether my picture of a topic is still accurate, at whatever moment I happen to wonder
- Keep a history that gets more valuable the longer it runs, without curating it by hand
- Know at a glance whether the whole machine is still working

**As the builder:**
- Add a source, an indicator, a visual or a report without it becoming a project
- Build something functional and elegant, and enjoy building it
- Show it to people as evidence of how I work

### 2.2 Non-Users (v1)

pulse is not built for anyone else to use, configure or extend. Strangers are expected to **read and rebuild** the repository, not to operate it against their own data. There is no multi-user model, no configuration UI, no onboarding path, and no support commitment.

### 2.3 Key User Journeys

Deliberately light — a single operator, no authentication, no multi-device handoff.

- **UJ-1. Yann checks a number he half-remembers.** Mid-conversation or mid-article, he wants the current French inflation rate. He opens pulse, lands on the homepage, clicks through to the macro report, and reads the figure in context alongside its recent trajectory and comparable countries. He closes it. The number is a little more familiar than it was.
- **UJ-2. Yann notices the machine is fine.** He opens pulse to reach a report and passes through the homepage, which shows every source's currency and the state of the last pipeline run. Nothing is marked. He continues to the report without having decided to check anything.
- **UJ-3. Yann notices the machine is not fine.** Same path, but a source is marked stale and another is flagged as having failed its assertions. He knows which figures on which reports not to trust, and has what he needs to investigate.
- **UJ-4. Yann adds a source.** He decides to track a new indicator. He follows the defined workflow for adding a source, and a coding agent performs it by following existing shape and calling shared implementation. The new source appears in the next pipeline run and on the homepage.
- **UJ-5. Yann adds a visual.** He works with an agent to author a purpose-built visual against a dataset, then wires it to the real query and registers it on a report. Existing visuals are unaffected.
- **UJ-6. A stranger rebuilds pulse.** Someone clones the public repository and rebuilds the warehouse and site end to end from the committed raw archive, without access to Yann's machine or any private data.

---

## 3. Glossary

Used verbatim throughout. Introducing a synonym anywhere in this document is a discipline violation.

- **Source** — a configured origin of data. **Public sources** are fetched automatically; **private sources** are supplied manually. Each source has its own ingestion cadence.
- **Snapshot** — an immutable copy of one source's data as fetched at one point in time. Never modified, never deleted.
- **Raw archive** — the complete set of snapshots across all sources. Durable; the only thing that must survive.
- **Dataset** — a wide, properly typed table curated for one question area, built from snapshots. The unit visuals query. Never a tall/long observations table.
- **Warehouse** — the built collection of datasets. Disposable; fully rebuilt from the raw archive on every pipeline run.
- **Visual** — a purpose-built rendering answering one question, authored against a dataset.
- **Purpose-built** — tailored to one question and its data, with no chart-type vocabulary and no visualization library. Describes the artifact, not its authorship.
- **Agent-authored** — produced by an AI agent rather than by a person. Describes authorship, not the artifact. See [addendum §C](addendum.md) for the concrete pipeline.
- **Report** — a curated, multi-angle page on one topic, composed of visuals.
- **Homepage** — the landing surface. Provides navigation to reports and displays pipeline state.
- **Data interface** — the contract by which a visual receives rows and registers on a report.
- **Visual language** — the store of conventions that apply across visuals and that the authoring agent consumes.
- **Annotation** — contextual information carried as data and joined to a visual: series breaks, definition changes, external events.
- **Pipeline run** — one execution of the chain: fetch → snapshot → warehouse build → report generation → publish.
- **Build profile** — public or private. Determines which reports and which data are included in an output.
- **Assertion** — a per-source check that data which arrived is plausible: row count, expected columns, latest period advanced.

---

## 4. Features

### 4.1 Sources and ingestion

**Description.** A source is a configured origin of data with its own cadence. Public sources are fetched automatically on a schedule; private sources are supplied manually because the data cannot leave the local machine. Two runtimes are therefore unavoidable and are a permanent property of the system, not a v1 compromise. Realizes UJ-4.

#### FR-1: Declare a source
A source is declared by configuration that identifies it, describes how its data is obtained, and states its cadence.

**Consequences (testable):**
- Adding a source requires no change to shared ingestion code.
- Sources are discovered from their declarations; there is no central registry to update.

#### FR-2: Per-source ingestion cadence
Each source declares its own cadence. No cadence is assumed system-wide.

**Consequences (testable):**
- Changing a source from monthly to weekly is a scheduling change only — no code and no architecture change.
- Sources with different cadences coexist in one pipeline run.

#### FR-3: Automatic ingestion of public sources
Public sources are fetched on schedule without human involvement.

**Consequences (testable):**
- A scheduled pipeline run completes with no manual step.
- A run that is delayed or dropped by the scheduler is recoverable by the following run; the pipeline is idempotent and a missed run is not data loss.

#### FR-4: Manual ingestion of private sources
Private data is supplied by placing files in a known location. `[ASSUMPTION: manual ingestion is a v1 Could, not a Must — see §7.]`

**Consequences (testable):**
- Private data never leaves the local machine and never enters the public repository.

---

### 4.2 Raw archive and replay

**Description.** Ingestion writes a snapshot before any transformation. Snapshots are immutable and permanent. Everything downstream is derived and replayable, which is what makes schema changes free and the warehouse disposable.

#### FR-5: Immutable snapshots
Every fetch writes a snapshot of the source's data as received, before any transformation.

**Consequences (testable):**
- A snapshot is never modified or deleted after being written.
- File-based sources are kept in their original format.
- Snapshots for public sources are committed to the repository and are available to anyone who clones it.

#### FR-6: Full rebuild from raw
The warehouse is rebuilt in full from the raw archive on every pipeline run.

**Consequences (testable):**
- Deleting the entire warehouse and rebuilding produces an equivalent result.
- A schema change requires no migration.
- A rebuild at any time reproduces the current warehouse from committed snapshots alone.

---

### 4.3 Datasets and transformation

**Description.** Snapshots are transformed into datasets: wide, properly typed tables curated per question area. Datasets carry their own documentation, which doubles as the semantic layer and as a machine-readable briefing for agents.

#### FR-7: Build datasets from snapshots
Datasets are built from snapshots through a declared transformation layer.

**Consequences (testable):**
- Datasets are wide and properly typed; no tall/long observations table exists at any level.
- Each dataset builds from the latest snapshot of its sources. `[ASSUMPTION: v1 builds from latest snapshot only; revision modelling is a non-goal — see §5.]`

#### FR-8: Dataset documentation as semantic layer
Every dataset and its columns carry descriptions alongside the transformation code.

**Consequences (testable):**
- Documentation is machine-readable and available to an agent without reading transformation code.
- A dataset added without documentation is detectable.

#### FR-9: Data tests and per-source assertions
Datasets carry data tests. Sources carry assertions that check whether arriving data is plausible.

**Consequences (testable):**
- A source whose row count collapses, whose expected columns are missing, or whose latest period failed to advance is flagged.
- A failing assertion does not stop the pipeline run — see FR-19.

---

### 4.4 Visual authoring

**Description.** A visual is purpose-built for one question and agent-authored. It is authored against a dataset and then wired to a real query. The data interface is what makes wiring mechanical: however unusual a visual is, it receives its rows and registers on its report the same predictable way. This section carries the brainstorm's most important architectural statement. Realizes UJ-5.

#### FR-10: The data interface contract
A visual receives its data, and registers on a report, through a single predictable contract.

**Consequences (testable):**
- Two visuals of entirely different design receive their rows identically.
- Wiring an authored visual to real data requires no change to the visual's rendering logic beyond the data source.
- An agent can register a new visual on a report by following the contract without reading other visuals' rendering code.

#### FR-11: Authoring against a declared schema
A visual is authored against a declared column schema before real data is available.

**Consequences (testable):**
- A visual renders during authoring without a built warehouse.
- Moving from authoring to wired is a substitution, not a rewrite.

#### FR-12: No visualization library
No chart or visualization library is used at any layer of the system.

**Consequences (testable):**
- No chart library appears in the dependency tree.
- No visual is produced by selecting a chart type.

#### FR-13: Cross-visual conventions
Conventions that apply across visuals live in one place that the authoring agent consumes.

**Consequences (testable):**
- The store of conventions starts close to empty and grows only as conventions emerge from use.
- Adding or changing a convention does not require touching every existing visual.
- A convention can be revised or withdrawn.

#### FR-14: Bounded cost of accommodating new kinds
Accommodating a new kind of data or visual may require development, but must not require reinventing the system.

**Consequences (testable):**
- Existing visuals continue to work when the system is extended for a new kind.
- The change required is additive and bounded, not a rewrite of shared machinery.

**Notes.** `[NOTE FOR PM]` This feature deliberately does not specify what visuals show — figures, precision, comparisons, layout and emphasis are decided per visual, in session, against real data. Attempts to derive such rules from the goals were raised and rejected four times during discovery; see [addendum §C.5](addendum.md).

---

### 4.5 Report composition

**Description.** A report is a standing analysis on one topic, composed of visuals. Annotations — series breaks, definition changes, external events — are content, not decoration, and are carried as data so they can be joined rather than hardcoded.

#### FR-15: Compose a report from visuals
A report declares the visuals it contains and their arrangement.

**Consequences (testable):**
- Adding a visual to a report is a declaration change.
- A report renders with visuals of arbitrary design.

#### FR-16: Annotations as data
Annotations are stored as data and joined to visuals.

**Consequences (testable):**
- No annotation is expressed as a hardcoded coordinate.
- An annotation added to the data appears without changing visual code.

#### FR-17: Report privacy
A report declares whether it is private. Build profiles determine what is included in an output.

**Consequences (testable):**
- A public build contains no private report and no data belonging to one.
- The same report code runs under both profiles.

---

### 4.6 Build, serve and deploy

**Description.** pulse produces static output. Locally it is served by a small web server, which is mandatory rather than convenient because duckdb-wasm cannot run from `file://`. There is no always-running service. Realizes UJ-1, UJ-6.

#### FR-18: Static output, served locally and hosted remotely
A build produces static output servable both from a local server and from static hosting.

**Consequences (testable):**
- The output deploys to static hosting requiring no custom HTTP headers.
- Opening reports locally is a single command.
- Data is queried in the browser via duckdb-wasm on all pages.

---

### 4.7 Homepage and pipeline visibility

**Description.** The homepage does two jobs, and the second depends on the first. It is the way into the reports, so it is passed through habitually; because it is passed through habitually, it is the one place where the system's health will actually be seen. A status page nobody opens deliberately is worth nothing.

The failure this defends against is specific. A crashed pipeline announces itself — the job exits non-zero and notifies. The dangerous failure is the one that succeeds: an empty response with a 200 status, a silently renamed column, a period that never advanced. The run goes green, the site deploys, the report renders, and a wrong or stale figure sits on a page being read for five seconds. Realizes UJ-2, UJ-3.

#### FR-19: Pipeline state on the homepage
The homepage shows the state of the last pipeline run across ingestion, warehouse build and report generation, and the currency of each source.

**Consequences (testable):**
- A source that has gone stale is visibly marked.
- A source that failed its assertions is visibly marked.
- A failed stage of the last pipeline run is visibly marked.
- Reaching a report from the homepage passes the reader through this state.

#### FR-20: Data currency on the report
A report shows how current the data behind it is.

**Consequences (testable):**
- Each report states the date of the data it displays.

#### FR-21: Suspect data is published and marked
When data arrives but fails its assertions, the pipeline builds with it and marks it.

**Consequences (testable):**
- A failing assertion does not stop other sources, datasets or reports from building.
- The affected source is marked on the homepage.

**Notes.** `[NOTE FOR PM]` The alternative — rebuilding a failing source from its last snapshot that passed, so a wrong figure never reaches a page — was considered and deferred. It is more correct and the raw archive makes it possible, but it needs snapshot-selection machinery, per-source last-pass tracking and per-source rebuild for a rare event. Recorded as a future goal; see §8.

---

### 4.8 Authoring workflows

**Description.** Adding a source, an indicator, a visual or a report follows a defined workflow. The workflow is a deliverable of pulse, not documentation about it — the repository carries its own instructions as first-class artifacts. Realizes UJ-4, UJ-5.

#### FR-22: Defined workflows for extension
Each extension task has a defined workflow: a known sequence of steps against a known shape.

**Consequences (testable):**
- Each of "add a source", "add an indicator", "add a visual" and "add a report" has a workflow that can be followed without reading unrelated system code.
- Following a workflow requires no decisions that are not stated in it.

#### FR-23: Structure repeats, code does not
Workflows repeat *shape* — file layout, naming, wiring steps. Implementation is shared.

**Consequences (testable):**
- Two sources added at different times have the same shape.
- Adding a source does not duplicate ingestion logic.
- A shared abstraction is introduced only after the second or third instance of its kind, never designed up front.

#### FR-24: An exemplar to copy
The first complete instance of each kind serves as the exemplar the workflow refers to.

**Consequences (testable):**
- Each workflow points at a working instance in the repository.

---

## 5. Non-Goals (Explicit)

pulse is not, and will not become in v1:

- **A data lab.** Natural-language querying of the data by an agent was in the original pitch and is deferred entirely.
- **A promotion path from exploration to reports.** Follows from the above.
- **A revision or vintage model.** Datasets build from the latest snapshot. Because all snapshots are retained, revision archaeology remains possible later as an opt-in extra dataset per source, rather than a tax paid globally now.
- **Cross-filtering between visuals.** Deliberately excluded; it is also the only thing that would force the per-visual versus per-report query decision (§8).
- **Durable storage for private data.** Private raw is local-only on one machine in v1.
- **Multi-source datasets.** A dataset is built from one source in v1.
- **A BI tool for other people.** No multi-user model, no configuration UI, no onboarding.
- **A chart library, or a consumer of one.** At any layer, ever.
- **An always-running service.** Nothing runs continuously; builds are scheduled and serving is on demand.

---

## 6. Cross-Cutting Non-Functional Requirements

- **NFR-1 — Agent legibility is the ease metric.** A coding agent does the work, so "easy" means legible to an agent: a workflow it can follow, existing shape it can match, shared implementation it can call, and a description that fits in markdown. Conventional, widely-known tools beat bespoke pipelines for this reason; escape hatches stay rare and clearly marked. This binds §4.1 through §4.8 equally.
- **NFR-2 — Reproducibility.** Anyone cloning the public repository can rebuild the warehouse and the public site end to end, with no access to private data or to the author's machine. Reproducibility is portfolio evidence, so it is a requirement rather than a nicety.
- **NFR-3 — Privacy separation.** Private data and private reports never enter a public output. The separation is structural, not procedural — it does not depend on remembering to exclude anything.
- **NFR-4 — Cost.** pulse runs within free tiers of its hosting and automation providers under normal operation. See [addendum §D](addendum.md) for the storage-quota analysis and its watch item.
- **NFR-5 — No service continuity constraint.** pulse may break, change schema, or be rebuilt from scratch without notice or migration. Nothing depends on its uptime.

---

## 7. MVP Scope

### 7.1 In Scope

- One public source, declared as configuration
- Immutable snapshots committed to the repository via Git LFS
- A transformation layer producing one wide, documented, tested dataset from staging models
- One real report on French macroeconomic data, with several purpose-built visuals
- The data interface contract
- Datasets materialised to parquet *(promoted from Should: duckdb-wasm reads parquet directly, so the alternative is an extra export step on every page)*
- Build, local serve, and public deploy
- A scheduled pipeline run executing the whole chain
- Homepage with navigation and pipeline state *(promoted from Should: FR-19 has nowhere else to live)*
- Data currency shown on reports
- Defined authoring workflows, with the first vertical slice as their exemplar

### 7.2 Should

- Per-source assertions beyond the transformation layer's defaults
- Annotations as data
- A store for cross-visual conventions

### 7.3 Could

- Manual ingestion path for private data
- A second and third report

### 7.4 Out of Scope for MVP

Everything in §5, plus:

- ~~One duckdb-wasm flagship report~~ — **moot.** duckdb-wasm now runs on every page, so there is no flagship to designate. This removes the dual data-delivery path, the per-report flag, and two code paths that would both have had to keep working.

`[NOTE FOR PM]` The manual private-data path (§7.3) is emotionally load-bearing — personal finances were one of the four original motivating topics. It is a Could only because private data has no durable storage story yet, not because it matters less.

---

## 8. Success Metrics

**Primary**
- **SM-1 — Ambient familiarity.** Core indicators for tracked topics can be recalled to the right magnitude and direction without looking them up. Crude test: state them from memory, then check. This is the actual goal; everything else is instrumental. Validates FR-15, FR-18, FR-20.
- **SM-2 — Habitual opening.** pulse is opened as part of a rhythm rather than as a remembered task. Validates FR-18, FR-19.

**Secondary**
- **SM-3 — Reflex source.** For a tracked indicator, pulse is where Yann goes, ahead of a search engine. Validates FR-15, FR-18.
- **SM-4 — Flat extension cost.** Adding a source or a visual does not get slower as the system grows. Validates FR-13, FR-14, FR-22, FR-23.
- **SM-5 — Archive integrity.** After a year, the raw archive has no unexplained gaps. Validates FR-3, FR-5, FR-19.
- **SM-6 — Reproducibility.** A stranger can clone the repository and rebuild it end to end. Validates FR-6, NFR-2.
- **SM-7 — Craft.** The result is something Yann considers functional and elegant. Valid on its own terms, explicitly including the case where SM-1 and SM-2 disappoint. Validates goal 3.

**Counter-metrics (do not optimise)**
- **SM-C1 — Density over memorability.** Indicators per report rising while recall falls. More figures on a page is not better if none of them stick. Counterbalances SM-1's pull toward completeness.
- **SM-C2 — Breadth over repetition.** Topics covered rising while revisits per topic fall. Each new topic dilutes the exposure existing ones need. Counterbalances SM-4 — making extension cheap is exactly what makes over-extension tempting.

---

## 9. Risks and Watch Items

- **The v1 bet.** If purpose-built visuals prove slow to produce even with agent authoring, the accepted cost on harder reuse is the first decision to revisit (§1.3).
- **Portfolio versus personal tool.** Portfolio pressure can suppress useful-but-messy work and personally revealing indicators. The two goals pull in different directions; see §10 item 3.
- **Storage quota.** Snapshot bandwidth is billed to the repository owner and is most likely consumed by scrapers, bots and CI forks — which is uncorrelated with the project succeeding. Analysis, decision and mitigations in [addendum §D](addendum.md).
- **Scheduler decay and dropped runs.** Scheduled automation is disabled after prolonged repository inactivity, and individual scheduled runs may be dropped. Monthly snapshot commits defuse the first; FR-3's idempotence requirement covers the second.
- **Silent success with bad data.** Mitigated but not eliminated by FR-9, FR-19 and FR-21, which detect and mark rather than prevent.

---

## 10. Open Questions

1. **Architecture candidate.** Evidence.dev, Observable Framework, or a general build tool with pulse's own conventions. Carried forward with a per-candidate cost ledger in [addendum §B.2](addendum.md).
2. **Visual wrapping friction.** If Evidence.dev is selected, authored HTML/SVG must be wrapped as a component before wiring. Whether that is mechanical enough to encode in a workflow is the sharpest practical difference between the candidates, and should be settled by building one visual each way rather than by argument ([addendum §C.3](addendum.md)).
3. **Should the personal tool and the public repository eventually separate?** Carried from the brainstorm, unresolved.
4. **Per-visual or per-report query.** Safe to defer: the data interface hides the difference, and only cross-filtering — a non-goal — would force the choice early.
5. **Local serving specifics.** Static files plus a small local server; the details are architecture's.
6. **Durable storage for private raw data.** Deferred with the private ingestion path.

*Resolved during discovery:* host choice and duckdb-wasm threading — single-threaded operation requires no special HTTP headers, so ordinary static hosting remains viable ([addendum §A.2](addendum.md)).

---

## 11. Assumptions Index

- **§4.1, FR-4** — Manual private-data ingestion is a v1 Could rather than a Must, following the brainstorm's MoSCoW.
- **§4.3, FR-7** — Datasets build from the latest snapshot only; revision modelling is a non-goal.
- **§7.1** — One public source and one report constitute the v1 vertical slice, per the brainstorm's Must list.
- **§2.3** — User journeys are stated lightly because there is a single operator, no authentication and no multi-device handoff.
