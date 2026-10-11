# DAIX: an on-chain activity index and Ether volatility — data

Data and results for the paper

> Valentinov, S. G. (2026). *DAIX: an on-chain activity index as a real-time measure of Ether volatility*. Manuscript submitted to *Digital Finance*.

DAIX (DeFi Activity IndeX) is a daily Fisher quantity index of user activity on two DeFi protocols on Ethereum mainnet: the Uniswap V3 exchange and the Aave V3 lending market. The repository contains the daily aggregates from which DAIX is computed, the index and its components, ETH realised variance, the driver series, and the forecasts and test results reported in the paper. It contains no code and no address-level data.

- Sample: 18 July 2025 – 1 October 2026 (441 UTC days); 55,546,160 screened events.
- Hourly sample: 18 July – 17 September 2026 (1,488 hours).
- Forecast origins: 14 January – 30 September 2026 (260 days), horizons 1–14 days.

## Sources

| Data | Source |
|---|---|
| Protocol events | Uniswap V3 subgraph and Aave V3 subgraph (Messari schema) on The Graph network |
| Protocol state (TVL, fees, rates, utilisation, deposits, liquidations) | Daily snapshot entities of the same subgraphs |
| Gas | `eth_feeHistory` on a public Ethereum JSON-RPC endpoint |
| ETH prices | Binance ETHUSDT five-minute candles, 1 January 2023 – 1 October 2026 (only derived daily statistics are included) |
| Funding rate | Binance ETHUSDT perpetual, eight-hourly funding rates |

Each event is assigned to one primary actor: the transaction originator for exchange events, the account for lending events and the sender for aToken transfers. Lending accounts with deployed contract code are excluded, except 23-byte EIP-7702 delegation designators, which mark ordinary accounts.

## Index construction (summary)

- An elementary aggregate is one of nine event types (`swaps`, `mints`, `burns`, `deposits`, `withdraws`, `borrows`, `repays`, `flashloans`, `transfers`) in one of seven fixed USD value bands (`<$10`, `$10-100`, `$100-1k`, `$1k-10k`, `$10k-100k`, `$100k-1M`, `>$1M`): 63 priced aggregates.
- For aggregate *i* on day *t*: *q* = number of priced events, *p* = USD value / *q* (unit value).
- Base period: the average day of 18–24 July 2026 (*q*₀, *p*₀).
- Laspeyres QL = Σ p₀q / Σ p₀q₀; Paasche QP = Σ pq / Σ pq₀; **DAIX = √(QL·QP)**.
- Events without a USD value are excluded from the index (3.3% of events).
- Unit values above 10 × the aggregate's sample median are capped at that level (34 of 26,015 aggregate-days, all in the `>$1M` band; listed in `data/capped_unit_values.csv`). The cap corrects USD mispricing of some tokens by the subgraphs.

`data/daily_aggregates.csv` with `data/base_period_and_caps.csv` is sufficient to recompute DAIX exactly.

## Files

### `data/`

| File | Rows | Columns |
|---|---|---|
| `daily_aggregates.csv` | 27,717 | `date`; `aggregate` (`event_type\|band`, or `event_type` alone for events without a USD value); `count` (all events); `usd` (USD value of priced events); `unpriced` (events without a USD value). Priced events = `count − unpriced`. Values are raw, before the cap. |
| `base_period_and_caps.csv` | 63 | `aggregate`, `event_type`, `value_band`; `q0_base_events_per_day`, `p0_base_unit_value_usd` (base week, after the cap); `unit_value_cap_usd`; `base_value_share` (*p*₀*q*₀ / Σ *p*₀*q*₀) |
| `capped_unit_values.csv` | 34 | `date`, `aggregate`, `unit_value_reported_usd`, `unit_value_capped_usd` |
| `daix_daily.csv` | 441 | `date`; `daix` (Fisher); `laspeyres`; `paasche`; `count_index` (events relative to the base week, equal weights); `events`; `unpriced_share` |
| `component_weights.csv` | 9 | `event_type`, `base_value_share` *w*ₑ₀ |
| `daix_components_daily.csv` | 441 | `L_<type>`: Laspeyres sub-index of each event type, so that `laspeyres` = Σ *w*ₑ₀ · `L_<type>` (up to rounding); `contrib_<type>`: exact additive contribution to `daix − 1` (Reinsdorf, Diewert & Ehemann, 2002), summing to `daix − 1` on every day |
| `eth_realised_variance_daily.csv` | 1,370 | `date`; `rv_pct2` (sum of squared five-minute log returns, in %², UTC day); `ln_rv`; `return_pct` (daily log return, %); `n_returns` (288 on complete days) |
| `drivers_daily.csv` | 441 | see below |
| `hourly_series.csv` | 1,488 | `hour_utc`; `rv_pct2` (hourly realised variance); `daix_hourly` (hourly Fisher index, base = average hour of 18–24 July 2026, no cap); `count_index_hourly`; `dex_events`, `lending_events` (event counts) |

