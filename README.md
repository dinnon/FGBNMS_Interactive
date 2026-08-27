# FGBNMS CPCe Data Pipeline and Interactive Viewer

This repository converts Flower Garden Banks National Marine Sanctuary CPCe
summary files into traceable tidy tables, analytical summary products, static
figures, and a browser-based interactive viewer.

The workflow is intentionally conservative:

- Raw files are never edited by the pipeline.
- Missing observations are not imputed or interpolated.
- Source labels and source-file information are retained in the tidy tables.
- Validation must pass before derived products or plots are generated.
- New years can be added without changing the Python code when they follow the
  supported CPCe summary layout and filename conventions.

## Quick start

This section is for users who only need to run the established workflow. See
the full guide below for installation details, new-year preparation, validation
rules, configuration, and troubleshooting.

### Install the Python libraries

Use Python 3.12 and the Python environment or application of your choice.
Install these libraries in that environment:

- pandas
- NumPy
- Matplotlib
- PyYAML
- openpyxl
- xlrd

If your environment accepts `pip` commands, all six can be installed with:

```bash
python -m pip install "pandas>=2.2,<3.0" "numpy>=2.0,<3.0" "matplotlib>=3.8,<4.0" "PyYAML>=6.0,<7.0" "openpyxl>=3.1,<4.0" "xlrd>=2.0,<3.0"
```

### Run these Python files in order

In your preferred Python application, open each linked file below and use its
**Run**, **Execute**, or equivalent command. Wait for each file to finish before
opening the next one.

| Step | Click and run | What it creates |
|---:|---|---|
| 1 | [`01_ingest_cpce_outyear.py`](CPCE_Data_Pipeline/scripts/01_ingest_cpce_outyear.py) | Standard long-format tables in `03_tidy/` |
| 2 | [`01b_ingest_appendix_random_transects_2004_2008.py`](CPCE_Data_Pipeline/scripts/01b_ingest_appendix_random_transects_2004_2008.py) | Historical appendix tables for 2004–2008 |
| 3 | [`02_validate.py`](CPCE_Data_Pipeline/scripts/02_validate.py) | `04_products/validation_report.txt` |
| 4 | [`03_derive_metrics_with_crosswalk.py`](CPCE_Data_Pipeline/scripts/03_derive_metrics_with_crosswalk.py) | Three figure-ready summary CSVs in `04_products/` |
| 5 | [`04_plot_reports_fig1.py`](CPCE_Data_Pipeline/scripts/04_plot_reports_fig1.py) | Annual benthic-cover figures in `05_outputs/figures/` |
| 6 | [`04_plot_reports_fig2.py`](CPCE_Data_Pipeline/scripts/04_plot_reports_fig2.py) | Annual coral-species and CCA figures |
| 7 | [`04_plot_reports_fig3.py`](CPCE_Data_Pipeline/scripts/04_plot_reports_fig3.py) | The long-term benthic time-series figure |

After Step 3, open
[`validation_report.txt`](CPCE_Data_Pipeline/04_products/validation_report.txt).
Continue only when it says:

```text
[PASS] No critical validation errors were found.
```

### Where to find the results

| Folder | Plain-language description |
|---|---|
| `03_tidy/` | Clean, detailed data retaining the source and transect information |
| `04_products/` | Validation results and summary CSVs used by the plots and viewer |
| `05_outputs/figures/` | Finished static PNG figures |
| `Interactive/CPCe_Web/` | Browser-based interactive viewer |

### Refresh the interactive viewer

Open `Interactive/CPCe_Web/index.html`. If the page displays its manual CSV
selector, select all six `figure*.csv` files from
`CPCE_Data_Pipeline/04_products/`. The viewer will load those files directly.

That is the complete routine workflow. The sections below explain each part in
more detail.

## Contents

