# Capstone Readme - Capstone Project for DSI course with General Assembly
Overview
This project was completed as the final and capstone project of my Data Science Immersive course with General Assembly.
Combining the topics of health and air quality was as a result of personal interests in both the health and environment fields.

PLEASE NOTE: Since this project was part of an course, it is something that is being regularly revisited, adapted and extended (for more information on this, please see the limitations and future work sections).
The full CSV of the scraped data and that of the larger data frame have yet to be uploaded since they are too large for the repo.  Instead, a sample that was created for the machine learning (see modelling section) has been included

## Background

Air quality is linked to numerous acute and chronic conditions and according to the World Health Organisation accounts for approximately 7 million preventable deaths each year, with around 89% of these occurring in low- and middle-income countries.  The primary air pollutants are all linked to human activity and an ever increasing global population is only likely to reduce air quality.
While the links between air pollutants and the effects on health are well documented and accepted, these are mostly due to diagnoses after the fact.  Similarly, health services work on treatment and cure after the damage has been done.  Is there a way of detecting or predicting health concerns before they become a chronic or life affecting illness?
Parkrun organise 5km runs on a weekly basis across more than 700 different locations throughout the UK.  It’s entirely free and open to all demographics aged 10+.  This can be used as a metric for cardiovascular health, with a lower 5km time indicating a better level of health.

## Goals of the project

To use the coefficients or feature importances of certain models as a basis for the relationship between variables and health.
To use air quality as a factor in predicting the running times for a 5km run.

## Data Collection

The majority of data was scraped directly from the parkrun website.  The results of every event including the runner’s name, age group (a combination of their gender and the age range into which they fall) and running time are recorded on a unique page for each week at each location.
The Department for Environment, Food and Rural Affairs (DEFRA) has more than 1500 air monitoring sites across the UK, measuring and collecting hourly data on 8 different air pollutants - Carbon Monoxide, Nitric Oxide, Nitrogen Dioxide, Ozone, Silicon Dioxide, Ammonia and two sizes of Particulate Matter.  These records were accessed using the OpenWeather API.

## Data Cleaning & Wrangling

Scraping of the parkrun website was significant, iterating through almost 40,000 pages using Selenium.  As such, it was carried out in 10 iterations, each saving a separate CSV.  Each of these files had to be imported and cleaned before they could be concatenated together to give a full dataframe of all runs completed at parkrun events in 2022 (due to the time restrictions on data collection and EDA, the scope of the data was limited to 2022).  
The original air quality dataframe contained hourly readings for every day of the year and each location.  This was filtered down to include only readings taken on Saturdays at 9am (since this is the time of the runs) by accessing the day of the week and hour from the timestamp of each measurement.  These two dataframes were merged, dropping any duplicate values as well as any runs which had no air quality data applicable.   
This created a dataset with over 4.3 million observations and 20 variables.

## Feature Engineering

The following features were engineered before and after data cleaning:
Age of the runner - as the median value of the age range
Gender of the runner - binarised to “is_male”
Latitude and Longitude of the parkrun location using Nominatim

## EDA

Parkruns - age and gender distrubtions
EDA showed that the majority of the observations were for men, with almost 60 % of the runners being male and on average male runners recording faster times than female runners.  Since the mean time for male runners was 27.2 minutes and 32.8 for female runners, it suggests that gender may have an effect on running time.

