# Hotel occupancy and revenue forecaster using Machine Learning 

Author: Artem Lukinov

### [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Contents)Contents:

-   [Background](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Background) 
-  [Problem Statement](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Problem-Statement)
-   [Collecting the Data](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Collecting-the-Data)
-   [Data Cleaning](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Data-Cleaning)
-  [Initial EDA](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Initial-EDA)
-  [The Modeling Process](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#The-Modeling-Process)
-   [Conclusions and Recommendations](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Conclusions-and-Recommendations)

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#File structure)File Structure

**data**  - all data points, cleaned data file and supporting materials
**images** all imagery used in the project and presentation
**1_Data_Cleaning.ipynb** - data cleaning notebook
**2_EDA.ipynb** - Exploratory Data Analysis
**3_Modeling.ipynb** - ARIMA, SARIMA, Prophet, LSTM RNN and GRU RNN modeling process and interpretations
**4_NeuralProphet Models.ipynb** - NeuralProphet modeling process and interpretations
**requirements.txt** - required packages

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Background)Background

Amount of rooms sold, also known as Occupancy and Revenue gained are the most important metrics in hotel industry. They serve as top line numbers that determine the rest of the P&L statement. How much housekeeping do we need on a given day? When is the best time for renovation? These and many other questions demand an optimal forecast that typically falls on the shoulders of Revenue Directors or General Managers. A typical forecast starts with a rough baseline forecast, based on last year's performance or a typical weekly pattern. These first drafts are up to 40% off of the final numbers that hotels experience. The next step is to use intuition and domain knowledge to "true up" these forecasts and come up with the final version. Since these depend a lot on human factor and require lots of input, a good amount of time  is spent on this process and, even after completing it, there are always those that would question the methods used to come up with these important numbers. 

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Problem-Statement)Problem Statement

There are many variables that can affect hotel room prices each day. The goal of this project is to determine whether it is possible to use Machine Learning and improve the accuracy of the baseline forecast for a hotel in San Francisco market using historical data as core algorithm for a potential software project. 
 
## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Collecting-the-Data)Collecting the Data
The data was acquired from a high end hotel in San Francisco, CA and contains historical sales data for 2017-2021. The hotel representatives were given a pitch and volunteered their data to be anonymously used for this project with hope of improving their forecasting process. The reports were pulled from an [ORACLE's Opera Property Management System](https://www.oracle.com/industries/hospitality/products/opera-cloud-services.html) that is widely used by hotels around the world. 

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Data-Dictionary)Data Dictionary
After collecting and cleaning and concatenating the data, this was the final Data Dictionary:

| Column Name | Description |	Data type	|
|--|--|--|
| business_date | Date | DateTime |
| market_code | Market Segment Code| string |
| no_definite_rooms | Rooms sold | int64 |
| revenue | Revenue | int64 |

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Data-Cleaning)Data Cleaning
After the final dataframe was concatenated from all files, a thorough review of all columns was performed to identify potential features for the model. The data was pretty clean with no null values and just 6 rows with metadata information that were identified by their index and removed. Although data was broken down into 11 distinct market segments, a decision to aggregate data on the daily level was made in order to simplify modeling process. This data was still plenty to  support the problem statement. 

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Initial-EDA)Initial EDA

For project's purpose the data was looked at from two viewpoints: 
- Rooms sold on a daily basis represented in 'no_definite_rooms' column
- Revenue gained every day in 'revenue' column.

Rooms Sold:

PICTURE

There are couple of immediate observations here:

-   the sudden drop to the right is due to Covid-19 and the hotel only operating for essential personnel
-   there seems to be no overall trend from the annual perspective. The green line representing annual resample is relatively flat
-   there are no days where rooms sold number is higher than the capacity of the hotel (120)
-   there is some annual seasonality on a monthly level as seen in dips towards the end of each year (holiday season)
-   there may be weekly seasonality which cannot be determined from this graph.

Now let's take a look at revenue data:

PICTURE first_look_revenue

