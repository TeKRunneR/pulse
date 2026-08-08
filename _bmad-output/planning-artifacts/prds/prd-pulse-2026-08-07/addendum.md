# pulse — PRD Addendum

Depth that belongs downstream (architecture, solution design, UX spec) rather than in the PRD narrative.

---

## A. Infrastructure feasibility research (2026-08-07)

Web research commissioned during Discovery to resolve brainstorm open items 2 and several watch items. Findings are grouped by the question they answer. Where a finding contradicts a brainstorm decision, that is flagged.

### A.1 Raw snapshot storage — Git LFS retained (decision 2026-08-07)

**Outcome: the brainstorm decision stands.** Public raw snapshots are committed in-repo via Git LFS. The research below argued against it; the bandwidth objection was withdrawn on review as overstated at pulse's actual data volume. The findings are retained because two of them convert into requirements — see *Decision* at the end of this section.

**Research findings as reported:**

- Free/Pro Git LFS quota is 10 GiB storage + 10 GiB bandwidth/month, **per account, not per repo** — shared across all of Yann's repos.
- GitHub documents that LFS download bandwidth is charged to the **repository owner**, regardless of who clones. On a public portfolio repo, any stranger, scraper, or CI fork cloning with LFS burns Yann's monthly quota. This directly undermines the "anyone can clone and rebuild" portfolio goal — the more successful the portfolio piece, the faster the quota drains.
- Overage is post-paid metered (~$0.07/GiB-mo storage, ~$0.0875/GiB egress). With no payment method or a $0 budget cap, GitHub **hard-blocks** pushes and LFS pulls for the rest of the calendar month.
- `actions/checkout` defaults to `lfs: false` (pointer files only, no bandwidth) — but dbt would then read 130-byte pointers instead of data, so the monthly job must set `lfs: true`, which is reported to consume the owner's quota even from GitHub-hosted runners.
- At the stated snapshot volume (KBs–MBs per run, tens of MB per year), **plain git handles this comfortably**: delta compression applies, `git log` diffs stay meaningful, no metering, no pointer indirection. GitHub's repo soft limit is ~1 GB recommended / 5 GB warning; the hard per-file block is 100 MB.

*Original recommendation was: commit snapshots with plain git, reserving GitHub Release assets as an escape hatch above ~50 MB.*

**Decision (2026-08-07): retain Git LFS.**

The quota argument does not survive contact with pulse's actual volume. At KB–MB per snapshot, 10 GiB/month is on the order of two hundred full clones of a 50 MB LFS payload — a level of traffic a personal portfolio repo will not reach, and if it did, the overage is recoverable rather than destructive. GitHub's own guidance also does point toward LFS for large files, though that guidance targets large individual binaries that delta-compress poorly, which is the opposite of pulse's many-small-text-files shape.

The concern that survives is unrelated to quota: **LFS interacts badly with the clone-and-rebuild portfolio goal.** A visitor who clones without `git-lfs` installed receives 130-byte pointer files that superficially resemble data; dbt then parses pointer text where it expects CSV, and the failure presents as a confusing parse error rather than an obvious missing-data error. Secondarily, `git log -p` over a snapshot ceases to be readable, costing some of the archive's introspectability.

Two mitigations were accepted and are carried into the PRD as requirements rather than notes:

1. The README states the `git lfs install` prerequisite up front, **and** the build performs a pointer-file check that fails loudly rather than allowing dbt to parse pointer text.
2. Actions checkout sets `lfs: true` only on steps that require snapshots, and `lfs: false` everywhere else.

Retained as a watch item: GitHub LFS overage at a $0 budget hard-blocks pushes and LFS pulls for the remainder of the calendar month. Cheap insurance is to set a non-zero budget cap or a billing alert.

### A.2 duckdb-wasm hosting — resolves brainstorm open item 2

duckdb-wasm ships three bundles selected at runtime by `selectBundle()`:

