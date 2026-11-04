[README_GAHS.md](https://github.com/user-attachments/files/34252419/README_GAHS.md)
# Farm Profitability Analysis in R: From Messy Joins to Predictive Models

An end-to-end analysis of a farm-level and a market-level survey dataset (GAHS, 2,000 farms and 120 markets), written in R. It covers importing and joining, a missing-value audit, wrangling, visualisation (including a regional map), hypothesis testing, logistic regression, and decision tree and random forest classifiers.

**The question:** what farm, farmer, production and market characteristics are associated with whether a farm is profitable?

**Read this first:** almost every farm in the data is profitable (99.1% of the modelling sample), so the predictive results are fragile and I do not treat them as real findings. The most valuable parts of this project are the data-engineering and the statistical reasoning, and the [limitations section](#limitations-and-what-i-would-do-differently) explains the weaknesses openly.

## The data

| File | Rows | Columns | Contents |
|---|---|---|---|
| `gahs_farm.csv` | 2,000 | 43 | Farmer and household traits, farm characteristics, inputs, yield, revenue, cost, profit, credit, storage, market access |
| `gahs_market.csv` | 120 | 18 | Market type, road quality, infrastructure, traders, cold storage, commodity prices, efficiency score |

The two files share `Region` and `Survey_Year`. The data appears to be simulated (for example, some market names do not match the regions they are assigned to), so the results show a method, not conclusions about real Ghanaian agriculture.

## Highlights

### 1. Catching a silent join error

The task was a left join of markets onto farms. The naive version returned **4,889 rows instead of 2,000**, because `Region` and `Survey_Year` do not uniquely identify a market. Each farm was matched to several markets and duplicated, which would have quietly inflated every later statistic.

The fix: aggregate the market data to one row per region-year combination (46 of them, using group means), then join. The result is **2,000 rows and 51 columns**, checked against the original farm count.

### 2. A missing-value audit with reasons for each treatment

| Variable | Missing | Treatment | Why |
|---|---|---|---|
| `Credit_Amount_GHS` | 67.7% | Left as `NA` | Missing likely means no credit was taken, so imputing would invent values |
| `Tomato_Price_GHS`, `Cocoa_Price_GHS` | 8.2%, 4.3% | Group-level imputation | Prices differ by region and year |
| Yield, revenue, profit, input cost, farm size, fertiliser, post-harvest loss | 4% to 5% | Median imputation | Skewed continuous variables, so the median is robust |

### 3. Wrangling

- `pivot_longer()` on the four commodity-price columns to get tidy long-format price data
- `str_extract()` to pull a numeric farm number out of IDs like `GAHS-F0001`
- `str_detect()` to flag formal credit sources (bank, microfinance, SACCO)
- Filtering, selecting and mutating to build the final analysis dataset: **1,823 farms, 32 variables**, including profit per hectare, a farm-size category, and ordered factors for education and technology adoption

### 4. Five visualisations

1. Distribution of the outcome (profitable vs not)
2. Outcome by technology adoption level
3. Farm size against yield, with separate fitted lines for each profitability group
4. Heatmap of mean profit per hectare by crop type and farm-size category
5. Choropleth map of the share of profitable farms by region, with farm locations overlaid

### 5. Inference done properly

I tested whether farm size differs between profitable and unprofitable farms. Shapiro-Wilk tests and Q-Q plots showed non-normality, so I used a **Mann-Whitney U test** instead of a t-test:

- W = 10,468, p = 0.128: no significant difference
- Rank-biserial effect size r = 0.237 (small to moderate), flagged as unreliable because the groups are 15 unprofitable farms against 1,808 profitable ones

### 6. Logistic regression

Predictors: farm size, yield, input cost, distance to market and farmer sex, on 1,697 complete cases (1,682 profitable, 15 unprofitable).

- A formal check found **no complete or quasi-complete separation**, so the coefficient estimates are valid.
- Only **yield** was significant (p = 0.045), with a very wide confidence interval because there are only 15 events.
- On a stratified 70/30 split, the default 0.5 threshold gave 99.6% accuracy but **0% sensitivity**: the model labelled everything profitable. Lowering the threshold to 0.10 or 0.05 raised sensitivity to 100%, but the test set contains only **one** unprofitable farm.

### 7. Decision tree and random forest

- **Imbalance:** SMOTE applied inside the training resamples only, so the test set stays untouched
- **Validation:** 10-fold stratified cross-validation
- **Tuning:** a grid search over complexity, depth and minimum node size for the tree (27 configurations), and over predictors per split, number of trees and node size for the forest, with the best model chosen by ROC-AUC
- **Result:** both models scored 1.0 on accuracy, AUC, sensitivity and specificity on the held-out set. Given a single unprofitable farm in that set, this is not evidence of a good model.
- **Variable importance (forest):** yield by a large margin (1027), then input cost (70), farm size (44), distance to market (28) and farmer sex (3.5)

## Limitations and what I would do differently

- **The outcome is almost constant.** With 0.9% of farms unprofitable, the test set has one positive case. Perfect scores on it show the evaluation cannot discriminate between models.
- **Probable circularity.** Profit equals revenue minus input cost, and revenue depends on yield. Yield and input cost are therefore close to defining profitability, which explains why they dominate. A better design would drop these direct components, or predict continuous profit (`Farm_Profit_GHS`) from agronomic, access and market variables.
- **Imputation affects the profit variables.** Imputing yield, revenue, profit and cost separately by median can break the accounting relationship between them. A cleaner approach would impute yield and cost and recompute revenue and profit, or drop those rows.
- **Aggregating markets loses detail.** Averaging market data by region and year solved the duplication problem but removes variation between markets in the same region.
- **Simulated data.** Even a perfect pipeline cannot give real-world conclusions from simulated data.

## Tools

R, with the tidyverse (readr, dplyr, tidyr, stringr, ggplot2), tidymodels (resampling, tuning, workflows, yardstick), broom, gtsummary, flextable, detectseparation and vip.


## Author

**Yaw Asante** - Data Scientist, Statistician and Financial Economist

Email: vkpoasante@gmail.com

## License
**MIT**



