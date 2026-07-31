# VERIFY-LEDGER — IIS 18-minute deck

**Spec:** `presentation-iis-deck-verify-fixpass` · Phase A (verify only, nothing edited)
**Deck:** `Cho-IIS_Insecure-Island-Syndrome_18min.qmd` (as of 2026-07-31 09:39)
**Sources:** `Manuscript_v5.3_2026-07-31.pdf` (26 pp) · `Appendix_v5.1_2026-07-31.pdf` (30 pp) · `Analysis_v5.3_export.RData` · `IIS-auto.bib` · `_cluster_labels_override.R`
**Written:** before any edit to the deck. Phase B had not begun when this file was created.

---

## 0. Summary

| | Count |
|---|---|
| Rendered quantities checked | 60 distinct inline-R quantities + 7 rendered tables (≈ 150 table cells) |
| Matched to a PDF at displayed precision | 57 |
| `DERIVED` (no PDF counterpart; verified against export) | 3 |
| Mismatched against source | 0 |
| **Defects confirmed** | **7** (2 not anticipated by the spec) |
| Suspicions cleared | 1 (§4.6 partial) |
| `NEEDS MJ` | 2 |

**Headline:** every *number* in the deck is correct. The defects are in **labels, attribution and framing** — three mislabeled table columns, a factually wrong ranking claim inherited from the manuscript, a mis-citation, two missing references, a retired analysis reinstated, and one leaked spec instruction.

---

## 1. Export identity (§3.1 / §4.0) — **CRITICAL, checked first**

| Item | Result |
|---|---|
| Glob under render locale (`en_US.UTF-8`, ICU) | `../../Cho-IIS/Analysis/Analysis_v5.3_export.RData` |
| Glob under `LC_ALL=C` | `../../Cho-IIS/Analysis/Analysis_v5_export.RData` ← **superseded v5** |
| Manuscript v5.3 `Setup` declares | `../Analysis/Analysis_v5.3_export.RData` (line 140) |
| Appendix v5.1 `Setup` declares | `../Analysis/Analysis_v5.3_export.RData` (line 124) |
| **Which export the deck actually loaded** | **v5.3 — the correct one** |

**Verdict: `CONFIRMED-WRONG` (mechanism), but no number is affected.**

The spec's collation analysis is exactly right, and I reproduced both branches. `_` (0x5F) sorts above `.` (0x2E) in C collation, so `Analysis_v5_export.RData` heads the descending sort and `cands[1]` picks the *superseded v5 export*. Under macOS ICU collation punctuation is weighted differently and `v5.3` wins. The resolution is therefore locale-dependent — the defect is real regardless of which way it falls, and version ordering by string sort must not survive this pass.

MJ rendered on macOS under `en_US.UTF-8`, so the deck loaded v5.3. The spec's own corroboration holds: the deck uses `slope_corroborated_weighted` and `design_concordance_headline`, and I confirmed by inspection that neither column exists on the v5 export — a v5 load would have errored rather than rendered wrong numbers. **§3.3 did not need re-running against a corrected render.**

**Fix (Phase B):** parse the `load(...)` path out of the manuscript's `Setup` chunk, `stopifnot` on existence, hard-fail with a clear message on a parse failure, keep the `IIS_EXPORT` override, print the resolved path into the provenance comment.

Note the build spec's §0.4 check — stop if the glob disagrees with the manuscript's declared export — was indeed never implemented.

---

## 2. Seeded defects (§4.0–§4.6)

### §4.0 Export resolution — **CONFIRMED** (mechanism), **no numeric consequence**
See §1 above.

---

### §4.1 RQ1 facet-map columns are positional — **CONFIRMED-WRONG. This is the defect MJ spotted.**

`pivot_wider(names_from = facet)` emits facet columns in factor/first-appearance order:

