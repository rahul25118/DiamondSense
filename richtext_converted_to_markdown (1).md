1\. Problem overview
--------------------

The goal of **DiamondSense** is to build a clear, end‑to‑end machine learning workflow that predicts a diamond’s **price** from its physical characteristics and quality grades (carat, cut, color, clarity, depth, table, x, y, z). This is framed as a **supervised regression** problem where price is a continuous target.​

2\. Data and cleaning
---------------------

**Dataset**

*   53,940 raw rows, 10 columns:carat, cut, color, clarity, depth, table, x, y, z, price.​
    
*   cut, color, clarity are categorical; the others are numeric.
    

**Quality checks**

*   No missing values in any column.
    
*   Found 20 rows where at least one of x, y, z was 0 (physically impossible for a diamond dimension).
    
*   Found 146 exact duplicate rows (145 after removing the bad‑dimension rows).
    

**Cleaning decisions**

1.  **Remove invalid geometry**
    
    *   Dropped all rows with x == 0 or y == 0 or z == 0.
        
    *   After removal, minimums became:
        
        *   x\_min = 3.73, y\_min = 3.68, z\_min = 1.07, all realistic positive values.​
            
2.  **Drop exact duplicates**
    
    *   Removed 145 duplicated rows, then reset the index.​
        

**Final dataset**

*   Shape after cleaning: **53,775 rows × 10 columns**, no missing values, no impossible dimensions, no duplicates. This matches how the diamonds dataset is commonly prepared for analysis and modeling.​
    

3\. Exploratory data analysis (EDA)
-----------------------------------

3.1 Numeric summary
-------------------

*   **Carat**: 0.20–5.01, median 0.70 — most diamonds are small–medium, with a tail of large stones.
    
*   **Price**: 326–18,823, mean ≈ 3,931, median ≈ 2,401 — strong right skew with a long expensive tail.
    
*   **Depth & table**: centered around depth 61–62 and table 56–59 with small spread, indicating standard cutting proportions.​
    

3.2 Price distribution
----------------------

*   Histogram of price shows many low‑to‑mid priced diamonds and fewer high‑priced ones, confirming the skew seen in the summary statistics.
    
*   This skew suggests that log‑price or robust models could sometimes help, but tree‑based models handle it reasonably well.​
    

3.3 Relationships between features
----------------------------------

*   **Carat vs price** scatterplot: strong upward, non‑linear trend; price grows faster than linearly with carat, with visible vertical bands due to popular weight categories (e.g., ~1.0, ~1.5, ~2.0 carats).​
    
*   **Correlation heatmap (numeric features)**:
    
    *   carat correlates very strongly with x, y, z (~0.95–0.98) and with price (~0.92).
        
    *   x, y, z also show strong positive correlation with price (~0.87–0.89).
        
    *   depth and table have weak or near‑zero correlation with price and mild correlation with other features.​
        

3.4 Price vs categorical features
---------------------------------

*   **Cut**: boxplots of price by cut (Fair, Good, Very Good, Premium, Ideal) show different medians but with heavy overlap; better cuts tend to be pricier, but effect size is moderate compared with carat.​
    
*   **Color**: price distributions by color (D best to J worst) show that worse color can still be very expensive at high carat, meaning size often dominates color.​
    
*   **Clarity**: clearer stones (e.g., IF, VVS1) tend to have higher typical prices than included stones (I1, SI2), again with overlap and interaction with carat.​
    

4\. Modeling
------------

4.1 Feature setup
-----------------

*   **Target**: price.
    
*   **Predictors**:
    
    *   Numeric: carat, depth, table, x, y, z.
        
    *   Categorical: cut, color, clarity.
        

**Preprocessing**

*   Numeric features passed through without scaling (tree models are scale‑invariant; linear model still works decently).​
    
*   Categorical features encoded using **one‑hot encoding** with handle\_unknown="ignore" to safely handle all levels.​
    

**Train/test split**

*   80% train, 20% test with a fixed random seed for reproducibility.
    

4.2 Baseline: Linear Regression
-------------------------------

*   Model: standard multiple Linear Regression using the encoded features.
    
*   Test performance:
    
    *   **RMSE** ≈ **1128.91**
        
    *   **R²** ≈ **0.9194**
        

**Interpretation**

*   Explains about **92%** of variance in prices — a strong linear fit.​
    
*   Typical error (~1,129) is around 28–30% of the mean price (~3,931), so the model can be off by about $1,000 on single diamonds, especially at the extremes.​
    

4.3 Main model: Random Forest Regressor
---------------------------------------

*   Model: Random Forest Regressor with 300 trees, default depth, and all cores used.​
    
*   Same preprocessing and train/test split as Linear Regression.
    

**Test performance**

*   **RMSE** ≈ **527.63**
    
*   **R²** ≈ **0.9824**
    

**Interpretation**

*   Explains about **98%** of price variance, in line with high‑performing models on this dataset.​
    
*   Typical error (~528) is less than half of the linear model’s error, giving much more accurate price predictions for individual diamonds.​
    

**Comparison**

Model RMSER² Linear Regression1128.910.9194Random Forest527.630.9824

Random Forest clearly outperforms Linear Regression because it captures non‑linear relationships and interactions between carat, cut, color, and clarity.​

5\. Feature importance and key insights
---------------------------------------

5.1 Random Forest feature importance (top 15)
---------------------------------------------

RankFeatureImportance1carat0.57482y0.30863clarity\_SI20.01964clarity\_I10.01525clarity\_SI10.01426color\_J0.01037x0.00818color\_I0.00779clarity\_VS20.007010color\_H0.005511clarity\_VS10.005412z0.005113depth0.003014color\_D0.002715color\_G0.0024

​

5.2 Interpretation
------------------

*   **Carat dominates**: with importance ≈ 0.57, carat alone provides more than half of the model’s predictive power, confirming that weight is the primary driver of diamond price.​
    
*   **Physical size (y, x, z)**: y (length) is the second most important feature (≈ 0.31), followed by x and z with smaller but real contributions; overall face‑up size and volume matter a lot in pricing.​
    
*   **Clarity**: high importance for SI2, I1, SI1, VS2, VS1 shows that internal imperfections significantly adjust price once size is known; worse clarity (I1, SI2) has strong negative impact.​
    
*   **Color**: J, I, H, G, D all appear in the top 15, but with lower importance than carat and main clarity levels, so color fine‑tunes price within size/clarity groups rather than dominating.​
    
*   **Depth & table**: low, non‑zero importance; extreme proportions affect value but are a secondary signal after size and quality grades.​