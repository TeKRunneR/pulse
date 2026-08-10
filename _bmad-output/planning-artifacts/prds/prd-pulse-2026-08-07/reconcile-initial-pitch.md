# Input reconciliation — `docs/initial-pitch.md`

**Date:** 2026-08-08
**Source input:** `C:\Users\yfontana\Code\pulse\docs\initial-pitch.md`
**Targets:** `prd.md`, `addendum.md` (this folder)
**Corroborating records consulted:** `.memlog.md` (this folder), `_bmad-output/brainstorming/brainstorm-pulse-data-hub-2026-08-06/brainstorm-intent.md` and its `.memlog.md`

Scope note: per the finalize brief, the PRD's deliberate exclusion of *what an individual visual shows* (figures, precision, comparisons, layout) is an explicit, repeatedly reaffirmed decision (`.memlog.md` line 41 records it as the fourth correction of the same error) and is not treated as a gap anywhere below.

---

## Summary table

| # | Pitch element | Status | Severity |
|---|---|---|---|
| 1 | duckdb-wasm's stated purpose — "dynamic dashboards can slice & dice data near-instantaneously" | **Silently dropped** (rationale replaced) | **High** |
| 2 | "Executing ingestion" must be easy and straightforward | **Silently dropped** | **High** |
| 3 | Demographic indicators as a tracked topic | **Silently dropped** | Medium |
| 4 | Personal finances as a tracked topic (and its motivating question) | **Silently dropped** (mechanism kept, purpose lost) | Medium |
| 5 | Named concrete sources: World Bank Databank, EDGAR | **Silently dropped** | Medium-low |
| 6 | Monthly ingestion baseline | Transformed, but the mitigation that depends on it is now unsupported | Medium-low |
| 7 | "dashboard" → "report" rename not recorded in the superseded-terminology list | Transformed, traceability gap | Low |
| 8 | Static hosting stated as "not a hard requirement but highly preferred" | Transformed, hardened silently | Low |
| — | Broken cross-references to a deleted addendum section | Incidental defect found while reconciling | Medium-low |

Verified as properly carried or deliberately deferred (no finding): the data lab; collect-from-sources; create-visuals-and-pin-to-dashboards; the Claude Design → coding-agent authoring pipeline; personal-use-first framing; portfolio-not-contributions framing; public/private source split and manual CSV-into-a-folder mechanism; DuckDB as the store; no existing dashboarding engine (Superset/Dash); HTML/SVG/CSS/JS as the visual medium; cheap automatic triggering via GitHub Actions; "everything can be challenged"; ease of opening dashboards; ease of adding a source; ease of creating a visual and adding it to a dashboard.

---

## 1. duckdb-wasm's stated purpose — near-instantaneous slice & dice — replaced rather than carried

**Severity: High.**

**Pitch (line 29):** "I'd like dashboards to use duckdb-wasm as their cache, so that dynamic dashboards can slice & dice data near-instantaneously."

The pitch attaches a functional rationale to duckdb-wasm: it exists so dashboards can be *dynamic* and respond to slicing *near-instantaneously*. The brainstorm preserved this — "duckdb-wasm is kept, because in-browser real-time analysis is genuinely wanted", with "reports needing open-ended slicing" opting in (brainstorm-intent §4).

**What the targets say.** The technology survives; the reason does not.
- `prd.md` FR-18 states only: "Data is queried in the browser via duckdb-wasm on all pages." It is a delivery-mechanism statement with no user-facing consequence attached.
- `prd.md` §1.1 goal 3 recasts duckdb-wasm as wanted "for their own sake, not necessarily because they are the most efficient choice" — i.e. as playground, explicitly *not* for a functional reason. This is the opposite of what the pitch gave as the reason.
- `addendum.md` §C.4 layer D records the "duckdb-wasm on all pages" simplification purely as a code-path-count argument (removes the dual data-delivery path). The interactivity purpose the brainstorm's per-report opt-in was serving is not carried across the simplification.

