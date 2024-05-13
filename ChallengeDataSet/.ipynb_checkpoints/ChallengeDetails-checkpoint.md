# Source

This problem is taken from https://challengedata.ens.fr/challenges/63

# Challenge context

When computers are responsible for all trading decisions through algorithms, it is very important that their input data be of the best quality possible. However, sometimes values ​​may be missing or contain outliers.

Futures contracts are a promise to deliver some underlying asset (let’s say cotton) at a given expiry date and at a predetermined price; they are for instance useful to a company for locking a price for a future time when they will need the underlying asset (it could be a garment manufacturer, for instance, for cotton).


# Challenge goals

The goal of this challenge is to recover **missing values** in financial time series covering 300 futures contracts.

The financial time series in question contains “daily bid-ask spreads”, which are simply some daily average of the “bid-ask spreads” observed throughout the day. This bid-ask spread is the difference between the highest price a buyer is ready to pay for the futures contract (highest bid price) and the lowest price a seller will accept (lowest ask price) for a given asset.

A large price difference (large spread) between bid and ask reflects the fact that only a few participants are ready to sell or to buy (as they are not making efforts to give a price closer to what the other side would like). Conversely, a small bid-ask spread reflects a liquid market where participants are more willing to trade. Anticipating the average bid-ask spread of the next trading day is thus important for knowing how much one can expect to trade during the day.

In this challenge, we removed the daily spread for some days of a 10 years history, for each of about 300 futures contracts.

The goal of the challenge consists in predicting these missing values using other features of the time series.


# Data description

## Dimension

Less than 1,000,000 rows, including 200,000 bid-ask spreads to predict.

## Description

- product_id and liquidity_rank define a unique futures contract. The dataset contains around 100 product_id (corresponding for instance to a specific type of crude oil). For each product_id several liquidity ranks can exist: liquidity_rank 0 refers to the contract with the closest expiry date (which is often the most traded one), 1 to the next contract to expire, etc. Thus, a contract of rank 1 becomes a contract of rank 0 when the previous contract of rank 0 expires.
- dt_close represents the day number (they are therefore chronological) for each data sample. For a given dt_close there is 1 entry per (product_id, liquidity_rank) pair.
- dt_expiry similarly represents the date of the futures contract expiry.
- normal_trading_day is set to 0 when the market is closed or the market activity is reduced.
- open, close represent contract prices, resp. at market opening and close.
- high, low represent the highest and lowest price of the contract during the day.
- volume is the number of contracts exchanged during the day, up to a factor that depends only on product_id.
- open_interest is the number of active contracts at the end of the day, with the same factor applied as for the volume.
- spread is the “daily bid-ask spread”, which is the value we want to predict.
- tick_size is a proxy for 1 unit of spread (this is not directly price difference).
- fixed is set to 1 when one or more features have been fixed for various reason (outliers, missing values…).

Prices are all normalized in some way (the values are thus not in currency units), but they are consistent within each product_id.

# Benchmark description

## Metric

Root mean square error between the predicted daily bid-ask spread and the real one, measured over every missing spread.

## Benchmark

Whereas classical time series prediction problems only allow the use of past data for predictions, participants can exploit the complete time series “to fill the gap”.

A very simple benchmark would be:

`df['spread'].interpolate(method='linear')`
