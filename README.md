# Hill-2026-PEV-analysis

R code, CSV datasets, and an Excel input workbook for “Wolbachia-induced cytoplasmic incompatibility produces heritable chromatin modifications that suppress position-effect variegation”.

- GitHub repository: https://github.com/hunter-j-hill/Hill-2026-PEV-analysis
- Figures and associated materials on Figshare: https://figshare.com/articles/figure/Figures_for_Hill_et_al_2026/34059486

## Scripts and repository layout

| Script | Purpose |
| --- | --- |
| `Analysis` | Beta mixed models, selected Mann–Whitney comparisons, Kruskal–Wallis omnibus tests, within-figure pairwise BH correction, and Pearson correlations on vial means. |
| `Figure Plots` | Generate each data-driven figure panel separately and save 300 dpi PNGs. Replaces the earlier manual plotting workflow. |
| `Table 2 (CI strength)` | Binomial GEE analysis of egg-to-adult viability, clustered by experimental date. |
| `Supp_Table_2_downsampling` | Supplementary Table 2: downsampling for unequal sample sizes and a separate Figure 4B comparison restricted to the 2026 experimental round. |

The commands below use the exact filenames in the supplied repository: `Analysis`, `Figure Plots`, and `Table 2 (CI strength)`, plus the new `Supp_Table_2_downsampling` script. These files contain R code despite having no `.R` extension. Keep the CSVs in `data/` and start from the repository root.

The scripts can be run independently. Run the Table 2 script in a separate R session because its initial `rm(list = ls())` clears the current workspace.

## Requirements

Use R 4.1 or later. RStudio is optional. Install dependencies once:

```r
install.packages(c("tidyverse", "mgcv", "geepack", "readxl"))
```

The tidyverse includes ggplot2; the plotting script only requires ggplot2. It does not install packages automatically. The downsampling script requires `readxl`.

## CSV inputs

Column names and group labels are case-sensitive.

| CSV | Required analysis/plot columns | Use |
| --- | --- | --- |
| `1F.csv` | `name`, `red`, `date` | Figure 1F and its statistical comparisons |
| `2C.csv` | `name`, `red` | Figure 2C plot, Kruskal–Wallis test, and four planned Mann–Whitney comparisons |
| `2D.csv` | `name`, `red`, `date` | Figure 2D and its statistical comparisons |
| `3C.csv` | `vial`, `red`, `EAV` | CI: Figures 3C, S3A, S3C; joint CI/UU analysis |
| `3D.csv` | `name`, `red`, `EAV` | UU: Figures 3D, S3B, S3D; joint CI/UU analysis |
| `4A.csv` | `name`, `red`, `Date` | Figure 4A |
| `4B.csv` | `name`, `red`, `date` | Figure 4B |
| `s4.csv` | `name`, `red` | Figure S4B |
| `polytenes.csv` | `name`, `wl` | Figure S5E chromosome width/length plot |
| `yw.csv`, `w1118.csv`, `UBW.csv` | `date`, `treatment`, `eggs`, `adults` | Table 2 GEE and descriptive companion plots |

`red` is red pigmentation on a 0–1 scale; `EAV` is egg-to-adult viability on a 0–1 scale. `wl` is a chromosome width/length ratio. `name` generally identifies a treatment group, but identifies individual vials in `3D.csv`. `vial` identifies vials in `3C.csv`. Date fields specify experimental dates or blocks; preserve the capitalized `Date` header in `4A.csv`.

The full plotting run requires all 12 CSVs. The analysis script loads seven CSVs: 1F, 2C, 2D, 3C, 3D, 4A, and 4B. Its Figure 4A CI-versus-UU comparison reuses the combined 3C/3D data. The plotting script draws Figure 4A directly from `4A.csv`.

### Linking measurements to source images

Pigmentation CSVs contain `Photo_ID` values matching image basenames without extensions. Use `Folder_ID` where provided (`Folder ID` with a space in `3C.csv`) and retain group context, especially for 4A where folder/photo combinations repeat across CI and UU. Date folders may use hyphens where CSV values use slashes.

For `s4.csv`, obtain both the 1F image set (643 images) and the additional S4B set (275 images). Together these cover its 918 measurements.

