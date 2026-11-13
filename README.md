# Do Volatility Regimes Improve Forecast Combinations?
## Evidence from Energy, Copper, and Carbon Markets

**Research code and supplementary computational results**  
**Author:** Łukasz Sroka — University of Economics in Katowice, Poland  
**Related manuscript:** *Do Volatility Regimes Improve Forecast Combinations? Evidence from Energy, Copper, and Carbon Markets* (submitted to *Sustainability*).

## Overview

This repository documents the empirical analysis of **one-day-ahead volatility forecasting** in four futures markets: **natural gas, Brent crude oil, copper, and European Union Allowances (EUA)**. It examines whether using volatility-regime information to adjust forecast-combination weights or switch forecasting models improves out-of-sample accuracy relative to simpler alternatives.

The primary forecasting target is **next-day Parkinson range-based variance**. Market states are identified causally using the five-session average of Parkinson variance and rolling 33rd/67th-percentile thresholds calculated from the preceding 252 observations. The original regime definition, forecast origins, and model-selection rules are not changed by the supplementary analyses.

The evaluated methods include:

- **Individual models:** Naive, heterogeneous autoregression (HAR), GARCH, EGARCH, GJR-GARCH, Random Forest, XGBoost, and support vector regression (SVR).
- **Forecast combinations:** equal weighting of HAR/RF/XGBoost (EW3), static QLIKE-optimized weights, regime-dependent weights, and regime-based model switching.
- **Assessment:** primarily QLIKE; additionally MAE, RMSE, forecast-loss inference (including DM-HLN with multiple-testing adjustments), and robustness checks.
- **Supplementary work:** inverse-MSE combination weights, volatility-based Value-at-Risk (VaR) assessment, expanded Model Confidence Set (MCS), shifted log-HAR, and structural-stability diagnostics.

The analysis is deliberately **market-specific**: the study does not assume that regime-conditional combinations must outperform simpler methods in every market.

## Repository structure

```text
volatility-regime-forecast-combinations/
├── README.md
├── reproduction_guide.md       # Detailed EUA notebook execution guide
├── requirements.txt            # Observed Python package dependencies
├── notebooks/
│   ├── Probability-Weighted Regime - Neutral GAS.ipynb
│   ├── Probability-Weighted Regime - Brent.ipynb
│   ├── Probability-Weighted Regime - Copper.ipynb
│   └── Probability-Weighted Regime - EU ETS EUA.ipynb
└── robustness/
    ├── Natural Gas/
    ├── Brent/
    ├── Copper/
    └── EUA/
```

The `robustness/` directories contain selected CSV outputs from STEP 12 and STEP 13. Some dated, observation-level files require a redistribution review and should not remain publicly available without appropriate permission (see **Data access and redistribution** below). The directory tree above shows the intended organization once the accompanying guide and requirements file have been uploaded.

The notebook file named `Neutral GAS` contains the **natural-gas** analysis; the filename is preserved as used in the repository.

## Data access and redistribution

The study uses daily historical **continuous-futures OHLC** records obtained from **Investing.com**, covering the study period beginning **25 January 2010** and ending **22 September 2026**, subject to market-specific trading calendars and availability. The provider's conventions for constructing continuous contracts and subsequent historical data revisions could not be independently verified.

**The underlying third-party OHLC data are not provided for redistribution through this repository.** To reproduce the analysis, obtain corresponding historical data from the original data provider under its applicable access and licensing conditions. Identical numerical results may require the same historical vintage of the provider's data.

The original notebook readers expect UTF-8 with BOM (`utf-8-sig`) and the following Polish source headers:

| Original CSV column | Internal variable | Description |
|---|---|---|
| `Data` | `Date` | Date, parsed as `DD.MM.YYYY` |
| `Ostatnio` | `Close` | Closing price |
| `Otwarcie` | `Open` | Opening price |
| `Max.` | `High` | Daily high |
| `Min.` | `Low` | Daily low |
| `Wol.` | `Volume` | Volume; formatted suffixes may include `K`, `M`, or `B` |
| `Zmiana%` | `Change_pct` | Percentage change |

Prices are cleaned from the provider's locale-dependent number format; records are subsequently sorted chronologically. Do not silently drop or alter observations to force agreement with the original sample sizes.

The original local filenames referenced by the four notebook readers are:

| Market | Original CSV filename |
|---|---|
| Natural Gas | `Kontrakty terminowe na gaz ziemny.csv` |
| Brent | `Kontrakty terminowe na ropę Brent (LCOX6).csv` |
| Copper | `Kontrakty terminowe na miedź (HGZ6).csv` |
| EUA | `European Union Carbon Permits Allowance (EUA) Yearly Futures.csv` |

**Do not commit** raw provider-supplied OHLC data, unreviewed daily realized-volatility or return series, or notebook outputs displaying these data. Some generated intermediate CSVs contain `Target_Parkinson_t1` or other provider-derived daily observations and require a separate redistribution review before public release.

## Sample design and expected checks

The original analysis uses a chronological division: **Train through 31 December 2016**, **Validation during 2017–2019**, and **Test from January 2020 through the available end of the sample**. The expected **effective modeled** sample sizes are:

| Market | Train | Validation | Test |
|---|---:|---:|---:|
| Natural Gas | 1,765 | 790 | 1,766 |
| Brent | 1,772 | 774 | 1,736 |
| Copper | 1,735 | 770 | 1,772 |
| EUA | 1,766 | 770 | 1,728 |

