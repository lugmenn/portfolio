## Data transformation using Python

Full report (in Spanish) can be accessed [here](https://github.com/lugmenn/portfolio/blob/main/Fitbit_Analysis.ipynb).

Bellabeat executive team wanted to explore new ways in which the business could have a better approach with its users. They wanted to explore collected data on similar wearable smart-devices focused on health improvement. For this, [public data recorded on Fitbit devices](https://www.kaggle.com/datasets/arashnic/fitbit) was chosen as the data source consisting of 18 _csv_ files with daily and hourly data on physical activity, as well as sleep records, separated in two folders. The first one included records collected between March 12, 2016 and April 11, 2016. The second folder included data recorded between April 12, 2016 and May, 12, 2016.

Early data exploration was completed on **Microsoft Excel** to understand better the content in each of the files, produce quick JOINS with **Power Query** and summaries with the use of **pivot tables**.

Further exploration was done on **Python** which was also the tool chosen for data cleaning and transformation.

### Importing libraries and the dataset

Since data would be explored, cleaned, transformed and analyzed on Python due to its fast data processing and versatility, the required libraries and packages were imported first (including visualization tools which would be later used).


```python
import scipy.stats as stats
import numpy as np
import pandas as pd
import re
from matplotlib import pyplot as plt
import seaborn as sns
```

The data I chose to base the analysis on came from the following files: 

*   _dailyActivity_merged.csv_
*   _hourlySteps_merged.csv_
*   _sleepDay_merged.csv_


```python
# the process was similar for the rest of the files

# importing records from 03/12/2016 - 04/11/2016
daily_act1 = pd.read_csv('/content/Fitbit analysis/dailyActivity_merged1.csv')

# importing records from 04/12/2016 - 05/11/2016
daily_act2 = pd.read_csv('/content/Fitbit analysis/dailyActivity_merged.csv')

# merging both data sources into a single dataframe
dailyact = pd.concat([daily_act1, daily_act2], keys= [1, 2])
dailyact.info()
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    /tmp/ipykernel_227/3611578210.py in <cell line: 4>()
          2 
          3 # importing records from 03/12/2016 - 04/11/2016
    ----> 4 daily_act1 = pd.read_csv('/content/Fitbit analysis/dailyActivity_merged1.csv')
          5 
          6 # importing records from 04/12/2016 - 05/11/2016


    NameError: name 'pd' is not defined


### Data Processing

#### Duplicate elimination

How many different users were on each file?


```python
print('Users in dailyact:', len(dailyact['Id'].unique()))
print('Users in hourlysteps:', len(hourlysteps['Id'].unique()))
print('Users in sleep:', len(sleep['Id'].unique()))
```

Finding and eliminating duplicate rows in the dataset. Here, I show the code used in the _hourlysteps_ dataframe (a similar process was applied on the rest of the dataframes).


```python
# how many duplicate rows are there?
hourlysteps_duplicate = pd.DataFrame(hourlysteps.duplicated())
hourlysteps_duplicate.iloc[:].value_counts()
```


```python
# duplicate rows were dropped from the dataframe
hourlysteps.drop_duplicates(inplace=True)

# the remaining records were double-checked (175 should have been eliminated)
hourlysteps.info()
```


```python
# resetting the index on the dataframes and deleting any rows with missing values
hourlysteps = hourlysteps.droplevel(0)
hourlysteps.dropna(inplace=True)
hourlysteps.head()
```

### Filtering the sample

I wanted to know if all the users had the same number of records on the dataframes. For a better analysis I decided to leave out those users that did not wore the device for at least one month. This removed 2 users, leaving a sample of 33 people.


```python
# finding how many days each user used the device
records_by_user = dailyact.groupby('Id')['ActivityDate'].count().sort_values().to_frame('Days of use')
records_by_user.reset_index(inplace=True)
records_by_user.head()
```


```python
# list with the users to keep
users_to_keep = list(records_by_user['Id'][2:])

# filtering out the dataframe
dailyact_filter = dailyact.loc[dailyact['Id'].isin(users_to_keep)]
dailyact_filter.reset_index(inplace=True)
print('There are', len(dailyact_filter['Id'].unique()), 'users in the "dailyact" dataframe')
```

#### Cleaning the column names

For a quicker code writing, the column names were transformed into lowercase.I also used _strip()_ to eliminate any possible extra space on the column names.


```python
cols = dailyact_filter.columns
dailyact_filter.columns = [x.lower().strip() for x in cols]
```

### Formatting datetime columns

Each dataframe included a column with the date or date AND time of the record. however, al values were a string data type. To manipulate dates easily I had to convert them into a datetime date type.


```python
dailyact_filter['activitydate'] = pd.to_datetime(dailyact_filter.activitydate)
print(dailyact_filter['activitydate'])
```


```python
# then, the data was sorted by user and date of record
dailyact_clean = dailyact_filter.sort_values(by = ['id', 'activitydate'], ascending = [True, True], na_position = 'first', ignore_index = True)
dailyact_clean.drop('index',inplace=True, axis=1)
dailyact_clean.head()
```

The date column in the _hourlysteps_ dataframe included both the date and the time of record. Since the analysis would require to group data by each hour, I decided to split it into two columns. The hour record was in a 12 hour format, which, for a better data understanding, I converted into a 24 hour format.


```python
hourlysteps['activityhour'] = pd.to_datetime(hourlysteps['activityhour'], format='%m/%d/%Y %I:%M:%S %p')
hourlysteps['activitydate'] = hourlysteps['activityhour'].dt.date
hourlysteps['activitytime'] = hourlysteps['activityhour'].dt.time
hourlysteps['activitydate'] = pd.to_datetime(hourlysteps['activitydate'], format='%m/%d/%Y')
hourlysteps = hourlysteps[['id', 'activitydate', 'activitytime', 'steptotal']]
hourlysteps.head()
```

Finally, the _sleep_ dataframe also had the time in the date column. However it had "12:00:00 AM" in all records, so it was eliminated. Additionally, the unit used in the total sleep time and total time in bed was "minutes". To have a better understanding on the total time amount, they were converted into hours, renaming the column names in the process.


```python
sleep['sleepday'] = pd.to_datetime(sleep['sleepday'], format='%m/%d/%Y %I:%M:%S %p')
sleep['sleep_day'] = sleep['sleepday'].dt.date
sleep['sleephour'] = sleep['sleepday'].dt.time
sleep['sleepday'] = pd.to_datetime(sleep['sleep_day'], format='%m/%d/%Y')

sleep = sleep[['id', 'sleepday', 'totalsleeprecords', 'totalminutesasleep', 'totaltimeinbed']]
sleep['totalminutesasleep'] = sleep['totalminutesasleep']/60
sleep['totaltimeinbed'] = sleep['totaltimeinbed']/60

sleep.rename(columns={'totalminutesasleep':'hoursasleep', 'totaltimeinbed':'hoursinbed' }, inplace=True)
sleep.head()
```

#### Merging the dataframes

The analysis required to evaluate sleeptime and activity-time during each day. For this reason, the dataframes _sleep_ and _dailyact_ were merged into a single dataframe.


```python
# I changed the column name that was used for merging the dfs
sleep.rename(columns={'sleepday':'activitydate'}, inplace=True)

# inner join
act_sleep = pd.merge(dailyact_clean, sleep,
                     how='inner', on= ['id', 'activitydate'])
```

This allowed to have, by the end of the process, four dataframes ready to be analyzed. Keep reading more [here](https://lugmenn.github.io/portfolio/2024_06_bellabeat_analysis.html)

1.   dailyact_clean
2.   hourlysteps
3.   act_sleep
4.   sleep


```python

```
