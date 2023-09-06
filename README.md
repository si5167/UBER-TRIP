# UBER-TRIP
import numpy as np
import pandas as pd
import os

import matplotlib.pyplot as plt
import datashader as ds
import colorcet as cc
import folium

from collections import defaultdict, OrderedDict
from folium.plugins import HeatMapWithTime
from datetime import datetime
%matplotlib inline
plt.style.use('ggplot')
plt.rcParams['figure.figsize'] = (15,10)
#Load the datasets

df_apr14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-apr14.csv")
df_may14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-may14.csv")
df_jun14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-jun14.csv")
df_jul14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-jul14.csv")
df_aug14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-aug14.csv")
df_sep14=pd.read_csv("C:/Users/siddhika/OneDrive/Desktop/uber-raw-data-sep14.csv")

#Merge the dataframes into one

df = df_apr14.append([df_may14,df_jun14,df_jul14,df_aug14,df_sep14], ignore_index=True)
df.head()
	Date/Time	Lat	Lon	Base
0	4/1/2014 0:11:00	40.7690	-73.9549	B02512
1	4/1/2014 0:17:00	40.7267	-74.0345	B02512
2	4/1/2014 0:21:00	40.7316	-73.9873	B02512
3	4/1/2014 0:28:00	40.7588	-73.9776	B02512
4	4/1/2014 0:33:00	40.7594	-73.9722	B02512


# Disaggregate 'Date/Time' column
df['Date/Time'] = pd.to_datetime(df['Date/Time'])

df['Month'] = df['Date/Time'].dt.month
df['Day'] = df['Date/Time'].dt.day
df['Time'] = df['Date/Time'].dt.time

df.head()
	Date/Time	Lat	Lon	Base	Month	Day	Time
0	2014-04-01 00:11:00	40.7690	-73.9549	B02512	4	1	00:11:00
1	2014-04-01 00:17:00	40.7267	-74.0345	B02512	4	1	00:17:00
2	2014-04-01 00:21:00	40.7316	-73.9873	B02512	4	1	00:21:00
3	2014-04-01 00:28:00	40.7588	-73.9776	B02512	4	1	00:28:00
4	2014-04-01 00:33:00	40.7594	-73.9722	B02512	4	1	00:33:00


# Day of the week
df['Weekday'] = df['Date/Time'].dt.dayofweek

# Hour
df['Hour'] = df['Date/Time'].dt.hour

df.head()

	Date/Time	Lat	Lon	Base	Month	Day	Time	Weekday	Hour
0	2014-04-01 00:11:00	40.7690	-73.9549	B02512	4	1	00:11:00	1	0
1	2014-04-01 00:17:00	40.7267	-74.0345	B02512	4	1	00:17:00	1	0
2	2014-04-01 00:21:00	40.7316	-73.9873	B02512	4	1	00:21:00	1	0
3	2014-04-01 00:28:00	40.7588	-73.9776	B02512	4	1	00:28:00	1	0
4	2014-04-01 00:33:00	40.7594	-73.9722	B02512	4	1	00:33:00	1	0

# Map month
month_map = {
    4: 'April',
    5: 'May',
    6: 'June', 
    7: 'July',
    8: 'August',
    9: 'September'
}
df['Month'] = df['Month'].replace(month_map)
# Map weekday
weekday_map = {
    0: 'Monday',
    1: 'Tuesday',
    2: 'Wednesday',
    3: 'Thursday', 
    4: 'Friday', 
    5: 'Saturday',
    6: 'Sunday'
}
df['Weekday'] = df['Weekday'].replace(weekday_map)

final_columns = [
    'Base', 'Lat', 'Lon', 'Month', 
    'Day', 'Weekday', 'Hour']
df = df[final_columns]
df.head()
	Base	Lat	Lon	Month	Day	Weekday	Hour
0	B02512	40.7690	-73.9549	April	1	Tuesday	0
1	B02512	40.7267	-74.0345	April	1	Tuesday	0
2	B02512	40.7316	-73.9873	April	1	Tuesday	0
3	B02512	40.7588	-73.9776	April	1	Tuesday	0
4	B02512	40.7594	-73.9722	April	1	Tuesday	0
num_missing = df.isnull().sum().sum()
print('Number of missing values: {}'.format(num_missing))
