# 🔬 Real-World Stress Test #03 — Olist E-Commerce Stockout Stress Test

> **Don't just ask whether a forecast works. Ask how it behaves when the observations it depends on suddenly change.**

This repository is part of the **Real-World Stress-Test Series** — a set of focused analytical experiments designed to examine how data-driven methods behave under simulated changes in real-world conditions.

Instead of building another standard sales dashboard, this experiment asks:

> **How does a simple rolling-average forecast respond when recorded sales activity is artificially set to zero for a simulated 14-day period?**

The goal is **not** to claim that a real stockout occurred or to estimate actual lost sales. This experiment measures how a forecast changes when its input observations are deliberately modified.

---

## 🎯 Why This Experiment?

A forecast can be useful under historical conditions but may respond differently when the observations used to produce it change.

In e-commerce, zero recorded sales can have several explanations. A product might have no purchases on a particular day, or the data may not capture the reason for that lack of activity. A sales dataset alone does not prove that an item was unavailable.

This experiment therefore focuses on:

- Forecast sensitivity
- Time-series behaviour
- Controlled stress testing
- Forecast error comparison
- Reproducibility
- Responsible interpretation
- Business implications of incomplete sales and inventory information

The broader question is:

> **Is a forecast's historical performance enough, or should we also examine how it responds when its input data changes?**

---

## 📊 Experiment at a Glance

| Component | Description |
|---|---|
| **Dataset** | Olist Brazilian E-Commerce Public Dataset |
| **Business Domain** | E-commerce / Sales Analytics |
| **Data files** | `olist_orders_dataset.csv` and `olist_order_items_dataset.csv` |
| **Time-series measure** | Daily delivered-order item-record count for a selected product |
| **Forecast baseline** | 7-day rolling average |
| **Stress variable** | Recorded daily item activity |
| **Stress scenario** | Replace 14 consecutive daily observations with zero |
| **Training/test approach** | Chronological 80/20 split |
| **Model retraining** | Not applicable; this is a rolling-average baseline |
| **Primary focus** | Forecast sensitivity and error under simulated zero observations |

---

# 🧪 Experimental Design

### 1. Inspect and prepare the data

The orders dataset contains order timestamps and statuses. The order-items dataset identifies products associated with orders.

The files are joined using `order_id`. The primary analysis keeps delivered orders and counts the selected product's item records by calendar day.

### 2. Build a daily time series

The product with the largest number of delivered-order item records is selected for the main experiment.

Daily item-record counts are aggregated into a calendar-day series. Missing calendar dates within the observed date range are filled with zero so that a seven-observation window represents seven consecutive days.

These counts are used as a **proxy for recorded sales activity**. They are not a verified measure of unit demand or lost demand.

### 3. Establish a baseline

The time series is split chronologically: the earliest 80% of dates are used as the training period and the latest 20% as the test period.

A simple forecast is produced by taking the mean of the previous seven observations. The forecast for a given date uses only observations before that date.

### 4. Apply controlled stress

A 14-day window inside the test period is selected. In a separate copy of the series, recorded activity during this window is replaced with zero.

```text
Original series      Recorded daily activity
Stress scenario      Same series, but 14 observations are set to zero
Forecast window      Previous 7 daily observations
Stress duration      14 consecutive calendar days
```

This is a **simulated zero-recorded-sales scenario**, not evidence of an actual inventory stockout.

### 5. Compare forecast behaviour

The same rolling-average method is applied to the original series and the stress-scenario series.

The experiment compares:

- Forecast paths
- Average forecast during the stress window
- Forecast differences after the stress window
- Baseline test mean absolute error (MAE)
- Stress-scenario test MAE
- Sensitivity across the five most frequently recorded products

---

# 🧠 Key Methodological Decision

A tempting interpretation would be to treat the simulated zero period as proof of a stockout and calculate the difference as actual lost demand.

That would not be supported by the available data.

The Olist files used here do not directly provide reliable item-level inventory availability, stockout timestamps, or verified lost-demand labels. A zero in the constructed daily series means **no delivered-order item record was observed for that product on that date under this data preparation**, not that customers wanted to buy the item but could not.

Therefore, this experiment does **not** claim to measure:

- Actual stockout frequency
- Actual units of lost demand
- Revenue lost because of unavailability
- The causal business impact of inventory shortages

Instead, it asks:

> **How does a rolling-average forecast change when some of its input observations are artificially replaced with zero?**

This distinction is central to the experiment.

---

# 📈 Results

The notebook generates the metrics from the actual data when it is run. Numerical results should be reported only after the notebook has completed successfully.