These are **model-ready** row counts after feature/target construction, not raw OHLC record counts. Discrepancies should be investigated, not resolved by changing the observations or original modeling decisions.

## Python environment

The supplied notebooks record **Python 3.10.20** and a local kernel named `garch_fix`. The original third-party package versions were not embedded in the notebook metadata. Accordingly, the supplied [`requirements.txt`](requirements.txt) identifies the observed dependencies but is **not an exact historical environment lockfile**.

A possible local setup from the repository root is:

```bash
python -m venv .venv
# Windows PowerShell: .\.venv\Scripts\Activate.ps1
# Linux/macOS: source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

If the original environment is still available, its authentic versions can be recorded separately with `python -m pip freeze > requirements-lock.txt` **in that environment**. A freeze performed in a newly created environment must not be represented as the original environment.

## Reproducing the analysis

**Read [`reproduction_guide.md`](reproduction_guide.md) before executing the notebook.** The current detailed, indexed execution guide was prepared for the **EUA notebook**. The Natural Gas, Brent and Copper notebooks share the main methodological structure, but their exact cell indices, optional analyses, checkpoints and local file paths are not guaranteed to be identical.

### 1. Configure the local data location

Each original notebook begins with a **machine-specific Windows input path**. Replace this path with the location of your independently obtained CSV. For example, a reproducible local arrangement is `data/raw/EUA.csv` (not tracked in Git), provided the input still uses the headers and formats expected by the reader.

The notebook's working directory can differ depending on how Jupyter was launched. Determine it with `Path.cwd()` and resolve paths accordingly; do not assume the notebook's file directory is the current working directory.

### 2. Use one consistent results directory

The STEP 12 and STEP 13 sections write and read intermediate checkpoints using paths such as `REVISION_RESULTS/<market>/` and `REVISION_RESULTS/<market>/STEP13_NEW/`. In the original code, some later cells still refer to `C:\Users\UE\REVISION_RESULTS` or external module locations. **Update only these local I/O paths and the corresponding invocation cells** so that every stage uses the same output root. For natural gas, the output directory identifier used by some routines is `Natural_Gas`, not the display name `Natural Gas`.

### 3. Execute the core notebook stages in dependency order

| Order | Stage | Purpose |
|---:|---|---|
| 1 | Initial loading and STEP 2–4 | Import/validate OHLC, construct Parkinson variance and lagged features, classify volatility regimes |
| 2 | STEP 5–6 | Chronological Train–Validation–Test split and complete 27-predictor modeling dataset |
| 3 | STEP 7–9 | Original HAR, GARCH-family and ML benchmark estimation and Validation forecasts |
| 4 | STEP 10 | Freeze original combination weights and regime-switching rules using Validation |
| 5 | STEP 11 | Generate aligned original Test forecasts and evaluate their performance |
| 6 | Revised STEP 12.0–12.7 | Boundary/positivity audits, Validation-only calibration, additional combinations and VaR robustness |
| 7 | Final post-review STEP 13.1 | Expanded MCS, based on the **original** `12_5A_daily_forecasts.csv` Test checkpoint |
| 8 | Final post-review STEP 13.2 | Shifted log-HAR, with paired HAR comparison using `12_6_inverse_MSE_daily_forecasts.csv` |
| 9 | Final post-review STEP 13.3 | Predefined-date HAC structural-stability tests and exploratory CUSUM |

**Do not assume that “Run All” is safe.** Some original robustness/reviewer-response cells are optional, placed between main sections, use checkpoints from earlier executions, refer to other markets, or contain non-portable imports or terminal commands. Follow the EUA cell-by-cell guide and check the corresponding stage headings and dependencies in each other notebook. The last post-review STEP 13 definitions are available within the notebooks; the separate runner cells may require adaptation if external `.py` modules are not provided.

For the final expanded MCS, retain the frozen original forecast pool, strict-positive QLIKE admissibility rules, `method='R'`, stationary block bootstrap, **3,000 replications**, and its recorded random seed. Do not substitute revised forecasts or tune models on the Test set.

### 4. Validate the outputs

Check sample sizes, chronological forecasting origins, forecast/target date alignment, strictly positive admissible variance forecasts, and the preservation of Train/Validation-selected configurations. Compare the **aggregate** generated CSVs with the manuscript tables and the reviewed files under `robustness/`. Do not presume exact identity when the source-data vintage or software environment differs.

## Reproducibility status

This repository provides research notebooks and selected analytical outputs. **A fully automated, clean-environment, end-to-end reproduction has not been independently verified.** The original notebooks retain certain local path and checkpoint assumptions, and some supplementary cells require manual execution in dependency order. The detailed instructions are intended to make those requirements explicit rather than overstate reproducibility.

The STEP 12/13 analyses were performed as supplementary robustness checks. They should not be treated as changes to the original forecasting target, the frozen predictor set, the primary regime-classification protocol, or the original Test-based evaluation design.

## Citation and use

If you use this repository, please cite the associated manuscript and identify the specific GitHub commit or archived release used for replication. No license to third-party Investing.com market data is conveyed by this repository. The availability of source code does not imply that its third-party dependencies or input data are free of their own licensing conditions.

For methodology, results and limitations, refer to the manuscript and the detailed [reproduction guide](reproduction_guide.md).
