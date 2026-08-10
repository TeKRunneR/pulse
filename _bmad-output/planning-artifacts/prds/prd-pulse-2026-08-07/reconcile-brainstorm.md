# Input reconciliation — brainstorm-intent.md → prd.md + addendum.md

**Source:** `_bmad-output/brainstorming/brainstorm-pulse-data-hub-2026-08-06/brainstorm-intent.md` (2026-08-06)
**Targets:** `prd.md`, `addendum.md` (prd-pulse-2026-08-07)
**Date:** 2026-08-08

Method: every item of the source's six parts walked individually and classified as
**carried** (present in substance), **transformed** (present in changed form, intentionally),
**superseded** (consciously replaced — listed in the known-deliberate set), or **GAP** (silently dropped).

Known deliberate changes were excluded from the gap list up front: hand-built → purpose-built +
agent-authored; baked-JSON-default → duckdb-wasm everywhere (and the flagship-report Should as moot);
index page → homepage Must; cross-filtering removed from non-goals; the copy-existing-code /
don't-repeat-code contradiction resolved toward the latter; and the PRD's deliberate exclusion of
what any individual visual shows.

---

## Summary of findings

| # | Gap | Severity |
|---|---|---|
| G1 | In-browser interactive analysis has no requirement anywhere — duckdb-wasm survived as plumbing, the capability it existed for did not | High |
| G2 | Abstraction discipline rule dropped: shallow, domain-named, extracted after the 2nd/3rd visual, never designed up front | High |
| G3 | "Theme tokens" — the one brainstorm Should with no FR and no scope entry | Medium-High |
| G4 | No "change a dataset schema" workflow, although the brainstorm made a defined workflow the *condition* for accepting schema churn | Medium |
| G5 | Private-archive mortality ("dies with the laptop") not carried as a risk, despite contradicting archive-first | Medium |
| G6 | Open item 4 — whether personal tool and public repo eventually separate — dropped as an open question | Medium |
| G7 | "Ingestion stays outside dbt; raw registered as external sources" — boundary decision dropped | Medium |
| G8 | "Reports, **not tiles**" — the negative constraint half of the principle | Low-Medium |
| G9 | Per-visual/per-report query: the brainstorm's stated lean toward a dashboard-level slice dropped | Low |
| G10 | Topic scope narrowed: demographics and personal finances dropped from the goal statement | Low |
| G11 | "One-shot questions go to ChatGPT/Claude" — the positioning boundary | Low |
| G12 | Parquet materialization promoted to Must without carrying addendum A.5's smoke-test gate into scope | Low |
| G13 | Goal 1's "with the full history and its evolution visible" | Low |
| G14 | "Encodable as a skill" dropped from the agent-legibility ease criteria | Low |
| G15 | Archive/glance loop "orders the roadmap" — the sequencing consequence | Low |

---

## Part 1 — What pulse is

| Source item | Status | Note |
|---|---|---|
| Personal data hub + dashboarding tool | Carried | §1 |
| Ingests public automatically, private manually (bank CSV) | Carried | FR-3, FR-4 |
| Archives every fetch immutably | Carried | FR-5 |
| Builds curated datasets with DuckDB | Carried | FR-7, addendum §C.4 layer B |
| Publishes hand-built HTML/SVG/JS reports | Transformed | purpose-built; medium (HTML/SVG/CSS/JS) recorded in addendum §C.1 |
| Standing questions about **economy, demographics, climate, personal finances** | **GAP (G10)** | PRD §1.1 goal 1 names economics and "climate and emissions" only. Demographics disappears entirely. Personal finance survives only as the private-source Could — it is never named as a question area, so nothing in the PRD says pulse is *for* personal finances. |
| "One-shot questions go to ChatGPT/Claude; pulse is for the questions that never close" | **GAP (G11)** | PRD §1 keeps "not the ones closed in an afternoon", which preserves the timescale but loses the explicit *substitution boundary*. That boundary is the cheapest available answer to "why not just ask an LLM?" — a question a downstream reviewer will ask. |
| Public repo doubles as portfolio | Carried | §1, goal 2, NFR-2 |

---

## Part 2 — Goals