The main summary is saved to:

`outputs/results/stockout_stress_summary.csv`

It includes:

- Selected product identifier
- Simulated window start and end dates
- Baseline test MAE
- Stress-scenario test MAE
- Mean baseline forecast during the stress window
- Mean stress-scenario forecast during the stress window
- Forecast difference during the stress window
- Mean forecast values after the stress window
- Forecast difference after the stress window

The daily comparison is saved to:

`outputs/results/daily_forecast_comparison.csv`

### How to interpret the results

If the stress-scenario forecast decreases relative to the baseline, this indicates that the rolling-average forecast is responding to the zero-masked observations.

Because the forecast uses the previous seven observations, the impact may continue after the stress window ends. As original observations return to the input series, the two forecast paths may move closer together.

The actual size and duration of the difference depend on the selected product's recorded activity and the timing of the stress window.

**Do not insert numerical results into this README until you have run the notebook and verified the generated output files.**

---

# 🔍 What Was Actually Measured?

### Baseline forecast

The baseline is a 7-day rolling mean. For each date, it calculates the mean of the previous seven daily observations.

### Stress-test analysis

The experiment focuses on:

- Daily forecast values
- Difference between the two forecast paths
- Mean forecast during the simulated window
- Mean forecast difference after the window
- Baseline test MAE against the original recorded activity
- Stress-scenario test MAE against the same original recorded activity
- Differences across five frequently recorded products

MAE measures the average absolute difference between forecast values and the chosen reference observations. Here, the original recorded activity is used as the reference series. It is **not** a verified measure of true demand during a stockout.

---

# 🏗️ Analysis Pipeline

```text
Olist Orders Dataset + Order Items Dataset
                    ↓
            Data Quality Checks
                    ↓
       Filter Delivered Orders and Join
                    ↓
       Select a Frequently Recorded Product
                    ↓
        Build Daily Activity Time Series
                    ↓
          Chronological 80/20 Split
                    ↓
          7-Day Rolling-Average Baseline
                    ↓
             Simulated Zero Window
                    ↓
       Stress-Scenario Rolling-Average Forecast
                    ↓
       Compare Forecasts and Calculate MAE
                    ↓
          Five-Product Sensitivity Check
                    ↓
          Responsible Business Interpretation
```

---

# 💡 Business Interpretation

The experiment demonstrates how a simple forecasting method can respond to an artificial change in its input observations.

If recent daily values are set to zero, the rolling average can decrease because some of the values used in the calculation are now zero. Since the method uses a moving seven-day window, the effect may persist for a number of days.

In a business setting, this highlights the importance of understanding the meaning of zero sales before using a forecast to support decisions.

For example, zero recorded sales could reflect low demand, a product's normal purchase pattern, data coverage issues, or unavailability. These possibilities should not be treated as interchangeable.

Useful questions for a real inventory forecasting system include:

- Was the product available for purchase?
- Were inventory levels recorded accurately?
- Were there any periods when sales could not be observed?
- Does the forecasting data distinguish zero demand from unavailable stock?
- How stable is the forecast under alternative input conditions?
- What additional data is required before making a business decision?

---

# ⚠️ Interpretation Boundary

This experiment **does not establish**:

> “The product was out of stock for 14 days and lost sales as a result.”

Instead, it establishes:

> **“In a simulated scenario, 14 daily observations were replaced with zero, and the rolling-average forecast was compared with the forecast from the original series.”**

These are fundamentally different statements.

### Forecast sensitivity ≠ actual stockout impact

Estimating actual lost demand would require stronger supporting information, such as:

- Inventory availability and stock-level history
- Product availability timestamps
- Customer demand or attempted-purchase signals
- Reliable stockout records
- A suitable method for estimating unobserved demand

This project is a **forecast stress test**, not a causal stockout or lost-sales study.

---

# 📁 Repository Structure

```text
Olist-E-Commerce-Stockout-Stress-Test/
│
├── Data/
│   └── Raw/
│       ├── olist_orders_dataset.csv
│       └── olist_order_items_dataset.csv
│
├── Notebooks/
│   └── Week3_Olist_Stockout_Stress_Test.ipynb
│
├── outputs/
│   ├── figures/
│   │   ├── historical_observed_sales.png
│   │   ├── simulated_stockout_observed_sales.png
│   │   └── forecast_effect_of_stockout.png
│   │
│   └── results/
│       ├── stockout_stress_summary.csv
│       ├── daily_forecast_comparison.csv
│       └── product_sensitivity_check.csv
│
├── requirements.txt
├── .gitignore
└── README.md
```

