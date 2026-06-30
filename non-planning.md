# Non-Planning / Chart-Showcase Dataset Reference — Enterprise PBIX Model Builder

> **Read this file in full when Step 0 in SKILL.md determined the dataset is Non-Planning-shaped** —
> i.e. it has no scenario dimension, no account hierarchy, and no financial fact table at
> account × period × org-unit grain. Typically a chart-feature showcase (bubble, scatter,
> quadrant, positioning-map demos) or behavioural/correlation data.
>
> **What this file is, deliberately:** a worked extraction of one real, deployed model —
> the `SAAS_Bubble_Scatter_plot.bim` built for a fictional B2B SaaS company ("Engagely"),
> covering account-engagement scatter/bubble charts and a Gartner-style competitor
> positioning quadrant. It is **not** a generalized pattern library the way `planning.md`
> is — it's a concrete template. Use it as a structural reference for datasets shaped
> similarly (flat or near-flat fact/dimension tables driving specific chart visuals), and
> adapt the specifics (table names, column names, measure names, DAX) to whatever the
> actual dataset in front of you contains. Don't copy the SaaS vocabulary (Account, ARR,
> Churn Risk Tier) onto an unrelated non-planning dataset just because the structure matches.
>
> This file assumes you've already read the universal BIM generation rules, format string
> standards, generation rules summary, and shared output-file list in `SKILL.md`.
>
> © 2026 Logeshkumar Sivakumar. All rights reserved. Contact: elogu2001@outlook.com

---

## Table of Contents

