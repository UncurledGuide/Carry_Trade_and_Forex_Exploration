# Carry Trade & Forex Exploration

Research repo for a JPY/CHF/MXN/CNY carry trade analysis pipeline built for an economics research lab. The project progresses from single-pair currency analysis through multi-asset portfolio construction to formal robustness testing.

---

## Repository Structure

```
Carry_Trade_and_Forex_Exploration/
│
├── currency_analysis.ipynb       # Single-pair carry analysis (JPY/USD)
├── portfolio_construction.ipynb  # Multi-currency portfolio with weighting schemes
├── robustness_testing.ipynb      # Walk-forward, VaR/CVaR, Monte Carlo stress tests
├── portfolio_comparison.html     # Interactive dashboard: strategy performance comparison
└── robustness_dashboard.html     # Interactive dashboard: robustness test outputs
```

---

## Notebook Overview

### 1 — `currency_analysis.ipynb`
Builds and analyzes a JPY-funded USD carry trade over a 40-year window using FRED data.

- Pulls live data via the FRED CSV API: USD/JPY spot rate (`DEXJPUS`), US effective federal funds rate (`EFFR`), and Japanese short-term rate (`IRSTCI01JPM156N`)
- Computes daily carry returns using ACT/360 daycount convention: borrowing in JPY, investing in USD at the overnight rate
- Extends to three strategy variants:
  - **Overnight cash carry** — EFFR vs. JP short rate
  - **Bond carry** — US 2Y Treasury yield approximation vs. JP funding
  - **Equity carry** — JPY-funded long US equities (S&P 500), FX unhedged
- Plots rolling 1Q, 2Q, 1Y, and 3Y compounded returns across all three variants
- Computes full-sample and rolling Sharpe ratios (1Y and 3Y windows) for the equity carry strategy

### 2 — `portfolio_construction.ipynb`
Extends the single-pair framework to a multi-currency carry portfolio.

- Adds a CHF-funded USD carry leg (`DEXSZUS`, `IRSTCI01CHM156N`) alongside the JPY leg
- Further expands to include **MXN reverse carry** (borrow USD, invest MXN) and **CNY reverse carry** — capturing both funding and target sides of the carry spectrum
- Compares three portfolio construction methods:
  - **Equal weight** (50/50 JPY/CHF)
  - **Inverse volatility weighting** — 60-day rolling vol lookback, weights shift(1) to avoid lookahead
  - **Max-Sharpe optimization** — arrives at the optimal static allocation: **70% JPY / 27.4% MXN reverse / 2.6% CNY reverse** (in-sample Sharpe: 0.864)
- Outputs a performance summary table: annualized return, annualized volatility, Sharpe ratio for each strategy

### 3 — `robustness_testing.ipynb`
Stress-tests the optimal 70/27.4/2.6 portfolio using three complementary methods.

**Part 1 — Walk-Forward Optimization**
- Uses a 3-year training window, re-optimizing Sharpe-maximizing weights every 6 months
- Bounds: JPY weight constrained to [30%, 80%]; MXN/CNY weights to [0%, 50%]
- Compares out-of-sample walk-forward returns vs. static optimal vs. JPY-only across annualized return, vol, Sharpe, and max drawdown

**Part 2 — Value at Risk (VaR) & CVaR**
- Computes daily VaR and CVaR at 90%, 95%, and 99% confidence using three methods:
  - Historical (empirical percentile)
  - Parametric (Gaussian)
  - Cornish-Fisher (adjusts for skewness and excess kurtosis)
- Scales to 1-day, 1-week, 1-month, and 3-month horizons for a $100,000 notional portfolio

**Part 3 — Monte Carlo Simulation**
- Three simulation approaches:
  - Parametric multivariate normal
  - Block bootstrap (preserves autocorrelation and fat tails)
  - Regime-switching (separate calm/crisis distributions)
- Builds confidence intervals on max drawdown and worst-case loss across holding periods

---

## Data Sources

All data pulled live from FRED (St. Louis Fed) via public CSV API — no API key required.

| Series | Description |
|---|---|
| `DEXJPUS` | USD/JPY spot rate |
| `DEXSZUS` | USD/CHF spot rate |
| `DEXMXUS` | USD/MXN spot rate |
| `DEXCHUS` | USD/CNY spot rate |
| `EFFR` | US effective federal funds rate (daily) |
| `IRSTCI01JPM156N` | Japan short-term interest rate (monthly) |
| `IRSTCI01CHM156N` | Switzerland short-term rate (monthly) |
| `IRSTCI01MXM156N` | Mexico short-term rate (monthly) |
| `IRSTCI01CNM156N` | China short-term rate (monthly) |
| `DGS2` | US 2-year Treasury yield |
| `SP500` | S&P 500 price index |

---

## Key Findings

- The JPY-funded equity carry strategy (borrow JPY, hold US equities unhedged) significantly outperforms the pure overnight cash carry over multi-year horizons, with higher rolling Sharpe ratios across 1Y and 3Y windows
- Adding CHF and MXN reverse carry legs meaningfully improves diversification; inverse-vol weighting and max-Sharpe optimization both outperform equal weighting
- The optimal 70/27.4/2.6 portfolio achieves an in-sample Sharpe of **0.864** — walk-forward testing confirms the allocation is robust out-of-sample (weights remain stable across re-optimization periods)
- CVaR analysis highlights the fat left tail characteristic of carry strategies: Cornish-Fisher adjustments produce materially worse tail estimates than the Gaussian assumption

---

## Dependencies

```bash
pip install pandas numpy matplotlib scipy
```

---

## About

Work for an economics research lab at UC San Diego. Notebooks pull live FRED data and are designed to be re-run at any time for updated results.