The dataset files are kept locally and should not be committed to this repository. The folder names above reflect the intended structure; use the same capitalisation as your actual folders when configuring paths.

---

# 🚀 How to Run

### 1. Clone the repository

```bash
git clone <https://github.com/lisandihimara/Olist-E-Commerce-Stockout-Stress-Test>
cd Olist-E-Commerce-Stockout-Stress-Test
```

### 2. Install the required libraries

```bash
python -m pip install -r requirements.txt
```

The `requirements.txt` file should contain:

```text
pandas
numpy
matplotlib
jupyter
```

### 3. Download the dataset

Dataset source:

[Olist Brazilian E-Commerce Public Dataset — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Place the required files in:

```text
Data/Raw/
```

Required files:

- `olist_orders_dataset.csv`
- `olist_order_items_dataset.csv`

### 4. Open the notebook

Launch Jupyter from the repository root:

```bash
jupyter notebook
```

Open:

```text
Notebooks/Week3_Olist_Stockout_Stress_Test.ipynb
```

Run the notebook cells sequentially from top to bottom.

---

# 📊 Outputs

The notebook creates the following outputs.

### Historical Activity Plot

`outputs/figures/historical_observed_sales.png`

Shows daily recorded item activity for the selected product before applying the stress scenario.

### Simulated Activity Plot

`outputs/figures/simulated_stockout_observed_sales.png`

Compares original activity with the artificial zero-masked series around the simulated window.

### Forecast Comparison Plot

`outputs/figures/forecast_effect_of_stockout.png`

Compares the baseline and stress-scenario forecast paths over the test period.

### Stress Summary

`outputs/results/stockout_stress_summary.csv`

Contains the main forecast metrics and forecast differences.

### Daily Forecast Comparison

`outputs/results/daily_forecast_comparison.csv`

Contains daily original activity, stress-scenario activity, both forecast paths and their difference.

### Product Sensitivity Check

`outputs/results/product_sensitivity_check.csv`

Contains comparable stress-test results for the five most frequently recorded products.

---

# ❓ Questions Behind the Experiment

The analysis is designed around several questions:

1. How does a 7-day rolling-average forecast respond to a 14-day sequence of zero-masked observations?
2. How large is the difference between the baseline and stress-scenario forecasts?
3. Does the forecast difference persist after the simulated window?
4. How do forecast errors compare when both forecasts are evaluated against the original recorded activity?
5. Does the observed response vary across the five selected products?
6. What assumptions are introduced by replacing observations with zero?
7. What additional data would be needed to identify actual stockouts and lost demand?

---

# ⚠️ Limitations

This experiment has several important limitations:

- The zero period is simulated rather than observed as a confirmed stockout.
- The selected daily measure counts delivered-order item records and is only a proxy for sales activity.
- The dataset does not directly establish inventory availability or lost customer demand.
- The original observed series is a reference for comparison, not a verified counterfactual outcome.
- The 7-day rolling average is a simple baseline rather than a comprehensive forecasting system.
- The main experiment focuses on a product selected by frequency of delivered-order item records.
- The sensitivity check uses only five frequently recorded products.
- Results may differ with another product, time window, forecast method or stress duration.
- This analysis does not estimate the causal effect of stockouts on sales or revenue.

---

# 🧠 The Bigger Lesson

A forecast is shaped by the data it receives. When some of its recent observations change, its future predictions can change too.

Therefore, an important question is not only:

> **“How close is the forecast to the recorded observations?”**

but also:

> **“How does the forecast behave when the conditions or observations around it change?”**

This experiment is a small, reproducible example of that broader idea.

---

# 👤 My Perspective

This is intentionally a **focused analytical experiment**, not simply another dashboard or model-building project.

The emphasis is on:

- Asking a specific business question
- Designing a controlled stress scenario
- Building a transparent forecasting baseline
- Comparing outcomes with reproducible metrics
- Testing sensitivity across multiple products
- Making assumptions explicit
- Separating observed data from simulated conditions
- Communicating what the results can and cannot establish

> **The code supports the reasoning. The reasoning is the experiment.**

---

# 🛠️ Technologies

- Python
- Pandas
- NumPy
- Matplotlib
- Jupyter Notebook

---

# 👩‍💻 Author

**Lisandi Himara**

BSc (Hons) Data Science & Business Analytics

Areas of interest:

- Data Science
- Business Analytics
- Time-Series Forecasting
- Model and Forecast Robustness
- Decision Support
- Responsible Data Interpretation

---

## 📌 Final Takeaway

> **A forecast should not only be evaluated on historical observations. It should also be examined for how its predictions respond when the data it depends on changes.**