**Goal 1 — Personal utility → "Ambient familiarity."** Transformed, and the transformation is a sharpening
(it produced SM-1). One rider did not survive:

- **"with the full history and its evolution visible"** — **GAP (G13, Low).** Archive-first covers *retaining*
  history; nothing covers *displaying* it. UJ-1 reduces it to "recent trajectory". This sits close to the
  deliberately-excluded "what a visual shows" territory, but it is a product-level statement about what a
  report is for, not a spec of one visual — the archive is a portfolio and utility asset only if depth is
  actually rendered. Flagged for a judgement call rather than as a clear omission.

**Goal 2 — Portfolio.** Carried; "dbt DAG, documented datasets, agent-readiness are themselves the showcase"
carried near-verbatim in §1 and hardened into NFR-2.

**Goal 3 — Playground.** Carried, including the provenance ("unstated in the original pitch, surfaced only
under challenge") and the protection clause ("not to be optimised away"). PRD §1.1 softens the protection to
"unless absolutely necessary and after discussion" — a deliberate-looking edit, not reported as a gap.
But see **G1**: goal 3's *content* is "purpose-built visuals **and duckdb-wasm**", and only the first half
acquired a requirement.

---

## Part 3 — Core principles (10)

| Principle | Status |
|---|---|
| Archive-first | Carried — §1.2, verbatim |
| The archive/glance loop | Carried — §1 "Its engine is a loop", strengthened with external evidence (addendum §B.3). See G15 |
| Warehouse disposable, raw durable | Carried — §1.2, extended with the contracts/reports migration caveat |
| At-a-glance = zero assembly, not low density | Carried — §1.2, verbatim |
| Reports, not tiles | Partially carried — see G8 |
| Expressiveness-first visuals | Carried — §1.2, FR-12. See G2's neighbour note below |
| The data interface is the stable contract | Carried — §1.2, FR-10; "most important architectural statement" explicitly preserved in §4.4 |
| Predictable structure, shared implementation | Partially carried — see **G2** |
| Agent legibility is the ease metric | Carried — NFR-1. See G14 |
| Structure follows content | Carried — §1.2 |

### G2 (High) — the abstraction discipline rule

Brainstorm: *"Abstractions stay shallow and domain-named, and are extracted after the second or third visual,
never designed up front (over-abstraction is as agent-hostile as duplication)."*

PRD §1.2 keeps only "Shared render components, macros and functions carry the implementation, but must not get
in the way of purpose-built visuals." FR-23 states shape repeats and code does not. FR-13 carries an analogue
— "starts close to empty and grows only as conventions emerge from use" — but scoped to the *convention store*,
not to code abstraction. "Structure follows content" covers "no premature abstraction" in spirit.

What is lost is everything actionable:

- the **timing rule** (extract at the second or third instance, not before) — the only concrete test the
  brainstorm gave for when shared implementation is allowed to appear;
- the **quality bar** (shallow, domain-named);
- the **symmetry argument** (over-abstraction is *as* agent-hostile as duplication), which is what stops
  FR-23's "code must not repeat" from being read as a mandate to abstract early and aggressively.

This matters more than a normal missing rationale because FR-23 as written pushes in the opposite direction:
an agent told "what must not repeat is the code" and given no counterweight will generalise on the first
duplicate. The brainstorm anticipated exactly this failure and the PRD dropped the guard.

**Suggested home:** a consequence on FR-23, or a fourth bullet under the "Predictable structure, shared
implementation" principle.

### G8 (Low-Medium) — "not tiles"

The brainstorm's principle is a contrast: a page is a standing analysis, **not** a tile/KPI-widget dashboard.
PRD §1.2 renames it plainly to "Reports." and the glossary defines Report positively. The rejected alternative
— the metric-tile dashboard that every BI tool defaults to, and that an agent composing a page will otherwise
reach for — is nowhere in the document, including §5 Non-Goals which would be its natural home.
"Multi-angle", "dense" and "curated" imply it but do not state it.

Also dropped from this principle: **"refreshed monthly"** — correctly superseded by FR-2's per-source cadence,
not a gap.

### G15 (Low) — "orders the roadmap"

The brainstorm attached a sequencing consequence to the loop: it *orders the roadmap*. The PRD's MVP scope
does honour it (one real report is a Must alongside ingestion), but the ordering rule itself is not stated,
so a future scope negotiation has nothing to cite when someone proposes "ingest three sources first, visuals later".

### G14 (Low) — "encodable as a skill"

NFR-1 lists four ease criteria (workflow to follow, shape to match, shared implementation to call, description
that fits in markdown). The brainstorm had five; **"encodable as a skill"** is missing. Related: the brainstorm's
concrete artifacts — `CLAUDE.md` plus skills for add-a-source / add-an-indicator / add-a-visual, a v1 **Must** —
are generalised in §4.8 and §7.1 to "defined authoring workflows … the repository carries its own instructions
as first-class artifacts". The generalisation looks deliberate (tool-agnosticism is a stated PRD posture), but
it drops a testable bar: a workflow that cannot be encoded as a skill has failed NFR-1.

---

## Part 4 — Architecture decisions (14)

| Decision | Status |
|---|---|
| Sources as folders (`source.toml` + optional fetch script, discovery by directory) | Carried, generalised — FR-1 ("declared by configuration", "no central registry to update"); the concrete convention is preserved in addendum §C.4 |
| Immutable raw snapshots before any transform; original format kept, parquet only when not already a file; replayable | Carried in full — FR-5, FR-6 |
| Public raw in-repo via Git LFS; reproducibility as portfolio evidence; monthly commits keep the workflow alive | Carried — §7.1, NFR-2, §9, addendum §A.1/§A.4 (challenged and retained) |
| Private raw gitignored, local-only, one laptop, deferred | Carried — §5, NFR-3, OQ-5. See **G5** |
| Datasets wide + typed + per question area; no tall/long table anywhere; schema changes acceptable given a defined workflow | Partially carried — FR-7, NFR-5. See **G4** |
| dbt-duckdb: staging → datasets, tests, `schema.yml` as semantic layer, `manifest.json` for agents | Carried — FR-7/8/9, §7.1, addendum §C.4 |
| …**ingestion stays outside dbt; raw registered as dbt external sources** | **GAP (G7)** |
| …dbt source freshness covers the staleness half of silent breakage | Carried — FR-19 (mechanism correctly left to architecture) |
| No revision/vintage modelling; archaeology as opt-in per-source extra, not a global tax | Carried — §5, FR-7 assumption |
| Static site + local server; `pulse serve` mandatory because wasm cannot run from `file://`; no always-running service; dual local/remote model | Carried — §4.6, FR-18, §5, OQ-4 |
| Two ingestion runtimes unavoidable | Carried — §4.1, and upgraded to "a permanent property of the system, not a v1 compromise" |
| duckdb-wasm kept because in-browser real-time analysis is genuinely wanted (goal 3); optional per report; one flagship with bold slicing | Superseded *in mechanism only* — see **G1** |
| Dashboard privacy flag + data-root indirection + build profiles; same code both ways | Carried — FR-17, NFR-3, glossary |
| Visual contract: visual = query + render + manifest; report = manifest of visuals + layout; mock `data` array with declared column schema; wiring = substitution | Carried — FR-10, FR-11, FR-15 |
| Repo carries its own instructions; first vertical slice is the exemplar | Carried — §4.8, FR-24. See G14 |
| Candidate simplification (SHOULD): datasets materialised straight to parquet, collapsing publish/export | Transformed — promoted to Must, §7.1, with rationale. See **G12** |

### G1 (High) — in-browser interactive analysis lost its requirement

The brainstorm's wasm decision has two independent halves:

1. **Delivery mechanism** — baked JSON by default, wasm opt-in per report.
2. **A user-facing capability** — *"in-browser real-time analysis is genuinely wanted (goal 3)"*, concretised as
   one report being a deliberate flagship with **"bold slicing"** / **"open-ended slicing"**.

Only half 1 was superseded. Addendum §C.4 argues uniform wasm is simpler because it removes the dual data path,
the per-report flag and two code paths — an argument entirely about *plumbing*. It says nothing about slicing,
and correctly so; but the capability then fell through the gap between the two documents.

Net effect in the PRD as it stands:

- FR-18's only mention is a consequence line: "Data is queried in the browser via duckdb-wasm on all pages."
- No FR, no scope entry, no success metric and no user journey requires that a reader can *do* anything
  interactive with the data. UJ-1 is read-and-close.
- §1.1 goal 3 names duckdb-wasm as wanted for its own sake, but §1.3's non-negotiable is scoped to visual
  authoring only, so goal 3's second half has no constraint expressing it.

Consequence: an architect reading only the PRD can satisfy every requirement with fully static prerendered
output, and would be right to ask why duckdb-wasm is mandated at all — since the stated justification for it
(FR-18's local-server requirement) is circular: `pulse serve` is mandatory *because of* wasm. Half of goal 3
is currently unfalsifiable and unbuilt. Note also that the flagship-as-Should was the only thing keeping
interactivity *out* of the Must line; removing it as "moot" removed the scope control too, in both directions.

**Suggested resolution:** either an FR under §4.5 or §4.6 for at least one report supporting open-ended
in-browser slicing (Should, mirroring the brainstorm's flagship), or an explicit note that interactivity is
deferred and duckdb-wasm is v1 plumbing only. Either is fine; silence is not.

### G4 (Medium) — no schema-change workflow

Brainstorm: *"Schema changes are acceptable — there is no service-continuity constraint, just a need for a
defined workflow."* The permission and the obligation are one sentence; the PRD took the permission
(NFR-5, FR-6, §1.2) and dropped the obligation.

FR-22 enumerates workflows for "add a source", "add an indicator", "add a visual" and "add a report" — all
additive. Nothing covers change or removal. This is sharper than a generic omission because the PRD itself
raises the consequence: §1.2 notes that a schema change "might require migration of the contracts and reports".
It identifies the blast radius and then specifies no procedure for it, in a system whose whole risk posture
rests on schema churn being cheap.

### G5 (Medium) — private-archive mortality is not a risk

Brainstorm watch item: *"a local-only archive on one laptop dies with the laptop; accepted for v1, needs its
own backup story later."*

The PRD carries the **decision** (§5 non-goal, "Private raw is local-only on one machine in v1") and the
**follow-up** (OQ-5, "Deferred with the private ingestion path"), but not the **risk**. §9 lists five risks
and this is not among them. The framing loss is the point: it is the only item in the document that directly
contradicts the first core principle — "never losing history is a core requirement" applies to the raw
archive, and one half of the raw archive has no durability at all. Recorded as a non-goal, it reads as a
scoping choice; recorded as a risk, it reads as an accepted exposure with a known trigger (laptop loss) and
an owed mitigation. The brainstorm chose the second.

### G6 (Medium) — open item 4 dropped

Brainstorm open item 4: *"Whether the personal tool and the public repo should eventually separate."*

PRD §10 carries open items 1, 2 (resolved), 3, plus two of its own — but not 4. Its subject matter survives
only as the §9 risk "Portfolio versus personal tool". That is the *tension*; item 4 was the *decision the
tension eventually forces*. The brainstorm explicitly linked them ("see open item 4"), so the PRD has kept the
symptom and dropped the remedy under consideration. A risk with no candidate resolution recorded anywhere
will be re-derived from scratch the first time portfolio pressure actually bites.

### G7 (Medium) — ingestion outside the transformation layer

Brainstorm: *"Ingestion (fetch + snapshot) stays **outside** dbt, with raw files registered as dbt external
sources."*

Neither document states this. Addendum §C.4 lists A1 (fetch) and B (transformation) as separate layers, which
implies but does not assert the boundary, and §C.4's note that "build-time data loading and monthly archival
snapshotting are different jobs that … may share a mechanism or not" arguably loosens it further. §A.5 covers
dbt's `external` materialization on the *output* side; nothing covers registering raw as external sources on the
*input* side.

This is a real architectural constraint with product consequences the PRD does depend on — FR-5's
"snapshot before any transformation", FR-6's replay-from-raw, and the ability to swap the transformation layer
without touching ingestion all rest on it. It may be a considered deferral to architecture; if so it should be
said, because as written the architecture phase is free to fold fetching into dbt and break the FR-5 ordering
guarantee without noticing it has contradicted anything.

### G12 (Low) — the parquet promotion lost its gate

§7.1 lists "Datasets materialised to parquet" as in-scope with a promotion rationale. Addendum §A.5 recommends
adopting `external` materialization **"but gate it behind the smoke test above"** — whether an external model
also registers a catalog view for tests and `ref()` was *not confirmed*, and FR-9 requires datasets to carry
data tests. The Must-list entry does not carry the conditional, so the dependency between an unverified adapter
behaviour and a promoted-to-Must scope item is visible only to someone who reads the addendum. Also unstated in
the PRD: the brainstorm's framing that this makes "the warehouse a directory of parquet files", which is what
makes FR-6's "deleting the entire warehouse" concrete.

---

## Part 5 — v1 MoSCoW

**Must** — all nine items accounted for.

| Brainstorm Must | PRD |
|---|---|
| One public source (World Bank / public macro) with `source.toml` | §7.1 ✓ (the named candidate source is dropped; §7.1 says "one public source" and the report is French macro. Low impact — the choice is deferrable — but the concrete candidate was useful for the vertical slice) |
| Immutable raw snapshots via Git LFS | §7.1 ✓ |
| dbt-duckdb staging → one wide dataset, `schema.yml` docs + tests | §7.1 ✓ (tool-agnostic phrasing; dbt-duckdb settled in addendum §C.4) |
| One real France macro report, several purpose-built visuals | §7.1 ✓ |
| The data-interface contract | §7.1 ✓ |
| Build + `pulse serve` + public deploy | §7.1 ✓ |
| Monthly GitHub Action running the whole chain | §7.1 ✓ ("monthly" correctly superseded by FR-2 per-source cadence; the monthly *commit* survives in §9 as the scheduler-decay defusal) |
| "Data as of" freshness on the page | §7.1, FR-20 ✓ |
| `CLAUDE.md` + add-a-* skills | §7.1 ✓ generalised — see G14 |

**Should** — five of six accounted for; one unaccounted.

| Brainstorm Should | Disposition |
|---|---|
| Per-source assertions beyond dbt defaults | §7.2 ✓ FR-9 |
| Index page | Promoted to Must as "homepage" — deliberate ✓ |
| Chart annotations | §7.2 "Annotations as data" ✓ FR-16 |
| **Theme tokens** | **GAP (G3)** |
| Datasets materialised straight to parquet | Promoted to Must ✓ (see G12) |
| One duckdb-wasm flagship report | Declared moot — deliberate for the mechanism; see **G1** for the capability |

PRD §7.2 also adds "A store for cross-visual conventions" (FR-13), which has no brainstorm ancestor. That is an
addition, not a gap — but it should not be mistaken for theme tokens (below).

### G3 (Medium-High) — theme tokens

This is the only brainstorm MoSCoW item with no corresponding scope entry and no FR. The word "theme" does not
appear in `prd.md` at all.

The near-miss is FR-13 / §7.2's "store for cross-visual conventions" (glossary: *Visual language* — "the store
of conventions that apply across visuals and that the authoring agent consumes"). Theme tokens could be read as
a subset, but they are a different kind of thing: tokens are a concrete shared artifact (colour ramps, type
scale, spacing, axis and grid treatment) that visuals *reference at render time*, whereas the convention store
is guidance the *authoring agent reads*. FR-13's consequences make the distinction visible — "adding or changing
a convention does not require touching every existing visual" and "a convention can be revised or withdrawn"
describe editorial guidance, not a runtime dependency.

The addendum knows this: §B.2's cost ledger lists "Theming tokens across visuals — build yourself" across all
three candidates, and calls it one of the three rows that are "pulse's own regardless of choice". So the
research recognised it as unavoidable work, and the PRD's scope section never picked it up.

Why it matters beyond bookkeeping: with no chart library and every visual purpose-built, tokens are the *only*
mechanism by which a set of independently authored visuals will look like one product. Its absence interacts
badly with G2 — no shared tokens plus no abstraction-timing rule is the configuration in which visual drift is
most likely.

**Could** — both carried (§7.3): manual/private ingestion path; a second and third report.

**Won't** — all carried into §5: data lab ✓, lab-to-dashboard promotion ✓, revision modelling ✓, durable private
storage ✓, multi-source datasets ✓. Cross-filtering removed by the user (deliberate); note the PRD remains
internally consistent, since OQ-3 still references cross-filtering as the thing that would force the
per-visual/per-report choice early. §5 also adds three non-goals of its own (BI tool for others, chart library,
always-running service) — additions, not gaps.

---

## Part 6 — Open items, watch items and risks

### Unresolved, carried forward (4)

| # | Item | Status |
|---|---|---|
| 1 | Local vs remote serving specifics | Carried — OQ-4 |
| 2 | Host choice / COOP+COEP for SharedArrayBuffer | **Resolved** — addendum §A.2, noted under §10's "Resolved during discovery". Exemplary handling |
| 3 | Per-visual vs per-dashboard query | Carried — OQ-3, including the "safe to defer" reasoning and the cross-filtering trigger. See **G9** |
| 4 | Whether personal tool and public repo should eventually separate | **GAP (G6)** |

**G9 (Low)** — brainstorm open item 3 ends: *"Dense multi-angle reports tilt this toward one dashboard-level
slice queried many ways."* PRD OQ-3 keeps the deferral and the trigger but drops the lean. This is precisely the
kind of qualitative reasoning FR/OQ structure strips: the open question is preserved as *open*, but the prior
that the brainstorm had already established is gone, so architecture will approach it from zero rather than from
"dashboard-level slice unless something argues otherwise".

### Watch items and risks (6)

| Watch item | Status |
|---|---|
| **Git LFS budget** — free-tier quota, owner-billed clone bandwidth, `lfs:false` on checkout steps | Carried and deepened — §9 "Storage quota", addendum §A.1 (quota corrected to 10 GiB, decision re-challenged and retained, `lfs: true/false` mechanics recorded, README prerequisite noted) |
| **Scheduled-workflow auto-disable** — 60-day inactivity, monthly commits as defusal | Carried and deepened — §9, addendum §A.4 (defusal confirmed correct; the *dropped-run* failure mode is an addition, covered by FR-3's idempotence) |
| **Silent success with bad data** — 200-with-empty-body, silent schema change, stale period; mitigation = per-source assertions + freshness surfaced where it is looked at | Carried in full and made structural — §9, §4.7's description, FR-9, FR-19, FR-21, and the honest "detect and mark rather than prevent". The strongest carry-through in the document |
| **Private data retention** — local-only archive dies with the laptop | **GAP (G5)** — decision and deferral carried; risk framing dropped |
| **Public repo vs personal tool tension** | Carried — §9. But its linked resolution path is gone; see G6 |
| **The v1 bet** — if purpose-built visuals are slow even with agents, harder-reuse is the first decision revisited | Carried and strengthened — §9 and §1.3's "the condition under which this is wrong" |

Four of six watch items carried faithfully, one carried but demoted from risk to non-goal (G5), one carried with
its resolution path amputated (G6).

---

## Cross-cutting observation

The PRD's carry-through of *decisions* is strong — nearly every architecture bullet and every MoSCoW Must has a
home. Losses cluster in three predictable places:

1. **Negative and limiting statements.** FR structure states what the system *does*; the brainstorm's guards
   against overdoing it (G2's abstraction ceiling, G8's "not tiles") have no natural slot and fell out.
   Both would fit as FR consequences or Non-Goals.
2. **The second half of two-part decisions.** G1 (mechanism superseded, capability dropped with it) and G4
   (permission taken, obligation dropped) are the same failure: a sentence carrying both a change and a
   condition was carried across on the change alone.
3. **Leans and framings.** G9's dashboard-level-slice prior, G5's risk framing, G13's history-depth rider —
   material that is neither a requirement nor a question, and therefore has no receptacle in the target
   structure. These are the cheapest to reinstate and the easiest to lose again.

Recommended minimum action: G1, G2, G3. G4–G7 are worth a decision each (reinstate or explicitly defer);
G8–G15 are judgement calls that cost a sentence each.
