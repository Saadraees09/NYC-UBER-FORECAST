## Forecasting Uber Demand in NYC


The overall process for this project is shown in this flowchart:  

![](flowchart.png)

The notebooks and data are separated into their own folders. 

# Project Overview:
For my final project at Flatiron, I wanted to work on something that spanned across the following interests of mine:
- Urban transportation
- Geographic visualizations
- Timeseries forecasting

Therefore, I decided to see if I could forecast hourly Uber demand across NYC neighborhoods. In addition to time-lagged features (such as previous week’s demand), I added information specific to each neighborhood to improve my predictions. As a final result, I obtained relatively accurate unique forecasts for all neighborhoods in NYC.

# Data:
- The Uber data for this project came from FiveThirtyEight (https://github.com/fivethirtyeight/uber-tlc-foil-response), who obtained the data from the NYC Taxi & Limousine Commission (TLC) by submitting a Freedom of Information Law request on July 20, 2015.
- NYC’s Citibike data is provided on their website.
- The NYC subway station locations can be downloaded from data.ny.gov.
- The neighborhood geoJSON file was the same one used by Adrian Meyers’ excellent NYC taxi trips analysis.

This was a lot of data and to process it quickly, I relied upon using PySpark on a Google Cloud Dataproc cluster. Fortunately, Google’s tools make it easy to get up and running quickly.
I used a mix of this tutorial from Google and this writeup by 
Charles Bochet
 to get Jupyter notebook running on a cluster with 1 master node and 3 worker nodes.
Finally, I uploaded all my data to a Google Cloud Storage bucket which made it easy to access from multiple instances as well as avoided the headache of manually setting up a distributed filesystem (HDFS).

# Modeling:
For baseline modeling, I created forecasts using only time-lagged features. I used the following three techniques, with the goal of progressing further with the most promising forecast.
- Linear Regression
- Seasonal AutoRegressive Integrated Moving Average with eXogenous regressors (SARIMAX)
- Facebook Prophet

Models were evaluated using root-mean squared error (RMSE). This way, the error metric would be easily understandable as “number of pickups” the forecast was off by.


# Linear Regression:
Features:
- Day of the week
- Weekend or not
- Hour of day
- Number of days since 01/01/2015

The resulting forecast (plotted against test values from June) is shown below. The RMSE was 1,324.13. The forecast is certainly very good for such a simple model. However, it does not account for a lot of hour-by-hour variation and does not even try to address certain spikes.
![2021-03-25 (2)](https://user-images.githubusercontent.com/64773443/112430876-1f117700-8d15-11eb-8abd-1fc6f99b9b21.png)

# SARIMAX:
As with any ARIMA model, we have to stationarize the target variable with respect to time, and then use autocorrelation plots to determine the important number of lags and whether its a moving average (MA) or autoregressive (AR) model. The 4 plots below show that for the Uber pickups data.
![2021-03-25 (3)](https://user-images.githubusercontent.com/64773443/112431199-947d4780-8d15-11eb-8152-9025a13b7bbd.png)

- This chart shows the Uber pickups, a rolling mean, and a rolling standard deviation measure. While that data is not far from stationary, it can be better.
- Differencing the data with a lag of 168 (1 week in hours) really stationarizes the data.
- The autocorrelation plot shows important lags at 1 and 2, but also at 24 and at 48.
- The partial autocorrelation plots show important lags at 1 and 24.

Putting those parameters in, training the model, and creating a forecast leads to the following result.
![2021-03-25 (5)](https://user-images.githubusercontent.com/64773443/112431892-795f0780-8d16-11eb-8951-197ff39edd22.png)
The RMSE of 1,053.65 is quite a bit lower than the previous result with linear regression. However, there isn’t a lot of variance day-to-day in this forecast. That’s because the best predictor of demand on a Saturday, for example, is the previous Saturday. However, SARIMAX is extremely computationally expensive for so many lags, crashing my (relatively large) Google instance.

# Facebook Prophet:

![2021-03-25 (7)](https://user-images.githubusercontent.com/64773443/112433827-2f2b5580-8d19-11eb-86d2-fa6b86db6b42.png)

Not bad at all. It picked up on the weekly trend and hourly variance in each day. With an RMSE of 1,002.15, it’s already the best performing model. Since there is a lot of added customizability available on top of this, the following work was performed using Prophet.

Prophet allows adding additional regressors during the modeling process. I figured that there is signal in adding transit availability as another variable. Specifically:
- Subway train availability, defined as number of subway stops multiplied by the estimated number of passengers using the transit system at that location for that given hour.
- Bicycling activity, defined as number of citibike docking stations multiplied by number of rides originating from those stations.

![2021-03-25 (9)](https://user-images.githubusercontent.com/64773443/112434182-a52fbc80-8d19-11eb-9c32-4b12ae2c6847.png)

The increase in accuracy above indicates that there is signal in adding neighborhood-specific information to the timeseries forecasting model. We can zoom in to the neighborhoods themselves to see if all of them benefited with the new features.

Neighborhood Level Accuracy:

Each bar below represents one neighborhood of the 100 or so in this analysis. It’s clear that most neighborhoods see an increase in the forecast’s accuracy. However, there are some neighborhoods that see a decrease. Further analysis shows that these neighborhoods do not have a lot of Uber pickups within them so the added features increase the noise.
![2021-03-25 (11)](https://user-images.githubusercontent.com/64773443/112434660-37d05b80-8d1a-11eb-95d1-242f17365909.png)

# Conclusions
- Neighborhood-level data does help with time series forecasting, but it can also introduce additional noise.
- Long Short Term Memory (LSTM) Neural Networks perform rather well with timeseries forecasting and would be another tool to use in a future extension of this project.
- Bring ing in additional data — weather, real-time transit, sports events, population demographics, etc, would also be very interesting as I think there’s a potential for a lot of signal in those datasets.

# Future Work
- Apply LSTM
- Build a dashboard
- Look for more recent data and work on that

# Questions?
- Saadraees09@gmail.com
- https://www.linkedin.com/in/saad-raees-19b1231a9/

Presentation Link: https://docs.google.com/presentation/d/1NZnMN21gCKW5PwyD-CftOUO5Q2J7C1Noev1cMwqsWAg/edit#slide=id.g6d1c55ffe6_0_1







