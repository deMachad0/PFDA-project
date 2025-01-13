# Programming for Data Analytics

## CASEMENT Data Analysis 
This notebook focuses on manipulating and analyzing data from the following source: Historical Data - Met Eireann - The Irish Meteorological Service.

## Overview
Objective: Perform data cleaning, transformation, and visualization to gain insights from the Casement weather dataset.\
Data Source: The dataset provides historical weather information collected by Met Éireann.
`https://www.met.ie/climate/available-data/historical-data`

### Steps: 

1. Importing Libraries

```bash
import pandas as pd 
import numpy as np 
import matplotlib.pyplot as plt 
import seaborn as sns 
```

`pandas as pd`: Used for data manipulation and analysis.\
`numpy as np`: Provides support for numerical computations.\
`matplotlib.pyplot as plt`: Used for creating static, interactive, and animated visualizations in Python.\
`seaborn as sns`: Built on top of Matplotlib, it provides a high-level interface for drawing attractive statistical graphics.

---
2.  Loading the Dataset

```bash
url = "https://cli.fusio.net/cli/climate_data/webdata/hly3723.csv"
df = pd.read_csv(url, skiprows=23, low_memory=False)
```

`pd.read_csv()`: Reads a CSV file into a DataFrame.\
`skiprows=23`: Skips the first 23 rows of the file (often used to skip metadata or header information).\
`low_memory=False`: Prevents mixed data type warnings by processing data in chunks.

---
3. Converting date Column to Datetime

```bash
df['date'] = pd.to_datetime(df['date'], format="%d-%b-%Y %H:%M", errors='coerce', utc=True)
```

`pd.to_datetime()`: Converts the date column to a datetime object.\
`format="%d-%b-%Y %H:%M"`: Specifies the date format (e.g., "12-Jan-2023 14:30").\
`errors='coerce'`: Invalid parsing results in NaT (Not a Time).\
`utc=True`: Ensures the datetime is treated as UTC.

---
4. Extracting the Hour

```bash
df['Hour'] = df['date'].dt.hour
```

`df['date'].dt.hour`: Extracts the hour component from the date column and stores it in a new column Hour.

---
5. Normalizing the date Column

```bash
df['date'] = df['date'].dt.normalize()
```

`df['date'].dt.normalize()`: Removes the time component, keeping only the date portion (sets the time to midnight).

---
6. Converting wdsp Column to Numeric

```bash
df['wdsp'] = pd.to_numeric(df['wdsp'], errors='coerce')
```

`pd.to_numeric()`: Converts the wdsp column to numeric values.\
`errors='coerce'`: Non-numeric values are converted to NaN.

---
7. Dropping Rows with Missing Wind Speed Values

```bash
df = df.dropna(subset=['wdsp'])
```

`df.dropna(subset=['wdsp'])`: Removes rows where wdsp contains NaN.

---
8. Setting the date Column as the Index

```bash
df.set_index('date', inplace=True)
```

`df.set_index('date', inplace=True)`: Sets the date column as the DataFrame index, enabling time-based operations.

---
9. Calculating Hourly Mean Wind Speed

```bash
hourly_mean_wdsp = df.groupby('Hour')['wdsp'].mean().reset_index()
```

`df.groupby('Hour')['wdsp'].mean()`: Groups the data by hour and calculates the mean wind speed for each hour.\
`reset_index()`: Resets the index to ensure Hour remains a column in the DataFrame.

---
10. Calculating Monthly Mean Wind Speed

```bash
monthly_mean_wdsp = df.groupby('Month')['wdsp'].mean().reset_index()
```

`df.groupby('Month')['wdsp'].mean()`: Groups the data by hour and calculates the mean wind speed for each hour.\
`reset_index()`: Resets the index to ensure Hour remains a column in the DataFrame.

---
11. Plotting the Data

Hourly mean wind spped
```bash
sns.lmplot(data=hourly_mean_wdsp, x='Hour', y='wdsp')
plt.xlabel("Hour of Day")
plt.ylabel("Mean Wind Speed")
plt.show()
```

`sns.lmplot()`: Creates a linear regression plot for Hour vs. wdsp.\
`plt.xlabel()`: Sets the x-axis label to "Hour of Day".\
`plt.ylabel()`: Sets the y-axis label to "Mean Wind Speed".\
`plt.show()`: Displays the plot.

Monthly mean wind speed
```bash
plt.figure(figsize=(10, 6))
sns.barplot(data=monthly_mean_wdsp, x='Month', y='wdsp')
plt.title("Monthly Mean Wind Speed")
plt.xlabel("Month")
plt.ylabel("Mean Wind Speed")
plt.xticks(ticks=range(0, 12), labels=[
    "Jan", "Feb", "Mar", "Apr", "May", "Jun",
    "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"
])
plt.show()
```

`sns.lmplot()`: Creates a linear regression plot for Month vs. wdsp.\
`plt.xlabel()`: Sets the x-axis label to "Month of Day".\
`plt.ylabel()`: Sets the y-axis label to "Mean Wind Speed".\
`plt.xticks(ticks=range(0, 12)`: Sets the tick positions to be the integer values from 0 to 11. This represents the months of the year (0 corresponds to January, 1 to February, and so on)./
`plt.show()`: Displays the plot.

---
12. Prediction (the next 10 years)

```bash
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
```
`LinerRegression`: Used for the linear regression model to predict wind speed based on year and monh.\
`train_test_split`: Splits the data into training and testing sets for model validation.

---
13. Extracting Year and Month features

```bash
df['year'] = df.index.year
df['month'] = df.index.month
```
The year and month are extracted from the index of the DataFrame (which is a date column) and stored as new columns in the DataFrame `df`.