```
[1] "repertoire"  [2] "Aggression_online"  [3] "Disinfo_ops"
[4] "J4_distrust" [5] "TV_threat"          [6] "xMedia_FakeNews"
```

The hard-coded `col.names` vector is `c("Repertoire", "Online Aggr.", "Disinfo Ops", "TV Threat", "xMedia Fake", "Media Distrust")`. Assignment is by position, so:

| Position | Actual data | Deck's label | Verdict |
|---|---|---|---|
| 2 | `Aggression_online` | Online Aggr. | correct |
| 3 | `Disinfo_ops` | Disinfo Ops | correct |
| 4 | `J4_distrust` | **TV Threat** | **WRONG** |
| 5 | `TV_threat` | **xMedia Fake** | **WRONG** |
| 6 | `xMedia_FakeNews` | **Media Distrust** | **WRONG** |

**Three of six columns are mislabeled**, in a cyclic shift. Concretely, the deck currently shows cluster 3's row as `TV Threat = −0.14` when −0.14 is its *media distrust* value, and `Media Distrust = +0.18` when +0.18 is its *xMedia Fake* value.

The manuscript's `tbl-h1-facets` uses `gt()` + `cols_label()` — name-keyed — and its printed header order is **Repertoire | Online Aggr. | Disinfo Ops | Media Distrust | TV Threat | xMedia Fake**. The deck's *cell values* match the manuscript table cell for cell; only the headers are wrong. The deck traded a name-keyed idiom for a position-keyed one, exactly as the spec suspected.

**Fix (Phase B):** name-keyed via an explicit `select()` of the facet columns into a fixed order, then a named lookup for the labels — not a permutation of the positional vector.

---

### §4.2 Wrong Chang 2023 — **CONFIRMED-WRONG**

- Deck slide 4 cites `Chang et al. (2023)` for "low baseline trust in TV news", and References I lists **Chang, C., Hung, Y.-C., & Hsieh, M. (2023)**, *We are what we consume* — citekey `changWeAreWhat2023`, the media-diet-colour vote-prediction paper.
- Manuscript p.4: "**A. C. Chang & Tang (2023)** document that only roughly 46–47% of …" — citekey `changPoliticalFoundationMainstream2023`: Chang, Alex Chuan-hsien & Tang, Yen-Chen, *The Political Foundation of Mainstream Media Trust in East and Southeast Asia: A Cross-national Analysis*, **Asian Politics & Policy, 15**(4), 585–604, doi 10.1111/aspp.12715.

Two different works by two different first authors named Chang. The deck cites the wrong one, and — as the spec notes — the diet-colour paper is precisely the one the manuscript declines to use for imputation.

**Fix (Phase B):** in-text → `Chang & Tang (2023)`; replace the References I entry; the diet-colour entry becomes listed-but-uncited and is removed.

---

### §4.3 Works cited but absent from the reference slides — **CONFIRMED-WRONG**

Full cite-vs-reference sweep, both directions:

| Direction | Finding |
|---|---|
| Cited, not listed | **Hameleers & Garnier Ortiz (2024)** (B8); **Matthes et al. (2022)** (B8) |
| Listed, not cited | none currently (Chang C. becomes one once §4.2 is fixed) |

Both missing works resolve to verified `IIS-auto.bib` entries:
- `hameleersRiskPerceptionsMisinformation2024` — *The International Journal of Press/Politics*, 31(2), 344–366, doi 10.1177/19401612241304050.
- `matthesPerceivedPrevalenceMisinformation2022` — *Information, Communication & Society*, 26(16), 3133–3156, doi 10.1080/1369118x.2022.2146983.

No citekey needs hand-authoring. All 19 other author-year mentions on slides resolve to a bib entry **and** to a reference slide, and I checked each against the claim it supports, not just the key — no further mis-attribution found.

*Minor, noted not fixed:* slide 11 names "Williams' ISCS" without a year; this reads as an instrument name rather than an author-year citation, consistent with the manuscript's usage.