![alt text](https://github.com/jyirrell/capstone/blob/main/gender_dist.png)

![alt text](https://github.com/jyirrell/capstone/blob/main/time_dist_gender.png)

The spread of ages ranged from 10 years old to runners in their 90s so the age range of the dataset is quite considerable.  Indeed, there were over 280 runs completed by 20 different people, while almost 24,000 10 year olds took part.  Age seemed to have a bearing on running time, with 16-20 years olds having the best average running times and an obvious pattern of the average time increasing with age.  The exception to this being the age ranges that included 10-14 years, which were slower than the other teenagers.  This is hardly surprising as their bodies are still developing in strength, fitness and even height (giving a greater stride length and running efficiency).

![alt text](https://github.com/jyirrell/capstone/blob/main/time_dist_age.png)

## Air pollutants
The air pollutant data showed some interesting trends.  As expected, higher levels of air pollutant were measured around larger cities and higher density residential areas.  For example, the highest levels of particulate matter (shown in the heatmap below) were from Greater London, followed by Manchester and Merseyside, with lower levels around the coasts and into North West Scotland.  However, when compared with the fastest and slowest parkrun locations, there was little correlation between these variables.

![alt text](https://github.com/jyirrell/capstone/blob/main/pm_heatmap.png)

When looking at the variation for each pollutant across the year (measured by taking the mean value for the entire UK), it’s possible to see quite drastic changes in levels across different seasons.  For example, Carbon Monoxide, Nitrogen Dioxide and Ammonia all show annual lows around July.  While the highest concentrations for Carbon Monoxide, Sulphur Dioxide and Nitrogen Dioxide all occur at the beginning of the year - around the same time that the lowest measurement of Ozone occurs.

![alt text](https://github.com/jyirrell/capstone/blob/main/pollutant_annual.png)


## Relationships

The final stage of the EDA was to look for any correlation between the variables in the dataset.  The below heatmap shows the Pearson correlation for all of the variables and suggests that many of the pollutants may have collinearity - particulate matter of 2.5 um and 10um have a strong collinearity as do silicon dioxide and nitrogen dioxide.
However, all seem to have a Pearson correlation coefficient of 0 with time.  Indeed, the only variables that have any significant correlation with time are is_male (gender) and med_age.

![alt text](https://github.com/jyirrell/capstone/blob/main/correlation_heatmap.png)


## Modelling

Due to the size of the original dataframe, a sample of 1% (approx 43,000 observations) was created for the modelling stage.  The data was initially modelled on a training set from the sample using 3 different models:
Linear Regressor
Decision Tree Regressor
Random Forest Regressor

While there were other models that could have been utilised, these 3 were prioritised because the initial goal was to use the models to understand or explain the relationship between the air quality and running times.  This information can be gained by looking at the coefficients from  Linear Regressor models and the feature importances of the Decision Tree and Random Forest regressors, as shown below.

![alt text](https://github.com/jyirrell/capstone/blob/main/linreg_coeffs-2.png)

![alt text](https://github.com/jyirrell/capstone/blob/main/feature_importances.png)



Initially, R^2 training scores on both Random Forest and Decision Tree models were very high (over 0.8), however the R^2 scores on the test data were significantly lower (in some cases negative!), as were the cross validated scores.  This suggested that both models were heavily overfit on the training data and deploying these models on unseen data (such as the test data) would result in predictions that were a long way off the actual values.  As such, these models were then tuned with GridSearchCV ( while Linear Regressor was regularised and tuned with ElasticNetCV), to find the parameters that reduced overfitting and gave the best score on the test data.
However this also resulted in the effect of the air pollution on the predictions being completely removed.  The coefficients for each air pollutant in the LinearRegression model were turned down to 0 by ElasticNet regularisation and the GridSearchCV optimised DecisonTree and RandomForest models both had 0 for feature importance in all air pollutants


## Conclusions
With the original data and with the original methodology, the models had a low amount of success in predicting the running times (highest scores for the test data was approximately 0.24).  When looking at the feature importances and coefficients for these models, age and gender were the two most influential variables, suggesting the effect of the pollutants was limited.

## Limitations
The overall findings are greatly affected by the running times being more easily affected by individual running ability than almost any other variable. The data is also limited to those that ran only on those days. If the air pollution was particularly high then those greatest affected by it - the elderly and those with cardiac or respiratory conditions are less likely to run as they would feel more restricted. As such the data for this would be completely lacking and the models would be unable to include it.
Furthermore, it is limited by the range of air quality and pollutant measurements that can be observed in the UK. In future, air quality data and running times from a greater variety of countries and conditions could show more interesting findings.


## Key learning
This was my first experience of working with a dataset of over 1 million observations as well as how to visualise datasets with various dimensions and relationships.
I became far more adept at creating visuals to both investigate and show relationships within a dataset.
The modelling stage helped me improve my understanding and application of regression models and the methods with which their parameters can be tuned to improve overall performance.
Whilst I had scraped websites before, it was never to this scale.  The project taught me a lot about how to scrape websites on a larger scale within a short timeframe.


## Future work
Does increased air pollution actually increase observed running speed?
(because worse air pollution means that those who feel the affect of it are less likely to run)
Analysing only runs by the more regular runners
Importing weather data to merge with air quality
Looking at how air pollution affects total attendance and demographic


## Libraries used

- Pandas
- NumPy
- Selenium
- BeautifuSoup
- Geopandas
- Geopy
- SciKit Learn
- Folium
- MatPlotLib
- Seaborn
- Airpyllution


