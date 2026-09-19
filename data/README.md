# Data Documentation

This folder documents the public data sources, analytical scope, preparation logic, and key data-quality considerations used in the **Semaglutide Commercial Performance Tracker**.

The raw NHS prescribing files are not stored directly in this repository because of their size.

---

## Data Source

The project uses publicly available prescribing data from the:

**NHS Business Services Authority (NHSBSA) — English Prescribing Dataset**

The dataset records prescribing activity at GP-practice level and includes fields such as:

- Prescribing month
- Chemical substance
- Presentation
- Prescription items
- Quantity
- Total quantity
- Net Ingredient Cost (NIC)
- Actual Cost
- Presentation identifiers
- SNOMED codes

The analysis uses these fields to evaluate semaglutide and the wider defined GLP-1 market.

---

## Analytical Period

The full analytical dataset covers:

> **July 2024 – June 2026**

The primary headline comparison used in the dashboard is:

> **January–June 2026 vs January–June 2025**

This ensures that the year-on-year comparison uses equivalent six-month periods.

Because the dataset begins in July 2024, a full-year 2025 vs full-year 2024 comparison would not represent equivalent periods.

Historical trend visuals therefore use the full available period:

> **July 2024 – June 2026**

---

## Market Definition

For this project, the defined GLP-1 market includes the following five molecules:

- Semaglutide
- Dulaglutide
- Liraglutide
- Exenatide
- Lixisenatide

Semaglutide is treated as the focal product.

A stable market definition is used throughout the analysis to ensure that market size, market share, growth, and competitive comparisons use a consistent denominator.

---

## Analytical Grain

The source data is originally available at GP-practice level.

Because this project does not include geographic analysis, the data was aggregated to:

> **one row per month per presentation**

The final analytical grain is based on:

- YearMonth
- ChemicalSubstanceCode
- Molecule
- PresentationCode
- PresentationName
- SNOMEDCode

The following measures are aggregated using sums:

- Quantity
- Items
- TotalQuantity
- NIC
- ActualCost

This reduces model size while preserving the detail required for:

- Monthly trend analysis
- Molecule comparison
- Market share analysis
- Presentation mix analysis
- Recorded cost analysis

---

## Final Fact Table

The final `Fact_Prescribing` table contains:

- `YearMonth`
- `ChemicalSubstanceCode`
- `Molecule`
- `PresentationCode`
- `PresentationName`
- `Quantity`
- `Items`
- `TotalQuantity`
- `NIC`
- `ActualCost`
- `SNOMEDCode`

---

## Dimension Tables

### `Dim_Calendar`

Contains:

- YearMonth
- Year
- MonthNumber
- MonthName
- Quarter
- MonthYear
- YearMonthSort

The table supports time intelligence and chronological sorting.

---

### `Dim_Chemical`

Contains:

- ChemicalSubstanceCode
- Molecule
- Market
- FocalProductFlag

The `FocalProductFlag` identifies semaglutide for product-specific measures.

---

### `Dim_Presentation`

Contains:

- PresentationCode
- PresentationName
- ChemicalSubstanceCode
- Molecule
- SNOMEDCode
- Formulation
- Strength
- PresentationGroup

Presentation names were simplified into commercially readable groups such as:

- Injection 0.25 mg
- Injection 0.5 mg
- Injection 1 mg
- Tablet 3 mg
- Tablet 7 mg
- Tablet 14 mg

---

## Data Preparation Workflow

The main data-preparation process was completed in Power Query.

The workflow included:

1. Importing monthly prescribing files.
2. Separating source files into schema-compatible groups.
3. Harmonising column names across source schemas.
4. Standardising date fields.
5. Standardising chemical-substance fields.
6. Standardising presentation fields.
7. Correcting numeric data types.
8. Filtering to the defined GLP-1 market.
9. Appending the monthly source groups.
10. Aggregating the data to monthly presentation level.
11. Creating dimension tables.
12. Validating monthly coverage and commercial measures.

---

## Schema Harmonisation

The source files used across the analytical period did not all share an identical schema.

Two source groups were therefore harmonised before append.

Examples of schema variation included differences in fields such as:

- `BNF_CODE`
- `BNF_Presentation_Code`
- `BNF_DESCRIPTION`
- `BNF_Presentation_Name`
- `BNF_CHEMICAL_SUBSTANCE`
- `BNF_Chemical_Substance_Code`
- `YEAR_MONTH`
- `year_month`

These fields were standardised into a consistent analytical structure before combining the datasets.

---

## YearMonth Transformation

Source month fields stored in `YYYYMM` format were converted into proper date values representing the first day of each month.

Example Power Query logic:

```powerquery
#date(
    Number.FromText(Text.Start(Text.From([YearMonth]), 4)),
    Number.FromText(Text.End(Text.From([YearMonth]), 2)),
    1
)
```

This enabled:

- chronological sorting;
- year-on-year comparisons;
- monthly trend analysis; and
- Power BI time intelligence.

---

## Cost Field Data-Type Correction

An important data-quality issue was identified during development.

`NIC` and `Actual Cost` were initially converted to whole-number types during an early transformation step.

This caused decimal precision to be lost.

The issue was corrected at source level by parsing the fields as decimal numbers before append and aggregation.