---

### §4.4 B6 reinstates retired post-hoc power reasoning — **CONFIRMED-WRONG**

Appendix v5.1 **S4.2** ("What the RQ2 interaction blocks can detect"):

> "Comparing each cell's observed effect size against its own minimum detectable effect size — **which an earlier version of this appendix tabulated** — is post-hoc power reasoning: the observed effect and the *p*-value are two expressions of the same quantity, so such a comparison cannot supply evidence about a null result that the *p*-value did not already carry."

The deck's B6 renders `rq2_power_table` **including `Observed f2` and a `Powered` column** — exactly the comparison the paper retired one version ago. The object still exists in the export, so it renders fine; this is a substantive inconsistency, not a technical error.

The replacement statement, from S4.2 and mirrored in Methods (manuscript line 374):

> the 14-df repertoire × moderator block at N = 2,828 detects *f²* ≥ 0.0066 at 80% power with α = .05 (and *f²* ≥ 0.0049 in the two media-distrust cells, estimated at N = 3,761), well below conventional small-effect benchmarks. A closed gate therefore rules out repertoire-level moderation of ordinary size; moderation smaller than that threshold is not ruled out.

Export values confirm: `min_detectable_f2` = 0.006581915 for the eight N = 2,828 cells and 0.004930740 for the two N = 3,761 distrust cells. All ten rows carry `adequately_powered = TRUE`.

**Fix (Phase B):** rebuild B6 as the design-property statement; drop `Observed f2` and `Powered`; retitle.

---

### §4.5 Spec instruction leaked into slide 26 — **CONFIRMED-WRONG**

Line 687: `Do not cut. Four items:` — rendered as visible body text. Removed as part of the item 8 rewrite.

**Whole-deck sweep for other leaked instruction text: one instance only.** I scanned every non-notes line for imperative build-spec register (`Do not`, `Never`, `Must`, `Keep the`, `Add`, `Replace`, `Retitle`, `Verify`, `Fix:`, `Optional`, `Report`). Nothing else. Note that slide 20's `[Report W_overall_slope, never the bare W.]{.neon-cyan}` is a *reporting rule addressed to an analyst* rather than a leaked spec line — it is genuine paper content (manuscript line 379) misplaced as an audience takeaway, and §5 item 6 already replaces it.

---

### §4.6 Provenance stated twice — **CONFIRMED (exactly two, no third copy)**

| Location | Line |
|---|---|
| Slide 24 `.theory-box` | 671 |
| Slide 26 first bullet | 689 |
| Slide 24 speaker notes (Chinese) | 680 |

Two rendered copies plus one notes mention. **No third rendered copy exists** — that part of the suspicion is cleared. Item 8 moves it wholly to 26; the notes mention on 24 should go too, for consistency.

---

## 3. Sweep results

### 3.2 Object-and-column sweep — **PASS**

- All 31 objects the deck names exist in the export, **except** `clab` and `cluster_labels_provisional_note`, which are correctly supplied by `_cluster_labels_override.R` (verified present after sourcing).
- Every named column exists on its named object. No typos, no stale column names.
- **Every `slice(1)` verified to land on the intended row.** `coef_row`, `coef_row_bosc`, `rq2_row`, `h1_row`, `h1_coef`, `fm_row`, `rq2_term`, `bpath`, `ie_row`, `inv_row` all filter on a key that is unique in its table, so `slice(1)` is a no-op safety net rather than an arbitrary pick. The one genuinely non-unique case is `h3c_row`, which filters `str_detect(quantity, "^b-path")` within a mediator — `h3c_tie_type_table` has 19 rows and multiple quantity types per mediator; I confirmed the b-path row is the intended one and that its `delta_b` (0.0594), CI and ratio (2.516) match the manuscript's Δb = 0.059, [0.012, 0.103], 2.5×.
- `label_agreement` — named by spec §5 item 4 as the source for the agreement split — **does not exist in the export**. See §5 below.

