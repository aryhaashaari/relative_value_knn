# QQQ–SPY Relative-Value Strategy with kNN

Documentation for the Python strategy supplied in the attachment `Pasted text.txt`. The supplied file is unchanged. No numerical backtest results were included with it.

## Overview

The script explores whether a k-nearest neighbours (kNN) classifier can identify weeks when the QQQ/SPY spread's rolling z-score moves closer to zero.

It downloads QQQ and SPY prices, samples weekly closing prices, builds spread and individual ETF features, and trains a classifier on the first 75% of observations. The remaining 25% is used to generate signals and performance charts.

**Status:** research prototype. The current backtest has timing, direction, and portfolio accounting problems. Its reported returns should not be interpreted as evidence of a tradable edge.

## Requirements and execution

Use Python 3 with these packages:

```sh
python -m pip install yfinance pandas numpy matplotlib seaborn scikit-learn statsmodels
```

Save the supplied Python code as `strategy.py`, then run:

```sh
python strategy.py
```

Internet access is required for the price download. A graphical environment is needed to display the plots. These commands are usage instructions; the strategy has not been executed or validated as part of this documentation.

## Data and features

- **Assets:** QQQ and SPY.
- **Requested history:** January 1, 1990 onward; usable history begins when both assets have data.
- **Sampling:** last available close in each Friday-ending week.
- **Spread:** `log(QQQ / SPY)`.
- **Spread features:** spread level, 52-week z-score, one- and four-week changes, and four-week volatility.
- **ETF features:** one- and four-week returns, four-week volatility, and distance from 10- and 40-week moving averages.

The script also prints an Engle–Granger cointegration test on price levels. This test does not establish stationarity of the specific log-ratio spread. The comments describing the spread and all features as stationary are assumptions, not demonstrated findings.

The download has no fixed end date or explicit adjustment setting. Record the data cutoff, package versions, and adjustment settings when producing reproducible results. Exclude an unfinished weekly bar from evaluation.

## Model and signals

The target is 1 when next week's absolute spread z-score is smaller than this week's. Otherwise it is 0.

| Setting | Value |
| --- | --- |
| Training/test split | Chronological 75% / 25% |
| Scaling | StandardScaler fitted on training observations |
| Classifier | KNeighborsClassifier |
| Neighbours | 25 |
| Weighting | Inverse distance |
| Entry threshold | Predicted narrowing probability greater than 0.55 |
| Implemented position | Short QQQ / long SPY when active; flat otherwise |

The script also scans neighbour counts of 5, 10, 15, 20, 25, 35, 50, 75, and 100 on the same test period.

## Outputs

The script prints the cointegration p-value, observation count, target base rate, classification accuracy, signal frequency, annualised Sharpe ratios, maximum drawdowns, and terminal growth multiples.

It displays three plots:

1. Strategy and raw-spread cumulative growth.
2. Spread z-score with signal markers.
3. Sharpe ratio and investment frequency across neighbour counts.

The values labelled “Cumulative Return” are terminal wealth multiples: 1.20 represents a 20% gain under the script's accounting, not a 120% return. Classification accuracy uses the classifier's default decision rule, whereas trades use the separate 0.55 probability threshold.

## Known backtest limitations

### Signal and return timing

Features at week `t` include that week's closing prices, but the strategy applies the resulting signal to the spread change from `t-1` to `t`. That return occurred before the signal was available. Signals must instead be matched to returns after an achievable execution time. Simply shifting returns does not resolve execution assumptions if entry is still assumed at the same close used to calculate the signal.

### Target and position direction

The model predicts shrinking absolute z-scores, but always shorts QQQ and buys SPY. When the z-score is negative, a mean-reversion position generally requires the opposite direction. Rolling means and volatility can also cause the z-score to shrink without a profitable spread price move.

### Portfolio returns and benchmark

The script compounds a difference in log returns as though it were a portfolio's simple return. A valid account simulation needs explicit leg weights, capital, gross exposure, rebalancing, and financing assumptions, using each leg's actual return.

The raw-spread benchmark uses positive QQQ-minus-SPY log returns, while its chart label describes short QQQ / long SPY. The label and calculation have opposite directions.

Maximum drawdown is calculated without including the initial wealth of 1.0 as a starting observation, which can miss an initial loss from starting capital.

### Costs and validation

Transaction costs, slippage, short borrowing, financing, and dividend obligations are not explicitly modelled. Price adjustment settings must be consistent with the chosen dividend accounting to avoid omissions or double counting.

Selecting the best neighbour count from the test-period sensitivity scan would use the test set for model selection. Use time-ordered training and validation windows, ensure training labels are available before each forecast, and reserve a final untouched test period. The full-sample cointegration test is also unsuitable as an ex-ante selection rule without restricting it to information available at the time.

## Requirements for a credible evaluation

1. Correct signal timing and specify executable entries and exits.
2. Align position direction and prediction target with the intended spread trade.
3. Validate the chosen spread's mean-reversion behaviour using historical information available at each decision point.
4. Calculate account returns from explicit leg weights and include realistic trading and holding costs.
5. Tune parameters using time-ordered validation and evaluate once on untouched data.
6. Compare with cash, constant exposure in the same direction, and a simple z-score trading rule.
7. Report net CAGR, volatility, Sharpe, maximum drawdown, exposure, turnover, trade count, and performance by subperiod, with sensitivity to parameters and costs.

Until those changes are made and results are supplied, profitability and robustness remain unproven.
