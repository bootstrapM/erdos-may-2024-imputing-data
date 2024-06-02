# Time Series Imputation: Missing Financial Data

## Overview:
Imputing missing data in financial time series is crucial for maintaining data integrity, optimizing model performance, and ensuring the continuity necessary for accurate analysis. Missing values could lead to biased results, disrupt the sequential nature of financial data, and hinder effective risk management. By filling in these gaps, one can achieve more reliable insights and make better-informed decisions, ultimately enhancing the utility and precision of financial models and strategies.


## Motivation: 
Stock exchanges such as NYSE and NASDAQ average about 252 [trading days](https://en.wikipedia.org/wiki/Trading_day) a year (trading market remins closed on weekends and national holidays). Such markets have trading hours and non-trading hours nationally and internationally. Typically, trading begins at 9:30 AM and ends at 4:00 PM. The remaining hours is what we might call a period of missing data during which the `Close` price on the $i$'th day evolves to (usually different) `Open` price on $(i+1)$ 'th day as consequence of, for instance, developments in financial markets worldwide. Analyzing imputation for such time series could therefore yeild insight on correlations in international market and the relevent models and market predictors to use for the more practical problem of making forecast in price movements. Another situation of missing data that one could speculate is where an external shock (in the form of market news or geopolitical events) leads to huge jumps in the prices which might temporarily halt trading.

Though missing data in the daily stock prices is rare, in this project, we analyse a toy problem where we delete a few data points in the stock price time series by hand and attempt to impute it through various methods. The goal is to see which methods and what market indicators work best for such a dataset.


## Dataset:
We use `AAPL` (Apple Inc.) stock price data from January 1, 2023 until December 31, 2023. There are 250 data points. A sample is shown below.
![](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/dataframe.png)

We manually deleted the `Close` values for 7 windows of `[5, 4, 3, 2, 1]` consecutive days. The goal is to impute these "missing" data though various techniques. 

| ![Missing Close prices of Apple stock time series (7 windows with 5 consecutive missing points in each)](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/OriginalDataset.png) |
|:--:| 
| *Missing Close prices of Apple stock time series (7 windows with 5 consecutive missing points in each)* |

We also used 2023 data for `NVDA`, `MSFT`, `TSM`, `META`, `GOOG` stocks for performing cross-sectional analysis using Linear Regression and Vector Auto Correlation. A plot of the stock price series is presented below

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/multiple_prices.png) 

## Exploratory Data Analysis
He we give an exploratory data analysis of the `APPL` dataset. The plot below depicts the closing prices differences between consecutive trading days

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/PriceDifferences.png) 

While this looks sufficiently random by eye, better insights can be gained by looking at the summary statistics and the emperical cumulative distribution (ECDF) of the price differences. The summary statistics are 

| Statistic    | Value |
| -------- | ------- |
| Mean  | 0.272    |
| Standard Deviation | 2.11     |
| Skewness    | -0.257    |
| Excess Kurtosis | 1.82|

The ECDF is shown in the plot below (along with the ECDF of the normal distribution for which we used the mean and standard deviation given in the table above)

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/ECDF.png) 

Examination of the empirical cumulative distribution function reveals that price fluctuations largely follows a normal distribution across a large spectrum of values. This suggests that stock price movements follow a random walk pattern. There are outliers in this plot which also be seen in the excess Kurtosis (see the table of summary statistics above). In general, stock prices data are empirically known to follow a fat tailed distribution rather than a normal distribution. 

The plot below depicts the autocorrelation function across various lag intervals. The small autocorrelation values suggests that the price movements have little to no predictable relationship with past price movements.

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/ReturnSeriesAutocorrelation.png) 

The above analysis is reflective of the Efficient Market Hypothesis which states that that at any given time, asset prices fully reflect all historical information prior to that time. This implies Random Walk for asset prices -- stock prices follow a random walk, meaning that price changes are random and cannot be predicted with certainity based on historical data or other available information. Given the random walk behaviour the simplest thing to do would be to carry forward the last observation (this goes under the name of LOCF -- Last Observation Carried Forward. This is essentially the idea behind Martingale -- A stochastic process $\{X_1, X_2 \ldots, X_n, X_{n+1}\}$ is said to be a Martingale if $\mathbf{E}\left(X_{n+1} \mid X_1, \ldots, X_n\right)=X_n$). A random walk is an example of a Martingale. 

