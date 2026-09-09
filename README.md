# Market regime-aware BESS bidding: an ex-ante thin-market detection algorithm for the Italian MB

MSc thesis in Energy Engineering, Politecnico di Milano — defence Dec 2026.
Backtested on public TERNA, GME and ENTSO-E data (Jan 2023 – Jun 2026, IT-North).

<!-- img/excess_pnl.png -->

## Key finding

Bidding a smaller volume when the market is *thin* is worth **+4,861 € (+6.4%)**
over an all-or-nothing benchmark on H1 2026 — if the regime is known.
With the regime predicted ex ante, the same rule is worth **+420 €**.
The gap between the two curves is the priced cost of misclassification,
and it is the actual result of this work: the signal has value, the
classifier captures little of it.

A permutation control (random thin/thick labels) rules out the gain being
an artefact of the extra degree of freedom.
<img width="1443" height="497" alt="image" src="https://github.com/user-attachments/assets/dff30f1b-d9c0-4520-a67e-7b5fe566bdf7" />

---

## Why regimes

Balancing-price volatility spikes sharply as net zonal imbalance approaches
zero. When the system's balancing need is small relative to available
capacity the market is competitive and a single operator risks exclusion,
not price-making. **Thin** is therefore defined as |zonal imbalance| ≤ 50 MWh
— a conservative band containing the empirical peak — and **thick** as
everything else.

<img width="816" height="498" alt="image" src="https://github.com/user-attachments/assets/3b9e47c3-57c7-4961-99c3-a98e7fcb5013" />

The literature bids statically: calendar price patterns with a fixed
ex-post-optimised margin (Canevese), templated price menus updated
seasonally (Hosseini), real UVAM offers clustering near 400 €/MWh with a
0.05% acceptance rate (Schwidtal). None conditions on market state.

---

## Data and pipeline

| Source  | Data                                                        |
|---------|-------------------------------------------------------------|
| TERNA   | MB settlement, zonal imbalance, MSD6 MARG/MEDIO (API)       |
| GME     | Zonal day-ahead prices                                       |
| ENTSO-E | RES, load, generation by type — BZN IT-North                 |

Quarter-hourly slots on a canonical index. Train up to Dec 2025,
test on 2026.

Features are lagged to their real publication calendar: 24 h for imbalance,
price and regime; 1 h for load/RES/generation forecast errors, online
flexibility and gas share. Plus Ramp_60, the NORD−PUN spread (congestion)
and calendar terms.

**Classifier**: LassoCV on imbalance volume → linear classifier → Random Forest.
Balanced accuracy **0.609** vs 0.50 naïve. Balanced accuracy rather than
accuracy: with a 41% thin base rate, always predicting "thin" scores 0.41
accuracy and 0.50 BA.

---

## Strategies

Both strategies share identical price logic, acceptance proxy and market-share
constraint, so the P&L difference is attributable to the regime signal alone.
While in a Single-Price scheme the risk is to flip the imbalance sign (BRUNINX), 
in the Italian (pay as bid) balancing market the risk is to be excluded 
from the auction.

**A — all-or-nothing benchmark.** Bid = yesterday's MB price, offered only if
it beats the PUN. BESS 10 MW, Q = 2.5 MWh per quarter-hour. Price acceptance
via MSD6 proxy (MARG_UP / MARG_DOWN plus a `had_acceptance` flag); volume
acceptance only if q ≤ θ·|V| with θ = 2%. Result: **75,782 €**, 3.4% acceptance.

**B — regime-conditional volume.** Same price logic, q = α·Q_max with one α
per regime. Optimal α is 0.20 thin / 1.00 thick with the true label, and
0.65 / 1.00 with the predicted one — α moves toward 1 as the rational
response to classifier uncertainty.
<img width="577" height="408" alt="image" src="https://github.com/user-attachments/assets/0dce1838-b12a-4b11-adea-eeb55c88318d" />

---

## Declared limits

- The MSD6 proxy's competitive set is not identical to the real-time MB, and
  MARG is determined before imbalances materialise
- θ is calibrated, not measured (sensitivity run at 2 / 3 / 5%)
- Single hypothetical asset, no SoC constraint, energy only
- Single chronological holdout, no retraining during the test period —
  the ex-ante curve flattens after February, which is consistent with that

## Next

- **FEES API** (preliminary imbalance and prices every 15 min): cuts feature
  lag from 16 to 2–4 quarter-hours — the single largest improvement available
- Settlement-accepted volumes instead of the synthetic stack
- SoC constraint and cycle cost in the backtest (crossover c\* ≈ 27 €/MWh)

---

`Python` · `pandas` · `numpy` · `scikit-learn` · `statsmodels` · `matplotlib`

MSc thesis in Energy Engineering, Politecnico di Milano — defence Dec 2026.