**Why it is a silent drop, not a deferral.** There is no non-goal covering interactivity. Worse, `.memlog.md` line 53 records that cross-filtering — which *was* a brainstorm "Won't this time" item — "removed from non-goals but still referenced by open question 3". So interactive slicing is now stated nowhere: not as a requirement, not as a non-goal, not as an open question in its own right. The only surviving trace is the parenthetical in open question 3 ("only cross-filtering would force the choice early"), which presumes a decision the PRD no longer contains.

**Consequence if not fixed.** An architecture pass reading the PRD literally can satisfy every FR with a fully static, non-interactive site that instantiates duckdb-wasm merely to read parquet at load. It would pass FR-18, contradict nothing, and lose the capability the user named as the reason for the technology. There is also no responsiveness NFR anywhere, so "near-instantaneous" has no home — note that `addendum.md` §A.2 accepts a "roughly linear slowdown on scan/aggregate-heavy work" from the single-threaded EH bundle without anyone checking that against a stated interaction-latency expectation.

**Recommended fix.** Either (a) add a requirement that reports may support in-page slicing/filtering answered client-side without a round trip, with a stated responsiveness expectation, or (b) if interactivity really is out for v1, say so explicitly in §5 Non-Goals and restore the rationale trail — but note that (b) removes the pitch's only functional justification for duckdb-wasm and leaves goal 3 carrying it alone. Either way, restore cross-filtering to whichever list it belongs in so open question 3 stops dangling.

---

## 2. "Executing ingestion" has no ease requirement — and no requirement at all

**Severity: High.**

**Pitch (lines 21–25):** "It is important that the following are easy and straightforward: Executing ingestion / Opening dashboards / Adding a new ingestion source / Creating a new visual & adding it to a dashboard."

Three of the four are covered:
- *Opening dashboards* → FR-18, "Opening reports locally is a single command."
- *Adding a source* → FR-1, FR-22, FR-23, NFR-1.
- *Creating a visual and adding it to a dashboard* → FR-10, FR-11, FR-15, FR-22.

*Executing ingestion* is covered by nothing. FR-3 covers **automatic** fetching on a schedule ("without human involvement"). FR-4 covers private data being **placed** in a known location. No requirement anywhere states that a human can trigger a pipeline run — or one source's ingestion — on demand, in one step. Search of both targets confirms no FR, NFR or consequence addresses manual invocation.

**Why the redefinition of "easy" does not cover this.** NFR-1 redefines the ease metric as agent legibility: "A coding agent does the work, so 'easy' means legible to an agent." That reframe is legitimately recorded (brainstorm `.memlog.md` line 97, brainstorm-intent §3) and correctly covers the two *authoring* items. But it does not cover the two *operating* items, which are acts Yann performs, not an agent. Opening reports survived the reframe (FR-18) because it landed in a build/serve feature. Executing ingestion had no equivalent home and fell through unrecorded — no memlog entry discusses it.

**Concrete hole this leaves.** FR-4 says private data is supplied by dropping files in a known location, and stops there. Nothing states what turns a dropped CSV into a snapshot and a dataset. The pitch's private-source flow ("I extract a CSV file or similar into an input folder", line 18) is only half-specified without an on-demand local run.

**Recommended fix.** Add an FR under §4.1 or §4.6: a pipeline run — full, or scoped to one source — is executable on demand by a single local command, with the same behaviour as a scheduled run. This also gives §7.1's "a scheduled pipeline run executing the whole chain" a manually testable counterpart and is what the §7.3 "manual ingestion path" Could actually requires.

---

## 3. Demographic indicators dropped entirely

**Severity: Medium.**

**Pitch (line 12):** demographic indicators listed as one of four example topics.

Carried into the brainstorm verbatim ("about the economy, demographics, climate and personal finances", brainstorm-intent §1; brainstorm `.memlog.md` goal line). Absent from `prd.md` and `addendum.md` — a full-text search for "demograph" across both returns nothing.

`prd.md` §1.1 goal 1 names only two of the four topics: "Economic indicators are the worked example; climate and emissions indicators are equally in view." The PRD `.memlog.md` line 37 shows the elision happening: "Applies to economic indicators, climate/emissions, and others." The "and others" was never expanded.

