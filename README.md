# Momentum Index Construction (CRSP/S&P 500)

A from-scratch backtest of a classic **cross-sectional momentum strategy** on U.S. equities, built on CRSP daily stock data combined with historical S&P 500 index membership. The project replicates the well-documented 12-2 momentum anomaly (Jegadeesh & Titman, 1993; further studied by AQR Capital Management, e.g. Asness, Moskowitz & Pedersen, *"Value and Momentum Everywhere"*), and stress-tests it against realistic transaction costs.

## Context

This notebook implements the **"Establishing a Momentum Index"** and **"Implementing the Momentum Strategy"** sections of a university report — *"The UMD Factor: Evidence, Robustness, and Mutual Fund Implications"* (Theory of Finance course project, 2026) — which evaluates the economic viability of an AQR-style momentum mutual fund from the perspective of an investment decision made in early 2009.

The full report additionally covers the historical performance of the Fama-French UMD factor (1927–2008) and a long-only factor regression, which are **not** part of this repository. This repo only contains the code that builds the long-only momentum index and tests it against transaction costs — i.e. the part of the analysis that produces Figures 5–8 of the report.

## What it does

1. **Data cleaning** — loads daily CRSP returns and market cap for S&P 500 constituents, filters out stock-months with less than 75% trading-day coverage.
2. **Monthly aggregation** — compounds daily returns into monthly returns; tracks start- and end-of-month market equity.
3. **Momentum signal** — classic 12-2 momentum: cumulative return from month *t-12* to *t-2*, skipping the most recent month to avoid short-term reversal. Requires at least 13 consecutive months of valid history.
4. **Benchmark** — builds its own value-weighted S&P 500 benchmark directly from the dataset (start-of-month cap weights).
5. **Strategy** — each rebalance date, ranks all eligible stocks by momentum and goes long the **top decile (10%)**, weighted by start-of-month market cap. Tested at three rebalancing frequencies: **monthly, quarterly, annual** — quarterly is adopted as the reference specification, as it offers the best trade-off between return, turnover, and robustness to transaction costs for a mutual-fund-like implementation.
6. **Transaction costs** — a second, more realistic version adds:
   - a linear cost (in bps) on portfolio turnover at each rebalance,
   - an additional cost for "forced exits" (stocks dropping out of the investable universe mid-holding-period),
   - sensitivity analysis at **15 / 25 / 30 bps** one-way cost, calibrated on real-world execution cost estimates from Frazzini, Israel & Moskowitz (2018), *"Trading Costs"*, based on $1.7 trillion of AQR's live executed trades.
7. **Performance metrics** — total/annualized return, annualized volatility, Sharpe ratio, max drawdown, and the **momentum premium** (strategy return minus S&P 500 benchmark return) over the common backtest period.

## Results (common period: 1992–2008, 204 months)

**Gross of transaction costs**

| Rebalancing | Ann. Return | Ann. Vol | Sharpe | Max DD |
|---|---|---|---|---|
| 3 months (reference) | 13.14% | 20.27% | 0.464 | -60.4% |
| 1 month  | 14.61% | 19.57% | 0.556 | -42.0% |
| 12 months| 10.83% | 21.01% | 0.338 | -66.4% |

S&P 500 benchmark over the same period: **7.27%** annualized.

**Net of transaction costs (30 bps one-way)**

| Rebalancing | Ann. Return | Sharpe | Momentum premium (ann.) |
|---|---|---|---|
| 3 months (reference) | 11.80% | 0.397 | +4.53pp |
| 1 month  | 11.77% | 0.411 | +3.05pp |
| 12 months| 10.32% | 0.314 | +4.50pp |

**Quarterly strategy vs. UMD factor vs. S&P 500** (gross of costs, 1992–2008):

| | Strategy (3M) | UMD factor | S&P 500 |
|---|---|---|---|
| Annualized return | 13.1% | 10.1% | 7.2% |
| Volatility (σ) | 20.3% | 14.2% | 20.2% |
| Sharpe ratio | 0.464 | 0.446 | 0.170 |

**Takeaway:** quarterly rebalancing offers the most robust trade-off once transaction costs are introduced — monthly rebalancing wins gross of costs but loses most of its edge as costs rise, while annual rebalancing is the most cost-resilient but structurally lower-return. The quarterly strategy also slightly outperforms the long-short UMD factor on a risk-adjusted basis, and both decisively beat the S&P 500.

## Repository structure

```
.
├── notebooks/
│   └── momentum_index_construction.ipynb   # main analysis notebook
├── README.md
├── requirements.txt
└── .gitignore
```

> **Note on data:** the underlying dataset (`final.csv`) comes from CRSP/WRDS, which requires an institutional subscription and cannot be redistributed. The data file is intentionally excluded from this repository (see `.gitignore`) — to reproduce the results you need your own WRDS/CRSP access (daily stock file + S&P 500 historical membership list).

## How to run

```bash
pip install -r requirements.txt
```

Place your own `final.csv` (with columns `PERMNO`, `MbrStartDt`, `MbrEndDt`, `DlyCalDt`, `DlyRet`, `DlyCap`) in the project root, then run the notebook top to bottom in Jupyter, JupyterLab, or Google Colab.

## Methodology notes & limitations

- Momentum is computed only for stocks with a full, gap-free history of at least 13 months — this introduces a mild survivorship/continuity bias.
- The Sharpe ratio uses a fixed average annualized risk-free rate (3.738%) rather than a time-varying one.
- Transaction costs are modeled linearly in turnover; market impact and short-side costs (this is a long-only strategy) are not modeled.
- The backtest window (1992–2008) is constrained by the data continuity requirements above — it does not include the post-2008 period.

## References

- Jegadeesh, N., & Titman, S. (1993). *Returns to Buying Winners and Selling Losers: Implications for Stock Market Efficiency.* Journal of Finance.
- Asness, C., Moskowitz, T., & Pedersen, L. (2013). *Value and Momentum Everywhere.* Journal of Finance.
- Frazzini, A., Israel, R., & Moskowitz, T. (2018). *Trading Costs.* Available at SSRN: https://ssrn.com/abstract=3229719

## Author

**Raffaele Perillo**
Code for *"The UMD Factor: Evidence, Robustness, and Mutual Fund Implications"* — Theory of Finance course project, 2026.

