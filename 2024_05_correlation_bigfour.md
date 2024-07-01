## Metropolitan Area Population-Teams' Win Ratio Correlation


```python
import pandas as pd
import numpy as np
import scipy as sp
import scipy.stats as stats
import re
from matplotlib import pyplot as plt
from matplotlib.ticker import FuncFormatter
import seaborn as sns
```


```python
nhl_df=pd.read_csv("assets/nhl.csv")
nba_df=pd.read_csv("assets/nba.csv")
mlb_df=pd.read_csv("assets/mlb.csv")
nfl_df=pd.read_csv("assets/nfl.csv")
```


```python
cities=pd.read_html("assets/wikipedia_data.html")[1]
cities = cities.iloc[:-1,[0,3,5,6,7,8]]
cities.rename(columns = {'Population (2016 est.)[8]' : 'Population (2016 est.)'}, inplace=True)
cities[['NFL','NBA','NHL','MLB']] = cities[['NFL','NBA','NHL','MLB']].replace(to_replace="\[.*\]$", value="",regex=True).replace(to_replace="—", value="",regex=True)

population = cities[['Metropolitan area','Population (2016 est.)']]
population['Metropolitan area'].str.strip()
population['Population (2016 est.)'] = population['Population (2016 est.)'].astype(float)
```

### National Hockey League (NHL)


```python
citiesNHL=cities.iloc[:,[0,-1]]
citiesNHL['NHL'].replace("",np.nan, inplace=True)
citiesNHL.dropna(inplace=True)
citiesNHL['NHL']= citiesNHL['NHL'].str.split(" ").str[:]
citiesNHL_split = citiesNHL.iloc[0:2].explode('NHL').reset_index(drop=True)
citiesNHL['NHL'] = citiesNHL['NHL'].apply(lambda x: ', '.join(x))
citiesNHL['NHL'] = citiesNHL['NHL'].str.replace(',','')
citiesNHLcorrect = pd.concat([citiesNHL, citiesNHL_split]).reset_index(drop=True)
citiesNHLcorrect.drop([0,1], inplace=True)
citiesNHLcorrect.rename(columns={'NHL': 'team'}, inplace=True)

nhl = nhl_df[nhl_df['year']==2018]
# cleaning the teams column
# removing the city from the name ('city + team') (example : Florida Panthers to only Panthers)
nhl['team'] = (nhl['team'].str.split(n=1)
               .str.get(1)
               .replace(to_replace="\*$",value="",regex=True)
              )
nhl.drop([0,9,18,26],inplace=True)
nhl['team'].replace({'Bay Lightning': 'Lightning',
                     'Jersey Devils' : 'Devils',
                     'York Islanders' : 'Islanders',
                     'York Rangers' : 'Rangers',
                     'Louis Blues' : 'Blues',
                     'Jose Sharks' : 'Sharks',
                     'Angeles Kings' : 'Kings'},
                    inplace=True)
nhl = nhl[['team','W','L']]
nhl[['W','L']] = nhl[['W','L']].astype(int)

citiesNHLratio=pd.merge(citiesNHLcorrect, nhl, how = 'inner', on = 'team')
citiesNHLratio=citiesNHLratio.groupby('Metropolitan area').sum()
citiesNHLratio['Ratio'] = citiesNHLratio['W'] / (citiesNHLratio['W']+citiesNHLratio['L'])

populationratioNHL =  pd.merge(population, citiesNHLratio, how ='inner', on='Metropolitan area')
populationratioNHL = populationratioNHL.set_index('Metropolitan area')
populationratioNHL['Population (2016 est.)'] = populationratioNHL['Population (2016 est.)'].astype(float)
populationratioNHL.head()
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
      <th>Population (2016 est.)</th>
      <th>W</th>
      <th>L</th>
      <th>Ratio</th>
    </tr>
    <tr>
      <th>Metropolitan area</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>New York City</th>
      <td>20153634.0</td>
      <td>113</td>
      <td>105</td>
      <td>0.518349</td>
    </tr>
    <tr>
      <th>Los Angeles</th>
      <td>13310447.0</td>
      <td>89</td>
      <td>54</td>
      <td>0.622378</td>
    </tr>
    <tr>
      <th>San Francisco Bay Area</th>
      <td>6657982.0</td>
      <td>45</td>
      <td>27</td>
      <td>0.625000</td>
    </tr>
    <tr>
      <th>Chicago</th>
      <td>9512999.0</td>
      <td>33</td>
      <td>39</td>
      <td>0.458333</td>
    </tr>
    <tr>
      <th>Dallas–Fort Worth</th>
      <td>7233323.0</td>
      <td>42</td>
      <td>32</td>
      <td>0.567568</td>
    </tr>
  </tbody>
</table>
</div>



### National BasketBall Association (NBA)


```python
citiesNBA=cities.iloc[:,[0,-2]]
citiesNBA['NBA'].replace("",np.nan, inplace=True)
citiesNBA.dropna(inplace=True)
citiesNBA['NBA']= citiesNBA['NBA'].str.split(" ").str[:]
citiesNBA_split = citiesNBA.iloc[0:2].explode('NBA').reset_index(drop=True)
citiesNBA['NBA'] = citiesNBA['NBA'].apply(lambda x: ', '.join(x))
citiesNBA['NBA'] = citiesNBA['NBA'].str.replace(',','')
citiesNBAcorrect = pd.concat([citiesNBA, citiesNBA_split]).reset_index(drop=True)
citiesNBAcorrect.drop([0,1], inplace=True)
citiesNBAcorrect.rename(columns={'NBA': 'team'}, inplace=True)

