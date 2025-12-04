# [MLP Project 4–2] Cross-Market Extension: KOSPI Excess Return Prediction

## 📌 Course Information
| Category | Detail |
|---------|--------|
| **Course** | CS 53744 — Machine Learning |
| **Instructor** | Prof. Jongmin Lee |
| **Project** | MLP Project 4–2 — *Cross-Market Extension* |
| **Objective** | Extend the S&P 500 forecasting pipeline to a new market (KOSPI) |

---

## 📌 Team Information
| Role   | Name (KOR) | Student ID | GitHub ID |
|--------|------------|------------|-----------|
| Member | 박원규     | 20203955   | `@keiro23` |
| Member | 이유정     | 20214677   | `@yousrchive` |
| Member | 정승환     | 20201799   | `@whan0767` |

---

## 🎯 Project Goal

This repository contains a fully reproducible implementation of the **Cross-Market Extension** bonus project.  
The goal is to adapt the *Hull Tactical S&P 500 prediction pipeline* to a different financial environment —  
specifically, the **KOSPI** index — and evaluate:

1. Whether the **short-horizon predictability** found in the U.S. market transfers to Korea  
2. How **market microstructure differences** affect ML-based forecasting  
3. How to build a **volatility-constrained allocation strategy** using the predicted excess returns

**Target variable:**  
> `market_forward_excess_returns` — next-day excess return relative to a 5-year rolling mean  

**Allocation rule:**  
> w_t = clip(1 + k*z_t, 0, 2) with volatility ≤ 1.2× benchmark  

---

# Repository Structure

```
proj4-2/
├── data/
│   ├── 뉴스심리지수.xlsx                 # BOK Snapshot에서 수동 다운로드한 NSI
│   ├── kospi_df.csv                      # 수집 + feature engineering된 KOSPI 원본 데이터
│   └── kospi_yahoo_AB_full_feature.csv   # 최종 모델 학습용 full feature set
│
├── src/
│   ├── 01_data_preprocessing.ipynb       # 데이터 로딩, 정제, NSI merge, feature 생성
│   ├── 02_EDA.ipynb                      # 탐색적 분석, 시장 구조 분석
│   ├── 03_Modeling.ipynb                 # PCA + ElasticNet 모델링 + Backtesting
│   └── requirements.txt
│
└── README.md         
```

---

# 3. Dataset Card

### **Data Components**

| Category             | Columns                                    | Source                          | Notes                       |
| -------------------- | ------------------------------------------ | ------------------------------- | --------------------------- |
| Price Data           | date, open, high, low, close, volume       | Yahoo Finance (^KS11)           | via `yfinance`              |
| Returns / Momentum   | ret_1d, ret_5d, ret_22d, momentum_10/20/60 | Derived                         | Sliding-window calculations |
| Trend Indicators     | ma_* , MACD, signal, histogram             | Derived                         | Captures regime behavior    |
| Volatility / Risk    | vol_22d, ATR, Bollinger Bands, RSI         | Derived                         | Reflects uncertainty        |
| Drawdowns            | drawdown_60, max_drawdown_60               | Derived                         | Downside risk behavior      |
| External Indicators  | VIX, USD/KRW, Gold price                   | Yahoo Finance                   | Global risk signals         |
| News Sentiment Index | NSI                                        | **BOK Snapshot Excel download** | Added manually    |

### **Collection Notes**

* All financial price series collected automatically with `yfinance`.
* NSI (뉴스심리지수.xlsx) was **manually downloaded** from:
  [https://snapshot.bok.or.kr/dashboard/C8](https://snapshot.bok.or.kr/dashboard/C8)
* The dataset contains **all raw and engineered features** required for modeling.

---

# 4. Modeling Pipeline

### ✔ PCA (Dim = 5)

* Explained ~85% of variance.
* PC1 = market trend
* PC2 = volatility regime
* PC5 = sentiment + FX shock loading (Korea-specific)

### ✔ ElasticNet on PCA

* Best-performing linear model
* Nonlinear models (LightGBM/XGB) **overfit** KOSPI’s high-noise structure

### ✔ Time-Series Cross-Validation

* 5-fold `TimeSeriesSplit`
* PCA fit only on train split → **leakage-free**

### ✔ Volatility-Constrained Allocation

```
z_t = standardize(pred_t)
w_t = clip(1 + k·z_t, 0, 2)
scale if volatility exceeds 1.2× benchmark
```

---

# 5. Results Summary

### **Test Out-of-Sample (OOS) Results**

| Metric            | Benchmark | PCA+ElasticNet    |
| ----------------- | --------- | ----------------- |
| Annualized Return | –6.7%     | +2.8%             |
| Annualized Vol    | 16.4%     | 19.7%             |
| Vol Ratio         | 1.00      | **1.20 (capped)** |
| Sharpe            | –0.41     | **0.14**          |
| Max Drawdown      | –39.2%    | –33.0%            |

### Interpretation

* Predictability in KOSPI exists but is **very weak**.
* However, PCA+ElasticNet extracts small signals that translate into a **Sharpe improvement under a volatility cap**.
* NSI & USDKRW contribute to Korea-specific factors → not present in S&P 500.
* Nonlinear models degrade performance due to overfitting.

---

# 6. How to Run Everything

### Install Dependencies

```
pip install -r src/requirements.txt
```

### Run the workflow

1. `01_data_preprocessing.ipynb`
   → Load Yahoo Finance price data
   → Merge NSI
   → Create full feature dataset

2. `02_EDA.ipynb`
   → Plot rolling stats, vol regimes, sentiment correlation

3. `03_Modeling.ipynb`
   → PCA, ElasticNet, time-series CV
   → Backtest (cumulative return / drawdown / rolling vol)
   → Export results figures for report appendix

All notebooks produce reproducible outputs using only files inside `data/`.

---

# 7. Compliance with Course Requirements

✔ Dataset card (included above)

✔ Reproducible source code (in `src/`)

✔ Figures & results for Appendix (generated in notebook 03)

✔ Volatility-constrained strategy

✔ EMH interpretation & qualitative discussion

✔ No external API requiring credentials

✔ No leakage in modeling (verified via PCA per-fold training)

---

# 8. License & Disclaimer

None of the models, signals, or strategies should be used for real-world investment.