**Why it matters beyond example-listing.** The topic set is what §5's "Multi-source datasets" non-goal and the "one dataset per question area" framing are sized against, and it is the only signal downstream has about how heterogeneous the source landscape will be. Two topics reads as a narrower system than four. This is not an MVP-scope question — §7.1 correctly ships one source and one report — it is a vision-scope question, and vision is where the PRD chose to enumerate topics.

**Recommended fix.** Restore the full topic set in §1.1 goal 1 (economic, demographic, climate/carbon, personal finances), keeping MVP scope unchanged.

---

## 4. Personal finances: the mechanism survived, the purpose did not

**Severity: Medium.**

**Pitch (lines 14, 18):** "Personal finances indicators" as a tracked topic; "private sources, like my bank" as the collection route.

The **mechanism** is well covered: private sources (§3 glossary, FR-4), private reports and build profiles (FR-17), privacy separation (NFR-3), private-raw durability as a non-goal (§5), and the §7.3 Could.

The **purpose** is gone. Personal finances is never named as a topic pulse tracks. The closest the PRD comes is §9's risk that "personally revealing indicators" may be suppressed by portfolio pressure — which references the sensitivity without ever establishing the subject. A reader of the PRD alone learns that pulse supports private data but never learns what it is for.

The brainstorm carried the motivating question explicitly — brainstorm `.memlog.md` line 16: "Am I on track, or do I need to do something about it (personal finances)?" Note that this is a *different shape of question* from the public-indicator case: it is a personal-status question, not an ambient-familiarity question. §2.1 Jobs To Be Done contains nothing resembling it, and §1.1 goal 1 (ambient familiarity — carrying numbers in mind) does not describe it well. That mismatch is itself unrecorded.

**Recommended fix.** Name personal finances alongside the other topics in §1.1, and either add a JTBD covering the "am I on track" shape or record explicitly that the private-data path serves ambient familiarity in the same way public indicators do.

---

## 5. Named concrete sources dropped: World Bank Databank and EDGAR

**Severity: Medium-low.**

**Pitch (line 17):** "public databases, like the World Bank Databank or EDGAR".

- **World Bank** survives only obliquely: brainstorm MoSCoW named it ("One public source (World Bank / public macro)"), but `prd.md` §7.1 reduces it to "Ingestion of one public source" with no name. In `addendum.md` it appears only twice incidentally — as OWID's upstream in §B.3, and as a filename in an Observable Framework example in §B.4 (`data/wdi.parquet.py`).
- **EDGAR is absent from every artifact** — pitch, brainstorm and PRD. It never survived even the first hop.

**Why EDGAR's absence is more than a lost example.** In the pitch's context EDGAR sits directly beside the climate/carbon topic. `prd.md` §1.1 elevates climate and emissions indicators to co-equal status with economics in goal 1, while the only candidate source for that data has vanished from the record. The result is a stated goal with zero identified supply.

Naming the MVP's public source also matters for §7.1: "one public source" is unfalsifiable as a scope statement, whereas "the World Bank Databank" makes the vertical slice concrete and makes the FR-1/FR-22 exemplar checkable.

**Recommended fix.** Name the MVP public source in §7.1, and record EDGAR (or an equivalent emissions source) somewhere — vision, addendum, or an open question if the choice is genuinely undecided.

---

## 6. Monthly cadence: correctly generalised, but a mitigation now rests on nothing

**Severity: Medium-low.**

**Pitch (line 19):** "Ingestion should occur regularly, probably monthly."

The generalisation to per-source cadence (FR-2) is a **recorded decision** — `.memlog.md` line 45, explicitly "Replaces the brainstorm's implied global monthly default." That part is correct and is not a finding.

The problem is a consequence nobody tracked. `addendum.md` §A.4 and `prd.md` §9 both defuse the scheduled-workflow-decay risk with monthly snapshot commits: "The brainstorm's intended defusal — monthly snapshot commits — is therefore the correct one and makes this a non-issue, **provided the monthly job commits to the default branch**." But after FR-2, no requirement anywhere establishes that any job runs monthly, or that any run commits to the default branch. §7.1 says only "a scheduled pipeline run". If every source were declared quarterly, the mitigation silently fails — and the failure mode (workflows auto-disabled after 60 days of inactivity) is exactly the one §9 claims to have defused.

