# SVI Calibration of the SPY Implied Volatility Smile

This project calibrates the raw SVI (Stochastic Volatility Inspired) model to the implied-volatility smile of SPY options for a single expiry.

The aim is to construct a clean volatility smile from market option quotes, estimate the relevant forward price, calibrate the SVI parameters using constrained nonlinear least squares, and evaluate how well the fitted curve reproduces observed implied volatilities.

## Overview

The analysis uses SPY option-chain data for options expiring on **18 December 2026**.

The workflow includes:

- Filtering the option chain to a selected strike range
- Removing invalid bid and ask quotes
- Constructing call and put mid prices
- Restricting the analysis to a single expiry
- Estimating the SPY forward price using near-ATM put-call parity
- Using OTM puts below the forward and OTM calls above the forward
- Converting strike into forward log-moneyness
- Converting implied volatility into total implied variance
- Calibrating the five raw SVI parameters
- Comparing fitted and observed implied volatilities
- Checking fitted total-variance positivity
- Testing for butterfly arbitrage across the calibrated moneyness range
- Analysing calibration residuals

## Raw SVI Model

For a fixed maturity, raw SVI models total implied variance as

$$ w(k) = a + b\left[\rho(k-m) + \sqrt{(k-m)^2+\sigma^2} \right] $$

where

$$ k = \log\left(\frac{K}{F}\right) $$

is forward log-moneyness.

The five parameters control the level, slope, skew, horizontal location, and curvature of the implied-volatility smile.

## Methodology

### 1. Option-chain cleaning

The analysis focuses on SPY strikes between **650 and 850**.

Call and put mid prices are calculated from quoted bid and ask prices, and observations with invalid quotes are removed.

### 2. Single-expiry selection

SVI is calibrated independently for each maturity, so the dataset is restricted to options expiring on **18 December 2026**.

### 3. Forward estimation

The forward price is estimated from near-at-the-money call and put prices using put-call parity:

$$ F \approx K + e^{rT}(C-P) $$

A median estimate across nearby strikes is used to reduce sensitivity to individual quote noise.

The estimated forward price is approximately **768.56**.

### 4. Smile construction

To construct the market smile:

- OTM puts are used below the forward
- OTM calls are used above the forward

Implied volatility is then converted into total implied variance:

$$ w = \sigma_{\text{imp}}^2 T $$

### 5. SVI calibration

The SVI parameters are estimated using constrained nonlinear least squares by minimizing the difference between fitted and observed total variance.

## Results

The calibrated parameters are approximately:

- `a = -0.001664`
- `b = 0.058292`
- `rho = -0.572245`
- `m = 0.017535`
- `sigma = 0.118705`

The final implied-volatility RMSE is approximately:

**0.000213**

The fitted SVI curve closely tracks the observed SPY implied-volatility smile across the selected strike range.

The negative value of `rho` is consistent with the pronounced downside volatility skew typically observed in broad equity-market options.

## No-Arbitrage Diagnostics

A low calibration error does not by itself guarantee that a fitted volatility smile is economically admissible.

The calibrated SVI slice is therefore evaluated on a dense log-moneyness grid.

Across the calibrated range:

- Fitted total variance remains positive
- The minimum fitted total variance is approximately **0.00401**
- The butterfly-arbitrage diagnostic satisfies \(g(k) > 0\) throughout the evaluation grid
- The minimum observed value of \(g(k)\) is approximately **0.302**

These checks indicate that the fitted single-expiry SVI slice is free of butterfly arbitrage over the evaluated moneyness grid.

Because the project considers only one maturity, calendar-arbitrage conditions across expiries are outside the scope of this analysis.

## Interpretation

The project illustrates how an option volatility smile can be represented using a compact parametric model rather than a separate implied-volatility observation at every strike.

Residual analysis is used to identify regions where the fitted parametric curve differs from observed market prices and to assess the quality of the calibration.

In addition to achieving a low calibration error, the fitted slice passes basic static-arbitrage diagnostics over the calibrated range. This provides an economic validity check beyond statistical goodness of fit.

## Tools

- Python
- pandas
- NumPy
- SciPy
- Matplotlib

## Data

The analysis uses a manually downloaded SPY option-chain snapshot from Cboe delayed market data.

The market-data CSV used in the analysis is included in this repository so that the notebook can be reproduced directly.

## Notebook

See [`svi_spy_calibration.ipynb`](svi_spy_calibration.ipynb) for the full analysis.