In the checked 3C and 4A archives, the CI folder `1-16-16` corresponds to CSV date `1/16/26`. See the raw-image README for full paths, group mappings, and ilastik/Fiji instructions. Image identifiers support tracing measurements to source files and are not model predictors.

## Generate all plots

Set the R working directory to the repository root, then run:

```r
source("Figure Plots")
```

The script looks for input CSVs in `data/`, falling back to the working directory if no `data/` folder exists. You can set `data_dir` explicitly near the top of the script.

It creates 16 separate plots:

- Main panels: 1F, 2C, 2D, 3C, 3D, 4A, and 4B.
- Supplemental panels: S3A, S3B, S3C, S3D, S4B, and S5E.
- Three descriptive Table 2 companion plots: yw, w1118, and UBW.

By default, it writes PNGs only to `figure_plots/`, at 300 dpi, plus `session_info.txt`. Re-running replaces same-named PNG outputs. Set `save_plots <- FALSE` to disable file output. Existing PDFs from earlier runs are not removed.

Plots use a grey panel background with white gridlines. Boxplots display every input observation with horizontal jitter only; boxplot outlier symbols are suppressed to avoid plotting those observations twice. Jitter uses a fixed seed of 2026. Group plots retain the order of first appearance in each CSV.

Figures 3C and 3D show separate vials ordered by mean EAV; equal EAV values do not merge different vials. S3A and S3B order vials by numeric month/day and vial ID. S3C and S3D use one point per vial, averaging both pigmentation and EAV within each vial. The displayed Pearson r and p-value use those same vial means, matching the aggregation in the analysis script. Linear fits include confidence bands.

The Table 2 companion plots show the observed ratio `adults / eggs` for each CSV row. They are descriptive plots, not GEE estimates or manuscript Table 2 replacements.

Plot objects are retained in the `plots` list:

```r
print(plots$Figure_1F)
print(plots$Figure_S3C)
print(plots$Table2_descriptive_yw)
```

The script checks required columns and values and stops on invalid required inputs rather than silently dropping them. It generates data plots; assembled artwork, microscopy panels, and statistical significance brackets are not included.

## Run the pigmentation analyses

The analysis script reads bare CSV filenames from the working directory. From the repository root, run:

```r
setwd("data")
source("../Analysis")
write.csv(all_results, "../analysis_results.csv", row.names = FALSE)
writeLines(capture.output(sessionInfo()), "../analysis_session_info.txt")
setwd("..")
```

If execution stops before the final line, restore the working directory to the repository root before running another script.

The analysis creates `fig1f`, `fig2c`, `fig2d`, `fig3`, `fig4a`, `fig4b`, and `all_results`. It also creates `fig1f_kw`, `fig2c_kw`, `fig2d_kw`, `fig4a_kw`, and `fig4b_kw` for the omnibus tests. All of these figure comparisons and omnibus results are included in `all_results`; a separate 1F export is optional. Pearson correlation results are stored in `cor_ci` and `cor_uu` and printed separately.

The console prints a combined final table and a separate Kruskal–Wallis summary with sample size, chi-squared statistic, degrees of freedom, and unadjusted p-value. In the combined table:

| Column | Meaning |
| --- | --- |
| `n` | Sample sizes for the two groups; total observations for an omnibus test |
| `clusters` | Numbers of date/vial clusters for each group, where available |
| `method` | Selected test: `beta_GLMM`, `MW`, or `KW` |
| `raw_p` | Unadjusted p-value from the selected test (`best_p` in saved results) |
| `BH_p` | Within-figure BH-adjusted pairwise p-value (`bh_p`) |
| `MW_p` | Unadjusted Mann–Whitney p-value (`mw_p`), also supplied alongside beta-model results |

A dash indicates an unavailable or inapplicable value. For MW comparisons, `raw_p` and `MW_p` match. The saved CSV retains numeric values and additional fields, including `statistic` and `df` for KW rows; console p-values are formatted for display.

### Analysis methods and interpretation