| Bundle | Needs SharedArrayBuffer / COOP+COEP | Notes |
|---|---|---|
| MVP | No | Maximum compatibility |
| **EH** | **No** | WASM exception handling; the practical default |
| COI | Yes | pthread worker; multithreading is still flagged experimental upstream, with open issues on extension loading and OPFS |

Single-threaded EH loses intra-query parallelism (roughly linear slowdown on scan/aggregate-heavy work) but **loses no SQL features** over parquet.

Host header support: GitHub Pages **cannot** set custom headers (open request since 2021, no ETA). Cloudflare Pages, Netlify and Vercel all can, via `_headers` / `vercel.json`.

`coi-serviceworker` is a workaround but a fragile one: separate same-origin file, HTTPS or localhost only, forces a page reload on first visit, scope must match subdirectory deploys, and enabling COEP then blocks cross-origin subresources unless they send CORP.

**Recommendation:** target the single-threaded EH bundle, require no special headers, keep GitHub Pages viable. If profiling later proves threads are needed, migrate to Cloudflare Pages and set COOP/COEP there. Do not adopt coi-serviceworker.

### A.3 Parquet over static hosting — range reads work

DuckDB combines parquet footer metadata with HTTP range requests to fetch only the needed row groups and column chunks; no backend required. The host must send `Accept-Ranges: bytes` and honor `Range` — GitHub Pages and all major CDNs do.

Cross-origin parquet additionally needs `Access-Control-Allow-Origin` **and** `Access-Control-Expose-Headers: Content-Range, Accept-Ranges, Content-Length`. GitHub Pages sends `ACAO: *` on public sites but exposed-header support is unverified. Serving parquet **same-origin** sidesteps the question entirely.

Performance caveat from the duckdb-wasm issue tracker: remote reads are effectively sequential/blocking XHR, so for files up to ~50–100 MB a whole-file fetch often beats many small range requests. Range reads win on larger files with selective access.

**Recommendation:** ship parquet same-origin alongside the site; target roughly 5–50 MB per file, partitioning by year if a dataset outgrows that.

### A.4 Scheduled GitHub Actions decay — brainstorm defusal confirmed correct

GitHub disables scheduled workflows in public repos after 60 days with no repository activity (admins are emailed first). GitHub does not define "activity" in its docs; community consensus is that only **new commits to the default branch** reset the timer — issues, PR merges, releases, tags and workflow runs (including `workflow_dispatch`) do not. A known bug also disables the workflow's non-schedule triggers at the same time.

The brainstorm's intended defusal — monthly snapshot commits — is therefore the correct one and makes this a non-issue, provided the monthly job commits to the default branch. Fallback tooling exists (`gautamkrishnar/keepalive-workflow`, `PhrozenByte/gh-workflow-immortality`) but should not be needed.

Cron reliability: GitHub documents that `schedule` runs can be delayed under high load, and that queued jobs **may be dropped entirely**. Minimum interval is 5 minutes. Mitigation: avoid `:00`, pick an odd minute, add `workflow_dispatch`, and make the job idempotent so a dropped run self-heals the following month.

### A.5 dbt-duckdb `external` materialization to parquet

Supported and stable (adapter targets dbt-core ≥1.8, duckdb ≥1.0). Configured via `location`, `format` (parquet inferred from extension since 1.4.1), `options` passed through to `COPY`, and `external_root` in the profile.

Gotchas:
- **Incremental strategies are explicitly unsupported for `external` models** — every run is a full rewrite. Consistent with the archive-first "warehouse disposable, raw durable" principle, so not a real cost here.
- Reading parquet back is first-class via `external_location` on sources, with path templating.
- Whether an external model also registers a catalog view for tests and `ref()` was **not confirmed** — smoke-test one external model with a `not_null` test before building the pipeline on this pattern.
- Known issue: wildcard `location` patterns can pin a stale schema when columns are added.

