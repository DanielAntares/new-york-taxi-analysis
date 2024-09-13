# New York Taxi Spending Analysis

## Problem definition
Predict the average money spent on taxi rides for each region of New York per given day and hour.

This problem is a supervised regression problem. Supervised because we have the actual value of the value we’re trying to predict and regression because what we’re trying predict is a continuous variable (as opposed to categorical).

## Data problems I fixed

![Negative and zero values graph](/images/remove_negative.png)

Removed negative total_amount values: They are likely faulty data points and there weren’t many of them. Zero values from total_amount are also removed during the Data Cleaning process.

![Graph showing the too high values](/images/remove_too_high.png)

Removed too high values from total_amount: Some values for total_amount were too high. Because of this, I decided to come up with an upper limit. There were only 1378 data points for total_amount higher than $150. Compared to 2463931 data points, this is not a huge loss of information. Thus, I decided to remove data points with a total_amount value higher than $150.

## Original features of the model
Here is the list of features that can be used for model development that came with the original data: ['PULocationID','transaction_date','transaction_month','transaction_day','transaction_hour', 'trip_distance', 'total_amount'].

You can refer to [this document](https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf) for the meaning of each feature.

## Feature Engineering
I’ve added 3 sets of new features to the model. First set is time-based feature. These include, weekend and holiday boolean.

Second set is location-based information. We have Location IDs per region but there is a higher level abstraction for regions called Boroughs. This information came from the source of the main data.

The last set is weather related data. I’ve downloaded this data from [this website](https://www.kaggle.com/datasets/aadimator/nyc-weather-2016-to-2022). It’s free to use for development.

Here is a list of all features used in the final model: ['PULocationID', 'transaction_date', 'transaction_month', 'transaction_day', 'transaction_hour', 'trip_distance', 'total_amount', 'count_of_transactions', 'transaction_week_day', 'weekend', 'is_holiday', 'Borough', 'time', 'temperature_2m (°C)', 'precipitation (mm)', 'cloudcover (%)', 'windspeed_10m (km/h)']

## The algorithms I tried and the results
I tried Decision Trees, Random Forest, and Gradient Boosting. The benchmark model is a Decision Tree. In the benchmark model, I only included the original features of the model as stated above. And on the normal models I used all original features plus the newly created ones.

Here are the performance results before tuning:
| Algorithm         | MAE   | RMSE   | R2    |
|-------------------|-------|--------|-------|
| Benchmark model   | 9.056 | 14.017 | 0.192 |
| Decision tree     | 7.311 | 12.173 | 0.391 |
| Random forest     | 2.862 | 5.534  | 0.874 |
| Gradient boosting | 5.848 | 9.721  | 0.611 |

The Random Forest model is selected to be tuned. The best parameter values are: n_estimators: 1800 min_samples_split: 10 min_samples_leaf: 2 max_features: auto max_depth: 50 bootstrap: False

However, because of the high n_estimator value, I’ve decided to go with the parameter values that give the 3rd best performance with a bit of changes, which is not very different than the best performance: n_estimators: 200 min_samples_split: 5 min_samples_leaf: 1 max_features: sqrt max_depth: 100 bootstrap: False

The performance compares to previous models is:
| Algorithm           | MAE   | RMSE   | R2    |
|---------------------|-------|--------|-------|
| Benchmark model     | 9.056 | 14.017 | 0.192 |
| Decision tree       | 7.311 | 12.173 | 0.391 |
| Random forest       | 2.862 | 5.534  | 0.874 |
| Gradient boosting   | 5.848 | 9.721  | 0.611 |
| Tuned Random Forest | 5.011 | 8.786  | 0.682 |

Here is the True vs. Predicted value plot for the tuned random forest model. X-axis is the true values and y-axis the predicted values.

![Performance graph of tuned Random Forest](/images/tuned_random_forest.png)

## Next steps
Based on the plot above, the performance can be improved.  There are a few ways that wasn’t tried in this notebook:
- Limiting the regions included in this analysis. Removing the regions that do not normally get a lot of taxi traffic can be omitted. This might be a good action to take depending on the problem at hand. (If the goal is to service all of NYC no matter what, we should keep those data points)
- Hand selecting borough that should be included in the model. Again this should be decided based on the problem at hand and how this model is going to be used. But if the goal is solely to increase model performance, only including boroughs with the most transactions can increase the performance since likely most mistakes come from boroughs with fewer data points. Though this assumption should be checked before taking this action.
