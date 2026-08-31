# DCA vs Lump-Sum Crisis Backtester

A Python script that compares two core investment strategies — **Dollar-Cost Averaging (DCA)** and **Lump-Sum** — using real historical S&P 500 data (via the **SPY** ETF), across two different market regimes: a crash-and-recovery period and a steady bull market.

The goal isn't to declare one strategy universally "better," but to show **when** and **why** each one wins, using correct financial math (XIRR, cash-flow-adjusted returns, drawdown).

---

## What DCA and Lump-Sum mean here

- **Lump-Sum**: you invest the entire amount at once, on the first available day.
- **DCA**: you split the same total amount into equal monthly installments over a set number of months (default: 60 months / 5 years).

Both strategies invest **the same total amount** ($10,000 by default) — the only difference is the timing.

---

## How the script works

1. **Downloads price data** (adjusted close, dividends included) from Yahoo Finance via `yfinance`, for each scenario defined in the `SCENARIOS` list.
2. **Computes the Lump-Sum portfolio**: buys all shares on day 1 and tracks their value through to the end.
3. **Computes the DCA portfolio**: buys new shares every month with the same fixed amount, accumulating over time.
4. **Computes correct returns**:
   - **XIRR** (annualized return) for both strategies, accounting for exactly when each dollar was invested (money-weighted return).
   - **Cash-flow-adjusted daily returns** for DCA, so a new monthly contribution isn't mistaken for actual investment performance.
5. **Computes risk metrics**: annualized volatility and maximum drawdown.
6. **Generates an interactive chart** (Plotly) showing both portfolios' evolution and DCA's purchase points.
7. **Prints a summary table** combining all scenarios.

---

## Why SPY instead of the S&P 500 index (^GSPC)

SPY is a real, investable ETF — something you could actually buy. Its *adjusted* prices also include dividends, so the comparison reflects total return, not just price return.

## Why XIRR instead of a simple % return

With DCA, capital enters gradually — so each installment has a different amount of "time in the market." A simple `(final value / total invested) - 1` doesn't capture the true annual rate of return. **XIRR** solves for the rate where NPV = 0 across irregular cash flows, giving the correct annualized, money-weighted return — the one you'd actually see in a real account.

---

## Results (current scenarios)

### Scenario 1: 2007-2012 (Crash & Recovery)

| Metric | Lump-Sum | DCA |
|---|---|---|
| Invested | $10,000 | $10,000 |
| Final Value | $10,384.59 | $13,072.24 |
| Profit | $384.59 | $3,072.24 |
| Total Return | 3.85% | 30.72% |
| Annualized Return (XIRR) | 0.76% | 10.69% |
| Volatility | 26.43% | 26.29% |
| Max Drawdown | -55.19% | -54.37% |

**Winner: DCA** — because the investment started just before the 2008 crash, lump-sum got "locked in" at high prices. DCA, on the other hand, kept buying shares at much lower prices throughout the downturn, lowering the average cost basis and achieving a much better final return.

### Scenario 2: 2015-2019 (Bull Market)

| Metric | Lump-Sum | DCA |
|---|---|---|
| Invested | $10,000 | $10,000 |
| Final Value | $17,289.49 | $14,220.05 |
| Profit | $7,289.49 | $4,220.05 |
| Total Return | 72.89% | 42.20% |
| Annualized Return (XIRR) | 11.59% | 14.07% |
| Volatility | 13.43% | 13.38% |
| Max Drawdown | -19.35% | -19.39% |

**Winner: Lump-Sum** (by total return) — in a steadily rising market, investing everything immediately means more capital spends more time in the market. DCA missed part of the rally because it spread purchases out over 5 years.

> **Note:** DCA has a *lower* total return but a *higher* XIRR (14.07% > 11.59%). This isn't a contradiction: total return measures how much the capital grew overall, while XIRR measures the annual rate of return weighted by when each installment was invested. Because DCA's money went in later on average, it spent less time invested but at a more efficient annualized rate.

---

## Overall takeaway

With only two scenarios, no universal rule can be drawn, but the results confirm something widely documented (e.g. in Vanguard studies):

- **Lump-Sum** tends to win in steadily rising markets, because more time in the market usually means more return.
- **DCA** specifically protects you when the market drops shortly after you start investing, because it avoids locking in all your capital at high prices.
- Risk (volatility, max drawdown) ends up nearly identical between the two strategies once all purchases are complete, since both portfolios end up holding the same asset. The main difference is **entry-timing risk**, not the subsequent volatility.

For a stronger conclusion, you'd need a rolling backtest across many decades (e.g. every possible 5-year window from 1990-2024) to see what percentage of the time each strategy wins.

---

## Installation & Usage

```bash
pip install yfinance plotly pandas numpy scipy
python DCA_vs_LumpSum_Crisis_Backtester.py
```

The script runs best in a Jupyter/Colab notebook because of the `fig.show()` (Plotly) and `display()` (pandas) calls at the end.

### Configuration

At the top of the file you can change:

```python
SCENARIOS = [
    {"label": "...", "ticker": "SPY", "start": "YYYY-MM-DD", "end": "YYYY-MM-DD"},
]
INITIAL_INVESTMENT = 10_000.0   # total amount invested
N_MONTHS = 60                    # number of monthly installments for DCA
```

You can add as many scenarios as you like (different tickers, dates, crises) — the script runs all of them and prints a combined summary table at the end.

---

## Limitations / What it does NOT do

- Doesn't account for taxes, transaction fees, or bid-ask spread.
- Doesn't simulate dividend reinvestment separately (it's already baked into SPY's adjusted prices).
- Not investment advice — this is an educational/research tool built on historical data, which does not guarantee future returns.