nba = nba_df[nba_df['year'] == 2018]
nba = nba[['team', 'W', 'L']]
nba['team'] = (nba['team'].replace(to_replace="\(.*\)$",value = "",regex=True)
                  .replace(to_replace="[\*]",value = "",regex=True)
                  .str.split()
                  .str[-1]
                 )
nba['team'] = nba['team'].str.strip()
nba['team'].replace({'Blazers': 'Trail Blazers'},inplace=True)
nba[['W','L']] = nba[['W','L']].apply(pd.to_numeric, axis=1)

citiesratioNBA = pd.merge(citiesNBAcorrect, nba, how = 'inner', on = 'team')
citiesratioNBA = citiesratioNBA.groupby('Metropolitan area').sum()
populationratioNBA = pd.merge(citiesratioNBA, population, how = 'inner', on = 'Metropolitan area')
populationratioNBA.eval('Ratio = W/(W+L)',inplace =True)
#populationratioNBA.rename(columns={'Ratio' : 'NBA Ratio'}, inplace=True)
populationratioNBA = populationratioNBA.set_index('Metropolitan area')
populationratioNBA.head()
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
      <th>W</th>
      <th>L</th>
      <th>Population (2016 est.)</th>
      <th>Ratio</th>
    </tr>
    <tr>
      <th>Metropolitan area</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Atlanta</th>
      <td>24</td>
      <td>58</td>
      <td>5789700.0</td>
      <td>0.292683</td>
    </tr>
    <tr>
      <th>Boston</th>
      <td>55</td>
      <td>27</td>
      <td>4794447.0</td>
      <td>0.670732</td>
    </tr>
    <tr>
      <th>Charlotte</th>
      <td>36</td>
      <td>46</td>
      <td>2474314.0</td>
      <td>0.439024</td>
    </tr>
    <tr>
      <th>Chicago</th>
      <td>27</td>
      <td>55</td>
      <td>9512999.0</td>
      <td>0.329268</td>
    </tr>
    <tr>
      <th>Cleveland</th>
      <td>50</td>
      <td>32</td>
      <td>2055612.0</td>
      <td>0.609756</td>
    </tr>
  </tbody>
</table>
</div>



### Major League Baseball (MLB)


