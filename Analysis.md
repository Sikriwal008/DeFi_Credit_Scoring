# Analysis of Wallet Credit Scores

This document analyzes the 3,495 wallet credit scores generated from Aave V2 transaction data. Scores range from **0 to 1000**, where higher values indicate more responsible, reliable on‑chain behavior.

---

## Score Distribution

Below is the histogram of all wallet credit scores. Wallets forced to **0** (due to at least one liquidation) form a distinct cluster at the left, while the remainder span the 1–1000 range.

![Score Distribution](output/score_distribution.png)

---

## Key Observations

1. **Cluster at Zero**  
   - Wallets with ≥1 `liquidationcall` are automatically assigned a 0.  
   - In our sample, **101 wallets (2.89%)** fall here — the highest‑risk segment.

2. **Mid‑Range Concentration**  
   - The vast majority of non‑liquidated wallets score between **400 and 700**.  
   - These represent typical users who deposit collateral, borrow moderately, and repay responsibly.

3. **High‑Score Scarcity**  
   - Very few wallets exceed **900**.  
   - These “prime” wallets demonstrate the best mix of low leverage, consistent repayments, and steady activity.

---

## Behavior of Wallets in the Lower Range (Score 0–200)

Wallets scoring in this bottom band exhibit clear risk signals:

- **Liquidations (Score = 0):**  
  The single greatest penalty. Any liquidation event immediately forces a wallet’s credit score to zero.

- **High Leverage:**  
  Wallets with scores just above zero often have extremely elevated `borrow_to_deposit_ratio` (>1), meaning they borrow more than they deposit.

- **Poor Repayment Discipline:**  
  A low or zero `repayment_to_borrow_ratio` indicates infrequent or no repayments.

- **New & Aggressive:**  
  Very small `wallet_age_days` combined with bursty `tx_per_day`—a pattern often seen in bots or high‑risk traders.

---

## Behavior of Wallets in the Higher Range (Score 800–1000)

Top‑tier wallets share these characteristics:

- **Zero Liquidations:**  
  No history of `liquidationcall` events.

- **Healthy Collateralization:**  
  Very low `borrow_to_deposit_ratio`, indicating conservative borrowing against ample collateral.

- **Strong Repayment History:**  
  A `repayment_to_borrow_ratio` often ≥ 1, showing they repay at least as much as they borrow.

- **Established Presence:**  
  High `wallet_age_days`, reflecting long‑term, stable protocol participation.

- **Organic Activity:**  
  `tx_per_day` close to 1, signaling steady, human‑driven usage rather than erratic, high‑frequency trading.

These wallets form the stable core of the lending pools and are considered the most trustworthy.

---

## Next Steps

- **Expand Features:**  
  Incorporate governance votes, gas‑fee behavior, or token‑holding patterns.

- **Adjust Weights & Scaling:**  
  Tune health, stability, and activity weights to refine score granularity.

- **Real‑Time Scoring:**  
  Stream new transactions into this model for live credit updates.

*End of analysis.*
