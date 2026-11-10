# Option Pricing Engine

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/andrew-lynch3/Option_Pricing_Engine/blob/main/Option_Pricing_Engine.ipynb)

A Python notebook that prices options three independent ways (Black-Scholes, a Cox-Ross-Rubinstein binomial tree, and Monte Carlo simulation), checks the methods against each other, and then applies the engine to a **real option chain**: it backs out implied volatilities, plots the volatility smile, compares model prices with market quotes, and simulates **delta-hedging** a short option to show what the implied-vs-realized volatility gap is worth in dollars.

## What's inside

1. **Pricing engines:** closed-form Black-Scholes with all five Greeks, a binomial tree for European and American options, and Monte Carlo with antithetic variates (including an arithmetic Asian option with no closed-form price).
2. **Validation:** put-call parity, finite-difference checks on the Greeks, and convergence tests for the tree and Monte Carlo.
3. **Live market data:** a real AAPL option chain from `yfinance`, implied volatility solved with Brent's method, and the volatility smile against realized volatility.
4. **Model vs. market:** all three engines priced with one flat volatility and compared with bid/ask mid-prices.
5. **Trader outputs:** expected move, per-contract Greeks, and a P&L scenario grid.
6. **Delta-hedging simulation:** a short call hedged by simulation under different realized volatilities, with the P&L distribution and the effect of hedging frequency.

## Key results

Inputs for the live section come from **AAPL quotes captured on Nov 10, 2026 at about 3:47 pm ET** (spot $349.22, Dec 11 expiry, 31 days, 20 contracts priced from bid/ask mids). The 13-week T-bill yield was 4.12% and the trailing dividend yield was 0.23%.

### The three methods agree
The binomial tree converges to Black-Scholes as steps increase, with a characteristic oscillation. Monte Carlo falls within its own confidence interval.

![Binomial convergence](images/binomial_convergence.png)

### The market shows a skew, so one flat volatility is not enough
Out-of-the-money puts trade at much higher implied volatility than out-of-the-money calls, from about 34% at the 300 put to roughly 20% for upside calls. At-the-money implied volatility was 22.3% against 23.0% 30-day realized volatility.

![Volatility smile](images/volatility_smile.png)

Pricing every strike with one flat volatility misses the market by up to roughly $1.25 per share, well outside the bid-ask half-spread. All three engines agree with each other, so the gap comes from the model's constant-volatility assumption and not from a numerical error.

![Model vs. market](images/model_vs_market.png)

### Delta-hedging: the volatility gap in dollars
Short one 350 call sold at 22.3% implied volatility (a $925 premium per contract), delta-hedged daily (21 hedges, 50,000 simulated paths), results per contract:

| Realized volatility | Mean P&L | Std dev | 5th percentile | P(loss) | Vega × (IV − RV) |
|---|---|---|---|---|---|
| 23.0% (30-day realized) | -$25 | $174 | -$319 | 56% | -$25 |
| 22.3% (equal to implied) | about $0 | $169 | -$280 | 50% | $0 |
| 27.9% (1.25 × implied) | -$227 | $231 | -$658 | 88% | -$226 |

- The simulated average P&L matches the rule of thumb **vega × (implied − realized)**, and is about zero when realized volatility equals implied.
- Implied volatility was slightly *below* recent realized volatility here (by 0.6 of a volatility point), so the hedged seller loses a little on average: the volatility premium is not guaranteed.
- Hedging error shrinks like 1/√N in the number of rebalances, but the average is set by the volatility gap alone.

![Hedging P&L distribution](images/hedging_pnl_distribution.png)

![Hedging sweeps](images/hedging_sweeps.png)

## Run it

- **Colab:** click the badge above and choose Runtime → Run all.
- **Locally:** `pip install -r requirements.txt`, then open `Option_Pricing_Engine.ipynb` in Jupyter.

### Reproducibility
Market data changes daily, so the notebook has two modes. With `USE_SNAPSHOT = False` it pulls live data from `yfinance` and saves a snapshot to `data/`. With `USE_SNAPSHOT = True`, or automatically if the live pull fails, it loads the saved snapshot and reproduces the numbers above offline. Snapshots are only saved when real bid/ask quotes are available, never from stale last-traded prices.

## Limitations
- All three engines assume constant volatility and geometric Brownian motion (no jumps, no stochastic volatility).
- `yfinance` data is free and delayed; quotes can be stale or wide, and the filtering choices (OTM only, ±15% of spot, spread cap) affect the results.
- The spot price and option quotes may not be perfectly synchronized. This likely explains the small step in the smile where it switches from puts to calls at spot, and why the implied volatilities here differ from Yahoo's by up to about a volatility point or two. Backing out the forward price from put-call parity would reduce the effect.
- US equity options are American, but implied volatility is solved with a European model. This is a good approximation for out-of-the-money options and less so for in-the-money options and dividend payers.
- The hedging simulation uses one constant realized volatility, no transaction costs, and perfect fills.
- Model-vs-market differences are a diagnostic, not a trading signal.

## Possible extensions
Volatility surface across expiries, Longstaff-Schwartz for American Monte Carlo, a closed-form control variate for the Asian option, Heston stochastic volatility calibrated to the chain, transaction costs in the hedging study, and a C++ Monte Carlo core.
