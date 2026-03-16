# FlightSense Pro — Diwali 2025 Flight Price Predictor

<p align="center">
  <img src="outputs/route_comparison.png" width="100%" alt="Route Comparison">
</p>

A multi-model ML system that predicts flight prices around Indian festival peaks (Diwali), generates booking signals (BUY NOW / WAIT / SET ALERT), and compares fares across 4 airlines on 2 routes.

---

##  Features

| Feature | Description |
|---|---|
|  Live + Simulated Data | Amadeus API with local disk cache; no API key needed in simulate mode |
|  3 ML Models | LightGBM/XGBoost, Holt-Winters (SARIMA proxy), MLP Neural Network |
|  Ensemble Forecasting | Inverse-RMSE weighted blend of all three models |
|  90% Confidence Intervals | Bootstrap resampling (80 iterations default) for uncertainty quantification |
|  Competitor Comparison | IndiGo vs Air India vs SpiceJet vs Akasa — 4-airline fare tracking |
|  Booking Engine | BUY NOW / WAIT / SET ALERT / UNCERTAIN signals with confidence scores |
| Export | JSON (API-ready) + CSV recommendations output |

---

## Installation

```bash
git clone https://github.com/YOUR_USERNAME/flightsense-pro.git
cd flightsense-pro

# Install Python dependencies
pip install pandas numpy matplotlib scikit-learn scipy xgboost lightgbm requests

# Optional: Jupyter
pip install jupyter
```

---

##  Quick Start

### Simulate Mode (no API key needed)
```bash
jupyter notebook flight_tracker_pro.ipynb
```
In **Cell 4 (Configuration)**, ensure:
```python
DATA_MODE = 'simulate'
```
Run all cells. Outputs saved to `flightsense_outputs/`.

### Live Mode (Amadeus API)
1. Register at [developers.amadeus.com](https://developers.amadeus.com) — free tier available
2. In **Cell 4**, set:
```python
DATA_MODE      = 'live'
AMADEUS_ID     = 'your_client_id'
AMADEUS_SECRET = 'your_client_secret'
AMADEUS_ENV    = 'test'  # or 'production'
```

---

##  Project Structure

```
flightsense-pro/
├── flight_tracker_pro.ipynb          # Main notebook (run this)
├── flightsense_outputs/
│   ├── recommendations.csv            # Booking recommendations table
│   ├── flightsense_output.json        # Full API-ready JSON output
│   └── price_cache/                   # API response cache (auto-created)
├── outputs/
│   ├── BLR_IXR_analysis.png          # Bengaluru → Ranchi full analysis
│   ├── BLR_LKO_analysis.png          # Bengaluru → Lucknow full analysis
│   └── route_comparison.png          # Side-by-side route comparison
└── README.md
```

---

##  Model Architecture

```
Historical Data (2022–2024)  +  Current Year (2025)
          │
          ▼
   Feature Engineering (21 features)
   ├── Temporal: days_to_diwali, day_of_week, is_weekend
   ├── Surge: diwali_surge_exp, diwali_surge_quad, return_rush
   ├── Load Factor: load_factor, lf_shortfall, lf_urgency
   └── Price Memory: lag_1/3/7/14, roll3/7/14, std7
          │
          ├── GBM (LightGBM / XGBoost)     — RMSE ₹442 (BLR→IXR)
          ├── SARIMA proxy (Holt-Winters)   — RMSE ₹2,281
          └── Neural Net (MLP 128→64→32)   — RMSE ₹350
          │
          ▼
   Inverse-RMSE Ensemble
   (Neural 51.4%, GBM 40.7%, SARIMA 7.9%)
          │
          ├── 90% Bootstrap CI (80 iterations)
          └── Booking Signal Engine
              ├── BUY NOW   — within 5% of predicted floor
              ├── WAIT      — >15% above predicted floor + ≥14 days to Diwali
              ├── SET ALERT — high uncertainty (CI width > 20%)
              └── UNCERTAIN — <7 days to Diwali
```

---

##  Sample Output

| Date | Route | Price | Signal | Confidence | Cheapest Airline |
|------|-------|-------|--------|------------|-----------------|
| 2025-11-04 | BLR→IXR | ₹14,777 | UNCERTAIN | 94.6% | Akasa (₹14,112) |
| 2025-11-05 | BLR→LKO | ₹16,181 | UNCERTAIN | 94.1% | IndiGo (₹16,074) |
| 2025-11-10 | BLR→IXR | ₹14,712 | UNCERTAIN | 91.4% | IndiGo (₹14,336) |

---

##  Integration

### Webhook on BUY signal
```python
import requests
for rec in recommendations:
    if rec['signal'] == 'BUY_NOW' and rec['confidence'] > 0.70:
        requests.post('https://your-app.com/webhook/price-alert', json=rec)
```

### Daily Cron (Linux/Mac)
```bash
0 7 * * * cd /path/to/project && jupyter nbconvert --to notebook --execute flight_tracker_pro.ipynb
```

---

##  Performance

| Route | GBM RMSE | Neural RMSE | Ensemble RMSE |
|-------|----------|-------------|---------------|
| BLR → Ranchi | ₹442 | ₹350 | ₹451 |
| BLR → Lucknow | ₹658 | ₹473 | ₹644 |

---

##  Key Configuration (Cell 4)

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DATA_MODE` | `'simulate'` | `'simulate'` or `'live'` |
| `N_BOOTSTRAP` | `80` | Bootstrap iterations (↑ = tighter CI, slower) |
| `CI_LEVEL` | `0.90` | Confidence interval level |
| `CURRENT_LF` | `0.35` | Current seat load factor |
| `BUY_THRESHOLD` | `0.05` | Within 5% of floor → BUY NOW |
| `WAIT_THRESHOLD` | `0.15` | >15% above floor → WAIT |

---

##  License

MIT License — free to use, modify, and distribute.

---

*Built for Diwali 2025 travel planning. Extend to any route pair by editing `PRIMARY_ROUTES` in Cell 4.*