- Beta models use `mgcv::gam()` with `family = betar()` and a random-effect term for date or vial. Each comparison transforms the response to the open interval (0,1) using `(y * (n - 1) + 0.5) / n`.
- Selected comparisons use `wilcox.test()` defaults (Mann–Whitney tests).
- Kruskal–Wallis omnibus tests compare all groups within 1F, 2C, 2D, 4A, and 4B. Their p-values are unadjusted and excluded from pairwise BH correction. These rank tests use individual observations and do not account for date/vial clustering.
- BH correction is applied separately across the planned pairwise comparisons for each figure: four each for 1F, 2C, 2D, and 4B, and five for 4A. The standalone 3C/3D comparison and Pearson correlations are not BH-adjusted. The same CI-versus-UU comparison receives BH adjustment when included in the five-comparison Figure 4A family.
- Figure 2C uses only the following four planned two-sided Mann–Whitney comparisons, with BH correction across these four tests:

  | Label | First group | Second group |
  | --- | --- | --- |
  | p1 | `IA_females` | `UB_female` |
  | p2 | `IA_males` | `UB_males` |
  | p3 | `IAW_females` | `UBW_females` |
  | p4 | `IAW_males` | `UBW_males` |

  These compare stocks within each sex. They do not compare females against males or test all 28 possible pairs. The script validates the eight expected group labels and finite red values in [0,1] for 2C.
- Figure 4A compares `I fathers` vs `U fathers`, `CI` vs `II`, `CI` vs `UU`, `IU` vs `CI`, and `II` vs `IU`. The second comparison (p2), `CI` vs `II`, uses a two-sided Mann–Whitney test because the groups come from separate experimental blocks. It replaces the earlier `UU` vs `U fathers` comparison. The script normalizes the CSV label `U Fathers` to `U fathers`.
- Figure 4B compares `I fathers` vs `U fathers`, `I mothers` vs `U mothers`, `CI` vs `UU`, and `CI` vs `II`. The earlier `II` vs `UU` comparison is excluded.
- These pairwise changes do not remove groups from the Kruskal–Wallis omnibus tests. Rerun the analysis after changing a comparison family, since BH-adjusted p-values for other comparisons in that family may also change.
- The X/O comparisons in 1F and 2D are explicitly assigned to Mann–Whitney.
- For comparisons assigned to the beta mixed model, the script attempts that model and also calculates a Mann–Whitney p-value. If the beta-model p-value is available, `best_p` uses it and `method` is labeled `beta_GLMM`. Otherwise, `best_p` uses the Mann–Whitney p-value and `method` is labeled `MW`. The `status` field retains any caught error message. Comparisons explicitly assigned to Mann–Whitney use that test directly. Review model diagnostics before interpreting results; obtaining a p-value does not by itself establish model validity.
- The supplied analysis does not implement every statistical comparison for every plotted CSV. Generating a plot does not perform or validate its manuscript significance annotations.

## Run Table 2: egg-to-adult viability

Each input must contain nonnegative integer counts, positive egg totals, and `adults <= eggs`. Treatments are CI and UU. Check dates and treatment labels before fitting; unknown treatment labels become missing in the reviewed script, and blank date strings need explicit checking.

The Table 2 script sorts observations inside `run_gee()`, after data cleaning and immediately before `fit <- geeglm(...)`:

```r
df <- df[order(df$date), ]
```

This keeps all observations from the same date contiguous, as required for date-based clustering. Sorting only reorders rows. Use the corrected results after rerunning; results from the earlier unsorted workflow are superseded.

In a separate R session, starting at the repository root:

```r
setwd("data")
source("../Table 2 (CI strength)")
print(summary_table)
write.csv(summary_table, "../table_2_results.csv", row.names = FALSE)
writeLines(capture.output(sessionInfo()), "../table_2_session_info.txt")
setwd("..")
```

The model uses a binomial response `cbind(adults, eggs - adults)`, date clusters, and an exchangeable working correlation. With factor levels `c("CI", "UU")`, the odds ratio compares UU with CI. Outputs include log-odds coefficients, standard errors, odds ratios, approximate 95% confidence intervals, and unadjusted p-values. Odds ratios are not ratios of survival percentages.

## Run Supplementary Table 2: downsampling

Run the script from an interactive R session, starting at the repository root:

```r
source("Supp_Table_2_downsampling")
```

