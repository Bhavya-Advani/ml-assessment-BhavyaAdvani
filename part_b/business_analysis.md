
B1. Problem Formulation
(a) Machine Learning Formulation

Target Variable: items_sold (a continuous numerical value).

Candidate Input Features:

Store-related: store_size, location_type, competition_density.

Promotion-related: promotion_type.

Temporal/Calendar: is_weekend, is_festival, month, year.

Problem Type: This is a Regression problem.

Justification: The goal is to predict a specific quantity (number of items) rather than a category or class. By predicting a continuous value, the business can rank different promotion types based on the predicted volume and choose the one with the highest estimated output.

(b) Target Variable Selection: Revenue vs. Volume
Using items_sold (volume) is more reliable than revenue because revenue is heavily influenced by the unit price of items and the depth of the discount. For example, a 50% flat discount might generate high revenue but actually represent a lower number of individual customer "conversions" or inventory clearance compared to a BOGO offer.

Broader Principle: This illustrates the principle of isolating the behavior you want to influence. If the business goal is "Promotion Effectiveness," volume is a cleaner signal of customer response to the offer itself, whereas revenue is a "noisy" signal conflated with pricing strategy and profit margins.

(c) Alternative Modeling Strategy
Instead of a single global model, I propose a Clustered Modeling or Grouped Modeling strategy.

Justification: Stores in "Rural" locations likely have different shopping patterns and price sensitivities than those in "Urban" centers. By training separate models (or using a Hierarchical/Mixed-Effects model), we can capture the interaction between location_type and promotion_type more effectively. This prevents the high-volume urban stores from "washing out" the patterns specific to rural stores.

B2. Data and EDA Strategy

(a) Data Joining and Final Dataset DesignTo build the modeling dataset, I would follow a star-schema join approach using the Calendar table as the primary reference to ensure no time periods are missed.Join Strategy:Transactions + Store Attributes: Join via store_id to attach location and size context to every sale.Transactions + Promotion Details: Join via promotion_id (or date/store if promotions are pre-scheduled) to identify which campaign was active.Transactions + Calendar: Join via transaction_date to pull in holiday and weekend flags.Final Grain: The grain of the dataset will be one row per store, per month.Aggregations: Since the raw data is at the transaction level, I will aggregate it to the monthly grain by calculating:Sum of items_sold (Target).Mean of competition_density (to account for any mid-month changes).Sum of is_weekend and is_festival (to capture the "opportunity" for sales in that month).Mode (most frequent) or Max of promotion_type to identify the primary marketing driver for that store-month.

(b) EDA StrategyBefore modeling, I will perform the following four analyses to guide feature engineering:Boxplot of Items Sold by Promotion Type: I will look for the median and spread of sales for each of the five promotions. If the "whiskers" of the boxplots don't overlap, it confirms that the promotion type is a high-impact feature.Location-Promotion Interaction Heatmap: By plotting location_type vs promotion_type with items_sold as the intensity, I can see if "BOGO" works in Urban stores but fails in Rural ones. This would influence a decision to include interaction terms in the regression.Monthly Sales Line Chart (3-Year View): I will look for recurring peaks (seasonality). If sales spike every December regardless of promotion, I will ensure Month is treated as a categorical feature to prevent the model from misattributing seasonal organic growth to a specific promotion.Correlation Matrix: I will check for multicollinearity between features like store_size and footfall. If they are highly correlated ($r > 0.8$), I may drop one to improve the stability of the Linear Regression model.

(c) Handling Promotion ImbalanceWith 80% of the data representing "No Promotion" periods, the model risks becoming a "baseline predictor" that ignores the nuances of different marketing campaigns.Model Bias: The model might achieve high overall accuracy by simply predicting the average baseline sales, effectively "ignoring" the 20% of cases where a promotion actually caused an uplift.Addressing the Imbalance:Stratified Sampling: Ensure that the 20% test set contains a proportional representation of all five promotion types, not just the majority "No Promotion" class.Weighted Loss Functions: Assign higher "importance" weights to rows where a promotion was active during the training process.Uplift Modeling: Instead of predicting total sales, I would calculate a "Baseline Sales" feature (average sales for that store when no promotion is active) and use the model to predict the delta (uplift) caused specifically by the promotion.


B3. Model Evaluation and Deployment
(a) Train-Test Split Strategy
Recommended:
Time-based split
Example:

Train: Years 1–2
Validation: First half of Year 3
Test: Final months of Year 3
Why Random Split is Inappropriate:
Causes data leakage
Future data may influence past predictions
Ignores seasonality and temporal structure
Evaluation Metrics:
1. RMSE
Measures prediction error magnitude
2. MAE
Average forecast deviation
3. R²
Variance explained by model
4. Promotion Recommendation Accuracy
Whether model selects highest-performing promotion
Business Interpretation:
Metrics should focus on both prediction precision and strategic recommendation quality.


(b) Explaining Different Monthly Recommendations
Feature Importance Analysis:
Methods:

SHAP values
Feature importance plots
Partial dependence plots
Investigation:
December:

Festivals
Holiday shopping
Loyalty retention
March:

Off-season purchasing
Price sensitivity
Inventory clearance
Communication:
Explain to marketing team that:

Seasonal effects
Customer demand patterns
Promotion-store interactions
drive recommendation differences.


(c) Deployment Process
Step 1: Save Model
Use:joblib
pickle
Step 2: Monthly Data Pipeline
Collect latest:Store data
Promotion options
Calendar inputs
Apply same preprocessing
Generate predictions for each store-promotion combination
Recommend highest predicted sales option
Step 3: Monitoring
Track:

Prediction accuracy
Actual vs predicted sales
Drift in customer behavior
Promotion effectiveness shifts
Step 4: Retraining Triggers
Retrain when:

Error metrics worsen
Feature distributions drift
New promotions emerge
Major market changes occur
Governance:
Monthly performance dashboards
Alert thresholds
Scheduled quarterly retraining review
