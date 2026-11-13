**Volatility-Regime Forecast Combinations**

**About the Study**

This repository contains the Python implementation and supporting reproducibility materials for the manuscript:

Do Volatility Regimes Improve Forecast Combinations? Evidence from Energy, Copper, and Carbon Markets

The study evaluates one-day-ahead Parkinson variance forecasts for four futures markets:

Natural Gas

Brent Crude Oil

Copper

European Union Allowances (EUA)

The methodology includes HAR, GARCH-family models, Random Forest, XGBoost, SVR, and several forecast-combination strategies.

**Research Design**

The sample covers 2010–2026 and follows a chronological Train–Validation–Test framework:

Train: observations through December 2016

Validation: 2017–2019

Test: 2020–2026

Forecast performance is evaluated using QLIKE, MAE, RMSE, statistical inference procedures, and supplementary robustness analyses.

**Data Availability**

The original daily OHLC observations were obtained from Investing.com. These third-party market data are not redistributed through this repository.

Researchers wishing to reproduce the analyses should independently obtain the corresponding historical futures series from the original provider.

The provider's continuous-futures contract construction and subsequent historical data revisions may affect exact numerical replication.

**Reproducibility**

The repository is intended to provide model specifications, analytical code, hyperparameter settings, statistical evaluation procedures, and documentation of the required input data.

The complete execution order, input requirements, and environment specifications will be documented before the repository's public release.

