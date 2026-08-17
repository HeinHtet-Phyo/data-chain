# 🔗 DATA_CHAIN — Supply Chain Optimisation & Demand Forecasting

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square&logo=python)
![Gurobi](https://img.shields.io/badge/Gurobi-ILP_Solver-red?style=flat-square)
![SARIMA](https://img.shields.io/badge/SARIMA-Time_Series-green?style=flat-square)
![XGBoost](https://img.shields.io/badge/scikit--learn-ML-orange?style=flat-square)
![UWE Bristol](https://img.shields.io/badge/UWE_Bristol-Big_Data_Analytics-purple?style=flat-square)

**Three-echelon supply chain optimisation over a 12-month horizon.**  
Combines SARIMA + Linear Regression demand/price forecasting with Gurobi integer linear programming to maximise profit across 2 factories, 4 depots, and 10 UK retailers.

**💰 Optimal Profit Achieved: £7,402,632**

</div>

---

## 📌 Project Overview

Current supply chains waste millions through suboptimal routing, poor demand forecasting, and inefficient depot selection. **DATA_CHAIN** solves this by:

1. **Forecasting** 2026 demand and price for 10 UK retailers using SARIMA and Linear Regression
2. **Optimising** factory production, depot selection, and transport routes using Gurobi ILP
3. **Extending** the model with logical business constraints (exclusive supplier rules, single-source requirements)

> **Module:** Big Data Analytics Portfolio — Section 4  
> **University:** UWE Bristol | **Student ID:** 25036746

---

## 🏗️ Supply Chain Network

```
  FACTORIES          DEPOTS              RETAILERS
  ─────────          ──────              ─────────
  Swindon  ────────► Leicester ────────► Nottingham
  (1,000/mo)         (300 cap)           Cambridge
                                         Cardiff
  Sheffield ───────► Coventry ─────────► Norwich
  (1,250/mo)         (200 cap)           Portsmouth
                                         Birmingham
             ──────► Bristol ──────────► Exeter
                     (300 cap)           Preston
                                         Leeds
             ──────► Bedford ──────────► Oxford
                     (150 cap)

  Direct routes: Factory → Retailer (bypassing depots)
```

---

## 📊 Task 4.1 — Demand & Price Forecasting

### Models Compared

| Model | Approach | Best For |
|-------|----------|----------|
| **Linear Regression** | Trend + monthly dummy features | Price forecasting (all 10 retailers) |
| **SARIMA** | Seasonal ARIMA via `auto_arima` (m=12) | Demand forecasting (8/10 retailers) |

### Validation Strategy
- **Walk-forward validation** — train on first 24 months, test on last 12
- **Best model selected per retailer** based on test RMSE
- SARIMA wins demand forecasting for 8/10 retailers; LR wins all price forecasts

### 2026 Forecast Summary (10 Retailers)

| Metric | Value |
|--------|-------|
| Total predicted demand (2026) | **23,901 units** |
| Price range | **£369 – £753 / unit** |
| Peak demand month | **July** (summer surge) |
| Lowest demand month | **January** |

---

## ⚙️ Task 4.3 — Supply Chain Optimisation (Gurobi ILP)

### Objective Function

```
Maximise Z = Revenue
           − Production Cost  (£225/unit, both factories)
           − Transport Cost   (factory→depot, factory→retailer, depot→retailer)
           − Depot Cost       (Leicester £16K | Coventry £13K | Bristol £12.5K | Bedford £14K)
           − Holding Cost     (£20/unit/month at factories)
           − Shortage Cost    (£200/unit/month at factories)
```

### Decision Variables

| Variable | Description |
|----------|-------------|
| `x1_fdt` | Units: factory → depot (per period) |
| `x2_frt` | Units: factory → retailer direct (per period) |
| `x3_drt` | Units: depot → retailer (per period) |
| `Prod_ft` | Production quantity at factory f, period t |
| `Inven_ft` | End-of-period inventory at factory f |
| `Short_ft` | Shortage units at factory f, period t |
| `y_d` | Binary — depot d open (1) or closed (0) |

### Key Constraints

- ✅ Factory production capacity (Swindon: 1,000/mo | Sheffield: 1,250/mo)
- ✅ Depot transshipment capacity (no inventory storage at depots)
- ✅ Full demand satisfaction at all 10 retailers every period
- ✅ Factory inventory/shortage balance with carry-forward
- ✅ Non-negativity + binary depot selection

### Result

```
💰 Maximum Profit: £7,402,632
```

---

## 🔒 Task 4.4 — Additional Business Constraints

Three logical constraints added simultaneously using **Big-M method** (`M = 100,000`):

| Constraint | Rule |
|-----------|------|
| **i** | Leicester cannot supply **both** Nottingham and Portsmouth |
| **ii** | Any node supplying Birmingham **must also** supply Cardiff |
| **iii** | Cardiff must be supplied by **exactly one** factory or depot |

Binary variable `z_nr` = 1 if node n supplies retailer r (linked to flow via Big-M linking constraints).

---

## 📁 Repository Structure

```
data-chain/
├── 25036746_4.ipynb          ← Main Jupyter notebook (Google Colab)
├── 25036746_4.pdf            ← Mathematical model formulation (PDF)
├── 25036746_4a.csv           ← Output: 2026 demand forecasts (120 rows)
├── 25036746_4b.csv           ← Output: 2026 price forecasts (120 rows)
├── README.md
└── data/                     ← (add source data files here)
    ├── retailer_demand_3years.csv
    └── sales_price_2years.csv
```

---

## 🚀 How to Run

### Google Colab (Recommended)

```bash
# 1. Open the notebook in Google Colab
# 2. Upload data files to Google Drive at:
#    My Drive/BigData_Section4/
# 3. Run all cells in order — takes ~5 minutes
```

### Local Setup

```bash
# Clone
git clone https://github.com/HeinHtet-Phyo/data-chain.git
cd data-chain

# Install dependencies
pip install gurobipy pmdarima scikit-learn pandas numpy matplotlib

# ⚠️  Gurobi requires a licence (free academic licence available at gurobi.com)

# Run
jupyter notebook 25036746_4.ipynb
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python 3.10+** | Core language |
| **Gurobi + gurobipy** | Integer Linear Programming solver |
| **pmdarima (auto_arima)** | SARIMA seasonal time series forecasting |
| **scikit-learn** | Linear Regression with seasonal features |
| **pandas / numpy** | Data processing |
| **matplotlib** | Visualisation |

---

## 📈 Key Results

| Metric | Value |
|--------|-------|
| **Optimal profit** | **£7,402,632** |
| Total demand forecasted | 23,901 units across 10 retailers |
| SARIMA wins (demand) | 8 / 10 retailers |
| LR wins (price) | 10 / 10 retailers |
| Planning horizon | 12 months (Jan–Dec 2026) |
| Network size | 2 factories · 4 depots · 10 retailers |

---

## 📚 Academic Context

| Field | Detail |
|-------|--------|
| Module | Big Data Analytics — Portfolio Section 4 |
| University | UWE Bristol |
| Tasks | 4.1 Forecasting · 4.3 Optimisation · 4.4 Extended Constraints |
| Solver | Gurobi 10+ (academic licence) |
| Notebook | Google Colab |

---

## 👤 Author

**Hein Htet Phyo**  
BSc Data Science & AI — UWE Bristol (First Class Honours)  
[GitHub](https://github.com/HeinHtet-Phyo) · [LinkedIn](https://linkedin.com/in/hein-htet-phyo) · [Portfolio](https://hhp-portfolio-final.vercel.app)
