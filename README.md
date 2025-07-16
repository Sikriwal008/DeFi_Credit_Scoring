# DeFi Wallet Credit Scoring

Assign a **0–1000 credit score** to each Aave V2 wallet based solely on on‑chain transaction behavior. Wallets with any liquidations are forced to 0; all others are scored by health, stability and activity.

---

## Architecture & Flow

1. **Mount Google Drive**  
   - Uses `google.colab.drive` to access `user-wallet-transactions.json` in Drive.

2. **Load & Preprocess**  
   - Flattens raw JSON via `pd.json_normalize`.  
   - Renames fields, parses `timestamp`, coerces numeric types.  
   - Maps token symbols → decimals; computes USD value (`amount_usd`).  
   - Gracefully handles missing `actionData_debtAssetSymbol`/`actionData_debtToCover` in liquidation calls (issues a warning if absent).

3. **Feature Engineering**  
   - **Age**: days between first & last tx (`wallet_age_days`, min 1).  
   - **Volume**: sums of deposit/borrow/repay/redeem in USD.  
   - **Counts**: `transaction_count`, per‑type action counts, `liquidation_count`.  
   - **Ratios**:  
     - `borrow_to_deposit_ratio` = borrowed / deposited  
     - `repayment_to_borrow_ratio` = repaid / borrowed  
   - **Activity**: `tx_per_day` = transactions / wallet_age_days.

4. **Scoring Model**  
   - **Health** (40%):  
     \[
       1 - \ln\bigl(1 + \tfrac{\text{borrow_to_deposit_ratio}}{}\bigr)
     \]
   - **Stability** (40%):  
     \[
       \ln(1 + \text{wallet_age_days}) + \tfrac{\text{repayment_to_borrow_ratio}}{}
     \]
   - **Activity** (20%):  
     \[
       \frac{1}{1 + |\text{tx_per_day} - 1|}
     \]
   - **Raw** = 0.4·Health + 0.4·Stability + 0.2·Activity  
   - **Scale** raw → [100, 1000] via `MinMaxScaler`  
   - **Override**: `liquidation_count` > 0 ⇒ score = 0  

5. **Output**  
   - `wallet_scores.csv` (columns: `wallet`, `credit_score`)  
   - `score_distribution.png` (histogram of all scores)

---

## Dependencies

```bash
pip install pandas numpy scikit-learn matplotlib