```python
citiesMLB=cities.iloc[:,[0,-3]]
citiesMLB['MLB'].replace("",np.nan, inplace=True)
citiesMLB.dropna(inplace=True)
citiesMLB['MLB']= citiesMLB['MLB'].str.split(" ",n=1).str[:]
citiesMLB_split = citiesMLB.iloc[0:4].explode('MLB').reset_index(drop=True)
citiesMLB['MLB'] = citiesMLB['MLB'].apply(lambda x: ', '.join(x))
citiesMLB['MLB'] = citiesMLB['MLB'].str.replace(',','')
citiesMLBcorrect = pd.concat([citiesMLB, citiesMLB_split]).reset_index(drop=True)
citiesMLBcorrect.drop([0,1,2,3], inplace=True)
citiesMLBcorrect.rename(columns={'MLB': 'team'}, inplace=True)

mlb = mlb_df[mlb_df['year']==2018]
mlb = mlb[['team', 'W', 'L']]
mlb['team'] = (mlb['team'].str.split()
               .str[-1])
mlb.loc[0,'team'] = mlb.loc[0,'team'].replace('Sox','Red Sox')
mlb.loc[8,'team'] = mlb.loc[8,'team'].replace('Sox','White Sox')
mlb['team'] = mlb['team'].replace({'Jays' : 'Blue Jays'})
mlb[['W','L']] = mlb[['W','L']].apply(pd.to_numeric, axis=1)
mlb

citiesratioMLB = pd.merge(citiesMLBcorrect, mlb, how = 'inner', on = 'team')
citiesratioMLB = citiesratioMLB.groupby('Metropolitan area').sum()

populationratioMLB = pd.merge(citiesratioMLB, population, how = 'inner', on = 'Metropolitan area')
populationratioMLB.eval('Ratio = W/(W+L)',inplace =True)
#populationratioMLB.rename(columns={'Ratio' : 'MLB Ratio'}, inplace=True)
populationratioMLB = populationratioMLB.set_index('Metropolitan area')
populationratioMLB.head()
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
      <th>W</th>
      <th>L</th>
      <th>Population (2016 est.)</th>
      <th>Ratio</th>
    </tr>
    <tr>
      <th>Metropolitan area</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Atlanta</th>
      <td>90</td>
      <td>72</td>
      <td>5789700.0</td>
      <td>0.555556</td>
    </tr>
    <tr>
      <th>Baltimore</th>
      <td>47</td>
      <td>115</td>
      <td>2798886.0</td>
      <td>0.290123</td>
    </tr>
    <tr>
      <th>Boston</th>
      <td>108</td>
      <td>54</td>
      <td>4794447.0</td>
      <td>0.666667</td>
    </tr>
    <tr>
      <th>Chicago</th>
      <td>157</td>
      <td>168</td>
      <td>9512999.0</td>
      <td>0.483077</td>
    </tr>
    <tr>
      <th>Cincinnati</th>
      <td>67</td>
      <td>95</td>
      <td>2165139.0</td>
      <td>0.413580</td>
    </tr>
  </tbody>
</table>
</div>



### National Football League (NFL)


```python
citiesNFL=cities.iloc[:,[0,2]]
citiesNFL['NFL'].replace("", np.nan, inplace=True)
citiesNFL.dropna(inplace=True)
citiesNFL['NFL']= citiesNFL['NFL'].str.split(" ",n=1).str[:]
citiesNFL_split = citiesNFL.iloc[0:3].explode('NFL').reset_index(drop=True)
citiesNFL['NFL'] = citiesNFL['NFL'].apply(lambda x: ', '.join(x))
citiesNFL['NFL'] = citiesNFL['NFL'].str.replace(',','')
citiesNFLcorrect = pd.concat([citiesNFL, citiesNFL_split]).reset_index(drop=True)
citiesNFLcorrect.drop([0,1,2], inplace=True)
citiesNFLcorrect.rename(columns={'NFL': 'team'}, inplace=True)

nfl = nfl_df[nfl_df['year']==2018]
nfl = nfl[['team', 'W', 'L']]
nfl['team'] = nfl['team'].replace('\*$|\+$','', regex=True)
nfl.drop([0,5,10,15,20,25,30,35],inplace=True)
nfl['team'] = (nfl['team'].str
               .split()
               .str[-1])
nfl[['W','L']] = nfl[['W','L']].apply(pd.to_numeric, axis=1)

citiesratioNFL = pd.merge(citiesNFLcorrect, nfl, how='inner', on= 'team')
citiesratioNFL = citiesratioNFL.groupby('Metropolitan area').sum()

populationratioNFL = pd.merge(citiesratioNFL, population, how = 'inner', on = 'Metropolitan area')
populationratioNFL.eval('Ratio = W/(W+L)',inplace=True)
populationratioNFL.set_index('Metropolitan area', inplace=True)
#populationratioNFL.rename(columns={'Ratio' : 'NFL Ratio'}, inplace=True)
populationratioNFL.head()
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
      <th>W</th>
      <th>L</th>
      <th>Population (2016 est.)</th>
      <th>Ratio</th>
    </tr>
    <tr>
      <th>Metropolitan area</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Atlanta</th>
      <td>7</td>
      <td>9</td>
      <td>5789700.0</td>
      <td>0.4375</td>
    </tr>
    <tr>
      <th>Baltimore</th>
      <td>10</td>
      <td>6</td>
      <td>2798886.0</td>
      <td>0.6250</td>
    </tr>
    <tr>
      <th>Boston</th>
      <td>11</td>
      <td>5</td>
      <td>4794447.0</td>
      <td>0.6875</td>
    </tr>
    <tr>
      <th>Buffalo</th>
      <td>6</td>
      <td>10</td>
      <td>1132804.0</td>
      <td>0.3750</td>
    </tr>
    <tr>
      <th>Charlotte</th>
      <td>7</td>
      <td>9</td>
      <td>2474314.0</td>
      <td>0.4375</td>
    </tr>
  </tbody>
