# 📊 Trader Performance vs Market Sentiment
### Primetrade.ai — Data Science / Analytics Intern Assignment

---

## 📁 Repository Structure

```
primetrade-analysis/
│
├── primetrade_analysis.ipynb          ← Main notebook (fully executed with outputs)
├── README.md                          ← Setup, methodology, insights, recommendations
│
├── data/
│   ├── historical_data.csv            ← Hyperliquid trader data (211,224 rows)
│   └── fear_greed_index.csv           ← Bitcoin Fear/Greed Index (2,644 rows)
│
└── charts/
    ├── chart1_performance_fear_greed.png
    ├── chart2_behavior_fear_greed.png
    ├── chart3_leverage_segments.png
    ├── chart4_consistency_segments.png
    ├── chart5_pnl_vs_fgi_timeline.png
    ├── chart6_coin_sentiment_pnl.png
    ├── chart7_pnl_distribution.png
    ├── chart8_feature_importance.png
    ├── chart9_elbow_curve.png
    └── chart10_archetypes.png
```

---

## ⚙️ Setup & How to Run

### Prerequisites

```
Python >= 3.9
```

### Install Dependencies

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn jupyter
```

### Run the Notebook

1. Clone or download this repository
2. Place both CSV files inside a `data/` folder (or in the same directory as the notebook)
3. Launch Jupyter:

```bash
jupyter notebook primetrade_analysis.ipynb
```

4. Run all cells: **Kernel → Restart & Run All**

> All charts are saved automatically as `.png` files in the working directory.

---

## 🗃️ Dataset Overview

| Dataset | Rows | Columns | Date Range |
|---|---|---|---|
| `historical_data.csv` | 211,224 | 16 | May 2023 – May 2025 |
| `fear_greed_index.csv` | 2,644 | 4 | Feb 2018 – present |

**Key fields used:**
- `historical_data` — Account, Side, Size USD, Closed PnL, Timestamp IST, Direction
- `fear_greed_index` — date, value (0–100), classification (Extreme Fear → Extreme Greed)

**Data quality:** No missing values or duplicates in either dataset. Timestamp IST parsed with `dd-MM-yyyy HH:mm` format. `Extreme Fear` and `Extreme Greed` were collapsed into `Fear` and `Greed` for primary analysis, yielding a 3-class sentiment variable: Fear / Neutral / Greed.

---

## 🔬 Methodology

### Part A — Data Preparation

1. **Loaded & profiled** both datasets — shapes, dtypes, null counts, duplicates
2. **Parsed timestamps**: `Timestamp IST` → `datetime` → `date` (daily grain)
3. **Aligned datasets**: inner join on `date` between trader data and F/G index
4. **Derived metrics** at the trader × day level:
   - `daily_pnl` — sum of Closed PnL per account per day
   - `win_rate` — fraction of winning trades (PnL > 0) per account per day
   - `drawdown_proxy` — absolute value of negative daily PnL
   - `avg_size_usd` — mean position size in USD
   - `trade_count` — number of trades per account per day
   - `long_short_ratio` — BUY count / (SELL count + 1)
5. **Trader-level summary** computed for segmentation: total PnL, total trades, win rate, avg trade size, PnL std dev

### Part B — Analysis

- Compared performance metrics across sentiment regimes using **group means** and **independent t-tests**
- Segmented traders by **leverage proxy** (avg trade size relative to median), **trade frequency**, and **profitability consistency** — each split into Low / Mid / High tertiles
- Overlaid F/G index value as a continuous line against daily PnL bars on a timeline

### Bonus

- **Predictive model**: RandomForestClassifier on same-day behavioral + sentiment features to predict next-day PnL bucket (Big Loss / Small Loss / Small Win / Big Win). 80/20 train-test split.
- **Clustering**: KMeans (k=4, chosen by elbow curve) on standardised trader-level features → 4 behavioral archetypes

---

## 💡 Insights

### Insight 1 — Fear Days Drive Higher PnL Despite Lower Win Rates

| Sentiment | Avg Daily PnL | Win Rate | Avg Drawdown |
|---|---|---|---|
| **Fear** | **7,161** | 0.842 | 1,521 |
| Neutral | 4,684 | 0.836 | 1,047 |
| Greed | 5,764 | **0.856** | 1,288 |

Fear days produce the **highest average daily PnL** (+24% vs Greed), even though win rate is marginally lower. This means winning trades on Fear days are **disproportionately large** — traders who stay disciplined capture bigger dislocations. The higher drawdown proxy confirms greater two-way volatility.

![Performance Chart](https://raw.githubusercontent.com/Harshgup16/primetradeAI-assignment/main/chart1_performance_fear_greed.png)

---

### Insight 2 — Traders Take Bigger, More Frequent Positions on Fear Days

| Sentiment | Avg Trades/Day | Avg Position Size (USD) | Long/Short Ratio |
|---|---|---|---|
| **Fear** | **70.3** | **11,593** | 3.96 |
| Neutral | 65.8 | 8,574 | 4.50 |
| Greed | 54.5 | 6,822 | **2.70** |

Counterintuitively, traders are **more active** and use **larger positions** during Fear — not Greed. Experienced participants treat Fear as a buying opportunity, scaling in aggressively. Long/short ratio drops on Greed days, consistent with smart-money profit-taking as markets turn euphoric.

![Behavior Chart](https://raw.githubusercontent.com/Harshgup16/primetradeAI-assignment/main/chart2_behavior_fear_greed.png)

---

### Insight 3 — High-Leverage Traders Are the Most Sentiment-Sensitive Segment

Low-leverage traders show **stable PnL** across all regimes. High-leverage traders show the **highest PnL on Fear days** but also the **highest drawdown** — they capture the upside but absorb severe losses when wrong. The Loser segment bleeds most on Fear days due to over-leveraged, poorly-timed entries.

![Leverage Segments](https://raw.githubusercontent.com/Harshgup16/primetradeAI-assignment/main/chart3_leverage_segments.png)
![Consistency Segments](https://raw.githubusercontent.com/Harshgup16/primetradeAI-assignment/main/chart4_consistency_segments.png)

---

### Insight 4 — Four Distinct Behavioral Archetypes Respond Differently to Sentiment

KMeans clustering (k=4) identified four archetypes:

| Archetype | Characteristic | Best Sentiment Regime |
|---|---|---|
| 🏆 High-Profit Winners | High PnL, moderate trades | All regimes |
| ⚡ Hyperactive Traders | Very high trade count, thin per-trade edge | Greed |
| 🐋 Whale Traders | Very large position sizes | Fear |
| 📉 Consistent Losers | Negative cumulative PnL, high drawdown | None |

Whale Traders perform best on Fear days (large size × large price dislocations). Hyperactive Traders eke out more on Greed days when price action is smoother and directional.

![Archetypes](https://raw.githubusercontent.com/Harshgup16/primetradeAI-assignment/main/chart10_archetypes.png)

---

## 🎯 Strategy Recommendations

### Strategy 1 — Size Down on Fear for High-Leverage Traders

> *"High-leverage traders should cap position size to 50% of normal on Fear days."*

Although average PnL is higher on Fear days, the drawdown proxy is **18% higher** than on Greed days. The reward is real but so is the risk of ruin for traders without a demonstrated edge.

| Condition | Action |
|---|---|
| `sentiment = Fear` AND `leverage_tier = High` | Reduce position size by 50% |
| `sentiment = Fear` AND `consistency_tier = Loser` | Sit out entirely |
| `sentiment = Fear` AND `consistency_tier = Winner` | Maintain normal size — these traders have edge |

---

### Strategy 2 — Align Directional Bias with Sentiment Regime

> *"Increase long bias on Greed days; go net-neutral on Fear days unless you are a proven winner."*

Long/short ratio drops from ~4.0 on Fear to ~2.7 on Greed, suggesting smart money reduces long exposure as markets get euphoric. Consistent Winners sustain strong win rates across both regimes but compound faster on Greed with directional trades.

| Condition | Action |
|---|---|
| `sentiment = Greed` AND `archetype = High-Profit Winner` | Increase long bias (L/S ratio ≤ 2.5 acceptable) |
| `sentiment = Greed` AND `archetype = Hyperactive Trader` | Increase trade frequency — edge improves |
| `sentiment = Fear` | Reduce net directional exposure; favour mean-reversion over trend-following |
| `sentiment = Neutral` | Default strategy; no sentiment adjustment needed |

---

## 📈 Output Charts

| File | Description |
|---|---|
| `chart1_performance_fear_greed.png` | Avg PnL, Win Rate, Drawdown by Sentiment |
| `chart2_behavior_fear_greed.png` | Trade Frequency, Position Size, L/S Ratio by Sentiment |
| `chart3_leverage_segments.png` | PnL & Win Rate by Leverage Tier × Sentiment |
| `chart4_consistency_segments.png` | PnL by Consistency Tier × Sentiment |
| `chart5_pnl_vs_fgi_timeline.png` | Daily PnL bars overlaid with F/G Index line |
| `chart6_coin_sentiment_pnl.png` | Avg PnL per Trade by Coin × Sentiment (Top 10) |
| `chart7_pnl_distribution.png` | PnL distributions across Fear / Neutral / Greed |
| `chart8_feature_importance.png` | Feature importance from Random Forest classifier |
| `chart9_elbow_curve.png` | KMeans elbow curve for archetype selection |
| `chart10_archetypes.png` | Scatter of 4 behavioral archetypes |

---

## 🧰 Tech Stack

| Library | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, merging, aggregation |
| `numpy` | Numerical operations |
| `matplotlib` / `seaborn` | All visualisations |
| `scipy.stats` | T-tests for significance testing |
| `scikit-learn` | RandomForestClassifier, KMeans, StandardScaler |
| `jupyter` | Notebook execution environment |

---

*Submitted for Primetrade.ai — Data Science / Analytics Intern, Round 0*