1. [How This Dataset Shape Differs From Planning](#how-this-dataset-shape-differs-from-planning)
2. [Mode 1 — Integrated Intake](#mode-1--integrated-intake-non-planning)
3. [Mode 2 — Standalone Intake](#mode-2--standalone-intake-non-planning)
4. [The Date-Table Branch — Detect Before Building Relationships](#the-date-table-branch--detect-before-building-relationships)
5. [Worked Example — Model Overview](#worked-example--model-overview)
6. [Worked Example — Table Structure](#worked-example--table-structure)
7. [Worked Example — Relationship](#worked-example--relationship)
8. [Worked Example — Measure Architecture](#worked-example--measure-architecture)
9. [Worked Example — Full Measure Reference (All 24 Measures)](#worked-example--full-measure-reference-all-24-measures)
10. [Known Correction Applied to This Template](#known-correction-applied-to-this-template)
11. [BIM Mechanics Confirmed Identical to Planning](#bim-mechanics-confirmed-identical-to-planning)
12. [Adapting This Template to a Different Non-Planning Dataset](#adapting-this-template-to-a-different-non-planning-dataset)

---

## How This Dataset Shape Differs From Planning

Compared to a planning/financial model built from `references/planning.md`, this dataset shape is structurally simpler in every dimension:

| Aspect | Planning shape | Non-planning shape (this template) |
|---|---|---|
| Table count | 10–17 (Dim + Fact tables) | As few as 2 data tables + Measure table |
| Scenario dimension | Always present (`[Dim] Scenario`) | Absent |
| Account hierarchy | Always present (`[Dim] Account`) | Absent |
| Date table | Always present, enriched M expression, ~33 generated columns | Often absent — time field lives directly on the fact table |
| Relationship count | Often 10+ | Can be a single relationship, or none |
| Hierarchies | Common (Account, Geography, Product, Org) | Often none — flat dimension tables |
| Measure folder themes | Variance vs Plan, YTD/QTD, EBITDA-style ratios, Scenario Comparison | Per-chart-page folders: one folder per report page/visual, built around the specific axis/size/category fields that visual needs |
| Core measure pattern | P&L rollups filtered by Scenario | Filter-context-relative statistics (live averages used as quadrant split lines), correlation, concentration/Pareto share |

Do not try to force a Date dimension, scenario filter, or account hierarchy onto a dataset like this just because the planning skill always has them — check what's actually there (see the Date-table branch below) and build only what exists.

---

## Mode 1 — Integrated Intake (Non-Planning)

If a dataset-creator skill produced the CSVs in this conversation (e.g. `demo-dataset-creator`), read whatever table list it actually generated — there is no fixed expected table set the way Planning's Mode 1 expects `DimDate.csv` / `DimAccount.csv` / `FactFinancial.csv`. Read each CSV's header row to get columns, and read any documentation file the dataset-creator skill produced for descriptions, using the same description-parsing approach as Planning Mode 1 (look for `### TableName` blocks with a blockquote description and a column table) — that parsing logic is shape-agnostic and works here unchanged.

**Clarification questions for Mode 1 (ask together, once):**

```
Before I build the model:

1. What report pages / chart visuals is this dataset built to support?
   (e.g. "a scatter plot, a bubble chart, and two quadrant charts")
2. For each chart, which columns map to which visual encoding —
   X axis, Y axis, bubble size, category/legend, color/conditional formatting?
3. Does a separate date/calendar table exist, or does time live as a column
   directly on the fact table? (Check the CSV list first — only ask if unclear.)
```

These replace Planning's Use Case Checklist and Persona Table, which assume P&L-style reporting needs that don't apply here.

---

## Mode 2 — Standalone Intake (Non-Planning)

Same PBIX-parsing and description-file-parsing mechanics as Planning Mode 2 (`parse_pbix()`, `parse_description_doc()`, the naming-convention detector, the description resolution priority order) — none of that is planning-specific, reuse it as-is. The only difference is what you do with the result: don't assume the parsed tables will match a `DimScenario`/`DimAccount` shape, and don't fall back to Planning's built-in description catalogue (`BUILTIN_TABLE_DESCS`/`BUILTIN_COL_DESCS`) for unmatched columns — those entries are written for planning table names and will not match. If no description file is available for a non-planning table/column, fall straight to `auto_describe()` (split camelCase, capitalize) rather than a wrong catalogue hit.

---

## The Date-Table Branch — Detect Before Building Relationships

Check the dataset for a standalone calendar/date table before building anything:

**If a separate `[Dim] Date` table exists** (or equivalent — any table whose sole purpose is calendar enrichment): build it exactly as Planning does, using the enriched M expression in `references/planning.md` under "Date Table — Enriched M Expression" (Rule 7 in SKILL.md). Reuse it verbatim — the 33-column generation logic isn't planning-specific, it's just calendar math.

**If no separate date table exists** (this template's case): the time field lives directly on the fact table as an integer key (e.g. `Month Key`, `YYYYMM` format) paired with a hidden numeric sort-order column so it doesn't sort alphabetically. There is no relationship to build for time at all — `sortByColumn` on the label column is the only special handling needed. See the worked example below for the exact column pair (`Month Key` / `Month Key Sort Order`).

Ask the user directly if it's not obvious from the CSV/table list which case applies — don't assume either way.

---

## Worked Example — Model Overview

Source: `SAAS_Bubble_Scatter_plot.bim`, a model built for "Engagely" (fictional B2B SaaS, customer engagement platform), supporting three report pages: Scatter Plot, Bubble Chart, Account Quadrant, plus a fourth independent page, Vendor Quadrant (competitor positioning).

- **2 data tables** + 1 Measure table
- **1 relationship** (one-directional, fact → dimension)
- **24 measures** across 5 numbered display folders
- **No Date dimension table** — time lives on the fact table as `Month Key`
- **No hierarchies** — both dimension tables are flat
- `compatibilityLevel`: `1600`
- `defaultPowerBIDataSourceVersion`: `PowerBI_V3`
- `discourageImplicitMeasures`: `true`

These three model-level properties match SKILL.md's universal rules exactly — they don't change between planning and non-planning datasets.

---

## Worked Example — Table Structure

### `[Fact] Account Metrics` — account-month grain engagement data

> Account-month grain engagement data for approximately 2,500 customer accounts across 18 months. Drives the Scatter, Bubble, and Account Quadrant report pages. Each row represents one account's usage, sentiment, revenue, and risk position in a single month.

| Column | Type | Hidden | Role |
|---|---|---|---|
| Account ID | int64 | yes | surrogate key |
| Account Name | string | no | data point label |
| Month Key | int64 | yes | time field (YYYY-MM), no separate Date table |
| Industry Vertical | string | no | legend/category for Bubble chart |
| Plan Tier | string | no | slicer; sorted via Plan Tier Sort Order |
| Country | string | no | geo slicer |
| City | string | no | geo slicer |
| Usage Score | double | no | X axis (Scatter, Bubble, Quadrant) |
| NPS | **double*** | no | Y axis (Scatter, Bubble, Quadrant) |
| ARR | double | no | drives bubble size |
| Support Tickets Last 90d | int64 | no | optional supporting measure |
| Incumbent Vendor Key | int64 | yes | FK to `[Dim] Competitor Vendor`; blank = net-new business |
| Churn Risk Tier | string | no | conditional formatting overlay |
| Plan Tier Sort Order | int64 | yes | hidden sort helper |
| Month Key Sort Order | int64 | yes | hidden sort helper |

\* See "Known Correction Applied to This Template" below — the source BIM has this as `string`.

**Partition (M expression) — standard CSV load, same pattern as every non-Date table in Planning:**

```
let
    Source = Csv.Document( File.Contents( DatasetFolder & "\\FactAccountMetrics.csv" ),
        [ Delimiter = ",", Columns = null, Encoding = 65001, QuoteStyle = QuoteStyle.None ] ),
    #"Promoted Headers" = Table.PromoteHeaders( Source, [ PromoteAllScalars = true ] )
in
    #"Promoted Headers"
```

### `[Dim] Competitor Vendor` — sparse competitor positioning table

> Snapshot of approximately 25 competitor vendors (including one fictional self-positioned entry) used to build a sparse, Gartner-style competitive positioning quadrant. Deliberately low row count — this table is built for presentation clarity, not data density.

| Column | Type | Hidden | Role |
|---|---|---|---|
| Vendor ID | int64 | yes | surrogate key |
| Vendor Name | string | no | data point label |
| Vendor Category | string | no | internal grouping (not report-facing — see note below) |
| Is Self | boolean | no | flags the company's own product |
| Completeness Of Vision Score | double | no | X axis (Vendor Quadrant) |
| Ability To Execute Score | double | no | Y axis (Vendor Quadrant) |
| Market Presence Score | double | no | drives bubble size |
| Quadrant Label | string | no | derived category; sorted via Quadrant Label Sort Order |
| Vision Axis Split | **double*** | no | pre-computed mean, repeated every row — X split line |
| Execution Axis Split | **double*** | no | pre-computed mean, repeated every row — Y split line |
| Quadrant Label Sort Order | int64 | yes | hidden sort helper |

\* See "Known Correction Applied to This Template" below.

**Worth noting as a pattern:** `Vendor Category` exists in the data but is explicitly described as an internal/generation-time field, with `Quadrant Label` called out as "the field intended for report-facing categorisation." When a dataset has both a raw/internal version of a categorical field and a cleaned report-facing version, document which one the report should actually use — don't assume the report developer will guess correctly.

---

## Worked Example — Relationship

Only one relationship in the entire model:

```json
{
  "name": "Vendor-to-Account",
  "fromTable": "[Fact] Account Metrics",
  "fromColumn": "Incumbent Vendor Key",
  "toTable": "[Dim] Competitor Vendor",
  "toColumn": "Vendor ID",
  "crossFilteringBehavior": "OneDirection",
  "isActive": true
}
```

Note this follows Rule 3 from SKILL.md exactly — `fromColumn`/`toColumn` use the column's display `name` ("Incumbent Vendor Key", "Vendor ID"), not `sourceColumn`. With only two tables, there's exactly one relationship to get right; don't assume more structure than exists.

---

## Worked Example — Measure Architecture

Display folders here are organized **one per report page/visual**, not by financial theme:

```
01 | Scatter Plot          → Avg Usage Score, Avg NPS, Usage NPS Correlation, Account Count
02 | Bubble Chart           → Total ARR, Avg ARR per Account, Top 18 Pct ARR Share, Industry Vertical Count
03 | Account Quadrant       → Usage Axis Split, NPS Axis Split, Champions/At Risk/Dormant/Growing/Critical High Risk counts
04 | Vendor Quadrant        → Vendor Count, Avg Vision/Execution Score, Leaders Count, win/loss measures
05 | Navigation & Labels    → Latest Data Month, Selected Plan Tier
```

Three reusable **measure patterns** appear here, worth recognizing if a different non-planning dataset has a similar chart type — described concretely as they appear in this model, not abstracted into a generic library per your instruction:

**1. Filter-context-relative axis split** — instead of a fixed constant, the quadrant split line is computed live so it responds to slicers:
```dax
Usage Axis Split = AVERAGE('[Fact] Account Metrics'[Usage Score])
```
Then quadrant membership measures compare each row against that same live average via `CALCULATE` + `DISTINCTCOUNT`:
```dax
Champions Count =
CALCULATE(
    DISTINCTCOUNT('[Fact] Account Metrics'[Account ID]),
    '[Fact] Account Metrics'[Usage Score] >= [Usage Axis Split],
    '[Fact] Account Metrics'[NPS] >= [NPS Axis Split]
)
```

**2. Concentration / Pareto share** — share of total held by the top N% of rows by some value, using `SUMMARIZE` + `TOPN`:
```dax
Top 18 Pct ARR Share =
VAR __totalARR = [Total ARR]
VAR __n = DISTINCTCOUNT('[Fact] Account Metrics'[Account ID])
VAR __topN = ROUNDUP(__n * 0.18, 0)
VAR __topAccounts =
    TOPN(
        __topN,
        SUMMARIZE('[Fact] Account Metrics', '[Fact] Account Metrics'[Account ID], "@arr", [Total ARR]),
        [@arr], DESC
    )
VAR __topSum = SUMX(__topAccounts, [@arr])
RETURN DIVIDE(__topSum, __totalARR)
```

**3. Pearson correlation between two metrics** — full manual implementation since DAX has no built-in correlation function:
```dax
Usage NPS Correlation =
VAR __t = '[Fact] Account Metrics'
RETURN
IF(
    COUNTROWS(__t) > 1,
    CALCULATE(
        DIVIDE(
            SUMX(__t, ('[Fact] Account Metrics'[Usage Score] - [Avg Usage Score]) * ('[Fact] Account Metrics'[NPS] - [Avg NPS])),
            SQRT(SUMX(__t, ('[Fact] Account Metrics'[Usage Score] - [Avg Usage Score]) ^ 2) *
                 SUMX(__t, ('[Fact] Account Metrics'[NPS] - [Avg NPS]) ^ 2))
        )
    )
)
```

All three use `VAR`/`RETURN` (per the universal Generation Rules Summary), all use `DIVIDE()` not `/`, and all have a `formatString` set.

---

## Worked Example — Full Measure Reference (All 24 Measures)

### 01 | Scatter Plot

| Measure | Format | Description |
|---|---|---|
| Avg Usage Score | `0.0` | Average engagement score across filtered accounts. Plot against Avg NPS on a scatter plot. |
| Avg NPS | `0.0` | Average Net Promoter Score across filtered accounts. |
| Usage NPS Correlation | `0.00` | Pearson correlation between Usage Score and NPS for filtered accounts. ~0.5 = moderate trend; ~0 = no relationship; near ±1 = check filter context. |
| Account Count | `#,0` | Distinct accounts in current filter context — sanity check on chart data point count. |

### 02 | Bubble Chart

| Measure | Format | Description |
|---|---|---|
| Total ARR | `$#,0` | Sum of ARR across filtered accounts — drives bubble size. |
| Avg ARR per Account | `$#,0` | Average ARR per account; compare across Industry Vertical or Plan Tier. |
| Top 18 Pct ARR Share | `0.0%` | Share of total ARR held by top 18% of accounts by revenue. Dataset is deliberately right-skewed; expect 55–65% normally. |
| Industry Vertical Count | `#,0` | Distinct verticals — Bubble chart's legend/marker category. |

### 03 | Account Quadrant

| Measure | Format | Description |
|---|---|---|
| Usage Axis Split | `0.0` | Mean Usage Score — vertical quadrant split line, responds to filters. |
| NPS Axis Split | `0.0` | Mean NPS — horizontal quadrant split line. |
| Champions Count | `#,0` | Usage ≥ avg AND NPS ≥ avg — healthiest accounts. |
| At Risk Count | `#,0` | Usage < avg but NPS ≥ avg — satisfied but disengaging, a leading indicator. |
| Dormant Count | `#,0` | Usage < avg AND NPS < avg — lowest-engagement segment. |
| Growing Count | `#,0` | Usage ≥ avg but NPS < avg — engagement ramping faster than sentiment. |
| Critical High Risk Count | `#,0` | Accounts tagged Critical or High churn risk. Use as conditional formatting overlay. |

### 04 | Vendor Quadrant

| Measure | Format | Description |
|---|---|---|
| Vendor Count | `#,0` | Distinct vendors. Table is deliberately sparse — expect low double digits unfiltered. |
| Avg Vision Score | `0.0` | Average Completeness of Vision across filtered vendors; responds to slicers (unlike the static Vision Axis Split column). |
| Avg Execution Score | `0.0` | Average Ability to Execute across filtered vendors. |
| Leaders Count | `#,0` | Vendors classified Leaders (high vision, high execution). |
| Self Vendor ARR Won | `$#,0` | Total ARR of accounts migrated from the selected competitor(s). Use on a drillthrough from a selected vendor bubble. |
| Accounts Won From Vendor | `#,0` | Count of accounts migrated from selected competitor(s). |
| New Business Count | `#,0` | Accounts with no incumbent vendor — net-new vs. migration split. |

### 05 | Navigation & Labels

| Measure | Format | Description |
|---|---|---|
| Latest Data Month | *(none)* | Most recent month in the dataset, YYYY-MM. Use in dashboard subtitles for data freshness. |
| Selected Plan Tier | *(none)* | Selected plan tier name, or "All plan tiers" fallback. Use in page titles. |

Full DAX for every measure above is in the uploaded source BIM; reuse it as the literal reference implementation when a new dataset matches this shape closely, and treat it as a worked example to adapt otherwise.

---

## Known Correction Applied to This Template

The original uploaded BIM has three columns typed `"string"` that are numeric in both behavior and description:

- `[Fact] Account Metrics][NPS]` — averaged in three measures (`Avg NPS`, `Usage NPS Correlation` ×2 uses); described as "Net Promoter Score... from -100 to 100"
- `[Dim] Competitor Vendor][Vision Axis Split]` — described as "the mean Completeness Of Vision Score... same value repeated on every row"
- `[Dim] Competitor Vendor][Execution Axis Split]` — described as "the mean Ability To Execute Score... same value repeated on every row"

A string-typed column used inside `AVERAGE()` or arithmetic would error or behave unexpectedly in Power BI. This template corrects all three to `dataType: "double"`. If you generate a model from the literal source BIM file rather than this template, check for and fix this same class of bug — verify any column used inside a numeric DAX function actually carries a numeric `dataType`, regardless of dataset shape.

---

## BIM Mechanics Confirmed Identical to Planning

Verified directly against this model, confirming SKILL.md's universal rules apply unchanged:

- **Measure table placeholder** — `_` column has all three required properties (`type: "calculatedTableColumn"`, `sourceColumn: "[_]"`, `isNameInferred: true`), exactly per Rule 4.
- **Measure table partition** — `DATATABLE( "_", STRING, {{""}} )`, identical to Planning.
- **DatasetFolder expression** — present in `model.expressions`, `kind: "m"`, with `IsParameterQuery=true` meta annotation, per Rule 9.
- **Model-level properties** — `compatibilityLevel: 1600`, `defaultPowerBIDataSourceVersion: "PowerBI_V3"`, `discourageImplicitMeasures: true` — same as Planning (and confirms the `1600` figure in SKILL.md/Rule references over the stray `1550` previously found elsewhere in this skill).
- **Column annotations** — every column carries `{"name": "SummarizationSetBy", "value": "User"}`, same as Planning tables.
- **sortByColumn usage** — applied the same way: a label column points at a hidden numeric helper column in the same table (`Month Key` → `Month Key Sort Order`, `Plan Tier` → `Plan Tier Sort Order`, `Quadrant Label` → `Quadrant Label Sort Order`).

---

## Adapting This Template to a Different Non-Planning Dataset

When a new non-planning dataset doesn't match this SaaS example's specific vocabulary:

1. Run Step 0 and the Date-table branch above first.
2. Map each chart/report page to its own display folder, following the `01 | [Page Name]` numbering convention.
3. For each chart, identify which columns are axis fields, size field, category/legend field, and conditional-formatting field — ask the user explicitly if it's not obvious (see Mode 1 clarification questions above).
4. Reuse the three measure patterns above (live axis split, concentration share, correlation) wherever the new dataset has an analogous chart type — quadrant chart → axis-split pattern; skewed value distribution → concentration-share pattern; two correlated metrics on a scatter → correlation pattern — but write fresh DAX against the new dataset's actual table/column names. Do not paste this example's DAX with names swapped without checking the new dataset's grain and filter context match.
5. Apply the same BIM mechanics (Measure table placeholder, DatasetFolder expression, sortByColumn, relationship column-name rule) unchanged — those are universal regardless of dataset shape.
6. Generate the same five output files SKILL.md lists, using the documentation templates in `references/planning.md` under "Model Documentation Files" (those templates are shape-agnostic — substitute folders/measures/use-cases as needed).

<!--
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  END OF REFERENCE FILE — NON-PLANNING / CHART-SHOWCASE DATASETS
  Part of Enterprise PBIX Model Builder v3.0
  ─────────────────────────────────────────────────────────────────────────
  © 2026 Logeshkumar Sivakumar. All rights reserved.

  This reference file is the original intellectual property of
  Logeshkumar Sivakumar. It may not be reproduced, redistributed,
  resold, or published in any form without explicit written permission.

  See SKILL.md for universal BIM rules and references/planning.md
  for planning/financial datasets.

  Designed & Developed by : Logeshkumar Sivakumar
  Contact                 : elogu2001@outlook.com
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
-->