</table>
</div>



### Correlations

#### NHL CORRELATION


```python
NHLcorr= stats.pearsonr(populationratioNHL['Population (2016 est.)'],populationratioNHL['Ratio'])
print('r=',NHLcorr[0])
print('P value=',NHLcorr[1])
```

    r= 0.012308996455744289
    P value= 0.9504308637909502


#### NBA CORRELATION


```python
NBAcorr= stats.pearsonr(populationratioNBA['Population (2016 est.)'],populationratioNBA['Ratio'])
print('r=',NBAcorr[0])
print('P value=',NBAcorr[1])
```

    r= -0.17657160252844614
    P value= 0.36874741604463


#### MLB CORRELATION


```python
MLBcorr= stats.pearsonr(populationratioMLB['Population (2016 est.)'],populationratioMLB['Ratio'])
print('r=',MLBcorr[0])
print('P value=',MLBcorr[1])
```

    r= 0.1505230448710485
    P value= 0.4629669779770907


#### NFL CORRELATION


```python
NFLcorr= stats.pearsonr(populationratioNFL['Population (2016 est.)'],populationratioNFL['Ratio'])
print('r=',NFLcorr[0])
print('P value=',NFLcorr[1])
```

    r= 0.004922112149349409
    P value= 0.9797833458363694



```python
# correlation coefficient and p value for plot annotation
def annotate(visual, data, x, y):
    slope, intercept, rvalue, pvalue, stderr = scipy.stats.linregress(x=data[x], y=data[y])
    visual.text(.02, .9, f'r={rvalue:.2f}, p={pvalue:.2g}',transform=visual.transAxes)

# X axis formatter function
def millions(x, pos):
    return f'{x*1e-6:.1f}M'

sns.set_context("paper")
sns.set_theme('paper')
sns.set_style('whitegrid')

fig, axes = plt.subplots(2, 2, sharex=True, sharey=True, figsize=(8.5,5))
fig.suptitle("Metropolitan Areas' Population to Win-Ratio", fontweight='bold')


nhlreg= sns.regplot(ax=axes[0,0], x=populationratioNHL['Population (2016 est.)'], y=populationratioNHL['Ratio'], color='olive')
nbareg=sns.regplot(ax=axes[0,1], x=populationratioNBA['Population (2016 est.)'], y=populationratioNBA['Ratio'], color='green')
mlbreg=sns.regplot(ax=axes[1,0], x=populationratioMLB['Population (2016 est.)'], y=populationratioMLB['Ratio'], color='darkgoldenrod')
nflreg=sns.regplot(ax=axes[1,1], x=populationratioNFL['Population (2016 est.)'], y=populationratioNFL['Ratio'], color='olivedrab')


axes[0,0].set(xlabel='', ylabel='Win Ratio')
axes[0,0].set_title('NHL', fontweight='bold')
annotate(axes[0,0], populationratioNHL, 'Population (2016 est.)', 'Ratio')

axes[0,1].set(xlabel='', ylabel='')
axes[0,1].set_title('NBA', fontweight='bold')
annotate(axes[0,1], populationratioNBA, 'Population (2016 est.)', 'Ratio')

axes[1,0].set(xlabel='Population (Millions)', ylabel='Win Ratio')
axes[1,0].set_title('MLB', fontweight='bold')
annotate(axes[1,0], populationratioMLB, 'Population (2016 est.)', 'Ratio')

axes[1,1].set(xlabel='Population (Millions)', ylabel='')
axes[1,1].set_title('NFL', fontweight='bold')
annotate(axes[1,1], populationratioNFL, 'Population (2016 est.)', 'Ratio')

# Apply the formatter
formatter = FuncFormatter(millions)
for ax in axes.flat:
    ax.xaxis.set_major_formatter(formatter)

plt.tight_layout(rect=[0, 0, 1, 0.96])
plt.show()


plt.savefig("bigfour_corr.png")
```


    
![png](output_21_0.png)
    



    <Figure size 640x480 with 0 Axes>



```python

```