- [1. Repository structure](#1-repository-structure)
- [2. Software requirements](#2-software-requirements)
- [3. First-time setup](#3-first-time-setup)
- [4. Preparing input data](#4-preparing-input-data)
- [5. Running the pipeline](#5-running-the-pipeline)
- [6. Understanding the generated files](#6-understanding-the-generated-files)
- [7. Updating and opening the interactive viewer](#7-updating-and-opening-the-interactive-viewer)
- [8. Annual update checklist](#8-annual-update-checklist)
- [9. Configuration files](#9-configuration-files)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Data-handling and provenance notes](#11-data-handling-and-provenance-notes)

## 1. Repository structure

The repository has two main components:

```text
.
├── CPCE_Data_Pipeline/
│   ├── 01_raw/
│   ├── 02_intermediate/
│   ├── 03_tidy/
│   ├── 04_products/
│   ├── 05_outputs/
│   ├── config/
│   └── scripts/
├── Interactive/
│   └── CPCe_Web/
└── README.md
```

| Location | Purpose | Treatment |
|---|---|---|
| `01_raw/` | Original CPCe workbooks and CSV exports | Source data; do not edit |
| `02_intermediate/` | Optional temporary or diagnostic files | Not used by the normal run |
| `03_tidy/` | Long-format, source-faithful tables | Canonical machine-readable layer |
| `04_products/` | Validation report and derived summaries | Regenerable analytical products |
| `05_outputs/` | Static figures and presentation-ready files | Regenerable visual outputs |
| `config/` | Crosswalks, colors, paths, and figure settings | Review changes carefully |
| `scripts/` | Numbered Python processing steps | Run in the documented order |
| `Interactive/CPCe_Web/` | HTML viewer and its CSV snapshots | Browser-facing visualization layer |

The former desktop GUI is not required. Each numbered Python script is run
directly, one at a time.

## 2. Software requirements

### Required software

- Python 3.12 is recommended.
- Any Python-capable application, development environment, or command line.
- A current web browser for the interactive viewer.
- Git is optional but recommended when maintaining the project in GitHub.

### Python packages

| Package | Use |
|---|---|
| `pandas` | Reading, reshaping, validating, and writing tabular data |
| `numpy` | Numerical calculations |
| `matplotlib` | Static PNG and optional SVG figures |
| `PyYAML` | Reading pipeline and figure configuration |
| `openpyxl` | Reading `.xlsx` CPCe workbooks |
| `xlrd` | Reading the historical `.xls` appendix workbook |

The tested environment used Python 3.12 with pandas 2.2.3, NumPy 2.3.5,
Matplotlib 3.10.8, PyYAML 6.0.3, openpyxl 3.1.5, and xlrd 2.0.1.

The interactive page uses the open-source Plotly JavaScript library from a
content delivery network. A Python Plotly package is not required, but the
viewer needs internet access when it first loads Plotly.

## 3. First-time setup

### 3.1 Download or clone the repository

Download the repository as a ZIP and extract it, or clone it with Git. Keep the
folder structure intact so `CPCE_Data_Pipeline/` and `Interactive/` remain next
to each other.

Open the repository in the Python application or environment you intend to use.

### 3.2 Select a Python environment

A dedicated environment is recommended because it keeps the project libraries
separate from other Python work. Use any environment manager supported by your
Python application. A standard Python virtual environment can be created with:

```bash
python -m venv .venv
```

Select or activate that environment using the normal controls for your Python
application. An existing managed environment is also acceptable.

### 3.3 Install dependencies

```bash
python -m pip install --upgrade pip
python -m pip install "pandas>=2.2,<3.0" "numpy>=2.0,<3.0" "matplotlib>=3.8,<4.0" "PyYAML>=6.0,<7.0" "openpyxl>=3.1,<4.0" "xlrd>=2.0,<3.0"
```

Confirm that the imports are available:

```bash
python -c "import pandas, numpy, matplotlib, yaml, openpyxl, xlrd; print('Dependencies OK')"
```

Expected result:

```text
Dependencies OK
```

### 3.4 Choose how to run the scripts

Any application that can execute a normal `.py` file can be used. Open the
numbered files from `CPCE_Data_Pipeline/scripts/` and use the application's
**Run** or **Execute** command. Command-line users can run the same files with
`python path/to/script.py`.

The scripts determine the project path from their own locations, so the data
folders are resolved consistently regardless of the application used.

## 4. Preparing input data

### 4.1 Preserve the source files

Place new CPCe summary files under `CPCE_Data_Pipeline/01_raw/`. Subfolders are
allowed. Do not overwrite or manually clean the original source data; corrections
should be made through explicit, reviewed code or configuration changes.

Supported formats are:

- `.xlsx`
- `.xls`
- `.csv`

### 4.2 Required CPCe table structure

Standard inputs must contain one exact `TRANSECT NAME` header in the first cell
of the applicable summary-table row. For Excel files, the script first looks for
a worksheet named `Data Summary`. If that sheet is absent, it checks the other
worksheets for the same exact header.

The strict header check is intentional. It prevents a populated data row from
being mistaken for the header and stops the pipeline when the table structure is
ambiguous.

### 4.3 Filename conventions for new years

The standard ingest reads the year, bank, and survey scope from the filename and
folder path.

Include all three pieces of information in new filenames:

```text
2026_EFGB_StudySite_Random_Transects.xlsx
2026_WFGB_StudySite_Random_Transects.xlsx
2026_EFGB_Reefwide_RT_CPCe_Analysis.xlsx
2026_WFGB_Reefwide_RT_CPCe_Analysis.xlsx
```

- Use `EFGB` or `WFGB` for the bank.
- Include `StudySite` or `Study Site` for study-site surveys.
- Include `Reefwide` or use a clearly named reef-wide subfolder for reef-wide
  surveys.
- Include the four-digit year.

Files without a recognized scope are recorded as `UNKNOWN`. Historical unknown
scope files are retained for the long-term study-site products, but new files
should always identify their scope explicitly.

### 4.4 Historical appendix

The file below is handled by its own parser and is deliberately excluded from
the standard ingest:

```text
CPCE_Data_Pipeline/01_raw/Appendix 1 Random Transect Data 2004-2008.xls
```

Keep that filename unchanged unless the corresponding constant in the appendix
script is deliberately updated and tested.

## 5. Running the pipeline

Run the scripts in this exact order. A later step assumes that the earlier files
have already been created successfully.

| Order | Script | Main result |
|---:|---|---|
| 1 | `01_ingest_cpce_outyear.py` | Standard tidy tables |
| 2 | `01b_ingest_appendix_random_transects_2004_2008.py` | Historical appendix tables |
| 3 | `02_validate.py` | QA/QC report and pass/fail gate |
| 4 | `03_derive_metrics_with_crosswalk.py` | Figure-ready summary products |
| 5 | `04_plot_reports_fig1.py` | Annual benthic-cover figures |
| 6 | `04_plot_reports_fig2.py` | Annual coral-species and CCA figures |
| 7 | `04_plot_reports_fig3.py` | Long-term benthic time-series figure |

### Step 1: Ingest the standard CPCe summaries

```bash
python CPCE_Data_Pipeline/scripts/01_ingest_cpce_outyear.py
```

This script:

- Searches `01_raw/` recursively for supported files.
- Excludes the dedicated 2004–2008 appendix workbook.
- Requires one exact `TRANSECT NAME` header per source.
- Preserves source labels and source-file metadata.
- Writes four standard tables to `03_tidy/`.

For every source, the log should show a verified header row and parsed row
counts. A missing or ambiguous header causes the script to fail rather than
guess.

### Step 2: Ingest the 2004–2008 appendix

```bash
python CPCE_Data_Pipeline/scripts/01b_ingest_appendix_random_transects_2004_2008.py
```

This script processes the historical appendix independently and writes both a
source-faithful tidy table and appendix-specific summary products. It does not
replace or alter the standard tidy tables from Step 1.

### Step 3: Run validation

```bash
python CPCE_Data_Pipeline/scripts/02_validate.py
```

Open `CPCE_Data_Pipeline/04_products/validation_report.txt` immediately after
this step. Near the top of the report, confirm:

```text
[PASS] No critical validation errors were found.
```

Do not continue to Step 4 if the report shows `[FAIL]` or if the process returns
a nonzero exit code. The critical gate checks for:

- Missing or empty required tidy tables.
- Missing required columns.
- Missing or non-numeric analytical values.
- Percent-cover values materially outside 0–100.
- Accidental presence of the dedicated appendix in the standard tidy tables.

Warnings and informational diagnostics should still be reviewed. Known survey
gaps can be valid, but unexpected missing years or files should be investigated.

### Step 4: Derive analytical products

```bash
python CPCE_Data_Pipeline/scripts/03_derive_metrics_with_crosswalk.py
```

This script reads the tidy percent-cover table, applies the reviewed benthic
crosswalk, merges non-duplicating appendix coverage, and writes the three main
summary products to `04_products/`.

CCA is intentionally retained. Where CCA is recorded as a subcategory, the
derivation step produces a CCA group and adjusts colonizable substrate so the
same cover is not counted twice.

### Step 5: Generate Figure 1 outputs

```bash
python CPCE_Data_Pipeline/scripts/04_plot_reports_fig1.py
```

Creates one benthic-cover PNG per represented year in
`CPCE_Data_Pipeline/05_outputs/figures/`.

### Step 6: Generate Figure 2 outputs

```bash
python CPCE_Data_Pipeline/scripts/04_plot_reports_fig2.py
```

Creates one coral-species and CCA PNG per represented year. The number of taxa,
survey scopes, dimensions, colors, and other display settings are controlled by
the configuration files described below.

### Step 7: Generate Figure 3 output

```bash
python CPCE_Data_Pipeline/scripts/04_plot_reports_fig3.py
```

Creates the combined study-site time-series figure:

```text
CPCE_Data_Pipeline/05_outputs/figures/figure3_benthic_time_series.png
```

Missing survey years are displayed as breaks rather than filled or imputed.

## 6. Understanding the generated files

### `03_tidy/`: canonical data layer

These files retain observations at the source/transect level and should be used
for QA, traceability, or future analyses.

| File | Contents |
|---|---|
| `percent_cover_observations.csv` | Percent-cover observations by taxon, transect, source section, year, bank, and scope |
| `count_observations.csv` | Occurrence/count observations in the same long structure |
| `diversity_indices.csv` | Shannon and Simpson diversity metrics by transect |
| `metadata_observations.csv` | Source metadata and check rows that are not analytical cover/count/diversity records |
| `appendix_random_transect_percent_cover_2004_2008.csv` | Source-faithful long table from the historical appendix |

Treat `03_tidy/` as the canonical processed-data layer. It is simplified for
analysis but is not an imputed dataset.

### `04_products/`: analytical products

Products are derived from the tidy layer and can be regenerated.

| File | Contents |
|---|---|
| `validation_report.txt` | Human-readable QA/QC inventory and critical validation result |
| `appendix_major_category_by_transect_2004_2008.csv` | Appendix major categories summarized at the transect level |
| `figure1_appendix_random_transects_summary_2004_2008.csv` | Appendix input prepared for Figure 1 |
| `figure2_appendix_coral_species_summary_2004_2008.csv` | Appendix input prepared for Figure 2 |
| `figure3_appendix_benthic_time_series_summary_2004_2008.csv` | Appendix input prepared for Figure 3 |
| `figure1_random_transects_summary.csv` | Merged annual benthic-category means, standard deviations, standard errors, and transect counts |
| `figure2_coral_species_summary.csv` | Merged annual coral-species and CCA summaries |
| `figure3_benthic_time_series_summary.csv` | Study-site benthic time-series summary |

The three main `figure*.csv` products include a `data_source_branch` field so
rows originating from the standard CPCe branch and historical appendix remain
identifiable.

### `05_outputs/`: presentation files

`05_outputs/figures/` contains the static PNG figures created from the products.
These files are intended for reports, review, and publication workflows. They
are not the canonical data source.

`05_outputs/tables/` is currently a reserved location and may be empty.

### `02_intermediate/`: optional workspace

The standard pipeline does not currently write to this folder. It is available
for temporary diagnostic files but should not become a second source of truth.

## 7. Updating and opening the interactive viewer

Open `Interactive/CPCe_Web/index.html` in a current browser. The page first
tries to load the six CSV files stored beside it. If automatic loading is
blocked, the page displays a manual CSV selector. Select all six `figure*.csv`
files from `CPCE_Data_Pipeline/04_products/` and the viewer will load them
directly.

Before publishing the viewer, confirm that:

- No data-loading error is displayed.
- The newest year appears in the applicable year selectors.
- CCA appears where expected.
- EFGB/WFGB and study-site/reef-wide controls match the available data.
- Downloaded view CSVs contain the selected records.

## 8. Annual update checklist

Use this checklist whenever a new survey year is added:

1. Preserve an untouched copy of every new source file.
2. Confirm that each filename includes year, bank, and survey scope.
3. Place the files under `CPCE_Data_Pipeline/01_raw/`.
4. Confirm that the summary table has one exact `TRANSECT NAME` header.
5. Select the Python environment containing the required libraries.
6. Run Steps 1, 2, and 3.
7. Read the validation report and stop if it fails.
8. Run Step 4 and inspect the newest-year summaries.
9. Run Steps 5–7 to regenerate static figures.
10. Open the interactive page and select the six current figure-product CSVs.
11. Review the interactive charts and confirm the newest year is available.
12. Record the source files and reviewed outputs in the project change log or
    Git commit message.

For a new file that follows an existing CPCe layout, no source-code change
should be required.

## 9. Configuration files

| File | Purpose |
|---|---|
| `config/pipeline.yaml` | Project paths, output folders, plot DPI, and general export settings |
| `config/figure_settings.yaml` | Figure scopes, dimensions, axes, titles, colors, and display options |
| `config/benthic_crosswalk.csv` | Reviewed mapping from source labels to analytical groups and per-figure inclusion |
| `config/benthic_colors.yaml` | Shared benthic-category color definitions |
| `config/taxon_map.yaml` | Supplemental taxon mapping reference |

Changes to `benthic_crosswalk.csv` can change scientific categorization and
should receive domain review. In particular, the current crosswalk intentionally
retains CCA in the products and plots.

Files with `old` or `autosave_backup` in their names are retained as references;
the active plotting scripts read `figure_settings.yaml`.

## 10. Troubleshooting

### `ModuleNotFoundError`

Confirm that the intended Python environment is selected, then install the six
required libraries:

```bash
python -m pip install pandas numpy matplotlib PyYAML openpyxl xlrd
```

### The ingest cannot find a header

Confirm that the relevant worksheet or CSV contains exactly one row whose first
cell is `TRANSECT NAME`. Do not weaken the check or select a plausible-looking
data row without reviewing the source layout.

### The appendix script cannot read `.xls`

Confirm that `xlrd` is installed and that the appendix retains its expected
filename:

```bash
python -m pip install "xlrd>=2.0,<3.0"
```

### Validation returns a failure

Open `04_products/validation_report.txt` and resolve every item in the
`VALIDATION GATE` section. Rerun ingest and validation after correcting the
workflow issue. Do not edit the tidy CSVs by hand to force a pass.

### The interactive page is blank or reports a loading error

- Use the page's manual selector to choose all six `figure*.csv` files from
  `CPCE_Data_Pipeline/04_products/`.
- Confirm that the computer can reach the Plotly CDN.
- Confirm that no required filename was omitted from the selection.

### A new year is labeled `UNKNOWN`

Check that the filename contains a four-digit year, `EFGB` or `WFGB`, and an
explicit `StudySite` or `Reefwide` scope token.

## 11. Data-handling and provenance notes

- The pipeline does not fill missing years or missing observations.
- Numeric conversion is conservative; missing or non-numeric analytical values
  fail validation rather than being silently replaced.
- Raw source files remain unchanged in `01_raw/`.
- The standard tidy tables retain source filename, format, section, year, bank,
  scope, survey type, taxon label, and transect label.
- The historical appendix remains a separate source branch until the derivation
  step fills otherwise missing product keys.
- Analytical products contain means and uncertainty summaries, but the
  transect-level source values remain available in `03_tidy/`.
- Visual outputs should always be traceable back through `04_products/`,
  `03_tidy/`, and ultimately `01_raw/`.

When reporting or publishing results, archive the relevant raw inputs,
configuration files, validation report, and code revision together.
