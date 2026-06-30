# PBIX Model Builder

> A Claude AI skill that builds fully modelled Power BI semantic models for enterprise planning datasets.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Built for Claude](https://img.shields.io/badge/Built%20for-Claude%20AI-orange.svg)](https://claude.ai)
[![Maintainer](https://img.shields.io/badge/Maintainer-Logeshkumar%20Sivakumar-green.svg)](https://www.linkedin.com/in/logeshkumar-sivakumar/)

**© 2026 Logeshkumar Sivakumar. All rights reserved.**

---
 
## What this skill does
 
Given a dataset — whether produced earlier in the same conversation by a dataset-creator skill, or uploaded as an existing `.pbix` file with optional documentation — this skill produces:
 
| Output file | Purpose |
|---|---|
| `model.bim` | The semantic model itself: tables, columns, relationships, hierarchies, measures, descriptions |
| `build_model.py` | A Python script that generates everything; rerunnable any time the schema changes |
| `[name]_model_guide_public.md` | Plain-English guide for report developers — what's in the model, what to build |
| `[name]_model_guide_internal.md` | Full technical reference — private, for the model author |
| `measures_reference.md` | Quick lookup of every measure: folder, format, description, full DAX |
 
All five files are generated together in one run, every time — never partially, never on request only.
 
---
 
## Why it's split into three files
 
As of v3.0, the skill is no longer one monolithic file. It's split because the skill handles **two structurally different kinds of dataset**, and forcing both through one set of assumptions was producing a poor fit for one of them.
 
```
enterprise-pbix-model-builder/
├── SKILL.md                     ← always read first (≈330 lines)
└── references/
    ├── planning.md               ← read only for planning/financial datasets (≈5,360 lines)
    └── non-planning.md           ← read only for chart-showcase/behavioural datasets (≈371 lines)
```
 
**`SKILL.md`** contains everything genuinely universal regardless of dataset shape: the dataset-shape decision step, input-mode detection, the 9 critical BIM generation rules, format string standards, the generation rules summary, the attribution requirements, and the shared output-file list. It is intentionally thin — a router, not a rulebook.
 
**`references/planning.md`** is for datasets with a scenario dimension (Actual/Plan/Forecast), an account hierarchy, and a financial fact table feeding a P&L — Variance Analysis, YTD/QTD, EBITDA-style ratios, Scenario Comparison. It contains the full Mode 1/Mode 2 intake logic for that shape, the relationship map, the complete DAX measure architecture (including per-industry folder patterns like B2B SaaS billing metrics, Retail, Manufacturing), and a verbatim table/column description catalogue for the standard 13+ planning tables.
 
**`references/non-planning.md`** is for datasets with no scenario or account structure — chart-feature showcases, behavioural/correlation data, competitive-positioning quadrants. It is deliberately *not* a generalized pattern library; it's a concrete worked template extracted from a real deployed model (a B2B SaaS bubble/scatter/quadrant demo), covering its exact table structure, relationship, and all 24 measures, with guidance on adapting that structure to a different non-planning dataset.
 
This way, a non-planning dataset never gets pointed at planning-specific assumptions (a hardcoded relationship map built around `FactFinancial`/`FactPlan`/`FactPayroll`, or "universal" base measures that secretly require `[Dim] Scenario` to exist), and a planning dataset still gets the full depth of catalogue and pattern coverage it needs.
 
---
 
## How it decides which path to take
 
Every run starts the same way, regardless of dataset shape:
 
1. **Step 0 — Determine Dataset Shape.** Before asking any other question, the skill checks (or asks) whether the dataset is planning-shaped or non-planning-shaped. This is based on what's actually present — `DimScenario`/`DimAccount`/`FactFinancial` (or equivalents) signal planning; a small number of flat tables built to drive specific chart visuals signal non-planning. If it's not obvious, it asks directly rather than guessing.
2. **Load the matching reference file** — `references/planning.md` or `references/non-planning.md` — in full, before generating anything.
3. **Input Mode Detection.** Independently of dataset shape, the skill also detects *where the schema comes from*: Mode 1 (Integrated, downstream of a dataset-creator skill already run in this conversation) or Mode 2 (Standalone, from an uploaded `.pbix` and/or description markdown).
4. **Generate.** Once both the shape and the mode are known, the relevant reference file's intake questions, relationship logic, and measure design proceed, followed by the nine universal BIM rules from `SKILL.md` for the actual file generation.
Dataset shape and input mode are independent — a non-planning dataset can come from either mode, and so can a planning one.
 
---
 
## The 9 critical BIM generation rules
 
These live in `SKILL.md` and apply to every model this skill produces, regardless of shape. They cover the most common points where a generated BIM fails to deploy or deploys incorrectly:
 
1. Data types must be inferred from column name patterns, never from a 0-row pandas read
2. Hierarchies are created only where a real drill-path exists
3. Relationship columns use the column's display `name`, never `sourceColumn`
4. The Measure table's hidden `_` placeholder column needs three specific properties
5. Measure names must be unique across all display folders
6. `sortByColumn` is applied to every label column that has a paired numeric/ordinal column
7. A standalone Date table, if one exists, must use the enriched M expression (33 generated columns) — datasets with no separate Date table skip this entirely
8. Measures live only in a dedicated `Measure` table, never inside fact or dimension tables
9. A `DatasetFolder` M parameter must exist in `model.expressions`, or no partition can resolve its CSV path

---
## Related Skills

* [**Enterprise Planning Dataset Creator**](https://github.com/logeshkumar/enterprise-planning-dataset-creator) - generates the dataset that feeds into this skill (Mode 1)

---

## Changelog

| Version | Date | Key changes |
|---|---|---|
| 1.0.0 | May 2026 | Initial public release; production-ready model builder with dynamic model.bim generation, measure architecture, object descriptions, input mode detection, documentation outputs, dynamic industry support, and major BIM/measure table compatibility fixes. |
| 1.1.0 | May 2026 | BIM structure corrections; compatibility level raised to 1600; defaultPowerBIDataSourceVersion and discourageImplicitMeasures moved inside the model object; hidden measure table anchor column renamed from _Measures to _. |
| 1.2.0 | June 2026 | Production issue fixes; added Critical BIM Generation Rules; fixed relationship sourceColumn usage, calculated measure table column properties, duplicate measure validation, data type inference guidance, mandatory hierarchies, and expanded troubleshooting and README rules. |
| 1.3.0 | June 2026 | Added new Rules dor modelling, fixed versions, updated date table Logic |
---

## Attribution and Copyright

```
PBIX Model Builder
Designed and Developed by: Logeshkumar Sivakumar
Contact: elogu2001@outlook.com

© 2026 Logeshkumar Sivakumar. All rights reserved.

This skill file, including the DAX architecture, measure design patterns, display folder structure, 
relationship mapping logic, and documentation templates, is the original intellectual property of Logeshkumar Sivakumar. 
Unauthorised reproduction or redistribution is prohibited.
```

Every Python script and documentation file generated by this skill includes the above
attribution block automatically.

---

## License

See LICENSE file for full terms.

Copyright (c) 2026 Logesh Sivakumar. All rights reserved.
