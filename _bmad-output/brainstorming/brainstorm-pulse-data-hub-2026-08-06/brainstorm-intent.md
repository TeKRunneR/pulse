# pulse — Brainstorming Intent

## 1. What pulse is

pulse is a personal data hub and dashboarding tool: it ingests data from public sources (automatically) and private sources (manually, e.g. bank CSV exports), archives every fetch immutably, builds curated datasets with DuckDB, and publishes hand-built HTML/SVG/JS reports. It exists to answer *standing* questions — ones watched over months and years — about the economy, demographics, climate and personal finances. One-shot questions go to ChatGPT/Claude; pulse is for the questions that never close. The repo is public and doubles as a portfolio piece.

## 2. Goals

1. **Personal utility** — at-a-glance answers to standing questions, with the full history and its evolution visible.
2. **Portfolio** — a public repo that anyone can clone and rebuild end to end; the dbt DAG, the documented datasets and the agent-readiness are themselves the showcase.
3. **Playground for tech he enjoys** — hand-built visuals and duckdb-wasm are wanted for their own sake. This goal was unstated in the original pitch and surfaced only under challenge. It reconciles the choices that look inefficient under goal 1, and it is explicitly **not to be optimised away** by a future efficiency-minded pass.

## 3. Core principles

- **Archive-first.** Value compounds with time-series depth, so never losing history is the core requirement; dashboards are second.
- **The archive/glance loop.** The archive only compounds if the app is used, and it is only used if it is glanceable — so visualization is co-equal with ingestion, not downstream of it. This is the project's engine and orders the roadmap.
- **Warehouse disposable, raw durable.** Every run is a full rebuild from raw; only the raw archive must survive. Consequence: schema changes never need migrations.
- **At-a-glance means zero assembly, not low density.** The competition is the 15 minutes of finding, downloading and plotting. pulse wins by being pre-composed, however dense.
- **Reports, not tiles.** A page is a *standing analysis*: a curated, multi-angle, dense page on one topic, refreshed monthly. Multi-page and scrolling are fine. Annotations (series breaks, COVID, definition changes) are first-class content.
- **Expressiveness-first visuals.** Each visual is tailored to its data and its message; there is no fixed chart vocabulary. This is the crux of the app — a new way to visualize data, without established good practices, which will be discovered through use. Harder reuse is an accepted cost.
- **The data interface is the stable contract.** Maximum predictability below the line (how a visual receives rows, how it registers on a report); maximum freedom above it. This resolves expressiveness vs agent-legibility and is the session's most important architectural statement.
- **Predictable structure, shared implementation.** What repeats is the *shape* — file layout, naming, wiring steps, conventions. What must not repeat is the *code* — shared render components, dbt macros, a common visual library. Abstractions stay shallow and domain-named, and are extracted after the second or third visual, never designed up front (over-abstraction is as agent-hostile as duplication).
- **Agent legibility is the ease metric.** A coding agent does the work, so the target is: can the agent understand the workflow easily — straightforward, existing code to copy, describable in markdown, encodable as a skill. Conventional frameworks beat bespoke pipelines; escape hatches (e.g. `fetch.py`) stay rare and clearly marked.
- **Structure follows content.** No premature universal abstraction, ever imposed up front. Principles are agreed now; specifics are settled at design and implementation time.

## 4. Architecture decisions

