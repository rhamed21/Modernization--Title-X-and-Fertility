# Modernizing the Title X Family Planning Analysis

Re-estimates the effect of federally funded family planning programs on U.S. county
fertility (1960s-70s) using modern staggered-adoption difference-in-differences methods,
building on the data from Bailey (2012).

Estimators: Callaway and Sant'Anna (2021) with doubly robust estimation and not-yet-treated
controls, plus Sun and Abraham (2021) and a stacked event study as robustness checks.
Covariate balance uses generalized boosted model (GBM) propensity score weighting.

## Repository contents

| File | Purpose |
|---|---|
| `01_data_cleaning.Rmd` | Builds the analysis dataset from the original replication files |
| `02_analysis.Rmd` | Weighting, estimation, tables and figures |
| `data/` | Put the downloaded input files here (not tracked by git) |
| `report_outputs/` | Created when you run the analysis: `csv/` tables and `figures/` |

## Data

The data are **not** included in this repository. They come from the replication package for:

> Bailey, Martha J. Replication data for: Reexamining the Impact of Family Planning Programs
> on US Fertility: Evidence from the War on Poverty and the Early Years of Title X.
> Nashville, TN: American Economic Association [publisher], 2012. Ann Arbor, MI:
> Inter-university Consortium for Political and Social Research [distributor], 2019-10-12.
> https://doi.org/10.3886/E113821V1

1. Download the package from the DOI above (openICPSR may ask you to create a free account and accept terms of use).
2. Copy these two files into the `data/` folder:
   - `vs_fo_final.dta`
   - `table1data.dta`

## Requirements

R (4.1 or newer recommended) and these packages:

```r
install.packages(c(
  "haven", "dplyr", "tidyr", "ggplot2", "fixest", "cobalt", "purrr", "stringr",
  "readr", "WeightIt", "did", "tibble", "knitr", "kableExtra", "broom", "gbm",
  "rmarkdown"
))
```

Stata is **not** required.

## How to run

Open the project folder in R or RStudio (so the working directory is the repository root), then:

```r
rmarkdown::render("01_data_cleaning.Rmd")   # creates data/modernization_data.rds
rmarkdown::render("02_analysis.Rmd")        # creates report_outputs/ and the HTML report
```

Run them in this order. The analysis is slow because it fits a grid of GBM models for
each policy wave and bootstraps the Callaway and Sant'Anna estimates. The seed is fixed
(`set.seed(123)`), but small numerical differences across package versions and operating
systems are possible.

## What the data cleaning does

The panel `vs_fo_final.dta` stores the 1960 baseline covariates only as detrended terms
(baseline level x year), and `table1data.dta` holds additional covariates but has no FIPS
identifier. `01_data_cleaning.Rmd`:

1. Recovers the unrounded baseline covariates by dividing the detrended variables by `year`.
2. Attaches FIPS codes to `table1data.dta` by matching its seven demographic variables to the recovered 1960 values (one-to-one; the script stops if any county fails to match).
3. Merges the additional covariates (1960 population, two education measures, four region indicators) onto the full panel by FIPS.

This replaces an earlier Stata do-file. To confirm the two agree, save the Stata output as
`data/modernization_data.dta` before knitting; the last chunk compares every added column.

## Data notes

- Region indicators (`NE`, `MW`, `S`, `W`) are coded 0/100, not 0/1. The analysis treats any value above 0 as membership.
- `_60pcteducge12yr` has a maximum above 100, which cannot be a percentage. It comes from the source data and is not modified here.

## Author

Roa'a Hamed

