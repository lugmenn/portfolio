## Finding the correlation between different North-American metropolitan areas’ population and their Big Four teams’ Win/Lose ratio


The Big Four refers to the major professional sports leagues across North America (USA and Canada). These leagues include NFL, NBA, MBL and NHL, which contain 30 to 32 teams each, and are mostly based on the top populated metropolitan areas in both countries.

This analysis consisted of finding through the Pearson correlation how closely related are the Metropolitan Area population size and the Win/Lose ratio of the teams from each league, meaning the proportion of the won games against the total number of won AND lost games. Is it reasonable to expect a team to be more likely to win if the city it is based on is big, is the opposite to be expected or is there no relationship at all?

In this page, the full code for the NFL correlation analysis is included. To check the code for the NBA, MBL AND NHL, click here.

### Importing the libraries and data

Data was processed and analyzed with Numpy, Pandas, Re and SciPy. Then, Matplotlib and Seaborn were used for visualization.


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re
from matplotlib import pyplot as plt
import seaborn as sns
```

Source of data includes a cached table from Wikipedia with the 2016 population estimates in the Metropolitan Areas and a list of each Metro Area's teams from each league, as well as stats for all team from each league ranging from 2014 to 2018. The analysis was run on the results from the 2018 games.


```python
nhl_df=pd.read_csv("assets/nhl.csv")
cities=pd.read_html("assets/wikipedia_data.html")[1]
cities = cities.iloc[:-1,[0,3,5,6,7,8]]
cities.rename(columns = {'Population (2016 est.)[8]' : 'Population (2016 est.)'}, inplace=True)
cities[['NFL','NBA','NHL','MLB']] = cities[['NFL','NBA','NHL','MLB']].replace(to_replace="\[.*\]$", value="",regex=True).replace(to_replace="—", value="",regex=True)
cities.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Metropolitan area</th>
      <th>Population (2016 est.)</th>
      <th>NFL</th>
      <th>MLB</th>
      <th>NBA</th>
      <th>NHL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>New York City</td>
      <td>20153634</td>
      <td>Giants Jets</td>
      <td>Yankees Mets</td>
      <td>Knicks Nets</td>
      <td>Rangers Islanders Devils</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Los Angeles</td>
      <td>13310447</td>
      <td>Rams Chargers</td>
      <td>Dodgers Angels</td>
      <td>Lakers Clippers</td>
      <td>Kings Ducks</td>
    </tr>
    <tr>
      <th>2</th>
      <td>San Francisco Bay Area</td>
      <td>6657982</td>
      <td>49ers Raiders</td>
      <td>Giants Athletics</td>
      <td>Warriors</td>
      <td>Sharks</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chicago</td>
      <td>9512999</td>
      <td>Bears</td>
      <td>Cubs White Sox</td>
      <td>Bulls</td>
      <td>Blackhawks</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dallas–Fort Worth</td>
      <td>7233323</td>
      <td>Cowboys</td>
      <td>Rangers</td>
      <td>Mavericks</td>
      <td>Stars</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
