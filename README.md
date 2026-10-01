# Hill-2026-PEV-analysis
Analysis and figure-generation code for "Wolbachia-induced cytoplasmic incompatibility produces heritable chromatin modifications that suppress position-effect variegation".

# Statistical analyses and plots

This repository contains R scripts for statistical analyses and visualization of position-effect variegation (PEV) and egg-to-adult viability data. Each script contains dataset-specific filenames and comparisons that can be edited for the corresponding input files.

## Scripts

| Script | Purpose |
| --- | --- |
| `Analysis.R` | Analyze red pigmentation measurements using beta mixed models and Mann–Whitney tests; calculate Pearson correlations on vial means and apply Benjamini–Hochberg (BH) correction within specified figure comparison sets. |
| `plots.R` | Create boxplots with individual observations and scatterplots of red pigmentation versus egg-to-adult viability (EAV). |
| `Table 2 exchangable.R` | Analyze egg and adult counts using binomial generalized estimating equations (GEE) with date-based clustering and an exchangeable working correlation structure. |

The scripts are independent and do not need to be run in a particular order. Run the Table 2 script in a separate R session: its first line, `rm(list = ls())`, removes existing objects from the current workspace.

## Requirements

Use **R 4.1 or later** because `Analysis.R` uses the native pipe operator (`|>`). RStudio is optional.

Install the required packages once:

```r
install.packages(c("tidyverse", "mgcv", "ggplot2", "geepack"))
```

`ggplot2` is also included in the tidyverse. The installation lines at the start of `plots.R` can be skipped after setup.

## Input files

The scripts currently read CSV files from the R working directory. Set the working directory to the location of the input files, or edit the paths in the scripts. For files stored in a `data/` subfolder, use paths such as `"data/1F.csv"`.

Column names and group labels are case-sensitive. Retain the exact labels referenced by the scripts, or update the comparisons to match the data.

### Statistical analyses: `Analysis.R`

| CSV filename | Required columns |
| --- | --- |
| `1F.csv` | `name`, `red`, `date` |
| `2D.csv` | `name`, `red`, `date` |
| `3C.csv` | `vial`, `red`, `EAV` |
| `3D.csv` | `name`, `red`, `EAV` |
| `4A.csv` | `name`, `red`, `Date` |
| `4B.csv` | `name`, `red`, `date` |

- `name`: treatment/group label, except in `3D.csv`, where it identifies vials for clustering and correlation summaries.
- `red`: numeric red pigmentation response expressed on a 0–1 scale.
- `vial`: vial identifier.
- `date` or `Date`: experimental date/block identifier. `4A.csv` specifically requires capitalized `Date`.
- `EAV`: numeric egg-to-adult viability measurement.

All six files are loaded when the complete script is run. The CI-versus-UU comparison for Figure 4A reuses the combined `3C.csv` and `3D.csv` data.

### Visualization: `plots.R`

Change the filename in `read.csv("1F.csv", ...)` to the dataset you want to plot, then run the appropriate plotting section.

| Plot | Required columns | Figure references in the script |
| --- | --- | --- |
| Boxplot ordered by vial ID | `vial`, `red` | S3A, S3B |
| Boxplot in group order of first appearance | `name`, `red` | 1F, 2C, 2D, 4A, 4B, S6E |
| Boxplot ordered by mean EAV within each name | `name`, `EAV`, `red` | 3C, 3D |
| Scatterplot with linear fit and Pearson r | `EAV`, `red` | S3C, S3D |

Only the selected plot's columns are required when running sections individually. The script does not automatically loop over multiple CSVs.

### Egg-to-adult viability: `Table 2 exchangable.R`

The script currently reads `yw.csv`, `w1118.csv`, and `UBW.csv`. Each file requires:

| Column | Contents |
| --- | --- |
| `date` | Experimental date identifying the cluster |
| `treatment` | Treatment label; currently `CI` or `UU` |
| `eggs` | Number of eggs |
| `adults` | Number of adults emerging from those eggs |

Use nonnegative integer counts with `adults <= eggs` and positive egg totals for analyzed rows. Keep rows from the same date together in each CSV; the script does not sort clusters. Check labels and missing values before running: unrecognized treatment labels become missing and are excluded. Blank date strings are not explicitly excluded.

To analyze additional CSVs, add `run_gee()` calls and their corresponding entries in `summary_table`.

## Running and saving results

From a working directory containing the scripts and their expected inputs:

```r
source("Analysis.R")
```

`Analysis.R` prints results and creates objects including `fig1f`, `fig2d`, `fig3`, `fig4a`, `fig4b`, and `all_results`. It does not write output files automatically. For example:

```r
write.csv(all_results, "analysis_results.csv", row.names = FALSE)
write.csv(fig1f, "figure_1F_results.csv", row.names = FALSE)
```

Figure 1F is calculated and printed separately but is not included in the current `all_results` object or final summary.

For plots, open `plots.R`, load the desired CSV, and run the selected plot expression interactively. Save the displayed plot with, for example:

```r
ggplot2::ggsave("figure.pdf", plot = ggplot2::last_plot(),
                width = 8, height = 6, units = "in")
```

The script does not automatically save plots. Its jitter is unseeded and does not explicitly disable vertical jitter, so point positions can vary between runs.

In a separate R session, run the Table 2 analysis:

```r
source("Table 2 exchangable.R")
print(summary_table)
write.csv(summary_table, "table_2_results.csv", row.names = FALSE)
```

## Analysis details

- Beta models use `mgcv::gam()` with `family = betar()` and a random-effect term for vial or date. The response is transformed into the open interval (0,1) using `(y * (n - 1) + 0.5) / n` within each comparison.
- Mann–Whitney tests use `wilcox.test()` defaults. Selected comparisons use this test directly. Where a beta model returns a missing p-value, `best_p` uses the accompanying Mann–Whitney p-value; inspect model status before reporting results.
- BH correction is applied separately to the comparison sets for Figures 1F, 2D, 4A, and 4B. The standalone Figure 3C/3D result and the vial-mean Pearson correlations are not BH-adjusted in the script.
- The Pearson analyses in `Analysis.R` use vial means. The correlation shown in `plots.R` uses individual complete rows from the loaded CSV; these are different calculations.
- Table 2 uses binomial GEE with date as the cluster and an exchangeable working correlation structure. With treatment levels `c("CI", "UU")`, the reported odds ratio compares UU to CI. The script reports coefficients, standard errors, odds ratios, approximate 95% confidence intervals, and unadjusted p-values; it does not apply BH correction.

## Reproducibility and publication information

Document the observation unit, group definitions, data provenance, and any exclusions alongside the CSVs. Add the associated publication citation, data availability information, and code license before release.

Record the R and package versions used for the final analyses:

```r
writeLines(capture.output(sessionInfo()), "session_info.txt")
```

This README describes the supplied scripts as written. It does not establish that the models ran successfully or reproduce manuscript results; verify those against the final input data.
