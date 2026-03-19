# 🇳🇬 Nigerian Government Securities — Yield Curve Analysis
### NTB & FGN Bond Markets | 2001 – 2026 | 25 Years of Auction Data

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0%2B-150458?logo=pandas)](https://pandas.pydata.org/)
[![SciPy](https://img.shields.io/badge/SciPy-Optimization-8CAAE6?logo=scipy)](https://scipy.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Linear%20Algebra-013243?logo=numpy)](https://numpy.org/)
[![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualisation-11557c)](https://matplotlib.org/)
[![Platform](https://img.shields.io/badge/Platform-Google%20Colab-F9AB00?logo=googlecolab)](https://colab.research.google.com/)

---

## Project Overview

This project builds a **complete yield curve analysis system** for Nigerian government
securities using 25 years of Central Bank of Nigeria (CBN) primary market auction data.

Starting from raw, messy auction records, the project constructs a full yield curve
spanning the short end (91-day NTBs) through the long end (30-year FGN Bonds),
then applies three quantitative methods to extract economic insight:

- **Nelson-Siegel curve fitting** — smooth mathematical representation of the curve
- **Principal Component Analysis (PCA)** — decomposing yield movements into Level, Slope, Curvature
- **Bid-to-Cover (BTC) analysis** — measuring investor demand intensity at each auction

The project also performs a **five-regime historical analysis**, connecting yield curve
behaviour to major economic events in Nigeria's recent history — oil booms, recessions,
currency crises, and monetary policy cycles.

> **Why Nigeria?** Most yield curve research focuses on US Treasuries or European sovereign
> debt. Nigerian fixed income is underrepresented in quantitative finance literature
> despite being the largest economy in Africa. This project applies standard fixed income
> methodology to an emerging market context — and finds that the same universal structures
> hold (Level-Slope-Curvature decomposition) even in a high-inflation, oil-dependent economy.

---

## What Are These Instruments?

For readers new to Nigerian fixed income markets:

| Instrument | Issuer | Maturity Range | Mechanism | Purpose |
|---|---|---|---|---|
| **NTB** (Nigerian Treasury Bill) | CBN | 91, 182, 364 days | Discount — buy below face value | Short-term government liquidity |
| **FGN Bond** | DMO | 2 to 30 years | Coupon — regular interest payments | Long-term infrastructure financing |
| **OMO Bill** | CBN | 7 to 364 days | Discount | CBN liquidity management tool |

Together, NTBs and FGN Bonds form the **Nigerian sovereign yield curve** — the benchmark
against which all other Nigerian debt instruments are priced.

---

## What This Project Produces

### Chart 1 — Full Yield Time Series (2001–2026)
All 12 maturity buckets plotted together with crisis period shading.
Short-end NTBs (dashed lines) vs long-end FGN Bonds (solid lines),
plus a yield curve slope panel showing steepness over time.



---

### Regime Yield Curves
Average yield curve shape across five distinct economic periods.
Each line represents the *typical* curve shape during that regime.



> **Reading this chart:** The orange line (CBN Hiking Cycle 2022–2024) shows a partially
> inverted shape — the 1-year yield exceeded the 10-year yield — reflecting aggressive
> short-end rate hikes while long-run expectations remained anchored.

---

### Nelson-Siegel Rolling Parameters
β₀ (Level), β₁ (Slope), and β₂ (Curvature) estimated monthly across the full dataset.
Parameters constrained with economic priors to prevent optimizer divergence on sparse data.



---

###  Bid-to-Cover vs Yield
Demand intensity at NTB primary market auctions plotted against stop-out yields.
Colour-coded bars: green = oversubscribed (BTC > 2x), red = undersubscribed (BTC < 1x).



---

### BTC vs Yield Scatter
Correlation between auction demand and yield outcomes — testing whether strong
demand translates to lower yields (the fundamental demand-supply relationship).



---

### Nelson-Siegel Point-in-Time Fits
Fitted NS curves for two specific dates (June 2015 and March 2025),
showing how the curve shape evolved over a decade of monetary policy change.

| June 2015 | March 2025 |
|---|---|
| β₁ = −14.3 (steep upward slope) | β₁ = −3.0 (nearly flat) |
| β₀ = 20.4% (high level) | β₀ = 18.5% (compressed level) |

---

## Data

### Source
**Central Bank of Nigeria (CBN)** — Government Securities Auction Records
- URL: [CBN Statistical Database](https://www.cbn.gov.ng/rates/govtsecurities.asp)
- File hosted at: `data/Government_Securities.csv`

### Coverage
| Field | Detail |
|---|---|
| Date range | April 2001 — March 2026 |
| Raw records | 4,634 rows |
| Instruments | NTB, FGN Bond, OMO, and others |
| After cleaning | 3,067 rows (NTB + FGN Bond only) |
| Yield curve table | 1,482 dates × 12 tenor columns |

### Key Columns
| Column | Description |
|---|---|
| `auctionDate` | Date auction was held |
| `securityType` | Instrument type (cleaned from 16 variants to 3) |
| `tenor` | Duration label (91DAY, 182DAY, 5Y, etc.) |
| `rate` | Stop-out yield — the accepted interest rate (%) |
| `totalSubscription` | Total investor bids submitted (₦ millions) |
| `amtOffered` | Amount government offered to issue (₦ millions) |
| `maturityDate` | Date instrument matures |

### Data Quality Issues Encountered
- `securityType` column had 16 inconsistent labels for 3 instruments
- 21 records had negative tenor days (data entry errors — excluded)
- `trueYield` column was zero-filled (unusable — `rate` column used instead)
- Significant sparsity in FGN Bond tenors beyond 10Y prior to 2010

---

## Methods

### 1. Data Pipeline
```
Raw CBN CSV (4,634 rows)
    ↓ Standardise security type labels
    ↓ Parse dates, compute tenor in days
    ↓ Classify into 12 standard tenor buckets
    ↓ Filter: NTB + FGN Bond only
    ↓ Pivot to wide format (dates × tenors)
    ↓ Replace zeros with NaN, forward-fill (limit=7 days)
    → Clean yield curve table (1,482 × 12)
```

---

### 2. Nelson-Siegel Model

The Nelson-Siegel (1987) model fits a smooth yield curve using three economically
interpretable parameters:

```
y(τ) = β₀ + β₁·f₁(λτ) + β₂·f₂(λτ)

where:
  f₁(x) = (1 - e⁻ˣ) / x          ← loading for slope factor
  f₂(x) = (1 - e⁻ˣ) / x - e⁻ˣ   ← loading for curvature factor
```

| Parameter | Economic Meaning | Sign Convention |
|---|---|---|
| β₀ | Long-run yield level — where the curve anchors at ∞ | Always positive |
| β₁ | Slope — difference between short and long rates | Negative = upward sloping |
| β₂ | Curvature — hump or trough in medium maturities | Positive = hump shape |
| λ  | Decay rate — speed of transition from short to long | Always positive |

**Implementation note:** Parameters are estimated by minimising sum of squared errors
(SSE) using `scipy.optimize.minimize` with the L-BFGS-B algorithm.

**Prior constraints applied** (to prevent optimizer divergence on sparse data):
```python
bounds = [
    (5,   35),   # β₀ — Nigerian long-run yields have been in this range historically
    (-30, 30),   # β₁ — physically bounded slope magnitude
    (-30, 30),   # β₂ — physically bounded curvature magnitude
    (0.01, 10),  # λ  — must be positive and finite
]
```

> **Why bounds matter:** Without constraints, the optimizer fits the NTB-only data
> (max 1 year maturity) and extrapolates β₀ to physically implausible values like 1,200%.
> The bounds encode *prior knowledge* — a Bayesian concept — about what is economically
> reasonable for Nigerian long-run yields.

---

### 3. Principal Component Analysis (PCA)

PCA decomposes the covariance structure of yield changes across tenors into orthogonal
factors, answering: *"What are the hidden drivers behind correlated yield movements?"*

**Implementation:**
```python
# Standardise yields (zero mean, unit variance)
standardized = (yields - yields.mean()) / yields.std()

# Compute covariance matrix and eigendecompose
cov_matrix = standardized.cov()
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# Sort by descending eigenvalue
variance_explained = eigenvalues / eigenvalues.sum() * 100
```

**Results (NTB tenors — 91D, 182D, 364D):**

| Component | Eigenvalue | Variance Explained | Economic Interpretation |
|---|---|---|---|
| PC1 | 2.47 | **82.5%** | Level — all tenors move together |
| PC2 | 0.48 | **15.8%** | Slope — short vs long end divergence |
| PC3 | 0.05 | **1.7%** | Curvature — middle vs ends |

> This replicates the Litterman & Scheinkman (1991) three-factor structure — first
> documented for US Treasuries — using Nigerian emerging market data.

---

### 4. Bid-to-Cover Analysis

```
BTC Ratio = Total Subscription (investor bids) / Amount Offered (government supply)

BTC > 2x  →  Oversubscribed — government has pricing power → can accept lower yields
BTC = 1x  →  Break-even — market yield required
BTC < 1x  →  Undersubscribed — government must raise yields to attract investors
```

**Correlation results:**

| Tenor | BTC-Yield Correlation | Interpretation |
|---|---|---|
| 91D  | **−0.359** | Moderate — demand suppresses short yields |
| 182D | **−0.247** | Weak — same direction, noisier |
| 364D | **+0.060** | Near zero — demand and yield decouple |

The 364D result is economically interesting: the most popular NTB tenor attracts
structurally different investors (pension funds, banks) who bid regardless of yield
level — breaking the normal demand-yield relationship.

---

## Five Economic Regimes

| Regime | Period | Dominant Driver | Curve Shape |
|---|---|---|---|
| Pre-Crisis | 2001–2008 | Post-liberalisation adjustment | Steeply upward — β₁ avg −12.7 |
| Post-Crisis | 2009–2015 | Oil boom stability | Gently upward — β₀ lowest at 13.0% |
| Oil Recession | 2016–2017 | Naira crisis + recession | Hump shape — β₂ spiked to +17.1 |
| Stability | 2018–2021 | Recovery + COVID disruption | Normal upward, brief COVID dip |
| CBN Hiking Cycle | 2022–2024 | 30%+ inflation + aggressive hikes | Flat/partial inversion — β₀ = 24.9% |

---

## Key Findings

1. **The Level-Slope-Curvature structure is universal** — PC1 (82.5%), PC2 (15.8%),
   PC3 (1.7%) replicates the classic Litterman & Scheinkman decomposition on Nigerian data.

2. **Short-end yields are twice as volatile as long-end yields** — NTB standard deviations
   of 4.7–5.3% vs 2.2–2.5% for FGN Bonds. Short rates are CBN-driven; long rates are
   anchored by long-run inflation expectations.

3. **The 2022–2024 hiking cycle was historically unprecedented** — β₀ reached 24.9%,
   the highest long-run anchor in the 25-year dataset. Short-end NTB rates moved from
   ~5% to 20%+ in under two years.

4. **The Oil Recession (2016–2017) produced the highest curvature ever recorded** —
   β₂ averaged +17.1, reflecting a hump at the 1–2 year maturity as investors priced
   peak near-term uncertainty into medium-term instruments specifically.

5. **Demand signals work best at the very short end** — BTC-yield correlation of −0.36
   for 91D NTBs confirms the demand-supply mechanism. This correlation weakens at longer
   NTB tenors where structural investors dominate.

6. **Nigeria's NTB curve has never formally inverted** — the 364D−91D spread stayed
   positive throughout the dataset, meaning CBN raised all NTB tenors together rather
   than just the very short end during tightening cycles.

---

## Real-World Industry Applications

This analysis directly mirrors work done by professionals across Nigerian financial markets:

| Role | How This Analysis Is Used |
|---|---|
| **Fixed Income Trader** | NS parameters signal when the curve is cheap/rich at specific maturities — generating relative value trade ideas |
| **Portfolio Manager** | Regime classification informs duration positioning — reduce duration before hiking cycles, extend during easing |
| **Risk Manager** | PCA factors are used to compute yield curve VaR — measure portfolio sensitivity to Level, Slope, Curvature shocks |
| **CBN / DMO Analyst** | BTC trends inform auction sizing decisions — issue more when demand is strong, less when BTC falls |
| **Credit Analyst** | The sovereign yield curve is the risk-free benchmark — credit spreads for corporate bonds are measured against it |
| **Macro Strategist** | Curve shape (normal vs inverted) feeds into economic outlook reports and interest rate forecasts |

> **In simple terms:** Every Nigerian bank, pension fund, insurance company, and
> asset manager uses some version of this analysis to decide how much government
> paper to hold, at what maturity, and at what price. Understanding the yield curve
> is the foundation of fixed income investing.

---

## How to Run

### Requirements
```bash
pip install pandas numpy matplotlib scipy
```

### Steps
1. Clone this repository
2. Open `yield_curve_analysis.ipynb` in Google Colab or Jupyter
3. Run all cells in order — data loads directly from the GitHub-hosted CSV
4. Extension notebooks (`yield_curve_extensions.ipynb`, `ns_rolling_fix.ipynb`)
   must be run **after** the base notebook (they depend on variables in memory)

### File Structure
```
├── yield_curve_analysis.ipynb       ← Base notebook: data pipeline + core analysis
├── data/
│   └── Government_Securities.csv    ← Raw CBN auction data (4,634 rows)
├── charts/
│   ├── nigeria_yield_curve_full.png
│   ├── regime_yield_curves.png
│   ├── regime_comparison.png
│   ├── ns_rolling_parameters_fixed.png
│   ├── ntb_btc_vs_yield.png
│   ├── btc_yield_scatter.png
│   ├── Nelson-Siegel_Fitted_Yield_Curve.png
│   └── Nelson-Siegel_Fitted_Yield_Curve_2015-06-24.png
└── README.md
```

---

## Future Improvements

| Improvement | Description | Complexity |
|---|---|---|
| **Full-curve NS fit** | Include FGN Bond data points in the NS fit — currently NTB-only | Medium |
| **Svensson extension** | Add a fourth parameter (second hump) to improve long-end fit | Medium |
| **Yield curve VaR** | Use PCA factor loadings to compute portfolio Value-at-Risk | High |
| **Macro factor regression** | Regress NS parameters against oil prices, inflation, MPR, FX | Medium |
| **Real-time dashboard** | Connect to live FMDQ/DMO data feeds for current curve | High |
| **Corporate spread analysis** | Build credit spread curves on top of the sovereign baseline | High |
| **Forecasting model** | Use rolling NS parameters to forecast future yield levels | High |

---

## References

- Nelson, C.R. & Siegel, A.F. (1987). *Parsimonious Modeling of Yield Curves*. Journal of Business, 60(4), 473–489.
- Litterman, R. & Scheinkman, J. (1991). *Common Factors Affecting Bond Returns*. Journal of Fixed Income, 1(1), 54–61.
- Central Bank of Nigeria — [Government Securities Data](https://www.cbn.gov.ng)
- Debt Management Office Nigeria — [FGN Bond Issuance](https://www.dmo.gov.ng)
- FMDQ Exchange — [Secondary Market Yields](https://fmdqgroup.com)

---

## Author

**Olayinka Yusuf**
Student — Financial Engineering & Quantitative Finance
WorldQuant University — Module 1: Financial Data

*This project was developed as part of a structured learning programme in financial data
analysis and quantitative fixed income methods. All data is sourced from public CBN
auction records.*

---

*⭐ If you found this project useful, feel free to star the repository.*
