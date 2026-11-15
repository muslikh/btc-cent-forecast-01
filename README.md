# btc-cent-forecast-01

A forecaster for make.fun's **btc-cent-forecast-01**: predict the close of Coinbase
Wrapped BTC against USDC in the Orca Whirlpool `HxA6SKW...` 240 minutes after each
checkpoint, scored by the mean absolute error in cents (lower is better).

## The estimator

Absolute error is a proper scoring rule for a single number, so the optimal point
forecast is the **median** of the predictive distribution, never the mean. `main.py`
estimates that median two ways and blends them:

1. **Empirical median return** — the median of every realized 240-minute return inside
   the case's 72-hour window. Under MAE this is the L1-optimal estimator, and it is
   robust to the fat tails an Orca pool produces on a single large trade.
2. **SMA damping** — a pull toward the 240-minute simple moving average, so a wick at
   the checkpoint close does not drag the guess with it.

    guess = last + 2.0 * median_240m_return + 0.3 * (SMA_240 - last)

The weights were fit on the kit's five graded practice cases and confirmed on a
walk-forward replay of the 5,520-minute practice recording.

## Accuracy (practice)

| estimator | mean absolute error |
|---|---|
| house baseline (persistence) | 47,054.66 cents |
| **this entry** | **43,872.25 cents** |

## Run

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `*.in` in `$MAKEFUN_INPUT_DIR` (default `/input`), writes one
`<case>.out` per case to `$MAKEFUN_OUTPUT_DIR` (default `/output`). Python 3.13
standard library only, no network, deterministic.

## Licence

MIT — see `LICENSE`.