The above method of imputation (LOCF) is presented in the plot below

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/LOCF.png) 

When compared with the true data, we find a mean squared error of 11.38 (for the case where we have seven windows of 5 consequetive missing values in each window). 

In what follows we explore regression and time series models in search of other better performing frameworks.

## List of models that we tried:

The main models that we tried were the following (we also give a brief description of some of the models alongside):

- Baseline model: Linear Interpolation
- k-Nearest Neighbour: Weighted 2-Nearest Neighbors Regression with date as a feature. We want to generalize this idea by using more features, such as, open stock price and volume.
- Rolling average: Uses the average of a fixed number of points to the left and right of the missing values to make a prediction 
- Double Exponential Smoothing: A forecasting method that accounts for both the level and the trend in the data by applying exponential smoothing twice, once to the level and once to the trend.
- SARIMA: Incorporates seasonal patterns, trends, and autoregressive components. Can use both forward and backward forecasting for imputing missing data in the middle of the time series.
- Linear Regression: Useful for regressing price movement of different companies' stocks that have a strong correlation. 
- Granger Causality: This is a statistical hypothesis test used to determine whether one time series can predict another. In other words, if the past values of one time series `X` provide significant information about the future values of another variable `Y`  (beyond what is contained in the past values of `Y` alone), then `X` is said to <em>Granger-cause</em> `Y`. While this method is closely related to `cross-correlation`, is it more sophisticated since it provides a test for predictive causality, an information that can very useful for satatistical modeling. 
- Vector Auto Regression: A statistical model used to capture the linear interdependencies among multiple time series by allowing each variable to be a linear function of past values of itself and the past values of all other variables in the system.


### Summary of implimentation

- Linear Regression: We took linear interpolation as our baseline model. This model gives an MSE of 3.475 (for the case of 5 consecutive missing point) thereby outperforming LOCF. 

- Rolling Average, Double Exponential Smoothing, and SARIMA: These methods make predictions based on the closing prices alone. They performed comparably to linear interpolation for a single missing point but worsened as the number of missing data points increased. The optimal performance occurs when predictions based on data to the left and right of the missing points are weighted equally. An illustration of these methods is given below 

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/methods_illustration.jpg)

- Cross-Sectional Analysis:

| ![Correlation in daily percentage return for APPL against NVDA, MSFT, TSM, META, GOOG during 2020-2022](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/daily_ret_scatter_plot2020_2022.png) |
|:--:| 
|*Correlation in daily percentage return for APPL against NVDA, MSFT, TSM, META, GOOG during 2020-2022*|

| ![Correlation in daily return for APPL against NVDA, MSFT, TSM, META, GOOG in 2023](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/daily_ret_scatter_plot_2023.png)|
|:--:| 
|*Correlation in daily return for APPL against NVDA, MSFT, TSM, META, GOOG in 2023*|

After examining cross-correlation between Apple stocks and other tech companies stocks time series (shown in the correlation plots above), we observed that it might be reasonable to use the movement of the stock prices of other companies to impute the missing Apple stocks data. One such method of incorporating the stocks prices is by regressing daily returns of Apple stock on daily returns of other companies, and then using prediction of the daily returns for imputing missing values. The daily return is defined as follows $$r_t = \frac{P_t- P_{t-1}}{P_{t-1}}$$ where $P_t$ is the price of the stock at time $t$.

With this method, it is observed that when the linear regression on the daily returns has a good fit, the predictions of the missing stock values have lower error.

- Granger Causality (GC): As explained above, the idea behind Granger causality is that if `X` <em>Granger causes</em> `Y`, we can use `X` to predict `Y`. For example, if we want to predict stock for Apple, and we find that Google stock <em>Granger causes</em> Apple stock. Using Google stock in the modeling procedure will improve the prediction for Apple price imputation. We ran GC tests for 7 different companies: Apple, Google, Microsoft, NVIDIA, Amazon, Meta, TSMC. We found that the NVIDIA’s `Close difference` values Granger Causes Apple’s `Close difference` values. This information can thus be used to include NVIDIA in a statistical time series model such as Vector Auto Correlation (which is what we used below).

![alt_text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/GC_matrix.JPG)



## Final results

First we present MSE results of three methods (Rolling Average, Double Exponential Smoothing, and SARIMA) that make predictions based on the **`Close` prices series alone**. 

- The Rolling Average method

