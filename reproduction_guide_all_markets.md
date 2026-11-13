# Reproduction Guide — Natural Gas, Brent, Copper and EUA

**Manuscript:** *Do Volatility Regimes Improve Forecast Combinations? Evidence from Energy, Copper, and Carbon Markets*  
**Author:** Łukasz Sroka  
**Purpose:** Describe the input requirements, execution sequence, intermediate outputs, and validation checks for the four original Jupyter notebooks and their post-review robustness analyses.

> **Reproducibility status.** This guide was prepared by inspecting the four supplied notebooks and their actual cell order. It is a **code-level execution guide**, not a certificate that a clean end-to-end run succeeded. The original third-party OHLC files and every historical checkpoint were not available for an independent re-execution. Some notebook cells contain absolute Windows paths or rely on files created earlier. Users must configure these paths and supply legally obtained source data.

## 1. Files and market identifiers

The four notebooks use a common research design but have different physical cell indices and some market-specific exploratory blocks.

| Market | Notebook | `MARKET` value | Results subdirectory | Total cells in audited copy |
|---|---|---|---|---:|
| Natural Gas | `notebooks/Probability-Weighted Regime - Neutral GAS.ipynb` | `"Natural Gas"` | `Natural_Gas` | 239 |
| Brent | `notebooks/Probability-Weighted Regime - Brent.ipynb` | `"Brent"` | `Brent` | 257 |
| Copper | `notebooks/Probability-Weighted Regime - Copper.ipynb` | `"Copper"` | `Copper` | 250 |
| EUA | `notebooks/Probability-Weighted Regime - EU ETS EUA.ipynb` | `"EUA"` | `EUA` | 243 |

The audited uploaded copies included occasional filename suffixes, such as `(4)` or `(5)`, which are not part of the intended repository filename. **The cell indices below are zero-based notebook JSON positions, not Jupyter's `In [n]` execution counters.** If cells are inserted or deleted, identify the code by its `STEP` heading instead of relying on a numeric index.

**Execution rule:** process **one market notebook per fresh kernel**. Do not run all four market notebooks in the same live Python session; they reuse variable names (`df`, `model_df_complete`, `MARKET`, `BASE`, etc.).

## 2. Obtain and prepare input data

Obtain the corresponding **daily continuous-futures historical OHLC data from Investing.com** independently and in accordance with the provider's conditions. The source CSV filenames used by the audited notebook's first code cell are:

| Market | Original input CSV filename in the notebook |
|---|---|
| Natural Gas | `Kontrakty terminowe na gaz ziemny.csv` |
| Brent | `Kontrakty terminowe na ropę Brent (LCOX6).csv` |
| Copper | `Kontrakty terminowe na miedź (HGZ6).csv` |
| EUA | `European Union Carbon Permits Allowance (EUA) Yearly Futures.csv` |

All four input readers use `pd.read_csv(file_path, encoding="utf-8-sig")` and the same originally Polish column names:

| Source CSV column | Internal column | Handling |
|---|---|---|
| `Data` | `Date` | Parsed as `DD.MM.YYYY` (`%d.%m.%Y`) |
| `Ostatnio` | `Close` | Text price converted to float (comma decimals supported) |
| `Otwarcie` | `Open` | As above |
| `Max.` | `High` | As above |
| `Min.` | `Low` | As above |
| `Wol.` | `Volume` | Values with `K`, `M`, `B` suffixes converted to numbers |
| `Zmiana%` | `Change_pct` | `%` removed; converted to a numeric value |

The notebook sorts input observations by `Date` in ascending order and audits OHLC consistency, missing data and duplicate dates. **Do not silently repair, remove, winsorize, interpolate or replace observations** to force a desired result. These specific imported continuous series were used as supplied by the provider; contract maturities and historical roll adjustments were not independently reconstructed. Different source versions may give different numerical results.

### Chronological partitions and reference checks

- **Train:** through 2016-12-31.
- **Validation:** 2017-01-01 through 2019-12-31.
- **Test:** 2020-01-01 through the end of the available 2026 series.
- **Forecast target:** next-observation Parkinson range-based variance, `Target_Parkinson_t1`.
- **Main regime construction:** five-session recent Parkinson volatility, rolling 252-observation history, causal Q33/Q67 thresholds.
- **ML predictors:** the predefined set of 27 features.

After feature engineering and sample construction, the following **effective model sample counts** were reported in the original study:

| Market | Train | Validation | Test |
|---|---:|---:|---:|
| Natural Gas | 1,765 | 790 | 1,766 |
| Brent | 1,772 | 774 | 1,736 |
| Copper | 1,735 | 770 | 1,772 |
| EUA | 1,766 | 770 | 1,728 |