Driver columns (`drivers_daily.csv`, values at the end of each UTC day):

| Column | Definition |
|---|---|
| `lnvol_cex` | ln Binance ETHUSDT spot volume (ETH) |
| `lngas` | ln daily mean base fee (gwei) |
| `gasuse` | daily mean gas-used ratio (not used in the models) |
| `lntvl` | ln Uniswap V3 TVL (USD); one subgraph error on 14 Nov 2025 replaced by interpolation |
| `lnfees` | ln Uniswap V3 daily fees (USD) |
| `lnborrow_tot` | ln Aave V3 total borrows (USD) (not used in the models) |
| `lndeposit_tot` | ln Aave V3 total deposits (USD) |
| `lnliq` | ln(1 + Aave V3 daily liquidations, USD) |
| `r_borrow_usdc`, `r_borrow_usdt`, `r_borrow_weth` | variable borrow rates, % per year (`r_borrow_usdt` not used in the models) |
| `r_supply_usdc`, `r_supply_weth` | supply rates, % per year |
| `util_usdc`, `util_weth` | utilisation, borrows / deposits (`util_weth` not used in the models) |
| `fund` | daily sum of eight-hourly ETHUSDT perpetual funding rates, % |

The return and ln RV, which are also drivers in the paper, are in `eth_realised_variance_daily.csv`.

### `results/`

| File | Content |
|---|---|
| `descriptives.csv` | Table 1 |
| `decomposition_summary.csv` | Table 3: base weight, mean contribution, share of variance of daily changes, share of covariance with Δ ln RV, prewhitened same-day correlation with ln RV |
| `ccf_daily.csv` | Cross-correlations of ln DAIX and ln RV, lags −42…42 days (positive lag: DAIX leads): raw, day-of-week adjusted, prewhitened |
| `ccf_hourly.csv`, `leadlag_hourly_summary.csv` | Prewhitened hourly cross-correlations (±48 h) and summary tests |
| `volatility_forecasts.csv` | All volatility forecasts: `origin`, `h`, `target`, `model`, `forecast_rv`, `actual_rv` |
| `volatility_loss_qlike.csv`, `_se.csv`, `_ape.csv` | Mean losses by model and horizon (Table 4; W1/W2 = means of days 1–7 / 8–14) |
| `volatility_dm_tests.csv` | Diebold–Mariano tests of each model against each benchmark (QLIKE and squared error) |
| `volatility_mcs.csv` | Model confidence set p-values (QLIKE, 90%) |
| `daix_forecasts_win180.csv` | All DAIX forecasts, rolling 180-day window: `origin`, `h`, `target`, `model`, `forecast`, `actual` |
| `daix_mape_win180.csv`, `_expanding.csv`, `_win90.csv` | DAIX forecast MAPE (%) by model and horizon, three estimation windows (Table 5) |
| `daix_dm_tests_*.csv` | Diebold–Mariano tests on squared log errors (against the direct forecast unless stated in brackets) |
| `daix_mape_driver_variants.csv`, `daix_dm_driver_variants.csv` | Bottom-up forecasts without market drivers and with on-chain drivers only |
| `component_mape_win180.csv` | Component forecast MAPE without and with drivers |
| `driver_coefficients_h1.csv` | One-day-ahead driver coefficients and HC1 t-statistics, full sample (Table 6) |
| `ex_ante_origin_2026-09-17_win180.csv` | Forecasts made on 17 September 2026 for 18 September – 1 October 2026 |

Model names in the results files: `Naive` / `No change`; `Direct` (index on its own past); `Direct + all drivers (ridge)`; `Bottom-up, no drivers`; `Bottom-up, own drivers` (`BU own drivers`); `BU no market drivers`; `BU on-chain only`; `HAR-RV (full history)`, `HAR-RV (180 days)`, `GARCH(1,1)-t`; `Bridge: …` (DAIX forecast mapped into volatility); `HAR-X: …` (HAR plus DAIX forecast); `Bridge: realised index (oracle)` (uses the realised DAIX on the target day; not a feasible forecast).

## Licence

The data in this repository are released under the Creative Commons Attribution 4.0 International licence (CC BY 4.0). See `LICENSE`.

## Citation

Please cite the paper above (see `CITATION.cff`).