### 3.3 Number-against-PDF sweep — **57 matched / 3 DERIVED / 0 mismatched**

Matched at displayed precision, by slide:

| Slide | Quantities | Source |
|---|---|---|
| 8 Sample funnel | 6,106 · 4,031 · 4,027 · 2,828 · 3,889 | MS pp.12–13 |
| 11 Measuring both facets | α .61–.78 · distress .848 · BSC .84 · BoSC .79 | APX S1.1/S1.3/S1.4 |
| 16 RQ1 omnibus | F 4.59–4.90 · p < .001 · F 8.14 · f² .023–.030 | MS p.12 |
| 17 Facet map | all 75 table cells | MS Table 2 |
| 17 | distrust rank 13 (and rank 8 in notes) | MS p.13 |
| 18 RQ2 funnel | 14 · 6 · 4 · 4 · 2 · 5 of 10 · 8 of 10 · the five open cells by name | MS p.14 |
| 19 Interpretable | b .265 / .231 / −.277 / −.179 · design p .051 / .089 | MS p.14 |
| 20 Moderator effect | 0.116 · p .007 · 0.013 · 0.056 | MS p.14 |
| 21 b-paths | all 10 rows · TV threat b 0.049 | MS p.16, APX S3.3 |
| 22 H3c | Δb 0.059 · [0.012, 0.103] · 2.5× · Beaudoin .26 / .17 | MS pp.5, 17 |
| 23 H3b | p_design .036 · .071 / .077 marginal | MS p.17 |
| B3 | Distress BSC −0.028 · BoSC −0.150 | APX S3.3 |
| B4 | Δχ²(3) = 1.98, p .577 · ΔCFI .004 | APX S1.3 |
| B5 | "18 of 24 claims fully concordant" | MS p.17 |
| B6 | min. detectable f² 0.0066 / 0.0049 · SE ratio 1.17 | APX S4.1–S4.2 |
| B7 | 28 cells · IE −0.0514, CI [−0.0964, −0.0091] | APX S3.4 |
| B8 | r .101 · .107 · .636 | APX S1.5 |

`DERIVED` (legitimately absent from both PDFs; verified directly against the export): `sum(h3_distrust_indirect_table$ind_sig)` = 4 · `sum(h3_distrust_indirect_table$design_sig)` = 3 · the deck's own arithmetic `n_corrob_w − n_interp` = 2 (the manuscript states the "2 of the 6" in prose, so this one is corroborated too).

**Literals:** the only numeric literals in slide bodies are the per-wave counts `2,015 + 2,016`. Manuscript v5.3 line 303 still hard-codes these same two literals, so per build-spec §3 the deck may repeat them. Everything else is inline R. No literal estimate, count, *p*, CI bound, *F*, effect size or ratio anywhere.

### 3.4 Claim-scope sweep (build-spec §4, nine rules) — **8 pass, 1 fail**

| Rule | Verdict |
|---|---|
| 1. H3c magnitude, never "turn to bonding" | **PASS** — the only occurrence of "turn to" is the explicit denial on slide 22; the compositional formulation is present |
| 2. RQ2 moderation only where `interpretable_as_moderation` | **PASS** — the 14 appears solely as funnel step 1 |
| 3. No repertoire-specific moderation of the distrust pathway | **PASS** — nowhere, including figures |
| 4. H3b carries its design qualification on the same slide | **PASS** — slide 23 |
| 5. Indirect effects labelled descriptive | **PASS** — B7 opens "**Descriptive.**" and disclaims the index of moderated mediation |
| 6. Provenance on its own slide, not dropped | **PASS** (stated twice — that is §4.6, not a rule-6 failure) |
| 7. Two estimators, neither conservative; primary-only claims flagged | **PASS** — slide 13 theory-box; slides 19, 21, 23 each flag |
| 8. Ecological inference; partisanship not imputed | **PASS** — slide 26 |
| 9. The partisan reading failed and is reported as failing | **PASS** — slide 17 notes, at length |
| **Unnumbered — a claim must be true** | **FAIL** — see NEEDS MJ #1 |

