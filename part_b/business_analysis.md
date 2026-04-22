# **Part B**

## **B1. Problem Formulation**

### (a)  machine learning problem

The objective is to determine which promotion strategy maximizes the number of items sold at each store for a given month.

_Target Variable:_ - items_sold

_Input Features:_
* Store characteristics: store_id, store_size, location_type
* Promotion details: promotion_type
* Temporal features: month, year, is_weekend, is_festival, is_month_end
* Market factors: competition_density
* Customer behavior proxies: footfall, basket_size

_Type of ML problem_
This is a supervised learning regression problem, as the goal is to predict a continuous numerical outcome (item_sold) based on historical data. 

_Justification:_
Since past data contains both input (features) and output (items_sold), the model learned the relationship between them to predict future sales under different promotional strategies. 

### (b) Why Items Sold is Better than Revenue

Using item_sold (sales volume) is more reliable than total value for evaluating promotional effectiveness.
* Reason 1: removes prices bia
Revenue is influenced by product pricing and discount levels, which can distort the true effectiveness of a promotion.
For example, a heavy discount may increase revenue slightly but significantly increase the number of items sold. 

* Reason 2: Direct Measure of Demand
Item sold directly reflects customer demand and response to promotions, making it a more accurate indicator of promotion success.

* Reason #: compareable across promotions
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

* Justification:
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
* Since transactions are at a lower level(daily or per transaction) we aggregate
- items_sold- per store per month
- football- sum or average 
- basket_size- average 
- promotion_time- most frequent or assigned promotion for that month 
- is_weekend, is_festivaleducated- as counts or binary flags

Why this is important? Because aggregating to a store-month level ensures consistency between data structure and decision-making level, improving model relevance. 


### (b) EDA Strategy

Before modeling, the following EDA steps would be performed. 

1. Distribution Analysis (Histogram of items_sold)
* Purpose check: check skewness, outliners
* Impact: May apply log transformations if heavily skewed. 