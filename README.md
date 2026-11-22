# AI-Driven Supply Chain Demand Forecasting & Inventory Replenishment Planning

An end-to-end supply chain analytics project that transforms historical order data into SKU- and market-level demand forecasts, evaluates multiple machine-learning models, and derives inventory replenishment parameters such as Safety Stock and Reorder Point (ROP).

The project combines time-series feature engineering, statistical validation, machine-learning-based forecasting, and operational inventory calculations to support more informed demand and replenishment planning.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Project Scope](#project-scope)
- [Dataset Summary](#dataset-summary)
- [Project Architecture](#project-architecture)
- [Data Preparation and Exploratory Analysis](#data-preparation-and-exploratory-analysis)
- [Feature Engineering](#feature-engineering)
- [Feature Selection and Statistical Validation](#feature-selection-and-statistical-validation)
- [Demand Forecasting](#demand-forecasting)
- [Model Evaluation](#model-evaluation)
- [Inventory Replenishment Calculations](#inventory-replenishment-calculations)
- [Key Findings](#key-findings)
- [Analytical Outputs](#analytical-outputs)
- [Repository Structure](#repository-structure)
- [Technologies Used](#technologies-used)
- [How to Run](#how-to-run)
- [Assumptions and Interpretation Notes](#assumptions-and-interpretation-notes)
- [Limitations and Future Improvements](#limitations-and-future-improvements)
- [Project Outcome](#project-outcome)
- [Author](#author)

---

## Project Overview

Demand uncertainty is a major challenge in supply chain planning. Changes in customer purchasing patterns, product-level sales velocity, pricing, discounts, and seasonality make it difficult to estimate future demand accurately.

Inaccurate demand estimates can result in excess inventory, higher holding costs, stockouts, and delayed replenishment decisions. These issues affect working capital as well as the ability to maintain product availability.

This project develops a data-driven demand forecasting workflow using historical supply chain order data. It evaluates four regression models, selects the best-performing model using a chronological test period, generates demand forecasts, and calculates basic inventory replenishment parameters at SKU-market level.

### Project Objectives

- Convert transaction-level order data into a daily SKU-market demand dataset.
- Identify historical, calendar, product, and market-level demand patterns.
- Engineer lag-based and rolling statistical features for time-series forecasting.
- Compare multiple machine-learning and statistical regression models.
- Generate out-of-sample demand forecasts in original demand units.
- Calculate Safety Stock and Reorder Point to demonstrate the connection between forecasting and inventory planning.

## Business Problem

Supply chain planners need reliable estimates of future product demand to make better replenishment decisions. Four practical questions guide this project:

### Demand Planning

- How does demand vary across products, markets, and time periods?
- How much demand can be expected based on recent sales history?
- Which historical and calendar variables are most useful for predicting demand?

### Forecast Accuracy

- Which forecasting model performs best on unseen future observations?
- How can model performance be compared using consistent evaluation metrics?
- How can future information leakage be reduced during time-series validation?

### Inventory Replenishment

- What buffer stock is required to account for demand variability?
- At what inventory level should replenishment be considered?
- How can forecast outputs be connected to operational inventory parameters?

The project provides an analytical workflow to address these questions using historical demand data and clearly stated inventory assumptions.

## Project Scope

| Workstream | Analysis / Deliverable |
|---|---|
| Data preparation | Transaction-level data aggregation, quality checks, and demand transformation |
| Exploratory analysis | Demand distribution, skewness, and product/market demand patterns |
| Feature engineering | Historical demand baselines, calendar variables, lag features, and rolling statistics |
| Statistical validation | Correlation analysis, One-Way ANOVA, Two-Sample T-Test, and Chi-Square test |
| Forecasting | Linear Regression, Ridge Regression, Random Forest, and tuned XGBoost |
| Model evaluation | Chronological train-test split and comparison using R², RMSE, MAE, and log-space WAPE |
| Forecast generation | Out-of-sample predictions converted back to demand units |
| Inventory planning | SKU-market average daily demand, demand variability, Safety Stock, and ROP |

## Dataset Summary

The forecasting workflow is built using the **DataCo Supply Chain Dataset**. The raw transaction records are aggregated into a daily demand dataset at the product and market level.

### Dataset Profile

| Attribute | Description |
|---|---|
| Source dataset | DataCo Supply Chain Dataset |
| Aggregated observations | 25,513 daily SKU-market records |
| Unique products | 118 SKUs |
| Markets | 5 — LATAM, Europe, Pacific Asia, USCA, and Africa |
| Date range | 01 January 2015 – 31 January 2018 |
| Demand variable | `Demand_Units` |
| Product identifiers | SKU, Product Name, and Category |
| Additional variables | Product price, discount rate, sales, and order count |

### Daily Demand Aggregation

The raw transaction data is grouped using:

- `Date`
- `Product Card Id` (renamed as `SKU`)
- `Market`

The aggregation creates the following measures:

| Measure | Aggregation |
|---|---|
| `Demand_Units` | Sum of order item quantity |
| `Avg_Product_Price` | Mean product price |
| `Avg_Discount_Rate` | Mean discount rate |
| `Total_Sales` | Sum of sales |
| `Order_Count` | Number of unique orders |

This converts individual order transactions into a structured daily demand dataset suitable for SKU-market-level time-series analysis.

> **Dataset availability:** The main notebook currently reads `DataCoSupplyChainDataset.csv` from the repository root. The repository also contains `Dataset Sample.xlsx`, but the notebook does not load that Excel file. To reproduce the complete workflow, place the required CSV in the root directory or update the input path in the notebook. The raw CSV is not currently listed among the repository files.

## Project Architecture

```text
          DataCo Supply Chain Dataset
                      |
                      v
          Transaction-Level Data
                      |
                      v
       Daily SKU-Market Aggregation
                      |
                      v
          Data Quality Validation
                      |
                      v
       Exploratory Data Analysis
                      |
                      v
          Feature Engineering
      (Historical, Calendar, Lag,
          Rolling Statistics)
                      |
                      v
       Feature Selection & Statistical
                 Validation
                      |
                      v
        Chronological Train-Test Split
                  (80:20)
                      |
                      v
              Model Training
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
     XGBoost      Random Forest   Linear / Ridge
        |             |             |
        +-------------+-------------+
                      |
                      v
            Model Evaluation
                      |
                      v
             Tuned XGBoost
             (Best Model)
                      |
                      v
          Inverse Log Transformation
                      |
                      v
          Final Demand Forecasts
                      |
                      v
        Safety Stock and Reorder Point
                      |
                      v
          Final Forecast CSV Output
```

## Data Preparation and Exploratory Analysis

The workflow begins by converting transaction timestamps into daily dates and aggregating order quantities by SKU and market. Product name, category, average price, average discount, total sales, and unique order count are retained to preserve useful demand-related information.

### Data Quality Checks

The notebook performs basic data integrity checks before feature creation:

- Negative demand records are checked.
- Missing or null values are audited.
- Assertions are used to verify the cleaned demand data.
- Time-series-generated missing values are reviewed and handled during feature preparation.

### Demand Distribution and Transformation

Daily demand is positively skewed, with a small number of high-demand observations. To reduce the effect of this skewness during model training, the target variable is transformed using:

\[
\text{Log\_Demand\_Units} = \log(1 + \text{Demand\_Units})
\]

The notebook reports a reduction in target skewness from **2.0262** to **0.5298** after the Log1p transformation. Predictions are later converted back to original demand units using the inverse transformation.

## Feature Engineering

Feature engineering is performed to represent product-level demand behaviour, regional market patterns, calendar seasonality, and recent sales momentum.

### 1. Historical Demand Baselines

The following baseline variables are created:

- `SKU_Mean_Demand` — historical average demand at product level.
- `Category_Mean_Demand` — average demand at product-category level.
- `Market_Mean_Demand` — average demand at market level.

These variables provide a reference for distinguishing high-velocity and low-velocity products and for representing broader category and market demand patterns.

### 2. Calendar and Seasonality Features

Date-based features are extracted to capture recurring calendar patterns:

- Month and Day
- Day of Week
- Weekend indicator (`Is_Weekend`)
- Quarter
- Day of Year

### 3. Time-Series Lag Features

Lag variables represent historical demand observations for each SKU-market series:

| Feature | Description |
|---|---|
| `Lag_1` | Previous observation's demand |
| `Lag_7` | Demand 7 observations earlier |
| `Lag_14` | Demand 14 observations earlier |
| `Lag_30` | Demand 30 observations earlier |

### 4. Rolling Statistical Features

Rolling statistics capture recent demand trends and variability:

| Feature | Description |
|---|---|
| `RollingMean7` | 7-observation rolling mean |
| `RollingMean14` | 14-observation rolling mean |
| `RollingMean30` | 30-observation rolling mean |
| `RollingStd7` | 7-observation rolling standard deviation |
| `RollingStd30` | 30-observation rolling standard deviation |

The rolling calculations are shifted so that the current observation's demand is not included in its own historical rolling features. Lag and rolling feature missing values are handled using the SKU-level historical mean, while missing rolling standard deviations are filled with zero.

### 5. Pricing and Market Features

The model also uses average product price, average discount rate, market indicators, and one-hot-encoded product category variables. These features allow the models to consider differences in product, market, and commercial conditions.

## Feature Selection and Statistical Validation

Feature selection is used to examine the relationship between candidate predictors and the demand target before model training.

### Correlation Analysis

Pearson correlation is calculated between the log-transformed demand target and candidate predictors. A focused correlation heatmap is generated to inspect the relationships between demand and selected time-series features.

### Statistical Hypothesis Testing

Three statistical tests are conducted at a significance level of **α = 0.05**:

| Test | Objective |
|---|---|
| One-Way ANOVA | Examine whether mean daily demand differs across markets |
| Two-Sample T-Test | Examine whether weekday and weekend demand differ |
| Chi-Square Test of Independence | Examine the association between product-category groups and demand-volume groups |

The selected numerical and categorical variables are consolidated into a model-ready feature matrix. Market and category variables are one-hot encoded, producing **73 predictor features** for model training.

## Demand Forecasting

Four regression models are implemented to compare linear baselines with tree-based ensemble methods. The target used for model training is `Log_Demand_Units`.

### Models Evaluated

| Model | Purpose |
|---|---|
| Linear Regression | Basic linear regression baseline |
| Ridge Regression | L2-regularized linear model to help manage correlated predictors |
| Random Forest Regressor | Tree-based ensemble model capable of capturing non-linear feature interactions |
| Tuned XGBoost | Gradient-boosted tree model optimized using chronological cross-validation |

### Chronological Train-Test Split

The observations are sorted by date and split chronologically, rather than randomly, so the test set represents a later time period than the training set.

| Dataset | Records | Share |
|---|---:|---:|
| Training set | 20,396 | 79.94% |
| Testing set | 5,117 | 20.06% |
| Total | 25,513 | 100% |

- **Training period:** 01 January 2015 – 10 February 2017
- **Testing period:** 11 February 2017 – 31 January 2018

### XGBoost Hyperparameter Tuning

XGBoost hyperparameters are tuned using `TimeSeriesSplit(n_splits=3)`. This expanding-window validation approach preserves chronological order during cross-validation.

The notebook evaluates **16 parameter combinations**, resulting in **48 model fits** across the three folds.

| Hyperparameter | Selected Value |
|---|---:|
| `learning_rate` | 0.05 |
| `max_depth` | 6 |
| `n_estimators` | 100 |
| `subsample` | 0.8 |

**Best cross-validation RMSE:** 0.5334 (log-transformed target scale).

## Model Evaluation

All four trained models are evaluated on the same out-of-sample chronological test set. The comparison uses R², Root Mean Squared Error (RMSE), Mean Absolute Error (MAE), and Weighted Absolute Percentage Error (WAPE).

### Model Performance Comparison

| Model | R² (Log) | RMSE (Log) | MAE (Log) | WAPE (Log %) |
|---|---:|---:|---:|---:|
| **Tuned XGBoost** | **0.8932** | **0.4118** | **0.3230** | **14.78%** |
| Random Forest | 0.8809 | 0.4349 | 0.3382 | 15.48% |
| Linear Regression | 0.8485 | 0.4905 | 0.3931 | 17.99% |
| Ridge Regression | 0.8418 | 0.5012 | 0.4043 | 18.50% |

### Model Selection

Tuned XGBoost achieved the highest R² score and the lowest RMSE, MAE, and reported WAPE among the four evaluated models. It was therefore selected as the champion model for generating final demand predictions.

> **Metric interpretation:** The notebook calculates WAPE using the log-transformed target and log-scale predictions. Therefore, the reported WAPE values are **log-space WAPE**, not ordinary WAPE calculated on demand units. They should not be interpreted directly as unit-level business forecast error.

## Inventory Replenishment Calculations

After generating demand predictions, the workflow adds basic SKU-market inventory parameters to demonstrate how demand variability can inform replenishment planning.

### 1. Safety Stock

Safety Stock acts as a buffer against demand variability during supplier replenishment lead time.

\[
SS = Z \times \sigma_d \times \sqrt{L}
\]

Where:

- `Z` = service-level factor
- `σd` = standard deviation of daily demand
- `L` = replenishment lead time in days

The notebook uses a **95% service-level factor (Z = 1.65)** and an assumed **3-day lead time**.

### 2. Reorder Point (ROP)

The Reorder Point indicates the inventory level at which replenishment should be considered.

\[
ROP = (\bar{d} \times L) + SS
\]

Where:

- `d̄` = average daily demand
- `L` = replenishment lead time in days
- `SS` = Safety Stock

The resulting `Recommended_Safety_Stock` and `Reorder_Point_ROP` values are calculated for SKU-market combinations and included in the exported forecast dataset.

## Key Findings

1. **Tuned XGBoost performed best:** It achieved an R² score of 0.8932 on the log-transformed test target and outperformed the other evaluated models across the reported comparison metrics.
2. **Recent demand momentum was an important predictor:** In the Random Forest feature-importance analysis, `RollingMean7` ranked first, with an importance contribution of 89.63% in the reported ranking.
3. **Product-level demand baselines add context:** SKU-level, category-level, and market-level average demand features provide additional information about differences in demand velocity.
4. **Forecasts can be connected to replenishment parameters:** The final output combines actual demand, forecast demand, forecast error, Safety Stock, and ROP for further analysis.

Feature importance describes the model's predictive usage of variables; it does not establish that a feature causes demand to change.

## Analytical Outputs

The main notebook generates the following analytical outputs:

- Daily SKU-market demand dataset.
- Demand distribution and transformation analysis.
- Correlation analysis and statistical test results.
- Feature-engineered model input dataset.
- Chronological train-test split summary.
- XGBoost hyperparameter tuning results.
- Model performance comparison table.
- Actual versus forecasted demand comparison.
- Random Forest feature-importance ranking and chart.
- Actual-versus-forecast diagnostic visualizations.
- Final forecast dataset with inventory parameters.

### Final Export

The notebook exports `final_demand_forecast.csv` with **5,117 records and 85 columns**.

Important output fields include:

| Column | Description |
|---|---|
| `Date` | Forecast observation date |
| `SKU` | Product identifier |
| `Market` | Market/region identifier |
| `Product_Name` | Product description |
| `Actual_Demand_Units` | Actual demand in units |
| `Forecasted_Demand_Units` | Predicted demand in units |
| `Forecast_Error_Units` | Forecast minus actual demand |
| `Avg_Daily_Demand` | Average daily demand used in the inventory calculation |
| `Demand_Std_Dev` | Demand standard deviation used in the inventory calculation |
| `Recommended_Safety_Stock` | Calculated Safety Stock |
| `Reorder_Point_ROP` | Calculated Reorder Point |

The exported file is available here: [`final_demand_forecast.csv`](final_demand_forecast.csv).

## Repository Structure

```text
AI-Driven-Supply-Chain-Demand-Forecasting-/
│
├── Demand_Forecasting.ipynb       # Main data preparation, feature engineering,
│                                  # model training, evaluation, and inventory calculations
│
├── final_demand_forecast.csv      # Exported test-period forecasts and inventory parameters
│
├── Dataset Sample.xlsx            # Dataset sample available in the repository
│
└── Project.ipynb                  # Currently an empty notebook placeholder
```

**Input-file note:** The main notebook loads `DataCoSupplyChainDataset.csv`. That CSV is not currently listed in the repository. The `Dataset Sample.xlsx` file is not used by the current notebook code.

## Technologies Used

| Technology | Application |
|---|---|
| Python | End-to-end analysis and forecasting workflow |
| Pandas | Data manipulation, aggregation, and feature preparation |
| NumPy | Numerical operations and transformations |
| Matplotlib | Data and model-result visualizations |
| Seaborn | Exploratory analysis, correlation heatmaps, and feature-importance plots |
| SciPy | Statistical hypothesis testing |
| Scikit-learn | Regression models, evaluation metrics, and time-series cross-validation |
| XGBoost | Gradient-boosted demand forecasting model |
| Jupyter Notebook | Interactive development and documentation |

## How to Run

### 1. Clone the Repository

```bash
git clone https://github.com/tanish-6126/AI-Driven-Supply-Chain-Demand-Forecasting-.git
cd AI-Driven-Supply-Chain-Demand-Forecasting-
```

### 2. Install Dependencies

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn xgboost jupyter openpyxl
```

### 3. Add the Required Input Dataset

Place `DataCoSupplyChainDataset.csv` in the repository root, at the same level as `Demand_Forecasting.ipynb`. The notebook currently reads this exact filename using Pandas with `encoding="latin-1"`.

### 4. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `Demand_Forecasting.ipynb` and run the cells in sequential order. The notebook trains and evaluates the models, generates forecasts, performs the inventory calculations, and exports `final_demand_forecast.csv`.

## Assumptions and Interpretation Notes

- The forecasting analysis uses historical DataCo order data covering January 2015 to January 2018. It is a historical analytical exercise, not a live demand feed.
- The model target is log-transformed demand. Forecast predictions are converted back to demand units using the inverse `expm1` transformation, and negative predicted demand is clipped to zero.
- The inventory calculation assumes a fixed **3-day replenishment lead time** and a **95% service level (Z = 1.65)**. These are model assumptions, not supplier-specific observed values.
- In the current notebook, `Avg_Daily_Demand` and `Demand_Std_Dev` for Safety Stock and ROP are calculated from actual demand observations in the test-period data. They are not calculated from the forecast series alone.
- Model-comparison WAPE is calculated in log space and must not be presented as ordinary demand-unit WAPE.
- The dataset contains market-level information; it does not provide the warehouse-level stock positions and warehouse master data needed for a multi-warehouse inventory control system.

## Limitations and Future Improvements

- Add the required raw CSV dataset or provide a reproducible data-download/preparation step.
- Recalculate `SKU_Mean_Demand`, `Category_Mean_Demand`, and `Market_Mean_Demand` using training-period data only, then apply the learned values to future periods. In the current notebook, these full-dataset averages are created before the chronological split and may therefore expose information from the test period to model training.
- Evaluate forecast errors on the original demand-unit scale using suitable business metrics and segment-level diagnostics.
- Use forecast-based demand estimates and validated lead-time assumptions for a more operational Safety Stock and ROP policy.
- Extend the workflow with warehouse-level stock, supplier lead-time, ordering-cost, and holding-cost data before implementing warehouse-specific inventory optimization.
- Add automated model pipelines, saved model artifacts, and monitoring for future-period forecasting.

## Project Outcome

The project demonstrates an end-to-end analytical workflow that connects historical supply chain demand data with machine-learning forecasting and basic replenishment calculations. It compares four forecasting approaches, selects Tuned XGBoost based on the reported chronological test-set evaluation, and exports SKU-market-level forecast and inventory-planning fields for further analysis.

The result is a practical foundation for exploring how demand forecasting can support inventory planning and replenishment decisions.

## Author

**Tanish**  
GitHub: [tanish-6126](https://github.com/tanish-6126)