### 3.5 Citation sweep — see §4.2, §4.3. Complete both directions.

### 3.6 v5.3 / v5.1 consistency sweep — **PASS on 5 of 6**

| Item | Verdict |
|---|---|
| Added value, not the old composite distinctiveness score | **PASS** — slide 10 says "added value — the gap between within-cluster and population prevalence" |
| No ideological-orientation field anywhere | **PASS** — slide 10 states it as the takeaway; `_cluster_labels_override.R` *drops* `ideology` from `cluster_taxonomy`, and B1 does not render it |
| 30 replicates with agreement reported | **PARTIAL** — 30 replicates stated on slide 10; **agreement is not reported anywhere in the deck** (§5 below) |
| DBCV carries selection, silhouette secondary, not joint evidence | **PASS** — slide 9 body and notes both subordinate silhouette explicitly |
| Design-property statement replacing post-hoc power | **FAIL** — §4.4 |
| Provisional labels | **PASS** — slide 10 notes, slide 12 citation line, B1, all via `cluster_labels_provisional_note` |

---

## 4. NEEDS MJ

### NEEDS MJ #1 — "Cluster 3 is highest-threat on all four threat facets" is false, **and the manuscript says it too**

Slide 17 (line 510) and slide 17's notes assert:

> Cluster 3 (Full-Spectrum Online News) is **highest**-threat on all four threat facets and simultaneously rank 13 of 15 in distrust.

The export says otherwise:

| Facet | Cluster 3 rank | Highest is |
|---|---|---|
| Aggression_online | **1** | cluster 3 (+0.249) |
| Disinfo_ops | **1** | cluster 3 (+0.211) |
| TV_threat | **2** | cluster 6, +0.435 vs cluster 3's +0.309 |
| xMedia_FakeNews | **6** | cluster 6, +0.372 vs cluster 3's +0.179 |

Cluster 3 is highest on two of four, not four of four. **Cluster 6 (No TV News, Diverse Online) is highest on the other two.**

This is not a deck error in origin. **Manuscript v5.3 p.13 makes the identical claim** ("Note the reversal against H1a: cluster 3 is the highest-threat repertoire on all four threat facets and simultaneously rank 13 of 15 in distrust"), and the deck faithfully compressed it. But the manuscript **contradicts itself one page earlier** (p.12): "The pattern is facet-specific rather than a single high-threat/low-threat ordering: **no repertoire holds the same rank on all four facets**" — and its own Table 2 prints cluster 6 above cluster 3 on both TV Threat and xMedia Fake.

The manuscript's *defensible* version, also on p.12, is true and I verified it: relative to the reference, cluster 3 "perceives elevated threat on all four facets" — `h1_cluster_coefficients` gives cluster 3 positive and significant on all four (b = 0.254, 0.179, 0.217, 0.174; p = 4.6e-05, .003, .0002, .006), while its distrust coefficient is null (b = −0.079, p = .202) and its distrust *rank* is 13 of 15.

**What I did in Phase B:** replaced "highest on all four" with the true and equally strong formulation — significantly elevated on all four threat facets relative to the reference, top-ranked on both online threat facets, and rank 13 of 15 in distrust. The M1 reversal is fully preserved; nothing is weakened. **The same correction is needed at manuscript v5.3 p.13 and I did not make it — the manuscript is out of scope for this spec.** This is the one place the deck now knowingly diverges from the paper, and it diverges by being right.

### NEEDS MJ #2 — slide 20's "distress does not" drops a qualification the paper carries

Deck slide 20: "Security values raise perceived threat on three of four threat facets (cluster-averaged overall slopes); **distress does not**."