Since rooms and revenue are expected to be correlated (the more rooms sold, the higher the revenue), this graph looks almost identical albeit at a different scale.

-   the spikes represent big conventions in San Francisco that raise the hotel rooms demand and result in extreme room prices. January spikes fall on dates of JPMorgan conventions and October/November spikes represent Salesforce and Oracle.
-   the data (excluding 2020) seems to have no trend, but stil needs to be checked for stationarity for use in time series models.

## Null Hypothesis testing.

Null Hypothesis is that our data is non-stationary. 

**Augemented Dickey-Fuller Test**

| Metric|T-stat|Critical value at 1%|P-Value|
|-----------------|------------------------|---------------------------|--|
| Rooms         | (-6.8264844063432655)  | (-3.4310597571975685)| 1.9419934100737604e-09|
| Revenue      | (-9.727564233908703)  | (-3.4310598342409824)  |9.187072356670721e-17|

We can see that our statistic values of -6.8 and -9.7 are less than the critical values of -3.43 at 1%. Moreover, the p-values are insignificant. This suggests that we can reject the null hypothesis (data is non-stationary) with a significance level of less than 1%.

**Decomposition Chart**

This chart helps take another look at overall data and determine its characteristics:

PICTURE - decomp_roomsdecomp

From the trend view we see that there is a seasonal component on an annual level and there is seasonality present on rooms sold. Now let's check revenue data:

PICTURE - decomp_rev

Very similar findings to the ones in Rooms data: there is seasonality on an annual level. There is no trending observed until March 2020, when a downward trend starts to appear showing the impact of Covid-19.

## Autocorrelation checks

In order to determine the most optimal parameters for time series models, let's take a look at any autocorrelation and moving averages.

PICTURE ACF-room

Based on the ACF plot there is evidence of a trend since the small lag values have large, positive autocorrelations. The "scalloped" shape of the graph shows a seasonal cycle at every 5-7 lags (weekly) for MA(1). If it was MA(2) we would see two spikes lag1 and lag2, repeated in a weekly cycle. Because of this, we will use a value of MA(1) for the baseline model. Now let's take a look at PACF:

PCITURE PACF-room

This plots the correlation at a given lag (indicated by the horizontal axis), controlling for all of the previous lags. In the PACF plot we see some positive and some negative significant partial autocorrelations, which usually indicates strong seasonal fluctuations. There is a large dropoff after lag 1, which gives us the value for AR(1). This view also confirms seasonality at the weekly level as we see spikes on lag 7, 15, 21. However, this seasonality is not very strong. Let's explore daily revenue:

PICTURE ACF-rev

For an autoregressive (AR) time series, the ACF will go down gradually without any sharp cut-off, which is observed here. There is a MA(1) here, similar to rooms. This ACF tells us it is an AR series as well, then we turn to the PACF:


PICTURE PACF-rev

Very similar seasonal pattern, except this tells AR also looks to be AR(1). We are seeing one big spike, then starts to converse towards zero.

As graphing of the data shows, we are dealing with stationary, seasonal time series data and should explore time series models with AR(1) and MA(1). ARIMA would be a great choice as it incorporates both and SARIMA could be useful as we have seasonality in our data. As we have highly sequential data, other types of models that should be considered for this project are:

-   Long Short-Term Memory(LSTM) Recurrent Neural Network
-   Gated Recurrent Unit(GRU) Recurrent Neural Network

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#The-Modeling-Process)The Modeling Process

**Assumptions**

