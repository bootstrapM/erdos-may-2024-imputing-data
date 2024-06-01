# Imputing missing data from stock time series

## Overview:
Imputing missing data in stock time series is crucial for maintaining data integrity, optimizing model performance, and ensuring the continuity necessary for accurate analysis. Missing values can lead to biased results, disrupt the sequential nature of financial data, and hinder effective risk management and historical accuracy. By filling in these gaps, one can achieve more reliable insights and make better-informed decisions, ultimately enhancing the utility and precision of financial models and strategies.


## Motivation: 
The NYSE and NASDAQ average about 252 trading days yearly. What if someone accidentally deleted data from five (or more) trading days? Missing data is often ignored or removed when we analyze data, which could be problematic. One of the problems is that stock returns depend on that "missingness" (S. Bryzgalova, 2022).
Stakeholders: AAPL Inc., investors, financial companies


## Data:
AAPL (Apple Inc.) data from January 1, 2023 until December 31, 2023. There are 250 data points because trading market closes on Saturdays, Sundays, and national holidays. We want to predict the Close values.
A daily return is computed from the adjusted close as: 
$$
R_t = \frac{P_t-P_{t-1}}{P_{t-1}}
$$
where $P_t$ is the daily return on day $t$, and  is the adjusted close on day .


# Models:
- Baseline model: The linear interpolation method predicted the missing close values by taking the weighted sum of the two closest points.
- Rolling average
- Double Exponential Smoothing:
- SARIMA
- Linear Regression between different companies
- VAR: If company X Granger Causes Apple, then we can use X to predict Apple’s missing values. Granger Causality test only works well for stationary time series. Otherwise, the test will overestimate the causality. Therefore, we applied VAR on Apple’s close differencing values. Granger Causality test was applied for 7 different companies, including …. We found that the NVIDIA Granger causes Apple’s close difference at lag = 9. 
- KNN:


# Results: 
Linear interpolation remains a robust choice for both small and large gaps in stock time-series data compared to more complicated interpolation methods.