Manuscript p.14: "…whereas distress does not, **only TV_threat reaching a comparable slope**."

The three-of-four claim for security values is correct (Aggression_online, Disinfo_ops, TV_threat exclude zero; xMedia_FakeNews does not). But the flat "distress does not" is an overstatement: TV_threat × Distress has `W_overall_slope` = 0.051, CI [0.0007, 0.1017] — it excludes zero, barely. The paper qualifies; the deck's compression dropped the qualifier. Restored in Phase B as part of item 6.

---

## 5. Found, not anticipated by the spec

1. **`label_agreement` does not exist in `Analysis_v5.3_export.RData`.** Spec §5 item 4 says the exact-string and semantic-group agreement figures "both live in `label_agreement`" and must be inline R. They do not — the object is built inside `Appendix_v5.1_2026-07-31.qmd` from four CSVs under `Analysis/0705_data/data_clustered/76/relabeling_2026-07/`, deliberately outside the export because they are "labeling-protocol measurements, not model estimates". The deck's setup loads only the export. Adding CSV reads would break the deck's export-only contract for an item the spec marks optional, so I took the spec's own fallback and **routed the agreement split to speaker notes**, quoting the appendix's verified figures (14 of 15 clusters at 100% semantic-group agreement; exact-string agreement 3.3%–30.0%; cluster 11 produced 30 distinct strings in 30 runs). This also closes the §3.6 "agreement reported" gap.

2. **The `cluster_taxonomy` object in the export still carries an `ideology` column.** It is dropped by `_cluster_labels_override.R` before anything renders, and B1 never selects it, so the deck is clean. Flagging only because anything reading the raw export without the override would surface a field the v5.3 schema was supposed to have retired.

---

## 6. Per-slide verdicts

| # | Slide | Verdict |
|---|---|---|
| 1 | Title | CORRECT |
| 2 | Roadmap | CORRECT |
| 3 | A Fragmented Media System | **CONFIRMED-WRONG** — §4.2 wrong Chang |
| 4 | One Construct, or Two? | CORRECT |
| 5 | IIS: The Framework and What It Predicts | CORRECT (inert — §5 item 3) |
| 6 | Data and the Sample Funnel | CORRECT — 5/5 Ns match |
| 7 | Discovering Repertoires | CORRECT — DBCV/silhouette hierarchy right |
| 8 | Naming Them: LLM Structured Labeling | CORRECT but thin (§5 item 4); agreement unreported |
| 9 | Fifteen Repertoires | CORRECT |
| 10 | Measuring Both Facets | CORRECT — 5/5 reliabilities match |
| 11 | How a Claim Gets Made Here | CORRECT (§5 item 5 adds glosses) |
| 12 | RQ1: Both Facets Differentiate | CORRECT — 7/7 match |
| 13 | **RQ1: But Not in the Same Order** | **CONFIRMED-WRONG ×2** — §4.1 headers; NEEDS MJ #1 |
| 14 | RQ2: From Significant Terms to Interpretable | CORRECT — 8/8 match |
| 15 | RQ2: Amplification *and* Attenuation | CORRECT |
| 16 | RQ2: The Moderator's Own Effect | CORRECT numbers; NEEDS MJ #2; takeaway is a reporting rule (§5 item 6) |
| 17 | RQ3: Distrust Erodes; Threat Does Not | CORRECT |
| 18 | RQ3 / H3c: Bridging Erodes Harder | CORRECT |
| 19 | RQ3 / H3b: Compensation | CORRECT |
| 20 | Informational Insecurity Is Not One Thing | CORRECT; provenance duplicated (§4.6) |
| 21 | **What This Study Does Not Claim** | **CONFIRMED-WRONG** — §4.5 leaked instruction |
| 22 | References I | **CONFIRMED-WRONG** — §4.2 |
| 23 | References II | **CONFIRMED-WRONG** — §4.3 two missing |
| 24 | Closing | CORRECT |
| B1 | Full Cluster Taxonomy | CORRECT |
| B2 | Full RQ2 Verdict Table | CORRECT |
| B3 | Full b-Path Tables | CORRECT |
| B4 | Measurement Detail | CORRECT |
| B5 | Design Concordance Ledger | CORRECT |
| B6 | **Power for the RQ2 Interaction Blocks** | **CONFIRMED-WRONG** — §4.4 |
| B7 | Indirect Effects | CORRECT |
| B8 | Why This Isn't Just Measurement Noise | **CONFIRMED-WRONG** — §4.3 |