**Recommendation:** adopt `external` materialization as the publish layer (this is the brainstorm's "candidate simplification (SHOULD)"), but gate it behind the smoke test above.

### A.6 Infrastructure source links

[LFS billing](https://docs.github.com/en/billing/concepts/product-billing/git-lfs) ·
[LFS storage/bandwidth](https://docs.github.com/enterprise-cloud@latest/repositories/working-with-files/managing-large-files/about-storage-and-bandwidth-usage) ·
[Actions events/schedule](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) ·
[disabling/enabling workflows](https://docs.github.com/actions/managing-workflow-runs/disabling-and-enabling-a-workflow) ·
[keepalive-workflow](https://github.com/marketplace/actions/keepalive-workflow) ·
[duckdb-wasm](https://github.com/duckdb/duckdb-wasm) ·
[wasm instantiation](https://duckdb.org/docs/current/clients/wasm/instantiation.html) ·
[COOP/COEP on static hosts](https://blog.tomayac.com/2025/03/08/setting-coop-coep-headers-on-static-hosting-like-github-pages/) ·
[GH Pages headers request](https://github.com/orgs/community/discussions/13309) ·
[coi-serviceworker](https://github.com/gzuidhof/coi-serviceworker) ·
[Cloudflare Pages headers](https://developers.cloudflare.com/pages/configuration/headers/) ·
[dbt-duckdb](https://github.com/duckdb/dbt-duckdb) ·
[dbt-duckdb external data](https://deepwiki.com/duckdb/dbt-duckdb/6-working-with-external-data) ·
[DuckDB dbt post](https://duckdb.org/2025/04/04/dbt-duckdb) ·
[httpfs](https://duckdb.org/docs/current/core_extensions/httpfs/https) ·
[async HTTP reads issue](https://github.com/duckdb/duckdb-wasm/issues/1723) ·
[GH Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits) ·
[GH Pages CORS](https://github.com/orgs/community/discussions/157852)

---

## B. Competitive landscape and prior art (2026-08-07)

Web research commissioned during Discovery. Its purpose is not to reopen the build-it-yourself decision — that decision is anchored in goal 3, which the brainstorm explicitly protected from efficiency-minded revision — but to establish honestly what pulse reinvents, what it genuinely adds, and which conventions are worth adopting rather than inventing.

### B.1 The code-first / static-output landscape

| Tool | Output | Data layer | Authoring | Customization stance |
|---|---|---|---|---|
| **Evidence.dev** | Static site (SvelteKit prerender) + "Universal SQL": build emits a parquet cache, browser runs duckdb-wasm for filters/params | Any warehouse, or a local `.duckdb` / parquet / CSV directory | Markdown + SQL fences + `<LineChart/>` components | Opinionated component library over ECharts. Escape hatches: `echartsOptions`, raw `<ECharts>`, Svelte custom components. Expressive *within* a chart grammar. |
| **Observable Framework** | Static site generator, SSG + optional client JS | Data loaders in any language (`data/foo.csv.py`) run at build → static snapshots; duckdb-wasm client-side; page loaders can bake SVG at build time | Markdown + JS cells; Plot/D3/Mosaic/Vega available but not required | Maximal — it is a web app; raw SVG/HTML/CSS fully supported. |
| **Quarto** (+ dashboards) | Static HTML/PDF/docs; dashboards are a layout format | Whatever the R/Python/Julia kernel does | Literate `.qmd`; ggplot/matplotlib/plotly/OJS | Publishing-first; charts come from the language's libraries. |
| **Rill Developer** | Server app with embedded DuckDB; local dev then `rill deploy` | DuckDB/ClickHouse/MotherDuck, YAML metrics layer | YAML + SQL; dashboards auto-generated from metrics | Deliberately low — the exploration UX is the product. |
| **Datasette** | Server (1.0 still alpha as of 2026); Datasette Lite runs in-browser via Pyodide | SQLite | Canned queries in YAML, plugin ecosystem | Exploration and publishing, not visual craft. |
| **Streamlit / marimo** | Streamlit is server-based; marimo can `export html-wasm` (Pyodide, deployable to Pages) | Python in-process | Python widgets | Python-idiomatic, hard to make bespoke. |
| **Metabase / Superset** | Server + DB, GUI-built | Semantic-ish layers | Point-and-click | The contrast class. |

Also notable: **Mosaic/vgplot** (UW IDL) for duckdb-wasm-backed linked views at scale, and **Observable Notebook Kit** (2025) as an open notebook file format with static site tooling.

### B.2 What is already served, what pulse reinvents, what is genuinely uncovered

**Already fully served.** "Static site + DuckDB + code-authored charts + scheduled build" is *not* a gap in the market. Evidence.dev does precisely this: local `.duckdb` source, parquet cache, duckdb-wasm in the browser, deployed to any static host, driven by a GitHub Actions cron. Observable Framework does it as well, with a stronger ingest story. End-to-end write-ups of both patterns exist publicly.

**The cost ledger.** This section originally listed a flat set of things "pulse would reinvent". That framing was superseded on 2026-08-07 by the layer split (§C.4 below): visual authoring is the only closed layer, and everything below it may be adopted. The costs are therefore conditional on the candidate, not fixed. Revised:

| Piece | Evidence.dev | Observable Framework | Vite + own conventions |
|---|---|---|---|
| Dev server, hot reload | supplied | supplied | supplied |
| Build orchestration | supplied | supplied | supplied |
| Page routing, navigation | supplied | supplied | partial — Vite bundles, routing is yours |
| Data loading conventions | supplied (SQL fences) | supplied (file-based loaders) | build yourself |
| Parquet chunking + manifest for lazy duckdb-wasm loading | supplied (Universal SQL cache) | partial | build yourself |
| Freshness metadata plumbing | build yourself | build yourself | build yourself |
| Theming tokens across visuals | build yourself | build yourself | build yourself |
| The data-interface contract | build yourself | build yourself | build yourself |

Two readings follow. First, the build cost of the Vite path is materially higher than the framework paths, and Evidence's Universal SQL cache layer — a substantial piece of engineering — is the largest single item it would forgo. Second, and more useful: **the bottom three rows are pulse's own regardless of choice.** Freshness plumbing and theming are chores; the data-interface contract is not a chore but the actual contribution, and no candidate supplies it because no candidate has pulse's problem.

**Genuinely uncovered:**

- **No chart-library vocabulary at all.** Observable Framework *permits* hand-written SVG; nothing in the ecosystem *rewards* it. Every tool's unit of work is "a chart of type X"; pulse's unit is "a bespoke visual answering one question." The closest prior art is newsroom graphics and OWID's Grapher, neither of which is a reusable product.
- **AI-agent-as-author as a first-class design constraint.** The 2026 discourse acknowledges that agents now write SQL and visualization code, but the tooling response is agents *driving existing BI*, not repositories *shaped for* agent authorship. Uncovered, though also unproven as a differentiator.
- **Archive-first ingest semantics** (immutable dated snapshots, as-of querying, provenance) bundled with the publishing layer. No tool ships these together.

**Honest risk, stated plainly.** The two uncovered points above are craft and portfolio arguments, not capability arguments. Observable Framework plus a self-imposed "no plotting library" discipline would deliver roughly 85% of pulse at roughly 15% of the cost. The PRD should state why the remaining 15% is worth it rather than leave the question implicit — otherwise the first efficiency-minded reviewer, human or agent, reopens it.

### B.3 Prior art: personal data warehouses

- **Dogsheep / Simon Willison** — the canonical statement of the personal data warehouse. `*-to-sqlite` importers plus Datasette. Pattern: one tiny importer per source, one file format, one query UI.
- **Git scraping** (also Willison) — an Actions cron commits scraped data to git; every commit is a timestamped snapshot. This is pulse's archive layer, already proven and free, and it independently corroborates the plain-git recommendation in §A.1.
- **karlicoss/HPI** plus promnesia — personal data as a Python API rather than a warehouse; strong on source-adapter modularity.
- **QS Ledger** — quantified-self aggregator plus notebooks; dated, mostly Jupyter.
- **davidgasquez/datadex** — the closest philosophical sibling: a serverless, local-first open-data platform whose stated principles are Open / Modular / Simple / Data as Code / Glue. Notably it *moved off* Dagster + DuckDB; the reasons are worth reading before committing.
- **OWID's ETL** — a content-addressable DAG with a snapshot layer and per-indicator metadata; the professional version of archive-first over World Bank / WHO data.

The recurring pattern across all of them: git-as-storage, one importer per source, an immutable raw layer, dbt staging to marts, an Actions cron — **and the project usually dies at the presentation layer.** This corroborates the brainstorm's archive/glance loop as the correct organising principle, and is the strongest external evidence that visualization must be co-equal with ingestion rather than downstream of it.

### B.4 Conventions worth adopting rather than inventing

- **File-based data-loader routing** (Observable Framework): `data/wdi.parquet.py` *is* the build rule for `wdi.parquet`. Zero config, self-documenting, cache-invalidated by content hash. Directly compatible with pulse's "sources as folders" decision.
- **A single site manifest** for navigation, with shared code in a components directory rather than in pages.
- **The build-time / client-side split** (Evidence): build-time queries serve the narrative; client-side SQL is reserved for interaction. This is exactly pulse's "wasm optional per report" decision, arrived at independently.
- **Annotations as data** (Evidence's `ReferenceLine` / `ReferenceArea` / `ReferencePoint`): declarative, data-driven overlays — a table of events joined to the chart, never hardcoded coordinates. The single best convention in the space, and directly relevant to pulse's "annotations are first-class content" principle.
- **Freshness UX via dbt exposures**: `dbt source freshness` plus exposures (which report depends on which model) lets each visual render "as of `<date>`, source `<X>`, snapshot `<hash>`". Archive-first makes this cheap, and it is a visible differentiator.
- **OWID-style per-indicator metadata**: citation, unit, definition and processing notes travelling with the data.
- **A thin metrics definition file** (Rill's YAML metrics layer): even for a single user, it prevents metric drift across pages.

### B.5 Uncertainty flags

- Observable's relative investment in Framework versus Notebooks 2.0 / Notebook Kit is ambiguous. The Framework repository was active as of roughly May 2026, but 2.0 received the 2025 announcement effort. Verify before adopting any Framework convention deeply.
- Datasette 1.0 remains in alpha.
- Evidence's Universal SQL implementation details may have shifted since its launch post.

### B.6 Landscape source links

[Evidence.dev](https://evidence.dev) ·
[Why we built Universal SQL](https://evidence.dev/blog/why-we-built-usql) ·
[Evidence custom components](https://docs.evidence.dev/components/custom-components) ·
[Observable Framework](https://github.com/observablehq/framework) ·
[Framework page loaders](https://observablehq.com/framework/page-loaders) ·
[Observable Notebook Kit](https://observablehq.com/notebook-kit/) ·
[Quarto](https://quarto.org) ·
[Rill](https://rilldata.com) ·
[Datasette](https://datasette.io) ·
[Datasette Lite](https://lite.datasette.io) ·
[marimo WebAssembly export](https://docs.marimo.io/guides/exporting/webassembly_html/) ·
[Mosaic/vgplot](https://idl.uw.edu/mosaic/) ·
[Personal Data Warehouses](https://simonwillison.net/2020/Nov/14/personal-data-warehouses/) ·
[Dogsheep](https://dogsheep.github.io/) ·
[git scraping](https://simonwillison.net/tags/git-scraping/) ·
[karlicoss/HPI](https://github.com/karlicoss/HPI) ·
[promnesia](https://github.com/karlicoss/promnesia) ·
[QS Ledger](https://github.com/markwk/qs_ledger) ·
[datadex](https://github.com/davidgasquez/datadex) ·
[OWID ETL docs](https://docs.owid.io/) ·
[OWID Grapher](https://github.com/owid/owid-grapher) ·
[Vibe Coding Comes for BI](https://motherduck.com/blog/vibe-coding-comes-for-bi/) ·
[Evidence + DuckDB + Actions](https://kiliantscherny.substack.com/p/building-a-simple-data-app-with-evidence) ·
[dbt + DuckDB + Actions](https://chrsolv.substack.com/p/duckdb-dbt-analytics)

---

## C. Visual authoring pipeline — concrete implementation (2026-08-07)

The PRD states the requirement tool-agnostically: purpose-built visuals are **agent-authored**. That phrasing is deliberate — it must survive tool churn over a project measured in years. This section records the concrete pipeline as it stands today.

### C.1 The two steps

1. **Claude Design authors the visual** against a mock `data` array with a declared column schema. Output is HTML/SVG/CSS/JS. No chart or visualization library is involved at any point.
2. **A coding agent wires it to real data** — swapping the mock array for the real query result against a dataset, and registering the visual on a report.

The two steps are separable on purpose. Step 1 is a design act, judged on whether the visual answers its question; step 2 is a mechanical act, judged on whether it still renders with real rows. The data-interface contract exists precisely to make step 2 mechanical.

### C.2 Why authorship is stated separately from the artifact

"Purpose-built" describes the artifact: tailored to one question, no chart-type vocabulary, no library. "Agent-authored" describes its provenance. These are independent axes — a visual would still be purpose-built if a human drew it — and collapsing them into a single adjective (as "hand-built" did in the original pitch and brainstorm) creates two problems:

- It misdescribes the work. No human draws these.
- It hides a **design constraint** inside an adjective. Agent authorship is not an implementation detail of pulse; it is a constraint the repository is shaped around. A downstream reader — including the coding agent reading this PRD — that infers a human in the loop will make wrong decisions about how much structure and convention the repository needs.

Terminology consequence: "hand-built", "hand-written" and "hand-authored" are not used anywhere in pulse's documentation. Where the original pitch and brainstorm used them, read "purpose-built".

### C.3 Open question deferred to architecture

If Evidence.dev is selected, step 1's HTML/SVG output must be wrapped as a Svelte component before step 2 can wire it. Whether that wrapping is mechanical enough to be encoded as a skill, or whether it introduces enough friction to distinguish the candidates, is unresolved. It is the sharpest practical difference between the Evidence and Observable Framework paths and should be settled by building one visual each way rather than by argument.

---

## C.4 The layer split (2026-08-07)

Established in conversation with Yann and superseding the all-or-nothing framing of §B.2. pulse divides into layers; only one is closed.

| | Layer | Status |
|---|---|---|
| **A1** | Fetch mechanism | Open — framework-provided is fine |
| **A2** | Archive semantics: immutable dated snapshots, replayable from raw | Requirement; mechanism open |
| **B** | Transformation / warehouse | Settled — dbt-duckdb |
| **C** | Site build: dev server, HMR, routing, bundling, static output | Open |
| **D** | Data delivery to page | Open. duckdb-wasm on all pages (see below) |
| **E** | Visual authoring | **Closed** — purpose-built, no chart or visualization library at any level |
| **F** | Freshness / metadata plumbing | pulse's own regardless of choice |

Notes on specific layers:

- **A1 vs A2 are separable.** Build-time data loading (what Observable Framework's loaders do) and monthly archival snapshotting (what the cron does) are different jobs that happen to touch the same data. They may share a mechanism or not. "Sources as folders" is a structuring convention and carries no implication that ingestion must be bespoke.
- **D — duckdb-wasm on all pages.** This simplifies the brainstorm's "baked JSON default, wasm opt-in per report". The brainstorm chose baked-default for simplicity on low-data pages; uniform wasm turns out to be the simpler option because it removes the dual data-delivery path, the per-report flag, and two code paths that must both keep working. It also removes the last Evidence-specific mismatch, since Evidence's Universal SQL model is wasm-always.
- **E is Goal 3 stated as a constraint.** A tool that supplies the expressive visual layer removes the part of the project that makes it worth doing.

**Candidates carried forward to architecture:** Evidence.dev, Observable Framework, Vite + own conventions. Carrying named candidates forward is what stops the architecture phase from re-deriving this analysis from scratch.

**Serving is not a differentiator.** All three emit static prerendered output and deploy to GitHub Pages. Pages viability is determined by the wasm bundle — single-threaded EH needs no COOP/COEP (§A.2) — not by the framework choice. C and D can therefore be decided on authoring ergonomics alone. One gotcha applies to all three equally: GitHub project pages serve from `/reponame/`, so base-path configuration is required or asset URLs break silently on first deploy.

### C.5 Superseded terminology and framings

Recorded so that downstream readers encountering the older documents are not misled:

- "hand-built" / "hand-written" / "hand-authored" (pitch, brainstorm) → read **purpose-built**; authorship is stated separately as agent-authored.
- "baked JSON at build time by default, wasm opt-in per report" (brainstorm) → superseded by duckdb-wasm on all pages.
- "Public raw is committed in-repo via Git LFS" (brainstorm) → **retained** after challenge; see §D.
- §B.2's flat "what pulse would reinvent" list → superseded by the conditional cost ledger in §B.2 and the layer split above.

---

## D. Raw snapshot storage — Git LFS retained (2026-08-07)

**Decision: keep Git LFS for public raw snapshots, as the brainstorm specified.** §A.1's recommendation to drop it was raised, challenged by Yann, and not adopted. The decision is his; this section records the reasoning on both sides so it is not silently reopened.

### D.1 The argument for dropping LFS (§A.1)

LFS download bandwidth is billed to the repository *owner* regardless of who clones, against a 10 GiB/month per-account quota shared across all of Yann's repos. On a public portfolio repo, strangers cloning drain that quota. At $0 budget, GitHub hard-blocks pushes and LFS pulls for the remainder of the calendar month. Snapshots are KBs–MBs, which plain git handles with delta compression and readable diffs.

### D.2 Why it was not adopted

Yann's counter-arguments, which hold:

1. **The quota is very unlikely to bind.** 10 GiB/month of clone traffic on a personal project is a large amount of traffic.
2. **Both failure branches are acceptable.** If the quota binds because the data grew, plain git is no better — a large repo is a large repo. If it binds because the project became popular, that is a problem worth having and is solvable by paying for it.
3. **Plain git is not unlimited either.** GitHub publishes repository size guidance (under 1 GB recommended, warnings above 5 GB) and documents that repositories may be throttled; GitHub's own advice for large files is to move them to LFS. §A.1 compared metered LFS against an implicitly unmetered plain git, which is not the real comparison. **This is a correction to §A.1, not merely a difference of risk appetite.**

At the stated volume both mechanisms work. LFS additionally keeps clones and git history light, and is the documented path for data files.

### D.3 Retained as a watch item

One nuance to D.1 survives and is worth monitoring, though it was not decisive: the bandwidth-draining traffic is most likely to come from scrapers, bots and CI forks rather than from readers, so consumption is **not** well correlated with the project's success. The "if it's that popular, monetize it" escape valve therefore may not fire when the quota does.

Cheap mitigations if it ever binds, in order: set `lfs: false` on any Actions checkout that does not need snapshot contents (this is already the default and costs nothing); set a $0 budget cap so overage hard-blocks rather than bills; move any single artifact above ~50 MB to a GitHub Release asset (2 GB per asset, unmetered egress).

If Evidence.dev is selected, step 1's HTML/SVG output must be wrapped as a Svelte component before step 2 can wire it. Whether that wrapping is mechanical enough to be encoded as a skill, or whether it introduces enough friction to distinguish the candidates, is unresolved. It is the sharpest practical difference between the Evidence and Observable Framework paths and should be settled by building one visual each way rather than by argument.
