# **Institutional-Grade Portfolio Optimization with Walk-Forward Validation and Advanced Risk Metrics**

#### Run this Notebook in the Cloud

| Binder | Google Colab |
| :---: | :---: |
| [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/maxiveloso/python-portfolio-optimization/main?filepath=portfolio_optimization_case_study.ipynb) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/maxiveloso/python-portfolio-optimization/blob/main/portfolio_optimization_case_study.ipynb) |

#### Author
AI-assisted implementation by Maximiliano Veloso

---

### Purpose

This repository showcases an end-to-end, production-style portfolio optimization pipeline focused on empirical rigor, risk-aware modeling, and out-of-sample validation. It is designed as a technical case study for roles in financial analysis and data analytics, emphasizing:

- Robust data validation and reproducibility
- Advanced risk-adjusted optimization beyond mean-variance
- Systematic stock selection
- Walk-forward validation and performance monitoring
- Transparent benchmarking and attribution

*Executive highlights are summarized for completeness, but the README is primarily aimed at technical reviewers and hiring managers.*

---

### Key Results

- Annual Return: 48.48% (increase vs. benchmark: 27.91 pp)
- Annual Volatility: 39.30% (increase: 13.66 pp)
- Sharpe Ratio: 1.18 (increase: 0.46)
- Sortino Ratio: 1.65 (increase: 0.69)
- Max Drawdown: -62.58% (decrease: -26.96 pp)
- Information Ratio: 1.36 (increase: 1.36)
- Alpha (annual): 21.29% (increase: 21.29 pp)
- Beta: 1.36 (increase: 0.36)
- Out-of-Sample (OOS) Annual Return: 9.20%
- OOS Sharpe: 0.24

Benchmark used: QQQ. Risk-free rate: 2%.

Note on selected assets: The final portfolio constituents are selected via a documented selection algorithm (see “Stock Selection Methodology”) and depend on realized returns, covariance, correlation penalties, and constraints over the specified lookback. For the main run in this repo, example selected assets were:
- NVDA, META, NFLX, CRM
with a Max Sharpe constrained solution approximately:
- NVDA: 40.00%
- META: 24.11%
- NFLX: 27.62%
- CRM: 8.28%

---

### Data Insights and Analysis

From the ingested data, average daily returns across the tech universe were 0.147% with 2.738% volatility and 0.542 average correlation, informing the correlation penalty in stock selection. This analysis revealed growth-heavy assets like NVDA driving performance but increasing tail risks, as seen in CVaR metrics.

---

### Interpretation and Discussion

- The Max Sharpe solution under the given constraints delivered strong in-sample risk-adjusted performance (Sharpe ≈ 1.18; Sortino ≈ 1.65), with meaningful alpha (≈ 21.29%/yr) and IR (≈ 1.36) relative to QQQ.
- The profile is high beta (≈ 1.36) with elevated drawdowns (≈ -62.6%), consistent with a growth-heavy tech allocation and concentration constraints.
- OOS results are modest (Sharpe ≈ 0.24), highlighting the challenge of persistent alpha in live conditions and validating the need for robust walk-forward and monitoring.
- This repository is intentionally conservative in claims; it demonstrates process, controls, and measurement, not hindsight overfit performance.
- These results support risk-aware allocation strategies for institutional clients, e.g., capping weights at 40% to mitigate concentration risk while targeting alpha of 21.29% vs. QQQ.

---

### Why This Case Study Is Relevant for Financial/Data Analyst Roles

This case study is highly relevant for financial and data analyst roles because it demonstrates a complete analytical workflow from start to finish. 

It begins with **data cleaning**, moves on to building well-reasoned models, and then applies **constraint-aware optimization** to those models. The study emphasizes the importance of a robust validation process, using **out-of-sample (OOS) validation** and a **walk-forward analysis** to assess the model's ability to generalize to new data and to prevent **overfitting**. 

Beyond just performance metrics, it also shows an understanding of real-world constraints by incorporating **risk monitoring** and factoring in **transaction costs**. Finally, the entire project is structured to be production-friendly, featuring robust logging, clear parameterization, and a focus on reproducibility.

---

### What This Repo Contains

- Core notebook/script: portfolio_optimization_case_study.json
  - Configuration, parameters, and logging
  - Data ingestion and validation
  - KPI and advanced risk metrics
  - Optimization (Max Sharpe, Min Variance, Risk Parity) with constraints
  - Systematic stock selection (exhaustive/greedy, correlation penalty)
  - Rebalancing simulation with transaction costs
  - Walk-forward evaluation (train/test rolling windows)
  - Dashboard-style visualization functions

- Data dependency: CSVs under data/ for the specified tickers and QQQ. Prices are expected as Close Price column with Date index.

---

### Technical Stack