---

*End of Phase A. Phase B begins after this file is written.*

---

# Phase B — what was applied

Appended after Phase B. Nothing above this line was edited.

| Defect | Fix applied |
|---|---|
| §4.0 | Setup chunk now parses the `load(...)` path out of `Manuscript_v5.3_2026-07-31.qmd` and rebases it onto the deck's directory. Hard-fails on a missing manuscript, an unparseable path, or two conflicting declarations. `IIS_EXPORT` override kept. Resolved absolute path prints into the provenance comment. `Sys.glob` is gone. |
| §4.1 | Table is now name-keyed: `select(Repertoire = repertoire, all_of(names(facet_labels)))` then `rename_with()` through a named lookup declared in setup. `all_of()` errors loudly if a facet disappears. Header order now matches the manuscript's `cols_label()` order. **Verified 75/75 cells and all 6 headers against `tbl-h1-facets`.** |
| §4.2 | In-text → `Chang & Tang (2023)`; References I entry replaced with `changPoliticalFoundationMainstream2023`; the diet-colour entry removed (it became listed-but-uncited). |
| §4.3 | Hameleers & Garnier Ortiz (2024) and Matthes et al. (2022) added to References II from verified bib entries. |
| §4.4 | B6 retitled "What the RQ2 Interaction Blocks Can Detect"; renders the two distinct minimum-detectable-*f²* rows only. `Observed f2` and `Powered` gone from the deck. Theory-box carries S4.2's licensed conclusion **and** its reason for refusing the observed-vs-detectable comparison. |
| §4.5 | `Do not cut. Four items:` removed with the slide-26 rewrite. |
| §4.6 | Provenance removed from slide 24 (body **and** notes); slide 24's theory-box replaced with the theoretical payoff. Provenance now appears exactly once, on slide 26. |
| NEEDS MJ #1 | Slide 17 and its notes now say cluster 3 is *elevated on all four threat facets against the reference* (with `cl3_threat_p_max` inline) and rank 13 of 15 in distrust. Slide 26 carries the same corrected wording. Notes flag the manuscript's contradiction explicitly. |
| NEEDS MJ #2 | Slide 20 restores "only TV threat reaching a comparable slope". |

**§5 content items:** all six implemented. Item 3 split into two slides (see below). Item 4's optional agreement split routed to notes (`label_agreement` is not in the export). Item 7's optional slide-21 scale gloss **not added** — dropped first for budget, as §6 directs.

**Slides added: 1 of the 2 permitted** — "What's Actually at Stake" (item 3). The three contested pairs plus the bridging/bonding gloss would not fit at readable size beside the existing hypothesis table. No other slide added, split, reordered or removed.

**Render:** clean. 28 R warnings, identical in count to a render of the pre-edit `HEAD` version — pre-existing svglite font warnings, not introduced here.

**Budget: 20:30 against 18:00 — 2:30 over.** Recovery ladder applied as far as it goes: slide 21's gloss never added; the agreement split is in notes; slide 4 trimmed to a single column (0:50 → 0:35); slide 19's design-based *p*-values moved to notes (0:50 → 0:40). Everything remaining is either a §5 addition MJ asked for or a slide §6.4 forbids cutting, so per §6.5 the overage is reported rather than cut further.