The correct transformation approach used decimal numeric types and the appropriate locale.

Conceptually:

```powerquery
{"NIC", type number},
{"ACTUAL_COST", type number}
```

with locale handling applied during type conversion.

Fixed divisors were not used because the underlying cost values did not consistently contain the same number of decimal places.

---

## Data Quality Issue: Incomplete Monthly Coverage

The first reused prescribing fact table was found to contain incomplete monthly coverage for the required analytical period.

This was identified during quality assurance.

The dataset was therefore rebuilt directly from the underlying source files.

The final pipeline:

- re-harmonised the two source schema groups;
- appended the complete monthly series;
- revalidated cost fields; and
- rechecked the analytical period.

This ensured that the final dataset contained complete monthly coverage from:

> **July 2024 to June 2026**

---

## Quality Assurance

The rebuilt dataset was validated using several checks.

These included:

- Expected monthly coverage
- Prescription-item totals
- NIC totals
- Actual Cost totals
- NIC per Item
- Previous-year comparisons
- Presentation-level uniqueness
- Dimension-table uniqueness
- Relationship validation
- Market-share denominator checks

Example overall QA totals after rebuilding the pipeline:

| Period | Prescription Items | NIC | Actual Cost |
|---|---:|---:|---:|
| Jul–Dec 2024 | 1,214,843 | £108.26M | £107.74M |
| 2025 | 2,171,044 | £209.30M | £209.34M |
| Jan–Jun 2026 | 1,058,592 | £89.46M | £89.01M |
| Jul 2024–Jun 2026 | 4,444,479 | £386.02M | £386.09M |

These checks were used to confirm that the rebuilt data pipeline produced commercially plausible results.

---

## Primary Commercial Volume Measure

The primary volume measure used in the project is:

> **Prescription Items**

`Quantity` is not used as the primary cross-product volume metric because quantity units may differ between formulations and presentations.

Prescription Items therefore provide a more consistent basis for comparing molecules and presentations.

---

## Important Interpretation Notes

### Prescription Items Are Not Patients

Prescription items represent prescribing activity.

They should not be interpreted as unique patients.

One patient may generate multiple prescription items.

---

### NIC Is Not Revenue

Net Ingredient Cost is a recorded prescribing-cost measure.

It should not be interpreted as:

- Manufacturer revenue
- Net sales
- Profit
- Realised selling price
- Average selling price

---

### NIC per Item Is Not Product Price

`NIC per Item` is calculated as:

> Recorded NIC / Prescription Items

It is used as a diagnostic measure to compare recorded cost with prescribing volume.

It should not be interpreted as the manufacturer's commercial selling price.

---

### Presentation Mix Is Not Patient Switching

Changes in presentation mix describe changes in the proportion of prescription items across presentations.

They should not be interpreted as direct patient-level switching between formulations or strengths.

The source data is aggregated and does not contain longitudinal patient-level information.

---

## Data Limitations

The dataset and analytical design have several limitations.

### England Scope

The source data reflects prescribing activity captured within the English NHS prescribing dataset.

The results should not be interpreted as total UK pharmaceutical consumption.

---

### Clinical Indication

The prescribing data does not necessarily identify the clinical indication associated with every semaglutide prescription.

The project therefore analyses overall semaglutide prescribing activity rather than indication-specific performance.

---

### No Sales Data

The dataset does not contain:

- Manufacturer sales
- Commercial revenue
- Rebates
- Discounts
- Net realised price
- Profitability

The project is therefore a prescribing-performance analysis rather than a manufacturer financial-performance analysis.

---

### Market Definition

The project uses a defined five-molecule GLP-1 market.

All market share and market growth measures should be interpreted within this analytical definition.

---

### Historical Coverage

The available dataset begins in July 2024.

This means that full-year 2025 vs full-year 2024 comparisons would not be equivalent.

The primary headline comparison is therefore based on:

> **Jan–Jun 2026 vs Jan–Jun 2025**

---

## Why Raw Data Is Not Included

The original NHS prescribing files are large and are not stored in this repository.

This repository instead provides:

- documentation of the source;
- analytical scope;
- transformation logic;
- QA approach; and
- final Power BI analysis.

Users who wish to reproduce the project should download the relevant monthly prescribing files directly from the official NHSBSA public data source.

---

## Reproducibility Notes

To reproduce the analysis:

1. Download the relevant monthly NHSBSA prescribing files covering Jul 2024–Jun 2026.
2. Harmonise source schemas where required.
3. Convert month fields into proper dates.
4. Ensure NIC and Actual Cost are imported as decimal numeric fields.
5. Filter to the defined five-molecule GLP-1 market.
6. Aggregate practice-level records to monthly presentation level.
7. Build the calendar, chemical, and presentation dimensions.
8. Create the DAX measures documented in the main project README.
9. Validate monthly coverage and headline totals before building visuals.

---

## Disclaimer

The data used in this project is publicly available NHS prescribing data.

This analysis was developed independently for educational and portfolio purposes.

It is not affiliated with, endorsed by, or produced on behalf of the NHS, any pharmaceutical manufacturer, or any other commercial organisation.

The analysis should not be interpreted as clinical, financial, investment, or commercial advice.