|![Results from Rolling Average method](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/rolling_average.png)|
|:--:| 
|*Results from Rolling Average method*|
  
- Double Exponential Smoothing

|![Results from Double Exponential Smoothing](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/ARIMA.png)|
|:--:| 
|*Results from Double Exponential Smoothing*|

- SARIMA
  
|![Results from SARIMA](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/double_exponential_smoothing.png)|
|:--:| 
|*Results from SARIMA*|
  
The errors of these methods were measured relative to linear interpolation using a normalized mean squared error. One key insight that emerged from our analysis was that the best results are achieved by giving equal weight to predictions based on data to the left and right of the missing data. Interestingly, these methods perform similarly to linear interpolation when there is a single missing point, but their performance deteriorates as the number of missing points increases.


Next, in the plot below we present the performance of the various models that incorporate **additional predictors**. The performance is evaluated on the 2023 data only and provides a uniform comparison of our models. On the x-axis is the number of the consecutive missing days. On the y-axis the ratio of the mean squared error of the model to the mean squared error of the linear interpolation. We can see that in all circumstances for this data our models perform worse than linear interpolation though we point out that the performance for linear regression was better than linear interpolation in some years (see the [notebook]() **Evgeniya add link to your notebook here** )

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/PresentationAssets/MSE_ratio_2023data.png)

## Conclusion and Future Directions

- We found that among the various times series models that we tried, linear interpolation is a robust choice for imputing both small and large gaps in data. When there is sufficiently high correlation between price movements of two companies, one may be used to regress over the the other. 


- In a future work we would like to incorporate other predictors that include Trading Volume, Derived Indicators (such as moving averages, relative strength index (RSI), and Bollinger Bands derived from historical price and volume data), Dividends and Corporate Actions (such as stock splits, buybacks), Industry Trends (Sector-specific trends having relatively stong cross-correlation), Market Indices (S&P 500 or Dow Jones Industrial Average ) and Interest Rates (Changes in central bank policies and interest rates, which can affect borrowing costs and investment returns). 

- We would like to systematically explore under what conditions the methods that we used outperform linear interpolation.
 
- We also began exploring advanced techniques like State Space Models (Kalman Filter, Kalman Smoother) and Neural Network (Neural ODEs, Generative Adversarial Networks) based approaches whose applications go beyond the present context. A preliminary implementation of the Kalman filtering technique is presented below.

### Other advanced techniques

Kalman Filtering is an algorithm used in time series analysis to estimate the state of a dynamic system from a series of noisy measurements. It operates recursively by updating it's estimates of the current state based on both the previous state and new measurements, accounting for uncertainties in both. The two main steps in the algorithm are prediction and update. In the prediction step it forecasts the next hidden state based on a linear model. In the update step it adjusts this prediction using the latest observed data. Below we present an attempt to use Kalman filter for imputing

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/KalmanFilter.png)

While it looks this algorithm perofrmed significantly worse than other techniques mentioned above we haven't explored the full potential of the method and leave a more treatment it to future work.

## Code Description

Cleaned Jupyter Notebooks used for the results shown above can be found in this [folder](https://github.com/bootstrapM/erdos-may-2024-imputing-data/tree/main/Models). It consists of the following notebooks

- [RollingAverage_ExpSmoothing_ARIMA.ipynb](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/Models/RollingAverage_ExpSmoothing_ARIMA.ipynb): This notebook compares the accuracy of rolling average, double exponential smoothing and ARIMA interpolation methods to that of the baseline (linear interpolation).
- [Stationary_test.ipynb](): We need to check for stationary before applying the Granger Causality test.
- [Granger_Causality_test.ipynb](): Apply Granger Causality Test. Companies that Granger Causes Apple’s close differencing will be implemented in VAR. 
- [VAR.ipynb](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/Models/VAR.ipynb): VAR model with the lag found from Granger Causality test.
- [regression_on_daily_return.ipynb](): Regression analysis of the daily return values of the other tech companies is used to predict Apple stock closing values.
- [ForwardBackwardARIMA.ipynb](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/Models/ForwardBackwardARIMA.ipynb): 
- [summary_of_models.ipynb](): Table with relative MSE data of all models
- [KalmanFilter.ipynb](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/Models/KalmanFilter.ipynb): 

The [folder]() contains figures pertaining to EDA. It consists of the following:

- [eda_for_regression](): basic data analysis for choosing models and features for regression on other companies’ stock movement







