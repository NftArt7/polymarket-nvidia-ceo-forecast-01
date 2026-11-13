# polymarket-nvidia-ceo-forecast-01

Forecast, minute by minute and 4 hours ahead, the Yes price of every trading
market of Polymarket's "Next CEO of NVIDIA?" event, as five quantiles
(0.05, 0.25, 0.5, 0.75, 0.95) per minute, for the lowest mean pinball loss.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

`main.py` (Python 3.13, standard library only, no network) reads every
`<id>.in` in the input directory and writes `<id>.out` to the output
directory: `{"forecasts": [{"id": <conditionId>, "q": [[q05, q25, q50, q75, q95] x 240]}]}`.

## Model

Per market, from the case file alone:

- **Centre:** the market's last price, corrected across markets. The 24
  markets are mutually exclusive outcomes, so their probabilities sum to ~1.
  When the last prices sum to S > 1 the centres are shrunk proportionally
  halfway toward a sum of 1 (a quarter of the way up when S < 1).
- **Spread:** a Gaussian random-walk band,
  `q = centre + z_tau * sigma * sqrt(h)`, clamped to [0, 1], 6 decimals.
  `sigma` is per-market: the standard deviation of 1-minute price changes
  clipped to ±0.015, blending the last 120 minutes (weight 0.7) with the
  last 1440 minutes (weight 0.3), times 0.8, floored at 0.0003.

The parameters were calibrated on the practice cases p00–p04 (answers
recovered from each following case's prices): **mean pinball loss 0.298**,
vs **0.393** for the house persistence baseline (sigma 0.0025) on the same
cases, with 90% coverage 0.901 and median error 0.855 points. Heavier tails,
empirical per-horizon quantiles, logit-space bands, trailing-median centres
and 1-hour-trend mean reversion were tried and scored worse.

Deterministic: two runs on the same case produce byte-identical output.
Runtime is ~0.06 s per case, well under the 8 s cap.
