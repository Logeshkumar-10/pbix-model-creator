# PBIX Model Builder

> A Claude AI skill that builds fully modelled Power BI semantic models for enterprise planning datasets.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Built for Claude](https://img.shields.io/badge/Built%20for-Claude%20AI-orange.svg)](https://claude.ai)
[![Maintainer](https://img.shields.io/badge/Maintainer-Logeshkumar%20Sivakumar-green.svg)](https://www.linkedin.com/in/logeshkumar-sivakumar/)

**© 2026 Logeshkumar Sivakumar. All rights reserved.**

---

## What It Does
 
This skill turns a raw enterprise planning dataset - or an existing PBIX file - into a deployment-ready Power BI semantic model in a single session.
 
It generates a `model.bim` file (Tabular Editor compatible) containing:
 
- **Renamed tables** with `[Dim]` / `[Fact]` prefix convention
- **Renamed columns** with camelCase split, FK columns hidden
- **All relationships** wired - 30 to 70+ depending on the dataset
- **Industry-native DAX measures** in numbered display folders
- **Descriptions** on every table, column, and measure
- **Two documentation files** - a public model guide and an internal technical reference
The measures, folder names, and DAX formulas are **fully dynamic** - driven by the actual industry, real `DimScenario` values, `DimAccount` structure, and operational column names read from the dataset. No hardcoded strings. No generic `KPI & Ratios` folders.
 
---
 
## Two Input Modes
 
| Mode | When to Use | What It Reads |
|------|------------|---------------|
| **Mode 1 - Integrated** | Immediately after running the Enterprise Dataset Creator skill | CSV files + documentation guides. Carries industry, company, and column names forward automatically. |
| **Mode 2 - Standalone** | When uploading an existing PBIX or description document | Parses the PBIX `DiagramLayout` for table names. Resolves descriptions from uploaded docs or the built-in catalogue. |
 
---
 
## Output Files (All Generated in One Run)
 
| File | Purpose |
|------|---------|
| `[industry]_model.bim` | The semantic model - open in Tabular Editor, deploy to Power BI Desktop |
| `build_model.py` | The Python script that generated the BIM - rerunnable |
| `[industry]_model_guide_public.md` | Report developer guide - measure descriptions, report page blueprints |
| `[industry]_model_guide_internal.md` | Technical reference - full DAX, relationship map, column visibility, deployment notes |
| `measures_reference.md` | All measures by folder - name, format, description, DAX formula |
 
---
 
## Industries - Fully Dynamic
 
The skill does not have a fixed list of supported industries. It works for **any industry** by researching the industry's standard terminology, KPI vocabulary, and account structure at generation time.
 
The industry name you provide drives everything - folder names, measure names, DAX formulas, account type filters, operational KPI columns, and the industry-specific report page blueprint. Two models built for different industries share no measure names or folder names. A Pharmaceutical model and a Hospitality model have nothing in common except the BIM structure.
 
**How industry affects the model:**
 
| Element | How It Changes by Industry |
|---------|--------------------------|
| Display folder names | Derived from the industry's reporting domains - e.g., `Revenue by Therapy Area` for Pharma, `Room Revenue & Rate` for Hospitality, `Subscription Revenue` for SaaS |
| Measure names | Use the industry's own vocabulary - `RevPAR` not `Revenue per Unit`, `NIM %` not `Margin %`, `OEE %` not `Utilisation %` |
| DAX filters | Read actual `AccountType` values from `DimAccount.csv` - never assume `"Revenue"` or `"COGS"` |
| Scenario names | Read actual values from `DimScenario.csv` - work with `"Budget"` just as well as `"Plan"` |
| Operational KPIs | Read column names from the Operational fact table header - column names vary by industry |
| Report page blueprints | The public model guide's Section 4 generates pages specific to the industry's reporting cadence |
 
**Measure design is also dynamic by persona.** The same industry generates different measure naming styles depending on the selected end user:
 
| Persona | Naming Style | Focus |
|---------|-------------|-------|
| CFO / CEO | Concise - `EBITDA`, `Net Revenue`, `Margin %` | High-level P&L and variance |
| FP&A Analyst | Descriptive - `YTD Net Revenue vs Plan`, `FYE vs Full Year Plan %` | Full variance stack and forecasting |
| BI Developer | Technical - explicit table references, grain labels | Model coverage and edge cases |
| Sales Leader | Commercial - `Pipeline Revenue`, `Win Rate %`, `ASP` | Revenue and commercial performance |
| Operations Manager | Operational - `OEE %`, `Cost per Unit`, `Utilisation %` | Throughput and cost efficiency |
| Investor / Board | Finance-standard - `EBITDA`, `ROIC`, `FCF`, `EPS` | Profitability and capital efficiency |
 
---
 
## Prerequisites
 
| Tool | Purpose | Cost |
|------|---------|------|
| [Claude](https://claude.ai) | The AI platform where the skill runs | Subscription required |
| [Tabular Editor 2](https://tabulareditor.com) | Opens `model.bim` and deploys to Power BI Desktop | Free |
| [Power BI Desktop](https://powerbi.microsoft.com) | The report layer | Free |
| Python 3.10+ | Runs `build_model.py` to generate the BIM | Free |
 
---
 
## Installation
 
This skill uses the Claude Skills system. To install it:
 
1. Rename the downloaded file to `SKILL.md` and copy it to your Claude skills directory:
```
/mnt/skills/user/enterprise-pbix-model-builder/SKILL.md
```
 
2. The skill is auto-detected when Claude starts. No restart required.
3. Trigger it in any conversation with one of the phrases below:
```
create PBIX model
build semantic model
add DAX measures
set up relationships
create BIM file
build Power BI model
model my uploaded PBIX
```
 
---
 
## How to Use
 
### Mode 1 - After Running Enterprise Dataset Creator
 
If you have just generated a dataset with the [Enterprise Dataset Creator](https://github.com/logeshkumar/enterprise-planning-dataset-creator) skill in the same Claude session, simply invoke this skill:
 
```
Build the Power BI model from the generated dataset.
Use cases: Variance Analysis, YTD, Forecasting, Therapy Area Ranking.
Primary user: FP&A Analyst.
```
 
The skill detects the generated files automatically and asks only two questions: use cases and end user persona.
 
### Mode 2 - From an Existing PBIX
 
Upload your `.pbix` file (and optionally a description document) and invoke the skill:
 
```
Model my uploaded PBIX.
Industry: Branded Pharmaceutical.
Use cases: Variance, YTD, Forecasting, Scenario Comparison.
Primary user: CFO.
```
 
---
 
## Deploying the Generated BIM to Power BI Desktop
 
```
1. Run build_model.py  →  model.bim is generated
2. Open model.bim in Tabular Editor 2
3. Edit the DatasetFolder expression - update the CSV folder path
4. Press Ctrl+S  (mandatory before deploying)
5. Model → Deploy to Power BI Desktop
6. Select your running Power BI Desktop instance → OK
7. Switch to Power BI Desktop → Home → Refresh
8. Fields pane → Measure table → verify display folders
```
 
### DatasetFolder Path Format
 
In the BIM file, the path must use double backslashes:
 
```json
"expression": "\"D:\\\\Projects\\\\Dataset\" meta [IsParameterQuery=true, Type=\"Text\", IsParameterQueryRequired=true]"
```
 
---
 
## BIM Compatibility
 
| Property | Value | Location in BIM |
|----------|-------|----------------|
| `compatibilityLevel` | `1600` | Root level |
| `defaultPowerBIDataSourceVersion` | `PowerBI_V3` | Inside `model` object |
| `discourageImplicitMeasures` | `true` | Inside `model` object |
| Measure table anchor column | `_` (hidden, calculated) | `DATATABLE( "_", STRING, {{""}} )` |
| Hierarchy naming convention | `[H] ` prefix on all hierarchies | `[H] Date`, `[H] Geography`, `[H] Account Hierarchy` |
| Sort behaviour | `sortByColumn` applied to label columns | Months → Month Number, Quarter Label → Quarter, Account Name → Sort Order, Level N Name → Level N Key |
| Relationship column references | `fromColumn` / `toColumn` use `sourceColumn` value | `"AccountKey"`, `"DateKey"` - never `"Account Key"` or `"Date Key"` |
| Measure table placeholder | Calculated table column with three required properties | `type: "calculatedTableColumn"`, `sourceColumn: "[_]"`, `isNameInferred: true` |
| Duplicate measure check | Counter-based validation before BIM is written | Raises `ValueError` with offending measure names and folders if any duplicates found |
| `linguisticMetadata` / `cultures` | Not included | - |
| Tabular Editor 2 | Fully supported | - |
| Tabular Editor 3 | Fully supported | - |
| Power BI Pro / Premium / Fabric | Compatible | - |
 
---
 
## Design Principles
 
- **Industry-native, not generic** - folder names and measure names use the terminology of the specific industry
- **Reads data before writing code** - reads actual scenario names, account types, and column names from CSVs before generating any DAX
- **Descriptions on everything** - every table, column, and measure carries a `description` property in the BIM
- **Correct data type inference** - column types are resolved from name patterns (`infer_data_type()`), not pandas dtype inference. Keys → `int64`, dates → `dateTime`, amounts/rates → `double`, flag columns → `boolean`. Avoids the pandas default of typing all columns as `string` when reading zero-row CSV headers
- **`[H]` prefixed hierarchies** - dimension tables (Account, Product, Geography, Date, Business Unit, Department, Customer, Supplier, Asset, Employee) automatically get named hierarchies. All hierarchy names start with `[H] ` so they are immediately identifiable in the Power BI field list
- **Automatic `sortByColumn`** - label columns (Month Short Name, Quarter Label, Account Name in P&L order, hierarchy level names) get a `sortByColumn` property in the BIM so they sort correctly out of the box. No manual sort configuration in Power BI Desktop. Works for any dataset - known tables use the built-in `SORT_RULES` dict, unknown tables fall back to pattern-based detection (`X Label` → `X Number`, `X Short Name` → `X Number`)
- **`PBI_FormatHint` on all measures** - ensures Power BI Desktop respects the `formatString` and does not auto-format
- **Source-column-aware relationships** - BIM `fromColumn` / `toColumn` use `sourceColumn` (camelCase, matches CSV header) not display name. Prevents the silent relationship-wiring failure where the model loads but Model view shows no relationship lines
- **Strict measure name uniqueness** - `add_measure_annotations()` runs a `Counter`-based duplicate check before any BIM is written. Halts with a clear error listing the duplicate name and the folders it appears in. Prevents the cryptic Tabular Editor `Item 'XXX' already exists in the collection` deploy failure
- **Calculated-table column compliance** - the Measure table's hidden `_` placeholder column carries `type: "calculatedTableColumn"`, `sourceColumn: "[_]"`, and `isNameInferred: true`. All three are mandatory; without them Tabular Editor refuses to deploy
- **No reconciliation measures** - every measure answers a business question
- **No `linguisticMetadata`** - removed to prevent compatibility issues
---
 
## Example - What Dynamic Output Looks Like (Pharma)
 
The folders, measure names, and DAX below are generated specifically for a Branded Pharmaceutical company. Running the same skill against a Retail, SaaS, or Banking dataset produces entirely different folders and measures - no overlap.
 
Running against the VivaNova Pharmaceuticals dataset produces:
 
```
Tables        : 27  (13 dims + 13 facts + 1 Measure table)
Relationships : 71
Measures      : 86  across 13 pharma-native display folders
Visible cols  : 314
Hidden cols   : 145  (FK + hierarchy path columns)
```
 
Display folders generated:
 
```
00 | Base Measures
01 | Revenue by Therapy Area
02 | Gross-to-Net Revenue
03 | Commercial Margin
04 | R&D & Clinical Trial
05 | Patent Cliff & LOE Risk
06 | Operating Cost
07 | Variance vs Plan
08 | Period Trends & YTD
09 | Forecasting
10 | Scenario Comparison
11 | Therapy Area & Product Ranking
12 | Navigation & Labels
```
 
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