- current baseline accuracy is 60% which is an error of roughly **36 rooms/day** at 75% average occupancy. On the revenue side, the baseline forecast error is roughly **$10,000/day** We will compare our models' RMSE to these metrics.
- SARIMA should be a better choice since our data is seasonal,
- ARIMA model can be explored as the data has moving average and auto-regression based on ACF and PACF chart interpretations,
-  Root Mean Squared Error (RMSE) should be used as a common metric across all models in order to determine the strongest model,
- Additional types of models can be found and may be better performing depending on their specialty and complexity.
- 2020 is an outlier in the yearly data and should not be used in training or testing the models. There is enough domain knowledge that forecasts full rebound of hotel industry in H2 2021:

	-	[PwC Hotel Trend report](https://www.hotelmanagement.net/operate/pwc-report-finds-reasons-for-hope-2021)
	-	[Hospitality.net report](https://www.hospitalitynet.org/news/4102270.html)
	- [American Hotel Lodging Association Industry Outlook](https://www.ahla.com/sites/default/files/2021_state_of_the_industry_0.pdf)

2017-2018 years were used as a training data and 2019 was used as testing data. This should create a robust model that can predict normal levels of hotel KPIs when rebound happens.



## **Time Series Models**

Both AIC and RMSE measurements were used in time series modeling in order to identify the best model in this class. As the ultimate focus is on the predicted values (forecast) measuring model strength by the RMSE of the predictions seemed more suited for this project. 

**Rooms Sold**

| Model|AIC|RMSE|Improvement over Baseline|
|-----------------|------------------------|---------------------------|--|
| ARIMA (Baseline)| 4995.92 | 27.83| -|
| ***ARIMA Grid Search***     | **4888.26**  | ***27.11***  |***3%***|
| SARIMA| 5039.08|48.36|-74%|

**Revenue**

| Model|AIC|RMSE|Improvement over Baseline|
|-----------------|------------------------|---------------------------|--|
| ARIMA (Baseline)| 11615.94 | 17412.2| -|
| ***ARIMA Grid Search***     | ***11523.83***  | ***16481.31***  |***5%***|
| SARIMA| 11690.08|24786.68|-42%|

With both KPIs ARIMA w/Grid Search performed the best.

PICTURE ARIMA-gs-rooms

PICTURE ARIMA-gs-rev

The orange line in the graph represents the expected future data based on the forecasting model: amount of rooms sold and revenue. The results of this model do not look surprising - they represent the autoregressive nature of the data. The reason for a steady graph with lack of spikes shows that ARIMA deals poorly with outliers in the data and that was one of the reasons to try SARIMA - a lot of outliers in our data have a 365 day seasonality (annual conventions, holidays, etc.)

The biggest surprise was SARIMA model's performance with this data. This might be due to a more complex nature of seasonality in our data. As mentioned earlier, the seasonality was not obvious when interpreting ACF graphs. 
It does not keep the same range throughout the data and has multiple levels. Even though there is seasonality on annual and weekly levels, some events shift year to year, which skews the model's effectiveness. This further proves the theory that it is more efficient to use machine learning for **baseline** forecast and then augment it using domain knowledge. 

## Recurring Neural Networks

Due to our data being sequential, with no gaps, and including time series RNNs make good candidates for this project. Two types of RNNs were explored. 

**LSTM RNN Model**

LSTM helps with the neural network long-term memory problem due to vanishing gradient during back-propagation. 
It learns long-term dependencies and helps fine tune the network based on previous errors. 

Due to computational pressure, the grid search was performed manually with the following results:

**Rooms**
| neurons in a hidden layer | epochs trained| RMSE| improvement over baseline|
|--|--|--|--|
|8| 100 |30.26|-|
|***32***|***100***|***29.98***|***1%***|
|64|50|38.56|-27%|

**Revenue**
| neurons in a hidden layer | epochs trained| RMSE| improvement over baseline|
|--|--|--|--|
|***8***| ***100*** |***18918.36***|***-***|
|32|100|20001.84|-6%|
|64|50|21398.04|-13%|

**GRU RNN Model**

The GRU is the newer generation of Recurrent Neural Networks and is pretty similar to an LSTM. GRU is computationally more efficient due to less complex structure - GRU has just two gates: reset and update vs. LSTM having three gates: input, output and forget. This made it a worthwhile candidate for this project's data.

**Rooms**
| neurons in a hidden layer | epochs trained| RMSE| improvement over baseline|
|--|--|--|--|
|***8***| ***100*** |***29.87***|***-***|
|32|100|31.82|-7%|
|64|50|30.27|-1%|

**Revenue**
| neurons in a hidden layer | epochs trained| RMSE| improvement over baseline|
|--|--|--|--|
|***8***| ***100*** |***19246.39***|***-***|
|32|100|22128.54|-7%|
|64|50|20892.71|-1%|

Early Stops were considered, but the loss charts determined it to be risky. 100 epochs was deemed the most appropriate length of training. This parameter can be readdressed in the future for further optimization.

## Prophet and NeuralProphet Models

As part of the project, some time was set aside to research alternative models to tackle this project. Two specialized models were identified in the process and proved highly effective. The package installs were easy and the learning curve was rather short. In about 30 minutes, an average Data Science enthusiast can build a baseline model and start tweaking it. 

**Prophet**

Prophet is a procedure for forecasting time series data based on an additive model where non-linear trends are fit with yearly, weekly, and daily seasonality, plus holiday effects. It works best with time series that have strong seasonal effects and several seasons of historical data. Prophet is robust to missing data and shifts in the trend, and typically handles outliers well. [Prophet](https://facebook.github.io/prophet/) is open source software released by Facebook’s Core Data Science team.

**NeuralProphet**

NeuralProphet is a Neural Network based user-friendly time series forecasting tool. This is heavily inspired by Prophet. NeuralProphet is developed in a fully modular architecture which makes it scalable to add any additional components in the future. NeuralProphet is a decomposable time series model with the components, trend, seasonality, auto-regression, special events, future regressors and lagged regressors. Auto-regression is handled using an implementation of AR-Net, an Auto-Regressive Feed-Forward Neural Network for time series.

Similar to Prophet, the implementation was straightforward, yet multiple parameters have proven challenging to tweak the model further. 

**Rooms**

| Model | RMSE | Improvement over base |
|--|--|--|
|Prophet (baseline)| 24.43| - |
|Prophet Grid Search| 25.61| -5%|
|***NeuralProphet***| ***22.59***| ***8%***|

**Revenue**

| Model | RMSE | Improvement over base |
|--|--|--|
|Prophet (baseline)| 41479.04| - |
|Prophet Grid Search| 11015.68| 73%|
|***NeuralProphet***| ***9554.99***| ***77%***|


NeuralProphet showed a staggering 73% improvement in revenue RMSE and 8% RMSE improvement in rooms over baseline model. Since it is a new package, the default parameters are likely not the best and were optimized efficiently.

## [](https://github.com/ArtemLukinov/Hotel-Occupancy-and-Revenue-Forecaster#Conclusions-and-Recommendations)Conclusions and Recommendations

The top 3 most successful models were ranked:

**Rooms**
|Rank  | Model | RMSE|
|--|--|--|
|1|NeuralProphet |22.59|
|2|Prophet (baseline)|24.43|
|3|ARIMA Grid Search|27.11|

**Rooms**
|Rank  | Model | RMSE|
|--|--|--|
|1|NeuralProphet |9554.99|
|2|Prophet Grid Search|11015.68|
|3|ARIMA Grid Search|16481|

**Conclusions**
- It is an obvious conclusion that NeuralProphet is the top choice for this project. It is important to note that the RNNs used in this project are not complex and may be improved upon. However, given the overall time commitment, NeuralProphet should be still considered a winner in efficiency and results. 
- The project successfully proved that using Machine Learning we can get a better baseline forecast saving time and effort in the forecasting process and relying more heavily on data than intuition.
- During any project, there should be time budgeted to explore a constantly improving world of Data Science and mine for the newest and most effective ways to solve a problem. 

**Recommendations**

1. Explore NeuralProphet interface to further improve the model
2. Work on a the next version of the forecasting models to incorporate 11 market segments. Although a complex RNN with multiple inputs and outputs should be pursued, NeuralProphet's development should be followed as the team is working on a multivariate version of the model (global forecasting).
3. Wrap the best model into a user-friendly software package with Opera PMS report upload and baseline forecast download functionality.
