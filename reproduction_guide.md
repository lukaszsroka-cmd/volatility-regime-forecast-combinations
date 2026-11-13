# Reproduction Guide — EUA Futures

**Study:** *Do Volatility Regimes Improve Forecast Combinations? Evidence from Energy, Copper, and Carbon Markets*  
**Scope of this document:** The actual EUA notebook supplied for review. The other three market notebooks have not been audited for their exact cell sequence in this guide.

## 1. Reproducibility status and scope

This notebook contains the main EUA analysis and a series of later robustness and editorial-review exercises. It is **not yet a fully one-click, clean-environment reproduction package**: there are absolute Windows paths, external CSV/checkpoint dependencies, an external-module import near the end, and a final cell containing shell commands rather than valid Python. The instructions below identify those requirements; they do not claim that a fresh end-to-end execution has been verified.

The main scientific specification and Test-period results should be kept frozen. Any path or environment changes made for public reproduction must be limited to file loading, output destinations, and execution infrastructure—not to model selection, target construction, observation handling, or Test-based tuning.

## 2. Required input data

Obtain the **same historical daily continuous-futures OHLC series for EUA** independently from Investing.com, subject to the provider's applicable terms. The notebook's original input file is named:

`European Union Carbon Permits Allowance (EUA) Yearly Futures.csv`

The notebook's initial reader requires `utf-8-sig` and these **original Polish column labels**:

| Source header | Converted to | Role |
|---|---|---|
| `Data` | `Date` | Dates formatted `DD.MM.YYYY` |
| `Ostatnio` | `Close` | Closing price |
| `Otwarcie` | `Open` | Opening price |
| `Max.` | `High` | Daily maximum |
| `Min.` | `Low` | Daily minimum |
| `Wol.` | `Volume` | Volume; parser recognizes `K`, `M`, `B` suffixes |
| `Zmiana%` | `Change_pct` | Percentage change, with `%` removed |

Price fields use comma-to-dot conversion. Dates are parsed using `format="%d.%m.%Y"`; the original input is then sorted in ascending date order. The primary target is **next-day Parkinson variance**, derived from High and Low. The notebook also calculates close-to-close returns, Garman–Klass variance and auxiliary features.

The original study's sample spans **25 January 2010 to 22 September 2026**; the analytical split is Train through **31 December 2016**, Validation **2017–2019**, and Test beginning **1 January 2020**. The EUA notebook's effective modeled observation counts are **Train 1,766 / Validation 770 / Test 1,728**. These counts must be checked against the local input data; if they differ, do not silently remove, interpolate, alter or substitute observations to make them match.

**Do not commit the third-party OHLC CSV to the public repository without appropriate redistribution rights.** Do not upload saved notebook outputs displaying raw OHLC, complete daily returns, or actual daily Parkinson-variance series without checking applicable rights. Aggregate summary tables can be reviewed separately for publication.

## 3. Python environment

Notebook metadata reports **Python 3.10.20**, kernel `garch_fix` (`Python (GARCH FIX)`). The notebook does **not** record the installed versions of its third-party libraries. Therefore the accompanying `requirements.txt` lists observed package dependencies but does not pretend to reproduce the original, exact package versions.

From a terminal in the repository root:

```bash
python -m venv .venv
```

Activate on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

On Linux/macOS:

```bash
source .venv/bin/activate
```

