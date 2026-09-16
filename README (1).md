# Statistical Pairs Trading: KO–PEP

An out-of-sample statistical arbitrage study of Coca-Cola (`KO`) and PepsiCo (`PEP`), built in Python.

The project investigates whether deviations in the historical relationship between two economically related stocks can be traded through a market-neutral long–short portfolio. Rather than optimizing retrospectively for a profitable result, the analysis preserves and explains the strategy's out-of-sample failure.

## Project highlights

- Screened four economically related asset pairs using the Engle–Granger cointegration test.
- Verified the integration properties of KO and PEP log-prices with Augmented Dickey–Fuller (ADF) tests.
- Estimated the hedge ratio exclusively on a 2018–2022 training sample.
- Evaluated the strategy on unseen 2023–2025 data.
- Shifted positions by one trading day to prevent look-ahead bias.
- Compared a naïve mean-reversion strategy with a stop-loss version.
- Included turnover-based transaction costs.
- Measured return, volatility, Sharpe ratio and maximum drawdown.
- Tested nine entry/stop-loss parameter combinations without retuning the strategy on the test set.

## Research question

If KO and PEP share a stable long-run relationship, can temporary deviations from that relationship generate out-of-sample trading opportunities?

The strategy is based on the idea that an unusually high or low spread may eventually revert toward its historical equilibrium. The project tests both this hypothesis and the consequences of its failure.

## Data and experimental design

Daily adjusted closing prices are downloaded with `yfinance` for the period from January 2018 through December 2025.

Four candidate pairs are initially considered:

| Pair | Economic relationship |
|---|---|
| XOM–CVX | Integrated energy companies |
| KO–PEP | Global beverage companies |
| JPM–BAC | Large US banks |
| GLD–SLV | Precious-metal ETFs |

The observations are divided chronologically:

| Sample | Period | Purpose |
|---|---|---|
| Training | 2018–2022 | Pair selection and parameter estimation |
| Testing | 2023–2025 | Out-of-sample strategy evaluation |

No hedge ratio, spread mean or spread standard deviation is re-estimated using the test sample.

## Pair selection and spread construction

KO–PEP produced the lowest Engle–Granger p-value, approximately `0.056`. This is borderline evidence at the 10% significance level, but it does not reject the null hypothesis of no cointegration at the conventional 5% level. The pair is therefore selected for an **exploratory** backtest rather than treated as definitively cointegrated.

The training-period relationship is estimated by ordinary least squares:

```math
\log(P_t^{KO}) = \alpha + \beta\log(P_t^{PEP}) + \varepsilon_t.
```

The residual defines the spread:

```math
S_t = \log(P_t^{KO}) - \alpha - \beta\log(P_t^{PEP}).
```

It is standardized using the training-period mean and standard deviation:

```math
Z_t = \frac{S_t-\mu_{train}}{\sigma_{train}}.
```

## Trading logic

The baseline strategy uses the following rules:

| Z-score condition | Position |
|---|---|
| `Z ≥ 2` | Short KO and buy PEP (short the spread) |
| `Z ≤ −2` | Buy KO and short PEP (long the spread) |
| Return to `Z = 0` | Close the position |

Signals are shifted by one day before returns are calculated. A signal observed from today's closing prices can therefore affect the position only on the following trading day.

### Stop-loss extension

The risk-managed strategy follows the same entry and equilibrium-exit rules, but closes the position if the absolute z-score reaches `3.5`. After the stop-loss is triggered, the strategy remains inactive until `|Z| < 1`.

## Return calculation

The notebook includes a preliminary calculation based on log-returns for educational comparison. The final evaluation uses simple asset returns:

```math
r_t^{portfolio} = \frac{r_t^{KO}-\beta r_t^{PEP}}{1+|\beta|}.
```

Dividing by `1 + |β|` normalizes the return by the portfolio's total gross exposure. Portfolio wealth is compounded as

```math
W_t = W_{t-1}(1+r_t^{strategy}).
```

All final metrics and sensitivity results use simple returns and a transaction cost of `0.10%` per unit of portfolio turnover.

## Out-of-sample results

| Metric | Naïve strategy | Stop-loss strategy |
|---|---:|---:|
| Total return | **−11.56%** | **−4.58%** |
| Annualized return | −4.03% | −1.56% |
| Annualized volatility | 5.92% | 1.46% |
| Sharpe ratio | −0.67 | −1.07 |
| Maximum drawdown | −16.99% | −4.58% |
| Opened trades | 1 | 1 |

The stop-loss substantially reduced the loss, volatility and drawdown, but did not make the strategy profitable. Because the main specification generated only one trade, the Sharpe ratios and annualized statistics should be interpreted cautiously.

## Sensitivity analysis

The analysis evaluates every combination of:

- entry thresholds: `1.5`, `2.0`, `2.5`;
- stop-loss thresholds: `3.0`, `3.5`, `4.0`.

All nine configurations produced negative returns. Tighter stop-losses generally reduced losses and drawdowns, but no threshold choice restored profitability. The best observed configuration is not promoted as an optimized model because selecting it after observing the test sample would introduce data snooping.

## Interpretation

From 2024 onward, the test-period z-score crossed the positive entry threshold and continued to rise rather than reverting toward zero. The strategy consequently shorted KO and bought PEP while KO continued to outperform PEP.

The central result is therefore a structural one: the historical KO–PEP relationship estimated during 2018–2022 was not stable over 2023–2025. Risk management limited the damage, but it could not manufacture mean reversion after the underlying relationship changed.

This negative result is an important part of the project. It demonstrates why a convincing trading backtest requires:

- a strict chronological train/test split;
- protection against look-ahead bias;
- realistic trading costs;
- risk controls;
- parameter-sensitivity checks;
- caution when the number of independent trades is small.

## Repository contents

```text
.
├── pairs_trading.ipynb   # Complete analysis and visualizations
└── README.md             # Project overview
```

The submitted `.py` file is an export of the Google Colab notebook. For GitHub presentation, the `.ipynb` version is recommended because it preserves the Markdown explanations, tables and plots.

## Installation

```bash
pip install numpy pandas matplotlib seaborn statsmodels yfinance
```

Run the notebook from top to bottom so that the downloaded data, estimated parameters, positions and performance series are generated in the correct order.

## Tools

- Python
- NumPy and pandas
- Statsmodels
- Matplotlib and Seaborn
- yfinance
- Google Colab

## Potential extensions

- Re-estimate the hedge ratio on a rolling window.
- Retest cointegration periodically.
- Introduce explicit structural-break detection.
- Add a maximum holding period.
- Include borrowing fees, financing costs and execution slippage.
- Screen a larger universe with multiple-testing corrections.
- Test the strategy across additional market regimes.

## Disclaimer

This project is for educational and research purposes only. It does not constitute investment advice, and historical performance does not imply future results.
