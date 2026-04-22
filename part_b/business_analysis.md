# **Part B**

## **B1. Problem Formulation**

### (a)  machine learning problem

The objective is to determine which promotion strategy maximizes the number of items sold at each store for a given month.

_Target Variable:_ 
- items_sold

_Input Features:_
* Store characteristics: store_id, store_size, location_type
* Promotion details: promotion_type
* Temporal features: month, year, is_weekend, is_festival, is_month_end
* Market factors: competition_density
* Customer behavior proxies: footfall, basket_size

_Type of ML problem:_  
This is a supervised learning regression problem, as the goal is to predict a continuous numerical outcome (item_sold) based on historical data. 

_Justification:_  
Since past data contains both input (features) and output (items_sold), the model learned the relationship between them to predict future sales under different promotional strategies. 

### (b) Why Items Sold is Better than Revenue

Using item_sold (sales volume) is more reliable than total value for evaluating promotional effectiveness.
* Reason 1: removes prices bias  

  Revenue is influenced by product pricing and discount levels, which can distort the true effectiveness of a promotion.  
  For example, a heavy discount may increase revenue slightly but significantly increase the number of items sold. 

* Reason 2: Direct Measure of Demand  

  Item sold directly reflects customer demand and response to promotions, making it a more accurate indicator of promotion success.

* Reason 3: compareable across promotions  

  Direct promotions (e.g. BOGO vs Discount) affect priceing differently. using items sold to ensure fair  ensures fair comparison across strategies. 

* Broader Priciple:  
  This illustrates the importance of selecting a target variable that aligns closely with the business objective and avoids external distortion. In real-world ML, the target should measure the true outcome of interest, not a proxy affected by confounding factors. 

### (c) Alternative Modelling Strategy

Using a single global model across all stores may not capture the variability in customer behavior across different locations. 

_Proposed Strategy: Segmented or Hierarchical Modeling_

* Option 1: Segmented or Hierarchial Modeling
  * Trains separate models for R1-Semi, R1, and Ruler Restores 
  * Captures location-specific customer behavior and promotion response 

* Option 2: Include interaction effect
  * Use a single model but include an interaction term between promotion_type and location_type.
  * Allows the model to learn how different promotions perform in different regions. 

_Justification:_  
Customer preference, purchasing power, and compensation vary significantly across locations. A segmented or intersection-based approach ensures that the model captures these differences, leading to more accurate and actionable recommendations.   


## B2. Data and EDA Strategy

### (a) Data Joining, Grain, and Aggregation

The raw data is available in four tables:
     - transactions
     - store attributes
     - promotional details
     - calendar

1. Step 1: Joining Strategy  
    * Join transactions with store attributes using a store_id, promotion details using promotion_type, and the calendar table using transaction_date.

2. Step 2: Grain of Final Dataset
    * The final dataset should be at the store-month level, example, one row is= one store for a month. This aligns with the business goal of deciding which promotion to run per store per month. 

3. Step 3; Aggeegations
   Since transactions are at a lower level(daily or per transaction) we aggregate
    - items_sold- per store per month
    - football- sum or average 
    - basket_size- average 
    - promotion_time- most frequent or assigned promotion for that month 
    - is_weekend, is_festivaleducated- as counts or binary flags

Why this is important?  
Because aggregating to a store-month level ensures consistency between data structure and decision-making level, improving model relevance. 


### (b) EDA Strategy

Before modeling, the following EDA steps would be performed. 

1. Distribution Analysis (Histogram of items_sold)
     * Purpose check: check skewness, outliners
     * Impact: May apply log transformations if heavily skewed. 

2. Promotion effectiveness(Boxplot of item_sold vs promotion_type)
      * Purpose: compare performance of different promotions 
      * Impact: helps identify strong/ weak promotion - feature importance expectation.  

3. Time-based trend (line plot over months/years)
     * Purpose- identifies seasonality and trends 
      * Impacts- justifies using features like month, year, is_month_end. 

4. Location-based analysis (bar chart: item_sold vs location_type)
     * Purpose: compare urban, semi-urban, and rural performance
     * impact- supporters, segmentation, or interaction features. 