When the file chooser opens, select `unbalanced_sample_sizes.xlsx`. If the script is saved with a `.R` extension, use `source("Supp_Table_2_downsampling.R")` instead. For a noninteractive run, replace `file <- file.choose()` in the script with the workbook path.

### Excel input

The workbook must contain the following worksheets and group labels exactly as shown. Each sheet has no header row: column 1 contains the group label, column 2 contains the red fraction of eye surface area (0–1), and column 3 contains the experimental date or vial ID. The `4B Control v CI` sheet requires Excel-readable dates in column 3 for the round-specific comparison.

| Worksheet | Smaller group | n | Larger group | n |
| --- | --- | --- | --- | --- |
| `4a-control v rescue` | `II` | 174 | `UU` | 534 |
| `4B Control v CI` | `UU` | 83 | `CI` | 239 |
| `4B fathers` | `U fathers` | 25 | `I fathers` | 119 |
| `1F-C(1)DX X-Y` | `CI XY` | 56 | `U XY` | 117 |
| `2D X-O males` | `CI_XO` | 16 | `UxU_XO` | 22 |

Worksheet names are workbook identifiers and are preserved literally. The script stops if the selected groups do not match these expected sample sizes. For downsampling, columns are initially read as text and red fractions are converted to numeric; missing or nonnumeric red values cause the script to stop.

### Procedure

For each of the five comparisons, the script retains every observation in the smaller group and randomly samples the larger group without replacement to the same sample size. It repeats this procedure 1,000 times with `set.seed(2026)`. Each iteration uses a two-sided Mann–Whitney test with a normal approximation and continuity correction (`wilcox.test(..., exact = FALSE, correct = TRUE)`). It also calculates the full-data p-value and the median p-value across iterations. These p-values are unadjusted.

A separate Figure 4B Control-versus-CI test uses observations with dates in 2026. The script identifies these by calendar year; the intended January–February 2026 round therefore assumes that all 2026 observations on this sheet belong to that round. This comparison uses the same Mann–Whitney settings and is not downsampled. Unlike the downsampling import, this section reads the red column directly as numeric, so those cells must be stored as numeric Excel values.

### Outputs and interpretation

The script prints the supplementary table and saves two CSVs beside the selected workbook. Rerunning replaces files with the same names.

| Output | Contents |
| --- | --- |
| `downsampling_results.csv` | Comparison labels, group labels, original sample sizes, original group medians, percentage of iterations with p < 0.05 (`pct_sig`), and the separate 2026-round p-value (`p_2026_round`). |
| `downsampling_results_full.csv` | Comparison and group information, full-data p-values (`p_full`), `pct_sig`, and median iteration p-values (`median_p`). For the 2026-round row, `p_full` is the p-value for that restricted comparison. |

`pct_sig` and `median_p` are `NA` for the 2026-round row because it is not downsampled. `p_2026_round` is `NA` for the five downsampling rows. Reported sample sizes and medians describe the original groups used for each row, not individual subsamples.

`pct_sig` describes how often p < 0.05 is retained after balancing sample sizes in these observed data. It is a sensitivity summary, not an adjusted p-value or the probability that the biological effect is real. Downsampling operates on individual observations and does not account for date/vial clustering.

To record the software versions used, run `sessionInfo()` after the analysis and save its output with the final results.

## Release notes and reproducibility

The Supplementary Table 2 script adds reproducible downsampling summaries for five unequal-size comparisons and a separate 2026-round Figure 4B test.

The updated plotting script fixes dataset-specific column handling and aligns correlation plots with the vial-mean analyses. It saves PNGs only and replaces the previous manual plotting workflow.

The analysis now includes Figure 1F and the four Figure 2C comparisons in the final table, plus omnibus Kruskal–Wallis results for 1F, 2C, 2D, 4A, and 4B.

The documented analysis workflow uses R 4.1 or later, sorts Table 2 observations by date before fitting, and labels the selected p-value source as `beta_GLMM` or `MW`. The Figure 2D X/O comparison contains 16 CI and 22 UU observations in the supplied data. Check original experimental records before changing any date values.

Record the software versions used for final results and include the selected code license and manuscript DOI when available. The plotting script does not modify the source CSVs. Filename matching and documentation review do not validate image contents, model assumptions, or exact agreement with manuscript results.
