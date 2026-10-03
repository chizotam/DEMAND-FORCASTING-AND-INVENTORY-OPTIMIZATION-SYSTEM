# Demand Forecasting & Inventory Optimization System

A machine learning system that forecasts daily retail demand with LightGBM and turns those forecasts into safety stock, reorder points, and replenishment decisions. An interactive web page lets anyone try the decision logic without installing anything.

**[Try the live demo](https://chizotam.github.io/DEMAND-FORCASTING-AND-INVENTORY-OPTIMIZATION-SYSTEM/)**

![Reorder decision for FOODS_1_001 at store CA_1](INVENTORY%20OPTIMIZATION%20SNIPPET.png)

## What it does

Retailers need enough stock to meet demand without over-stocking. This project connects the two halves of that problem:

1. **Forecasting:** a LightGBM model predicts daily unit demand for each product-store pair.
2. **Inventory policy:** the forecast and its error are converted into expected lead-time demand, safety stock, and a reorder point.
3. **Backtest:** the policy is tested against real historical demand to measure fill rate and stockouts.
4. **Web interface:** a static page shows the full workflow and returns a REORDER or HOLD STOCK decision.

## Results

Validation covers the final 30 days of history (26 Mar to 24 Apr 2016), using a time-based split rather than a random one.

| Model | RMSE | MAE |
| --- | --- | --- |
| Lag-7 baseline | 2.70 | 1.22 |
| LightGBM (final, tuned) | ≈1.916 | ≈0.969 |

Prediction-to-actual correlation on the validation set is about 0.854.

Inventory policy backtest (7-day lead time):

| Policy | Fill rate | Stockout rate |
| --- | --- | --- |
| Baseline policy | 91.07% | 4.35% |
| 90% service level | 98.19% | 0.67% |
| 95% service level | 98.91% | 0.36% |
| 98% service level | 99.38% | 0.18% |

<!-- TODO: add one line saying what the baseline policy was -->

The service level (for example 95%) is the setting used to calculate safety stock. It targets the share of replenishment waiting periods with no stockout. The fill rate and stockout rate above are measured results from the historical backtest, and they are not guarantees of future performance.

## How it works

```
M5 sales data -> preprocessing and feature engineering
              -> LightGBM demand model
              -> forecast error and uncertainty
              -> inventory policy (safety stock, reorder point)
              -> backtest on validation demand
              -> exported policy data -> static web page
```

The policy formulas:

- Lead-time demand = average predicted daily demand × lead time
- Safety stock = z × forecast error standard deviation × √lead time
- Reorder point = lead-time demand + safety stock
- Reorder when inventory + units on order ≤ reorder point
- Order quantity = lead-time demand, rounded up, minimum 1 unit

The z values are 1.2816 (90%), 1.645 (95%), and 2.0537 (98%).

## The web interface

Choose a product, store, service level, lead time, and current inventory. The page shows the forecast, safety stock, reorder point, and the final REORDER or HOLD STOCK decision, with the calculation behind each step.

- The LightGBM model is **not** run in the browser. Forecasts were generated offline in Python and exported as compact data, so the page is fast, free to host, and needs no server.
- The demo includes **150 product-store pairs** (15 products across all 10 stores), not all 30,490.
- The forecasts shown come from the historical validation period. This is not a live forecasting service.

## Dataset

The [M5 Forecasting](https://www.kaggle.com/competitions/m5-forecasting-accuracy) retail data: 30,490 product-store series, 1,913 days of sales, plus calendar events and weekly selling prices. About 57.5 million product-store-day rows were created after reshaping, and about 68% of them are zero-sales days. The data is not included in this repository. It can be downloaded from Kaggle.

## What is in this repository

```
.
├── index.html          # the web interface (forecast data embedded)
├── images/             # screenshot used in this README
├── README.md
└── Demand_Forecasting_Inventory_Optimization_Documentation.docx
```

- `index.html` is the interactive page, with the exported policy data built in.
- The technical documentation covers the full workflow in detail, including the policy formulas and the backtest.

The full training notebooks and the trained model file are not included in this repository.

## Running it

Open the live demo above, or download `index.html` and open it in any browser. No installation or server is needed.

## Limitations

- Validation covers only 30 days, so the results are not a measure of long-term performance.
- The safety stock uses forecast errors from the same period the policy is tested on, which can make the backtest look better than it would on unseen data.
- The simulation assumes a fixed 7-day lead time and starts each product-store pair with inventory equal to its reorder point, not observed inventory.
- The model underpredicts high-demand days, which raises stockout risk for fast sellers.
- The order quantity is a simple lead-time-demand rule. It does not optimize holding, shortage, or ordering costs.

## Documentation

Full details, including exploratory analysis, feature engineering, and error analysis, are in the [technical documentation](Demand_Forecasting_Inventory_Optimization_Documentation.docx).

## Built with

Python, pandas, NumPy, LightGBM, HTML, CSS, JavaScript. Developed on a 16 GB RAM laptop with no GPU, so memory use shaped the preprocessing choices.

## Author

Zplays · [GitHub](https://github.com/chizotam)


Zplays · [GitHub](https://github.com/chizotam)
