# Time Series Imputation: Missing Financial Data

## Overview:
Imputing missing data in financial time series is crucial for maintaining data integrity, optimizing model performance, and ensuring the continuity necessary for accurate analysis. Missing values could lead to biased results, disrupt the sequential nature of financial data, and hinder effective risk management. By filling in these gaps, one can achieve more reliable insights and make better-informed decisions, ultimately enhancing the utility and precision of financial models and strategies.


## Motivation: 
Stock exchanges such as NYSE and NASDAQ average about 252 [trading days](https://en.wikipedia.org/wiki/Trading_day) a year. Markets at such exchanges have trading hours and non-trading hours nationally and internationally. Each day, we have 8 hours of trading data and 16 hours of what we might call missing data. The challenge appears when we analyze time series of international stock markets worldwide. Analyzing imputation for such time series could yeild insight on the kind of models and market predictors to use for the more relevant problem of making forecast in price movements. Another situation that one could speculate is where an external shock leads to huge jumps in the prices which might temporarily halt trading.

Though missing data in the daily stock prices is rare, in this project, we analyse a toy problem where we delete a few data points in the stock price time series by hand and attmept to impute it through various methods. The goal is to see which methods and what predictors work best for such a dataset.


## Dataset:
AAPL (Apple Inc.) data from January 1, 2023 until December 31, 2023. There are 250 data points because trading market closes on weekends and national holidays. We manually deleted the `Close` values for 7 wondows of `[5, 4, 3, 2, 1]` consecutive days. The goal is to impute these "missing" data though various techniques. 

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/OriginalDataset.png) 

We also used 2023 data for `NVDA`, `MSFT`, `TSM`, `META`, `GOOG` stocks for performing cross-sectional analysis using Linear Regression and Vector Auto Correlation. A plot of the stock price series 

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/multiple_prices.png) 

## Exploratory Data Analysis
He we give an exploratory data analysis (EDA) of our dataset. The plot below presents depicts the closing prices differences between consecutive trading days

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/PriceDifferences.png) 

While this looks sufficiently random by eye, better insights can be gained by looking at the summary statistics and the emperical cumulative distribution (ECDF) of the price differences. The summary statistics are 

| Statistic    | Value |
| -------- | ------- |
| Mean  | 0.272    |
| Standard Deviation | 2.11     |
| Skewness    | -0.257    |
| Excess Kurtosis | 1.82|

The ECDF is shown in the plot below (along with the ECDF of the Normal Distribution with mean and standard deviation given in the table above)

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/ECDF.png) 

Examination of the empirical cumulative distribution function reveals that price fluctuations largely follows a normal distribution across a large spectrum of values. This suggests that stock price movements follow a random walk pattern. There are outliers in this plot which also be seen in the excess Kurtosis (see the table of Summary Statistics above). In general, stock prices data are known to follow a fat tailed distribution rather than a normal distribution. 

The plot below depicts the autocorrelation function across various lag intervals. The small autocorrelation values suggests that the price movements have little to no predictable relationship with past price movements.

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/ReturnSeriesAutocorrelation.png) 

This analysis is reflective of the Efficient Market Hypothesis which states that that at any given time, asset prices fully reflect all historical information prior to that time. This implies Random Walk for asset prices -- stock prices follow a random walk, meaning that price changes are random and cannot be predicted with certainity based on historical data or other available information. Given the random walk behaviour the simplest thing to do would be to carry forward the last observation (this is the idea behind Martingale -- A stochastic process $\{X_1, X_2 \ldots, X_n, X_{n+1}\}$ is said to be a Martingale if $\mathbf{E}\left(X_{n+1} \mid X_1, \ldots, X_n\right)=X_n$ ). 

This method of imputation is presented in the plot below

![alt text](https://github.com/bootstrapM/erdos-may-2024-imputing-data/blob/main/HimanshuNotebooks/LOCF.png) 

When compared with the true data, we find a mean squared error of 11.38 in case where we have seven windows of 5 consequetive missing values in each window. In the following we explore regression and time series models that have better performance. 

## Models:
-Baseline model: Linear Interpolation
-k-Nearest Neighbour: Weighted 2-Nearest Neighbors Regression with the feature date. We want to generalize this idea by using more features, such as, open stock price and volume.
-Rolling average:  Uses the average of a fixed number of points to the left and right of the missing values to make a prediction
-Double Exponential Smoothing: A forecasting method that accounts for both the level and the trend in the data by applying exponential smoothing twice, once to the level and once to the trend.
-SARIMA: Incorporates seasonal patterns, trends, and autoregressive components. Can use both forward and backward forecasting for imputing missing data in the middle of the time series.
-Linear Regression between price movement of different companies' stocks
-Vector Auto Regression: A statistical model used to capture the linear interdependencies among multiple time series by allowing each variable to be a linear function of past values of itself and the past values of all other variables in the system. 


## Results:

First we present the results of three methods that make predictions based on the `close` prices time series data alone. 

- The Rolling Average method uses the average of a fixed number of points to the left and right of the missing values to make a prediction
  
- Double Exponential Smoothing considers the trends in the data whereas SARIMA incorporates seasonal patterns, trends, and autoregressive components to estimate missing values.
  
The errors of these methods were measured relative to linear interpolation using a normalized mean squared error. One key insight that emerged from our analysis was that the best results are achieved by giving equal weight to predictions based on data to the left and right of the missing data. Interestingly, these methods perform similarly to linear interpolation when there is a single missing point, but their performance deteriorates as the number of missing points increases.




In the plot below we present the performance of the various models that incorporate additional predictors. The performance is evaluated on the 2023 data only, which we decided to use to have a uniform comparison of our models. On the x-axis is the number of the consecutive missing days. On the y-axis the ratio of the mean squared error of the model to the mean squared error of the linear interpolation. We can see that in most circumstances for this data our models perform worse than linear interpolation. However, we also note that the performance of our models could be better for other years.


Linear interpolation remains a robust choice for both small and large gaps in stock time-series data compared to more complicated interpolation methods.