5. Correlation heatmap (numerical features)
      * Purpose- to identify relationships and multicollinearity.
      * Impact- It helps in feature selection and avoiding redundant variables. 

* Conclusion of EDA  
EDA helps uncover patterns in promotions, seasonality, and store behavior, guiding feature engineering and model selection.


### (c) Handling Promotion Imbalance

Since 80% of transactions occur without promotion, the data set is highly imbalanced. 

_Problem:_
  * Model may become biased toward no-promotion cases
  * Poor learning of promotion impact - weak recommendations. 

_Solutions:_
1. Re-sampling Techniques
  * Over-sample promotion cases or under-sample non-promotion cases.

2. Use Sample Vities
  * Assigning Higher Importance to Promotion Records 

3. Feature Engineering
  * Create a binary feature "has_promotion"
  * analyzes promotions or non-promotions separately. 

_Conclusion:_  
Addressing imbalance ensures the model properly learns the effect of promotions rather than being dominated by non-promotion data. 



## B3. Model Evaluation and Deployment

### (a) Train-Test Split & Evaluation Metrics

_Train-test split strategy:_  
  since the data is time series(monthly over 3 years), a temporal split should be used:
  * Train on the first 80% of months (early period) 
  * Test on the last 20% of months (most recent period).  
  _Example:_   
  * Train: year 1 + year2 + part of year 3
  * Test on the remaining months of year 3

_Why is a random split inappropriate?_  
A random split breaks the time order and causes data leakage, where future information is used to predict past outcomes.
  - Unrealistic evaluation
  - Over-optimistic performance
  - Not aligned with real-world deployment


_Evaluation Metrics_

1. RMSE (Root Mean Squared Error)
  - Penalize large error more heavily
  - Interpretations: large mistakes in predicting demand are costly (example: a stock out or overstocking)

2. MAE (Mean Absolute Error)
  - Measures average error in prediction
  - Interpretation: easy to understand - average deviation in items sold for a store

3. MAPE (Mean Absolute Percentage Error)
  - Measures error in percentage terms
  - Interpretation: useful for comparing performance across stores with different sales volume

_Conclusion:_  
RMSE captures the risk of large errors, while MAE reflects a risk prediction accuracy, making them suitable for business decision making. 

### (b) Explaining Different Recommendations

This model recommends different promotions for the same store because feature values change across months and influencing the prediction.

 _How to investigate using feature importance:_
1. Check key features for both months:
   - month (December vs. March seasonality)
   - is_festival (December likely festive, March may not be)
   - is_month_end
   - promotion_type_impact
  - Competition_density

2. Use feature importance/contribution:
   - Identify which features that have the highest influence (example: promotions,month, festival)
   - Compare how these features differ between December and March


_How to communicate to the marketing team?_  
Example:
In December, the model recommends a loyalty points bonus because historical data shows that during the festive period, customers respond better to reward-based promotions.
In March, the model recommends a flat discount because demand is lower and customers are more price-sensitive, making discounts more effective.

_conclusion:_  
The model adapts recommendations based on seasonality, customer behavior, and promotion effectiveness, not just store identity. 


### (c) Deployment Strategy

1. Model saving
    - save the train pipeline using  
     ```import joblib  
     joblib.dump(modelpipeline,"model.pkl")
     ```  

2. Monthly Prediction Workflow

   1 At the start of each month:
   - Collect new data.
   - Correct month info (month, festival, etc.).
   - Competition data.

   2 Apply same pre-processing:
   - Feature engineering.
   - Encoding and scaling via pipeline.

   3 Load model and predict  
     ```model = joblib.load("model.pkl")  
     predictions = model.predict(newdata)
     ```  

   4 Select promotion with highest predicted items_sold

3. Monitoring Strategy 
  A: Performance monitoring
     - Track RMSE/MAE over time
     - Compare predicted vs actual sales
  B: Data drift detection
     - Check if input features distribution changes (example: new customer behavior)
  C: Prediction Drift
     - Sudden change in predicted value


4. Retraining Trigger  
  Retrain the model when:
   - Performance drops significantly
   - Data distribution shift
   - New trend emerges (example., new promotion time)


_Conclusion:_  
A robust deployment pipeline ensures consistent prediction, while monitoring ensures the model remains accurate and relevant over time.