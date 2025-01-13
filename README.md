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
df = pd.read_csv("hly3723.csv", skiprows=23, low_memory=False)
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
