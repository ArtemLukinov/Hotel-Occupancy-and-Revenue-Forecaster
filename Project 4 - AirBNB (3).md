

# Project 4:  Airbnb New User Bookings

Author: Artem Lukinov

###Contents:

-   [Problem Statement](
-   [Executive Summary]
-   [Data Dictionary]
-   [Collecting the Data]
-   [Data Cleaning]
-   [Pre-processing]
-  [Initial EDA]
-  [The Modeling Process]
-   [Conclusions and Recommendations]

## Problem Statement

Given a list of Airbnb users, can we predict which country a new users' first booking destination will be.

Executive Summary

In this challenge, we are given a list of users along with their demographics, web session records, and some summary statistics. All the users in this dataset are from the USA.

There are 12 possible outcomes of the destination country: 'US', 'FR', 'CA', 'GB', 'ES', 'IT', 'PT', 'NL','DE', 'AU', 'NDF' (no destination found), and 'other'.  'NDF' is different from 'other' because 'other' means there was a booking, but is to a country not included in the list, while 'NDF' means there wasn't a booking.

## Data Dictionary
After collecting and cleaning the data, this was the final Data Dictionary:

| Column Name | Description |
|--|--|
| id | user id |
| date_account_created | the date of account creation |
|  timestamp_first_active | timestamp of the first activity, note that it can be earlier than date_account_created or date_first_booking because a user can search before signing up | 
| gender | gender | 
| age | age | 
| signup_method | signup method |
| signup_flow | the page a user came to signup up from | 
| language | international language preference | 
| affiliate_channel | what kind of paid marketing | 
| affiliate_provider | where the marketing is e.g. google, craigslist, other | 
| first_affiliate_tracked | whats the first marketing the user interacted with before the signing up | 
| first_device_type | first device type | 
| first_browser | first browser | 
| country_destination | **target variable**  to predict | 


## Collecting the Data
Data was provided by Kaggle exercise:

https://www.kaggle.com/c/airbnb-recruiting-new-user-bookings/data

## Data Cleaning
There were a lot of missing values in various columns. 

 - Date_first_booking was removed as it had over 58% of null values.
- Outliers in 'age' column were renamed as null values and then KNN Imputer was used to impute those values back (mean or median methods were deemed too risky to overstate the data). 
- Gender column: -unknown- was replaced with NaN as it represents missing values.
- After checking data types, 2 columns were transformed to DateTime type for further EDA.
- After cleaning the dataframe was exported to CSV.

## Initial EDA

-   Unique users
    
-   Relationship between variables
    
-   NDF imbalance in country column
    
-   Distribution by gender, country, language, device
    ![](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-04-main/Screen%20Shot%202021-03-18%20at%203.54.25%20PM.png)
-   Correlation table
- ![](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-04-main/Screen%20Shot%202021-03-18%20at%203.53.56%20PM.png)

## The Modeling Process

1. Some columns had to be encoded and dummified for the models to be able to take the numerical data in. 
2. The dataset was split into train and test sets in a 2/3 ratio. Stratification was required as our y-variable was not evenly distributed with NDF being more than half of all observations.
3. Four models were identified for this project: Multinomial Naive Bayes, Random Forrest, Extra Trees and SVC. The Random Forrest models are typically overfit but produce better classification results, and Multinomial NB do no suffer as much form being overfit. This polar scenario was very interesting to observe and compare. Also, Random Forests undersample the majority class by default and naturally address imbalance in data. 
4.  Pipeline was created to search the best parameters for the models and grid search was performed on both.
5. After fitting the models the following accuracy results were observed:
MNB: 59.28%
Random Forest: 59.71%

The Random Forest performed better than Random Forrest model. Both models are still not efficient, so parameter tuning should be performed.

Conclusions and Recommendations

As of now, the project helped develop basic models to predict the destination country. 

Recommendations would be to run more models, and then run GridSearch on the best performing model to improve performance.


