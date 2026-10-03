# Retail Investment Strategy Backtester

Interactive app for comparing common retail investment strategies on historical equity data. Pick a stock, sector or custom ticker list, set the strategy rules, and compare the strategies side by side on return and risk metrics. Built with Python and Panel.

**Live demo:** [Hugging Face Space](https://huggingface.co/spaces/mildemx/investment-strategy-backtester)

For educational and analytical purposes only. Not investment advice.

## Run locally

```bash
pip install -r requirements.txt
panel serve interface.py --show
```

This starts a local server and opens the app in your browser.

## Features

- Interactive dashboard built with **Panel**, **HoloViews** and **hvPlot**
- Single-asset and multi-asset analysis
- Three ways to select stocks: predefined ticker list, GICS sector, manual ticker input
- Side-by-side strategy comparison with key performance metrics

## Strategies

Any combination of the following can be selected for comparison:

- **Dollar-Cost Averaging (DCA)**
  Invests a fixed amount on the first trading day of each month, regardless of price.

- **Double Down DCA**
  Invests twice the monthly amount when the price is at least a user-defined percentage below its rolling 52-week (252 trading day) high.

- **Lump Sum**
  Invests the full amount that standard DCA would invest over the period (number of months x monthly contribution) on the first day.

- **SMA DCA - Momentum**
  Invests only when the price is above its simple moving average.

- **SMA DCA - Mean Reversion**
  Invests only when the price is below its simple moving average.

- **Value Averaging**
  Targets a predefined portfolio growth path. Invests the shortfall when the portfolio is behind the target and nothing when it is ahead. It never sells.

### Parameters

- **Monthly Contribution ($):** the fixed amount invested each month.
- **Double Down Threshold:** the relative price drop from the rolling 52-week high that triggers the doubled contribution. A value of 0.15 means a drop of at least 15%.
- **SMA Period:** the number of trading days used for the simple moving average.
- **Desired Monthly Growth Rate (Value Averaging):** the target monthly growth rate of the portfolio.

## Key metrics

For each strategy the app reports:

- Total invested capital and final portfolio value
- Return on Investment (ROI)
- Internal Rate of Return (IRR)
- Compound Annual Growth Rate (CAGR)
- Maximum drawdown
- Calmar ratio
- Investment horizon (years)

## Methodology and assumptions

- **Data:** daily prices from Yahoo Finance via `yfinance`, adjusted for splits and dividends, so results are on a total-return basis.
- **Execution:** contributions are made on the first trading day of each month at that day's closing price. Fractional shares are allowed. There are no transaction costs, taxes or slippage.
- **Signals:** the SMA and drawdown signals use the same day's closing price at which the trade is made, with no execution lag. The Double Down signal needs 252 trading days of history, so no doubling occurs in the first year.
- **Capital deployed:** in the SMA strategies, months without a signal are skipped and that cash is not invested. ROI and CAGR are measured against the capital actually invested, so total capital differs across strategies.
- **ROI:** final value / total invested - 1.
- **CAGR:** (final value / total invested)^(1 / years) - 1, using the calendar years between the first and last monthly observation. Because capital is deployed gradually, IRR is the better money-weighted measure of return.
- **IRR:** computed from the monthly cash flows and annualised as (1 + monthly IRR)^12 - 1.
- **Maximum drawdown:** largest peak-to-trough fall in portfolio value, observed at monthly frequency. Portfolio value includes new contributions.
- **Calmar ratio:** CAGR / |maximum drawdown|.
- **Multi-asset portfolios:** the app builds one synthetic price series by taking the simple average of the selected tickers' adjusted prices on each day, and runs the strategy on that series as if it were a single asset. This is an average of share prices, not an equal-weight portfolio: a stock with a higher share price moves the series more than a cheaper one, regardless of percentage return. Holdings are not tracked per stock and nothing is rebalanced. A ticker with no data yet at the start (for example a later listing) is left out until its data begins.

## Limitations

- Results depend on historical data and say nothing about future returns.
- `yfinance` rate-limits requests, so running the simulation many times in quick succession may return no data.

## Project structure

```text
retail-investment-strategy-backtester/
├── interface.py            Panel app: inputs, strategy comparison, charts
├── dca_simulator/
│   ├── data_loader.py      Download and load price data (single and multiple tickers)
│   ├── data_processing.py  Cleaning and preparing price data
│   ├── strategies.py       DCA, Double Down, Lump Sum, SMA and Value Averaging strategies
│   ├── backtest.py         Backtesting logic used by the strategies
│   ├── metrics.py          ROI, CAGR, IRR, maximum drawdown, Calmar ratio
│   └── plots.py            Chart helpers
├── Project_python.ipynb    Notebook used for development and exploration
├── requirements.txt        Python dependencies
└── README.md
```

## Authors

Maxim Milde and Zahid Pashayev