**Recommended fix.** Either state a default/baseline cadence, or make the risk mitigation self-supporting: a requirement that the scheduled run commits to the default branch at least monthly regardless of per-source cadence.

---

## 7. The "dashboard" → "report" rename is not in the superseded-terminology record

**Severity: Low.**

The pitch says "dashboarding tool", "dashboards" (10 occurrences), "pin those visuals to dashboards". The PRD renames this to **report**, with a substantive redefinition in §1.2 and §3 ("a standing analysis: a curated, multi-angle page on one topic... Multi-page and scrolling are acceptable") — a deliberate and well-motivated shift away from the tile/single-screen connotation of "dashboard".

The rename itself is fine. The gap is that `addendum.md` §C.5, whose stated job is "Recorded so that downstream readers encountering the older documents are not misled", records the `hand-built` → `purpose-built` rename but **not** `dashboard` → `report`. A downstream reader arriving via `docs/initial-pitch.md` (linked from `prd.md` §0) has no bridge for the project's most frequently used original noun. Note the brainstorm also still uses `dashboard.toml` and "per-dashboard query" in places, so the ambiguity is live in a linked artifact.

**Recommended fix.** One bullet in §C.5: "dashboard" (pitch, brainstorm) → read **report**; note the redefinition away from single-screen tiles.

---

## 8. Static hosting: a stated preference hardened into a requirement without note

**Severity: Low.**

**Pitch (line 31):** "I believe that the previous point should also make it possible to serve these dashboards from static storage. **This isn't a hard requirement but would be highly preferred**, as it would make it quite easy to quickly open them locally, or host them somewhere cheaply."

FR-18 makes static output a hard requirement, and the reasoning is preserved (local opening via FR-18's single command; cheap hosting via NFR-4). The transformation is defensible — every architecture candidate emits static output anyway (`addendum.md` §C.4, "Serving is not a differentiator"), so the preference is free.

The only loss is that the user's explicit softness marker is gone, which quietly closes the fallback: if some later requirement (interactivity, private-data serving, large datasets) argued for a server, FR-18 now forbids it and no record says the constraint was ever negotiable. Given §5 also lists "An always-running service" as a non-goal, the hardening is doubly locked in.

**Recommended fix.** Optional. A half-sentence in FR-18 or §11 noting that static output was a strong preference rather than an absolute in the source input, and what would justify revisiting it.

---

## Incidental defect found while reconciling

**Severity: Medium-low. Not a pitch reconciliation issue — a broken internal reference.**

`prd.md` links to a section of the addendum that no longer exists, in two places:
- §6, NFR-4: "See [addendum §D](addendum.md) for the storage-quota analysis and its watch item."
- §9, Storage quota risk: "Analysis, decision and mitigations in [addendum §D](addendum.md)."

`addendum.md` contains sections A, B and C only. `.memlog.md` line 51 confirms the cause: "Addendum: section D deleted (duplicated the LFS decision Yann had already rewritten into A.1); C.5 repointed from D to A.1". C.5 was repointed; these two PRD references were not. Both should point to §A.1.

Note also that §9's risk text promises "mitigations" and NFR-4 promises "its watch item", but per `.memlog.md` line 50 the mitigations-as-requirements and the watch item were deliberately dropped at Yann's direction. So the two sentences now over-promise as well as mis-link, and should be reworded, not just repointed.

---

## Method note

For each pitch element I checked: presence in `prd.md`; presence in `addendum.md`; presence as a §5 non-goal, §7.4 out-of-scope item, §10 open question or §11 assumption; and presence of a decision record in `.memlog.md` or the brainstorm artifacts. An element was classified **silently dropped** only when it was absent from both targets *and* no decision to drop it appeared in any record. Items 6, 7 and 8 are classified as transformations with residual loss rather than drops, and are reported because the loss has a downstream consequence.