---
14. Grouping Data by Year and Month

```bash
monthly_mean_wdsp = df.groupby(['year', 'month'])['wdsp'].mean().reset_index()
```
The dataset is grouped by year and month.\
The mean wind speed (wdsp) is calculated for each month of each year.

---
15. Preparing Features and Target Variables

```bash
X = monthly_mean_wdsp[['year', 'month']]
y = monthly_mean_wdsp['wdsp']
```

`X`: contains the features: year and month.\
`y`: contains the target variable: mean wind speed (wdsp).

---
16. Train-Test Split

```bash
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```
The data is split into training and testing sets.\
`test_size=0.2`: means 20% of the data will be used for testing, and 80% for training.\
`random_state=42`: ensures the split is reproducible.

---
17. Training the Linear Regression Model

```bash
model = LinearRegression()
model.fit(X_train, y_train)
```
A LinearRegression model is created and trained using the training data (X_train, y_train).

---
18. Forecasting for the Next 10 Years

```bash
future_years = np.arange(monthly_mean_wdsp['year'].max() + 1, monthly_mean_wdsp['year'].max() + 11)
future_months = np.arange(1, 13)
future_data = pd.DataFrame([(year, month) for year in future_years for month in future_months], columns=['year', 'month'])
forecast = model.predict(future_data)
```
`future_years`: Creates an array starting from the year after the last historical year to the next 10 years.\
`future_months`: Generates an array of months from 1 to 12 (representing January to December).\
`future_data`: A new DataFrame is created with all combinations of future years and months.\
`forecast`: The model is used to predict the wind speed (wdsp) for the future data (next 10 years).

---
19. Creating Date Index for Future and Historical Data

```bash
future_dates = pd.to_datetime(future_data['year'].astype(str) + '-' + future_data['month'].astype(str), format='%Y-%m')
historical_dates = pd.to_datetime(monthly_mean_wdsp['year'].astype(str) + '-' + monthly_mean_wdsp['month'].astype(str), format='%Y-%m')
```
`future_dates`: Converts the year and month columns in future_data to a proper datetime format (YYYY-MM).\
`historical_dates`: Converts the year and month columns in monthly_mean_wdsp to a datetime format for historical data.

---
20. Plotting Historical and Forecasted Data

```bash
plt.figure(figsize=(12, 6))
plt.plot(historical_dates, monthly_mean_wdsp['wdsp'], label='Historical Data')
plt.plot(future_dates, forecast, label='Forecasted Data', color='red')
plt.title("Wind Speed Forecast for the Next 10 Years")
plt.xlabel("Date")
plt.ylabel("Mean Wind Speed (wdsp)")
plt.legend()
plt.grid(True) # Add a grid for better readability
plt.tight_layout() # Adjust layout to prevent labels from overlapping
plt.show()
```

`plt.plot`: Plots historical data (historical_dates, monthly_mean_wdsp['wdsp']) and forecasted data (future_dates, forecast).
The title, labels, and legend are added to the plot.
`plt.grid(True)`: Adds a grid for better readability of the plot.
`plt.tight_layout()`: Adjusts the layout to prevent overlapping labels.
Finally, the plot is displayed using `plt.show()`.

---

### Conclusion

This study focuses on wind speed data collected from the CASEMENT station near Clondalkin in Dublin, spanning the period from 1964 to 2024. Analyzing and manipulating this dataset allows for a comprehensive examination of wind speed characteristics in the region. The analysis shows various temporal scales, including hourly and monthly variations, as well as long-term trends, with the goal of providing insights into past wind patterns and generating predictions for future wind speeds.

First plot demonstrates the daily cycle of wind speed, with peak speeds in the afternoon and lower speeds at night (Wind speeds are generally lower during the night and early morning hours (roughly 0-7) and increase during the late morning and afternoon (roughly 10-17)). However, it also shows that a simple linear relationship doesn't fully capture the complex pattern of wind speed variation throughout the day.

Second plot demonstrates a strong seasonal cycle in wind speed, with higher speeds in the winter and lower speeds in the summer. This pattern is typical in many regions due to differences in atmospheric pressure gradients and weather systems between seasons.

Third plot shows the historical data and a forecast, the forecast itself is not very informative. The linear regression model is too simplistic to capture the complexities of wind speed variation. The red line represents the forecasted wind speeds for the next 10 years. The forecast shows a nearly flat, very slightly declining trend.
It's predicting a nearly constant future, which is unlikely given the historical data. 

---

### References

Met Éireann Climate Averages: Provides annual, seasonal, and monthly average values based on high-quality datasets (Met Éireann).Retrieved November 28, 2024, from `https://www.met.ie/climate/available-data/historical-data`

Data Cleaning Techniques: Discusses methods for handling missing data and outliers (Codilime 4). Retrieved November 28, 2024, from `https://codilime.com/blog/data-cleaning-techniques/`

Pandas DateTime Functions. Retrieved November 28, 2024, from `https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html`

Plot Data Types: Overview of many common plotting commands provided by Matplotlib. Retrieved November 28, 2024, from `https://matplotlib.org/stable/plot_types/index.html`

Scikit-learn Linear Regression Documentation. Retrieved January 11, 2025, from `https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html`

Scikit-learn Train-Test Split Documentation. Retrieved January 11, 2025, from `https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.train_test_split.html`

OpenAI, 2025. ChatGPT. Version 4. Retrieved January 13, 2025, from `https://chat.openai.com`

Google DeepMind, 2025. Gemini AI. Retrieved January 13, 2025, from `https://gemini.google.com/`

---