- Python: pandas, numpy, scipy.optimize, matplotlib, seaborn, plotly
- Reproducibility: seeded RNG, deterministic selection/optimization settings
- Inputs: 10-stock tech universe by default (AAPL, MSFT, GOOGL, AMZN, NVDA, META, TSLA, NFLX, AMD, CRM)
- Constraints: long-only, no leverage, max weight = 40%, monthly rebalancing, 10 bps per side transaction costs
- Scalability: The pipeline handles larger universes via greedy selection (O(n) complexity) instead of exhaustive combinatorics, with seeded RNG for reproducible Monte Carlo simulations (20,000 scenarios).

---

### Data Pipeline and Validation

- Source: CSV files (Close Price) ingested via load_prices_from_csv
- Validation:
  - Date alignment, missing value handling (ffill/bfill)
  - Duplicate removal
  - Minimum data length
  - Outlier flags (>50% daily change)
  - Constant-series removal
- Returns: daily pct_change() with Inf removal and NA handling

---

### Stock Selection Methodology

Function: select_optimal_stocks(prices_df, N_stocks, lookback_days, method='exhaustive'|'greedy')

- Windowing: looks back over a configurable window (e.g., last 2 years or TRAIN_WINDOW)
- Objectives: for each candidate subset, solve a constrained Max Sharpe optimization
- Correlation awareness:
  - Exhaustive: evaluates all combinations if small universe
  - Greedy: iteratively builds the set; scores are penalized by average off-diagonal correlation:
    score = Sharpe × (1 − avg_corr)
- Output: selected tickers and the optimal weights for that subset

This is not a naive top-4-by-Sharpe-at-asset-level selection and does not default to [NVDA, TSLA, AAPL, MSFT]. The selection depends on realized returns, covariance, correlation penalties, and constraints over the specified lookback.

---

### Optimization Engine

Function: optimize_weights(returns_df, objective, bounds, sum_to, method)

- Objectives:
  - Max Sharpe (primary)
  - Min Variance
  - Risk Parity (equal risk contribution via variance-based RC)
- Constraints:
  - Long-only bounds [0, MAX_WEIGHT]
  - Sum to 100%
  - Optional turnover penalty
- Solvers:
  - SLSQP via SciPy (first attempt)
  - Monte Carlo fallback with Dirichlet sampling under bounds

Portfolio statistics are derived consistently from daily returns with proper annualization.

---

### Risk and Performance Metrics

Function: calculate_comprehensive_kpis(returns, benchmark_returns, rf)

- Annualized return, volatility
- Sharpe, Sortino (downside volatility annualized)
- Max drawdown (path-based)
- VaR(95%) and CVaR(95%) annualized from daily distribution
- Alpha/Beta relative to benchmark
- Information Ratio and tracking error (via kpis_vs_benchmark)


#### Statistical and Analytical Techniques
Incorporated statistical methods including percentile-based VaR/CVaR for tail risk assessment and linear regression for alpha/beta estimation, enabling benchmark-relative insights (e.g., portfolio beta of 1.36 indicating amplified market exposure).

---

### Walk-Forward Validation

Function: walk_forward_evaluate(prices_df, train_window, test_window, ...)

- Rolling folds:
  - Train on rolling window (default 252 days)
  - Select assets
  - Optimize weights
  - Simulate out-of-sample (default 63 days) with rebalancing and costs
- Aggregation:
  - Concatenate OOS NAV across folds
  - Report aggregate OOS KPIs (e.g., OOS Sharpe ≈ 0.24)

Interpretation: Solid in-sample efficiency does not always translate one-for-one to OOS; this is expected and is the reason walk-forward is included.

---

### Rebalancing and Costs

Function: simulate_rebalance(prices_df, weights_schedule, cost_per_side)

- Applies rebalancing at specified frequency (monthly by default)
- Computes turnover and transaction costs (cost_per_side + slippage) × turnover × NAV
- Produces net NAV and KPIs after costs

---

### Visualizations

- Performance dashboard (Plotly):
  - NAV vs Benchmark, allocation, rolling drawdown, metric comparison
- Risk radar:
  - Sharpe, Sortino, IR, VaR, CVaR, Max DD
- Walk-forward plots:
  - OOS NAV evolution, bar charts for IS vs OOS Sharpe by fold

These figures support technical diagnostics and communication during reviews. Interactive Plotly dashboards provide dynamic views of NAV evolution, risk radars, and fold-wise Sharpe comparisons, facilitating stakeholder presentations.

---

### Limitations and Next Steps

- Return distribution assumptions and non-Gaussian tails: VaR/CVaR derived from empirical daily returns; extreme events may be undercounted.
- Transaction costs are simplified (bps per side). Slippage modeling can be expanded.
- Alternative objectives (Sortino-optimized, CVaR-minimizing, drawdown-constrained) can be added.
- Regime-aware models and time-varying covariance estimates (e.g., EWMA, DCC-GARCH) could improve OOS stability.
- Feature-driven selection (fundamental/alternative data) and Bayesian shrinkage can reduce noise.


---

### Reproducibility

- Seeds fixed (RNG_SEED)
- Deterministic selection/optimization flows where possible
- Version-locked parameters in the configuration section

---

### Contact

For questions or to discuss extensions (e.g., factor models, turnover control, or regime detection), please reach out via the repository’s issue tracker or LinkedIn.