Then:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name volatility-eua --display-name "Python (volatility-eua)"
python -m jupyterlab
```

Select the new kernel in Jupyter. To capture the **actual versions in the environment where results were originally generated**, execute the following in that environment's **terminal**, not in a plain Python notebook cell:

```bash
python --version
python -m pip freeze > requirements-lock.txt
```

A lock file generated from a newly created environment records the *new* versions; it should not be misrepresented as the historical environment used for the published calculations. Operating-system and BLAS/numerical-library differences can also affect exact numerical replication.

## 4. Path configuration — required before execution

**The original notebook contains absolute paths** pointing to the author's Windows/OneDrive workspace and revision outputs. This is a **known portability blocker**. Do not expect a cloned GitHub notebook to run unchanged.

For a public, portable copy, place the independently obtained input locally at, for example, `data/raw/EUA.csv` (a local, ignored folder) and replace only the input path in the first cell. The exact path must be calculated relative to the **actual kernel working directory** (`Path.cwd()`), not assumed.

Example configuration if the Jupyter kernel starts inside `notebooks/`:

```python
from pathlib import Path
ROOT = Path.cwd().resolve().parent
file_path = ROOT / "data" / "raw" / "EUA.csv"
```

If the kernel starts in the repository root, use `ROOT = Path.cwd().resolve()` instead. The source file must preserve the header and value formats described above.

**Output-directory consistency is critical.** Many original STEP 12 cells write to `Path("REVISION_RESULTS") / "EUA"` relative to the kernel's working directory; later post-review STEP 13 cells instead use the hard-coded `C:\Users\UE\REVISION_RESULTS`. Before running these sections, point their output/input paths to one common location, e.g.:

```python
from pathlib import Path
BASE = Path("REVISION_RESULTS").resolve()
EUA_DIR = BASE / "EUA"
```

Use that same `BASE` in the final STEP 13 MCS and in the `base=` arguments to log-HAR and structural-break functions. Preserve the existing input/output file names; **do not change the statistical procedures**. The instructions below assume `REVISION_RESULTS/EUA/` consistently refers to one directory.

## 5. Cell-by-cell execution map (zero-based indices)

**Cell index** means position in the `.ipynb` JSON, starting from **0**, not Jupyter's `[In n]` execution counter. Locate the stage by its heading if the notebook is edited later. Several code cells have no headings, so complete consecutive groups in order. The sections described as *optional* are not prerequisites for the core frozen forecast pipeline unless they supply a saved checkpoint explicitly used by a later analysis.

### A. Main analysis: execute in physical notebook order

| Order | Cells (0-based) | Code headings / task | Required outcome |
|---:|---|---|---|
| 1 | **0–1** | Load source CSV; rename Polish columns, convert types, date sort, basic OHLC audit | `df_raw`, `df` |
| 2 | **2–10** | STEP 2 feature engineering; STEP 3 diagnostics; STEP 4 causal regime identification and summaries | Daily Parkinson variance, returns, features, regime labels |
| 3 | **11–15** | STEP 5.1–5.5 chronological Train / Validation / Test designation | `Sample` labels |
| 4 | **16–29** | STEP 6.1–6.13 final 27-feature model sample and chronological / missingness / target-alignment checks | `model_df_complete`, `X_train`, `X_validation`, `X_test`, corresponding targets |
| 5 | **30–43** | Common evaluation functions, fixed configurations; STEP 7 expanding-window HAR and Validation evaluation | Original HAR Validation forecasts |
| 6 | **44–64** | STEP 8 return-based GARCH / EGARCH / GJR-GARCH candidates; Training mean selection, Validation grid and diagnostics | Frozen GARCH-family specifications and Validation forecasts |
| 7 | **65–112** | STEP 9 RF / XGBoost / SVR; five-fold time-series CV; Validation walk-forward forecasts; performance and statistical tests | `all_validation_forecasts`, frozen ML hyperparameters |
| 8 | **113–122** | STEP 10: EW3, static QLIKE weights, regime-specific weights, regime-switching rules chosen on Validation | Frozen combination rules |
| 9 | **123–154** | STEP 11.1–11.13: Test setup, expanding HAR / ML / GARCH forecasts, alignment checks, apply frozen combinations | `all_test_forecasts` with original Test predictions |
| 10 | **155** | Reference-parameter audit against manuscript tables | Optional audit; inspect any discrepancies rather than changing frozen models |
| 11 | **167–180** | STEP 11.14–11.17: final Test performance, DM-HLN / Holm, regime-conditional tests | Main manuscript comparisons |

**Important:** In the uploaded notebook cells 156–166 (revised STEP 12 blocks) physically occur **before** cells 167–180. The above is the *logical* sequence of the main forecasting analysis. Do not indiscriminately click **Run All** to reproduce only the main results: the notebook contains later optional blocks with external-file prerequisites.

### B. Core revision blocks — execute after STEP 11.13

These cells are physically located immediately after cell 155. For a complete revision workflow, execute in the following order after the original forecasts have been created:

| Order | Cells | Task | Files / dependencies |
|---:|---|---|---|
| 1 | **156** | Revised STEP 12.0–12.2: boundary checks, strict-positivity audit, Validation-only GARCH-to-Parkinson scale calibration | Writes `12_0_*`, `12_1_*`, `12_2_*` to `REVISION_RESULTS/EUA/` |
| 2 | **157** | STEP 12.3: Validation boundary sensitivity for weights/switch rules | Writes `12_3_*` |
| 3 | **158** | STEP 12.4: Train/Validation chronology and GARCH-selection robustness | Writes `12_4_*` |
| 4 | **159** | Restore/verify `statsmodels.api` alias `sm` | Required by subsequent inference if `sm` was overwritten |
| 5 | **160** | STEP 12.5A: original full-Test forecast archive, DM/Holm and combination comparisons | **Creates `12_5A_daily_forecasts.csv`**, required by expanded MCS |
| 6 | **161** | STEP 12.5B: calibrated-GARCH performance and DM/Holm | Requires STEP 12.2 calibration outputs |
| 7 | **162** | *Optional fast-resume utility* for restoring state from saved CSVs | Not needed for a successful continuous run; may require independently saved original Validation input |
| 8 | **163** | STEP 12.6: inverse-MSE combination (Validation weights) and original HAR Test archive | **Creates `12_6_inverse_MSE_daily_forecasts.csv`**, used for the post-review log-HAR paired comparison |
| 9 | **164–165** | Diagnostic checks of weights / saved consistency | Auxiliary audits |
| 10 | **166** | STEP 12.7: return-variance mapping and one-day VaR backtests | Uses frozen forecasts and Validation-derived calibration |

After the above, run **167–180** for the original final summary/inference if it has not already been run. Do not change original forecasting and weighting rules on the basis of these sensitivity analyses.

### C. Further original robustness and editorial analyses

| Cells | Content | Status |
|---|---|---|
| **181–188** | STEP 12.0 helpers; STEP 12.1 temporal subperiod performance and stability | Supplementary; after original Test forecasts |
| **189–199** | STEP 12.2–12.3 alternate causal regime definitions (10-day, Q25/Q75) | Supplementary; maintain Validation-only rule fitting; cell 198 repeats a regime robustness call — inspect before rerunning |
| **200–210** | STEP 12.4 parsimonious ARCH variants and comparison | Supplementary; computationally demanding, separate from frozen primary selection |
| **211–214** | STEP 12.5 alternative-loss-function comparisons | Supplementary |
| **215–220** | An **earlier** notebook-labeled STEP 13.1–13.6 positivity sensitivity | **Different from the later, post-review STEP 13.1–13.3** |
| **221–228** | STEP 14 MCS / other editorial experiments, including Markov-switching and RF-with-regime features | Optional, with additional saved-file prerequisites; not the expanded final MCS |
| **229–234** | STEP 15 OHLC / rollover-candidate / alternative-proxy audits | Optional review outputs; some saved outputs contain provider-derived observations and should not automatically be public |
| **235** | STEP 16 Holm-family audit based on saved `.ipynb` materials for **multiple markets** | Optional, requires external original notebooks/checkpoints; cannot be reproduced from EUA alone |
| **236** | Ad hoc `model.get_params()` diagnostic | Skip in automated execution; the `model` binding is not guaranteed |

**Numbering caveat:** The final post-review STEP 13 scripts appear at the very end (cells 237–240). They are distinct from the older positivity sensitivity labeled STEP 13 in cells 215–220 and from the older three-model MCS in cell 221.

## 6. Final post-review STEP 13.1–13.3: executable procedure

### 6.1 Expanded full-pool MCS — cell 237

Prerequisite: the **original, unmodified** `REVISION_RESULTS/EUA/12_5A_daily_forecasts.csv` from cell 160. The code reads it and applies strict QLIKE admissibility (complete, finite, positive forecast vectors). The primary analysis does not substitute any alternative Tweedie forecasts or retune original models.

Before execution, set within cell 237:

```python
MARKET = "EUA"
BASE = Path("REVISION_RESULTS").resolve()   # same folder used by STEP 12
REPS = 3000
INCLUDE_INVERSE_MSE = False
SEED = 20261020
```

The cell defines `run_mcs()` and calls it when `__name__ == "__main__"` (as normally in a notebook). Alternatively invoke `run_mcs()` once explicitly; **do not inadvertently run it twice**.

Expected outputs in `REVISION_RESULTS/EUA/STEP13_NEW/`:

- `13_1_MCS_original_full_pool_results.csv`
- `13_1_MCS_original_full_pool_audit.csv`
- `13_1_MCS_original_full_pool_daily_losses.csv` — retain locally unless distribution rights have been reviewed

The final Test sample must contain **1,728** observations for EUA. Record the set of excluded strategies and their exact reasons rather than silently removing individual observations.

### 6.2 Shifted log-HAR — cell 238

Prerequisite: `model_df_complete` from STEP 6.4 and the **unaltered original** `REVISION_RESULTS/EUA/12_6_inverse_MSE_daily_forecasts.csv` from cell 163 (needed to produce the paired HAR comparison). The function is **defined inside cell 238**; executing that cell does not by itself run the calculation.

After running cell 238, execute a separate Python cell:

```python
from pathlib import Path
BASE = Path("REVISION_RESULTS").resolve()
loghar_daily, loghar_performance = run_loghar(
    model_df_complete,
    market="EUA",
    base=BASE,
)
```

Expected outputs:

- `13_2_LogHAR_Validation_Test_forecasts.csv` — local daily series; avoid public redistribution without review
- `13_2_LogHAR_performance.csv` — aggregate Validation/Test measures
- `13_2_LogHAR_vs_original_HAR_HAC.csv` — paired comparison **only if** the original HAR CSV exists and the dates/targets align

The implementation freezes the offset using Train only (`0.01 * median positive Train Parkinson variance`), refits the supplementary log-HAR with expanding historical data, and exponentiates log predictions without subtracting the offset. It does **not** replace the primary HAR benchmark.

### 6.3 Structural stability — cell 239

Prerequisite: the original processed `df` including `Date` and `Parkinson_Var`. Cell 239 defines `run_break_diagnostics`, but does not run it automatically.

After running cell 239:

```python
from pathlib import Path
BASE = Path("REVISION_RESULTS").resolve()
break_results, cusum_results = run_break_diagnostics(
    df,
    market="EUA",
    base=BASE,
)
```

Expected outputs:

- `13_3_predefined_event_break_tests.csv`
- `13_3_global_CUSUM_diagnostic.csv`

The EUA study uses predefined cutoffs **2020-03-11** (COVID-19), **2021-01-01** (EU ETS Phase IV), and **2021-09-01** (European energy-crisis onset proxy), HAC(7) Wald tests with Holm adjustment within the market, and exploratory OLS CUSUM. These are stability diagnostics, **not** causal attribution or Test-based model selection.

### 6.4 Why cell 240 currently needs adaptation

Cell **240** imports `run_loghar` from `step13_2_loghar` and `run_break_diagnostics` from `step13_3_structural_breaks` via a hard-coded `STEP13_CODE` directory. **Those external `.py` modules are not supplied by this `.ipynb` file**, although the function definitions themselves appear in cells 238 and 239.

To keep everything in the notebook, **use the direct function calls in sections 6.2 and 6.3 instead of running cell 240 as-is**. Alternatively export the complete definitions from cells 238/239 as correctly named `.py` modules and adjust the import directory. Do not claim the original cell 240 is portable without either action.

### 6.5 Cell 241 is invalid Python — do not run

The cell contains shell commands entered as Python, including the incorrect phrase `python -m pip install freeze`. This causes `SyntaxError` in a Python kernel. Use the **terminal** commands in Section 3 instead. Do not execute cell 241 as written.

## 7. Expected key outputs and validation checks

| Deliverable | Generating section | Public sharing guidance |
|---|---|---|
| Base Test QLIKE/DM/Holm tables | Original STEP 11.14–11.17 (cells 167–180) | Aggregate summaries generally lower risk |
| Validation-only calibration/positivity and boundary reports | Revised STEP 12 (cells 156–161) | Publish summary results; review dated observation-level details |
| Inverse-MSE summary and weighting results | STEP 12.6 (cell 163) | Summaries lower risk; **not** entire daily forecast/target archive by default |
| VaR summary/backtests | STEP 12.7 (cell 166) | Aggregate outputs generally lower risk |
| Expanded MCS | Final STEP 13.1 (cell 237) | Share results/audit; assess detailed daily loss series separately |
| Log-HAR and HAC summary | Final STEP 13.2 (cell 238 + function call) | Share performance and HAC summary; review daily series |
| Predefined-break Wald / CUSUM | Final STEP 13.3 (cell 239 + function call) | Share aggregate diagnostic outputs |

Verify without adjusting observations to force a match:

1. EUA modeled sample counts: **1,766 / 770 / 1,728** (Train / Validation / Test).
2. All forecast origins and training rows satisfy chronology; no current Test target is used for fitting or weight selection.
3. Strict QLIKE receives nonnegative actual Parkinson variances and strictly positive admissible predictions; invalid model series are reported as excluded from MCS rather than selectively deleting days.
4. The dates and target values in original HAR/Test CSVs match the corresponding log-HAR Test evaluation rows one-to-one.
5. Original model parameters/weights, MCS random seed, bootstrap method and replication count remain unchanged when preparing public material.
6. Compare aggregate outputs to the final manuscript tables; do not claim numerical identity if a different continuous-futures vintage was used.

## 8. Publication-safe repository preparation

- Keep provider-supplied OHLC files **out** of GitHub unless redistribution is permitted.
- Strip notebook **saved outputs** that display original OHLC, exact daily realized volatility, or returns. Keep the computational code.
- Avoid committing `REVISION_RESULTS/` wholesale: it includes CSVs containing dated realized targets and daily forecast/loss series.
- Review **Git commit history** if restricted data or notebook outputs were committed in earlier public versions; removing a file in the latest commit does not remove historical versions.
- Add `.gitignore` rules for local data, checkpoints, environment folders, and sensitive output series.
- For archival reproducibility, publish a reviewed GitHub release (and optionally a Zenodo DOI) identifying the code version corresponding to the manuscript.

## 9. Known limitations of this guide

This guide was compiled by inspecting the provided EUA notebook **without executing its full computational workflow** and without the proprietary raw OHLC source or all saved original intermediate inputs. Its cell indices refer **only to this uploaded revision**. In particular, the direct function-call pathway at the end must be validated locally after the common output base and prerequisite CSVs have been set consistently. Different markets may have different optional checkpoints and should receive their own verification before claiming a single universal run procedure.