- **Sources as folders.** A source is a directory (`source.toml` + optional fetch script); pulse discovers folders, so adding a source is adding a directory.
- **Immutable raw snapshots.** Ingestion writes a snapshot per run *before* any transform. File-based sources are kept in their original format; parquet is used only when the source is not already a file. Everything downstream is replayable from raw.
- **Public raw is committed in-repo via Git LFS.** Snapshots are small (KBs–MBs/yr) and make the whole warehouse reproducible by anyone cloning — reproducibility as portfolio evidence. Monthly snapshot commits also keep the scheduled workflow alive.
- **Private raw is gitignored and local-only on one laptop for v1.** Durable/online storage for it is deferred to a future study.
- **Datasets are wide, properly typed tables**, curated per question area, built from raw; visuals query datasets. No tall/long observations table anywhere, including within a dataset. Schema changes are acceptable — there is no service-continuity constraint, just a need for a defined workflow.
- **dbt-duckdb** is the transformation layer: staging models → datasets, with data tests, and `schema.yml` descriptions serving as the dataset metadata sidecar / semantic layer; `manifest.json` gives a machine-readable briefing for agents. Ingestion (fetch + snapshot) stays **outside** dbt, with raw files registered as dbt external sources. dbt source freshness covers the staleness half of silent breakage.
- **No revision/vintage modelling.** Datasets build from the latest snapshot only. All snapshots stay in raw, so revision archaeology is possible as an opt-in extra dataset, per source, rather than a global tax.
- **Static site + local server.** Build produces static HTML; `pulse serve` is mandatory rather than convenient, because duckdb-wasm cannot run from `file://`. No always-running local service. The dual model (local = static files + tiny web server, remote = static hosting) is settled at design time.
- **Two ingestion runtimes** are unavoidable: GitHub Actions for public sources, local runs for private data that can never leave the laptop.
- **duckdb-wasm is kept**, because in-browser real-time analysis is genuinely wanted (goal 3). It is made **optional per report**: default is query results baked to JSON at build time (instant, no wasm payload); reports needing open-ended slicing opt in. This lets one report be a deliberate wasm flagship with bold slicing instead of taxing every page.
- **Dashboard privacy is a flag** (`private=true` in `dashboard.toml`) plus data-root indirection and build profiles: the public build simply omits private reports and their data slices; the same dashboard code runs both ways.
- **Visual contract.** A visual = query + render file + manifest; a report = a manifest listing visuals + layout. Visuals are authored against a mock `data` array with a declared column schema; wiring is swapping the mock for the real query result.
- **The repo carries its own instructions** as first-class artifacts: `CLAUDE.md` plus skills for "add a source", "add an indicator", "add a visual". The first vertical slice's real job is to be the exemplar the agent copies.
- **Candidate simplification (SHOULD):** dbt-duckdb materialises datasets straight to parquet, making the warehouse a directory of parquet files and collapsing the separate publish/export step.

## 5. v1 scope (MoSCoW — confirmed)

**Must**
- One public source (World Bank / public macro) with `source.toml`
- Immutable raw snapshots via Git LFS
- dbt-duckdb: staging → one wide dataset, with `schema.yml` docs and tests
- One real France macro report with several hand-built visuals
- The data-interface contract
- Build + `pulse serve` + public deploy
- Monthly GitHub Action running the whole chain
- "Data as of" freshness on the page
- `CLAUDE.md` + add-a-* skills

**Should**
- Per-source assertions beyond dbt defaults (row count, expected columns, latest period advanced)
- Index page
- Chart annotations
- Theme tokens
- Datasets materialised straight to parquet
- One duckdb-wasm flagship report

**Could**
- Manual/private ingestion path for bank data
- A second and third report

**Won't this time**
- Data lab (agentic querying)
- Lab-to-dashboard promotion
- Revision modelling
- Cross-filtering
- Durable private-data storage
- Multi-source datasets

## 6. Open items

**Unresolved, carried forward**
1. Local vs remote serving specifics (static files + tiny web server locally, static hosting remotely).
2. Host choice — GitHub Pages vs Cloudflare Pages/Netlify — since SharedArrayBuffer needs COOP/COEP headers that Pages cannot set, which affects duckdb-wasm performance mode.
3. Per-visual vs per-dashboard query. Safe to defer because the visual contract hides the difference; only cross-filtering would force the choice early. Dense multi-angle reports tilt this toward one dashboard-level slice queried many ways.
4. Whether the personal tool and the public repo should eventually separate.

**Watch items and risks**
- **Git LFS budget** — GitHub free tier is ~1GB storage + 1GB/month bandwidth per account, and public-repo clones draw on the owner's bandwidth. Set `lfs:false` on Actions checkout steps that do not need snapshots.
- **Scheduled-workflow auto-disable** — GitHub disables scheduled workflows after 60 days of repo inactivity; monthly snapshot commits are the intended defusal.
- **Silent success with bad data** — the real failure mode is not a failing job (Actions notifies on non-zero exit) but a succeeding one: HTTP 200 empty file, silent schema change, stale period. Mitigation is cheap per-source assertions plus freshness surfaced in the UI (per-source freshness on the index, stale/failed sources visibly marked) so breakage is caught where it is looked at.
- **Private data retention** — a local-only archive on one laptop dies with the laptop; accepted for v1, needs its own backup story later.
- **Public repo vs personal tool tension** — portfolio pressure can suppress useful-but-messy hacks and personally revealing indicators; the two goals pull in different directions (see open item 4).
- **The v1 bet to watch** — if bespoke visuals prove slow to produce even with coding agents, the accepted-cost decision on harder reuse is the first one that gets revisited.