These are **modeled** row counts, not raw CSV row counts. Investigate discrepancies in the source vintage, trading calendars and preprocessing; do **not** modify observations to match the table.

## 3. Environment and installation

The four audited notebooks report **Python 3.10.20** and a `garch_fix` kernel. Their imported third-party packages are covered by the accompanying `requirements.txt`: `numpy`, `pandas`, `matplotlib`, `scipy`, `scikit-learn`, `statsmodels`, `arch`, `xgboost`, `IPython`, `ipykernel`, and `jupyterlab`. Some conditional paths call local `step13_2_loghar` and `step13_3_structural_breaks` modules; these are **local Python modules, not packages from PyPI**, and their function bodies are also embedded in the final notebook cells.

From the repository root, open a terminal and run:

```bash
python -m venv .venv
```

Activate the environment (Windows PowerShell):

```powershell
.\.venv\Scripts\Activate.ps1
```

Or activate it on Linux/macOS:

```bash
source .venv/bin/activate
```

Install and launch:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name commodity-volatility --display-name "Python (commodity-volatility)"
python -m jupyterlab
```

Select the `Python (commodity-volatility)` kernel. To record **versions in the author's original computational environment**, run in that original environment's **terminal**:

```bash
python --version
python -m pip freeze > requirements-lock.txt
```

Do not claim that a lock file generated in a *new* environment records the original published versions. The notebooks themselves do not establish precise historical versions for all dependencies.

## 4. Input paths and a single consistent results base

The supplied notebooks' first cells contain the author's absolute Windows/OneDrive input paths. **These paths must be edited by an external researcher**, or replaced with configurable relative paths in a public copy. For instance, if a Jupyter kernel runs from the repository's `notebooks/` directory:

```python
from pathlib import Path
ROOT = Path.cwd().resolve().parent
file_path = ROOT / "data" / "raw" / "EUA.csv"  # substitute market/input filename
```

If `Path.cwd()` points to the repository root, set `ROOT = Path.cwd().resolve()` instead. **This example changes only input paths, not the data definitions or methods.** The `data/raw/` directory must remain local and ignored by Git.

**Critical output-path issue:** original STEP 12 cells commonly use **relative** `Path("REVISION_RESULTS") / MARKET.replace(" ", "_")`, while the last post-review STEP 13 cells hard-code `C:\Users\UE\REVISION_RESULTS`. These can silently point at different directories. Choose a common absolute location before running any analysis, and adjust path-only assignments accordingly. For example:

```python
from pathlib import Path
BASE = Path.cwd().resolve() / "REVISION_RESULTS"  # check actual notebook working directory
BASE.mkdir(parents=True, exist_ok=True)
```

Make sure all STEP 12 outputs and subsequent STEP 13 inputs resolve to the **same** base and these folders:

```text
REVISION_RESULTS/
├── Natural_Gas/
├── Brent/
├── Copper/
└── EUA/
```

For original relative output references to resolve to that same base, the kernel working directory must be the chosen parent directory, or the output path assignments in STEP 12 cells must be changed to use the chosen `BASE`. **Setting `BASE` in one new cell does not automatically rewrite earlier hard-coded or relative paths.** Never change model parameters, data transformations, selection rules or dates merely for portability.

## 5. Main forecasting workflow — common to all four markets

Run each notebook in its own kernel and in the logical order below. The table lists **zero-based cell positions in the audited copies**. Unlabeled intervening cells belonging to a stage are part of that stage; execute its relevant consecutive code cells.

| Order | Task / notebook STEP | Natural Gas | Brent | Copper | EUA |
|---:|---|---:|---:|---:|---:|
| 1 | Load CSV and normalize source columns | 0–1 | 0–1 | 0–1 | 0–1 |
| 2 | STEP 2–4: features, returns/volatility and causal regime diagnostics | 2–10 | 2–10 | 2–10 | 2–10 |
| 3 | STEP 5: Train / Validation / Test labels and summaries | 11–15 | 11–15 | 11–15 | 11–15 |
| 4 | STEP 6: the fixed 27-feature modeling dataset and checks | 16–28 | 16–28 | 16–28 | 16–28 |
| 5 | STEP 7: expanding-window HAR and Validation evaluation | 29–40 | 29–40 | 29–42 | 29–43 |
| 6 | STEP 8: GARCH / EGARCH / GJR-GARCH specification selection on Validation | 41–59 | 41–61 | 43–63 | 44–64 |
| 7 | STEP 9: RF / XGBoost / SVR Train-only tuning, Validation forecasts and inference | 60–107 | 62–109 | 64–111 | 65–112 |
| 8 | STEP 10: Validation-estimated equal/static/regime weights and switching rules | 108–117 | 110–119 | 112–121 | 113–122 |
| 9 | STEP 11.1–11.13: frozen one-day Test forecasts and combination predictions | 118–152 | 120–150 | 122–153 | 123–154 |
| 10 | STEP 11.14–11.17: primary Test scorecards, DM / Holm and regime comparisons | 159, 164–177 | 162–175 | 165–177 | 167–180 |

**Important physical-order exception:** the revised STEP 12 cells were inserted **between** the STEP 11.13 code and most STEP 11.14–11.17 summaries. To reproduce the *complete revised analysis*, after finishing STEP 11.13 use **Section 6 below**, then execute the STEP 11.14–11.17 summary cells (if not yet executed). For a baseline-only run you can execute the main STEP 11 summary cells as long as their variables and references are defined. Do not use a blind `Run All` as a substitute for checking dependencies.

Main pipeline checkpoints should include `df` (processed daily OHLC/volatility), `model_df_complete` (final aligned feature/target rows), the frozen Validation choices and a final Test prediction table (commonly `all_test_forecasts`). Validate sample partitions, no look-ahead, finite forecasts, and target alignment before continuing.

## 6. Revised STEP 12 — prerequisites for final STEP 13

Run after original STEP 11.13, in the order below. These are **large combined code cells**, not necessarily standalone imported `.py` modules.

| Order | Revised analysis | Natural Gas | Brent | Copper | EUA | Key output / dependency |
|---:|---|---:|---:|---:|---:|---|
| 1 | Stage-1 revision: STEP 12.0–12.2 — data/date boundary audit, strict-QLIKE positivity and Validation-only calibration | 154 | 152 | 155 | 156 | `12_0_*`, `12_1_*`, `12_2_*` |
| 2 | STEP 12.3 — Validation-boundary combination sensitivity | 155 | 153 | 156 | 157 | `12_3_*` |
| 3 | STEP 12.4 — chronology and GARCH-selection robustness | 156 | 154 | 157 | 158 | `12_4_*` |
| 4 | STEP 12.5A — original Test daily forecast archive and DM/Holm / combination analyses | 158 | 156 | 159 | 160 | **`12_5A_daily_forecasts.csv`**, required by final expanded MCS |
| 5 | STEP 12.5B — calibrated-GARCH comparison | 160 | 157 | 160 | 161 | Depends on STEP 12.2 calibration |
| 6 | STEP 12.6 — inverse-MSE weights and original HAR/Test archive | 162 | 160 | 162 | 163 | **`12_6_inverse_MSE_daily_forecasts.csv`**, needed for log-HAR vs HAR |
| 7 | STEP 12.7 — supplementary return-variance mapping and VaR backtests | 163 | 161 | 163 | 166 | `12_7_*`; EUA has an additional weight-consistency diagnostic in cell 165 |

Intervening cells include minor audits, print statements and **fast-resume utilities** (e.g. Natural Gas 161, Brent 159, Copper 161, EUA 162). Fast resume is designed to restore previously saved state when restarting a session; it is **not a mandatory step during a successful continuous run**. Run only when its checkpoint prerequisites are available and you understand what state it replaces. Similarly, execute any intermediary small setup/audit cells *when a subsequent section explicitly uses their variables*; they do not represent alternative experimental specifications.

Inspect output directories after each of the two key archives:

```text
REVISION_RESULTS/<MARKET_FOLDER>/12_5A_daily_forecasts.csv
REVISION_RESULTS/<MARKET_FOLDER>/12_6_inverse_MSE_daily_forecasts.csv
```

Here `<MARKET_FOLDER>` is `Natural_Gas`, `Brent`, `Copper`, or `EUA`, as specified in Section 1. These archives may include dated true target values and should **not be uploaded wholesale to a public repository** without a redistribution-rights review.

## 7. Other original robustness and editorial checks

The notebooks also contain robustness and revision experiments appended after the core Test evaluation. These checks are **not substitutes for the original primary model selection** and are not all mandatory if the sole aim is to reproduce the original forecast/combination tables and the final three STEP 13 analyses.

| Block | Natural Gas | Brent | Copper | EUA | Description |
|---|---:|---:|---:|---:|---|
| Additional original STEP 12 | 178–211 | 176–209 | 178–211 | 181–214 | Subperiod stability, alternative regimes, parsimonious ARCH, alternative-loss comparisons |
| Earlier notebook-labeled STEP 13 | 212–217 | 210–216 | 212–219 | 215–220 | **Positivity sensitivity** — not the final post-review STEP 13 below |
| STEP 14 / editor-requested experiments | 218–225 | 217–238 | 220–237 | 221–228 | Original smaller MCS and/or alternate models, regime-window tests, Markov and RF-with-regime diagnostics |
| STEP 15 / editor-requested data checks | 226–231 | 239–244 | 238–243 | 229–234 | OHLC and roll-candidate diagnostics, alternative volatility proxies |
| Other single-market/editorial diagnostics | 232 | 245–251 | 244 | 235–236 | Ad hoc checks, figure generation or cross-market saved-notebook audit |

**Numbering collision:** the **older STEP 13.1–13.6** refers to a positivity-transformation sensitivity. The **final post-review STEP 13.1–13.3** appears in the last cells of each notebook and refers instead to **expanded MCS, shifted log-HAR, and structural-stability diagnostics**. Locate these final cells by the exact opening docstrings described in Section 8.

Not all editor-requested cells can be run from a single market's data alone. Some require historical saved Validation/Test checkpoints or other notebooks. Some Brent-only cells test a positive Tweedie alternative: **do not substitute Tweedie forecasts into the original primary expanded MCS**. Only run optional blocks that correspond to a declared robustness analysis; document their input dependencies and keep the original frozen selection intact.

## 8. Final post-review STEP 13.1–13.3 — four-market execution map

The three final analyses have matching function implementations in all four notebooks, but different cell positions:

| Final analysis | Natural Gas | Brent | Copper | EUA | Input needed |
|---|---:|---:|---:|---:|---|
| STEP 13.1 — expanded full-pool MCS | **233** | **252** | **245** | **237** | Original `12_5A_daily_forecasts.csv` |
| STEP 13.2 — shifted log-HAR | **234** | **253** | **246** | **238** | `model_df_complete`; original `12_6_inverse_MSE_daily_forecasts.csv` for paired comparison |
| STEP 13.3 — event-date Wald and CUSUM | **235** | **254** | **247** | **239** | Processed `df` with `Date` and `Parkinson_Var` |
| Combined runner with external `.py` imports — **skip if using embedded functions** | 236 | 255 | 248 | 240 | Tries to import local modules not provided by the notebook alone |

**The final run cells in some markets have a misleading copied `BRENT` comment**, despite passing the correct market argument. Do not use that comment as evidence of the market; inspect `MARKET`, `OUT` and `market=` before executing. The extra Natural Gas cell **237** repeats the structural-break call; avoid duplicating the run. EUA cell **241** contains shell commands incorrectly entered as Python and must **not** be run.

### 8.1 STEP 13.1: expanded full-pool MCS

1. Confirm `REVISION_RESULTS/<MARKET_FOLDER>/12_5A_daily_forecasts.csv` exists and contains the original, frozen **entire Test** sample.
2. In the relevant final STEP 13.1 cell set:

   ```python
   MARKET = "EUA"                 # change to the notebook's own market
   BASE = Path(r"YOUR_ABSOLUTE_REVISION_RESULTS_PATH")
   REPS = 3000
   INCLUDE_INVERSE_MSE = False
   SEED = 20261020
   ```

3. Execute that cell **once**. It defines and invokes `run_mcs()` via `if __name__ == "__main__":`. Do not invoke `run_mcs()` a second time unnecessarily.
4. Check the printed strict-QLIKE exclusion audit. An inadmissible **forecast strategy** is excluded from the full Test comparison when it has a nonfinite/nonpositive forecast; the code does not remove individual observations to make that strategy admissible.
5. Inspect files written under `REVISION_RESULTS/<MARKET_FOLDER>/STEP13_NEW/`:

   - `13_1_MCS_original_full_pool_results.csv`
   - `13_1_MCS_original_full_pool_audit.csv`
   - `13_1_MCS_original_full_pool_daily_losses.csv` (keep local unless sharing rights have been reviewed)

Method settings used by the embedded code: `arch.bootstrap.MCS` with method `"R"`, stationary bootstrap, 3,000 replicates, fixed seed and block length based on the Test sample. **This MCS uses original Test forecasts**, not the alternative Brent Tweedie sensitivity.

### 8.2 STEP 13.2: supplementary shifted log-HAR

1. Confirm the original data-preparation cells were executed through STEP 6.4 and `model_df_complete` exists.
2. Confirm the **matching market's** `12_6_inverse_MSE_daily_forecasts.csv` archive exists from STEP 12.6; it contains the frozen original HAR Test forecasts used for the paired comparison.
3. Execute the notebook's final STEP 13.2 **function-definition** cell from the mapping above. **Defining the function does not run it.**
4. In a separate code cell, set `BASE` to the same absolute results directory and execute:

   ```python
   from pathlib import Path
   BASE = Path(r"YOUR_ABSOLUTE_REVISION_RESULTS_PATH")
   loghar_daily, loghar_performance = run_loghar(
       model_df_complete,
       market="EUA",  # substitute Natural Gas / Brent / Copper / EUA
       base=BASE,
   )
   ```

5. Verify the returned Validation/Test counts, strictly positive forecasts, and one-to-one Test-date and target alignment against the original HAR archive. Do not force-match by deleting dates or replacing another market's CSV.
6. Expected outputs in `STEP13_NEW/`:

   - `13_2_LogHAR_Validation_Test_forecasts.csv` (daily, keep private pending license review)
   - `13_2_LogHAR_performance.csv`
   - `13_2_LogHAR_vs_original_HAR_HAC.csv` (provided the original HAR comparison file exists and aligns)

The supplementary log-HAR uses a Train-fixed positive offset and expanding-window fitting. Its original HAR comparator is **not** re-estimated or tuned using the Test set. The paired QLIKE differential uses an HAC(7) mean-loss test and is **not** the same as the original DM-HLN family.

### 8.3 STEP 13.3: predefined breaks and exploratory CUSUM

1. Confirm processed `df` exists with `Date` and `Parkinson_Var`; the appropriate market's Train-only volatility scale is computed by the embedded procedure.
2. Execute the final STEP 13.3 **function-definition** cell.
3. In a separate cell run:

   ```python
   break_results, cusum_results = run_break_diagnostics(
       df,
       market="EUA",  # substitute the active market
       base=BASE,
   )
   ```

4. Check outputs:

   - `13_3_predefined_event_break_tests.csv`
   - `13_3_global_CUSUM_diagnostic.csv`

Predefined event cutoffs are **2020-03-11** (COVID-19), **2021-09-01** (European energy-crisis proxy), and, for **EUA only**, **2021-01-01** (EU ETS Phase IV). Joint AR(1) parameter-stability Wald tests use HAC(7) and Holm adjustments within each market; CUSUM is exploratory. **These are retrospective diagnostics, not causal identification or an excuse to retune the primary models.**

## 9. Review output files and compare with manuscript tables

Validate the run independently for each market:

1. Check model sample sizes against Section 2 and Test dates against the source vintage.
2. Verify forecasting chronology and that the target of each Test row was unavailable at its forecast origin.
3. Check that frozen Validation-estimated hyperparameters, GARCH specifications, combination weights and switching choices were not changed using final Test information.
4. Verify that full-Test strict QLIKE admissibility is evaluated at the **strategy** level, with exclusion reasons saved in the audit file.
5. Verify log-HAR and original HAR are compared on exactly the same dates and target values, without dropping mismatch rows.
6. Compare summary values in `12_5A_performance.csv`, `12_6_inverse_MSE_performance.csv`, `13_1_MCS_original_full_pool_results.csv`, `13_2_LogHAR_performance.csv`, `13_2_LogHAR_vs_original_HAR_HAC.csv`, and `13_3_*` with the corresponding main-text or appendix tables.
7. Record the precise data version, environment, executed notebook revision, and any execution failures. **Do not assert numerical identity if the market data differ from the originally used continuous-futures history.**

## 10. Public release and data-license safeguards

- Do **not** publish the source OHLC CSV files without appropriate redistribution authorization.
- Keep full dated realized-variance, daily-return and daily-loss/forecast archives private unless their publication has been separately reviewed for compliance. Files named `12_5A_daily_forecasts.csv`, `12_6_inverse_MSE_daily_forecasts.csv`, `13_2_LogHAR_Validation_Test_forecasts.csv`, or equivalent are **not automatically safe to publish** merely because they are generated by Python.
- Clear saved Jupyter cell outputs that display original OHLC rows or other sensitive/provider-derived series; do not remove the computation cells themselves.
- Preserve aggregate model-performance tables, audit summaries and tested configurations where rights and research-integrity checks allow publication.
- `.gitignore` does not erase data committed in earlier Git history. Inspect public commit history before advertising a repository as sanitized.
- When the reproducibility package is stable, identify its Git revision or release tag in the manuscript Data Availability Statement; a Zenodo DOI may be added if archived.

## 11. Current limitations

The notebooks contain local-path assumptions, duplicated stage identifiers and optional exploratory cells that are not a single guaranteed `Run All` workflow. The present guide identifies their logical execution and interdependencies, but **a clean-environment run has not been performed** using the original historical data. Any portability changes should affect file paths, module loading, or output placement only; they must not silently alter the study design, observations or primary inference.
