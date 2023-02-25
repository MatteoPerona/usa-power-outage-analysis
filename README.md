# Analyzing Power Outages in The Continental US
### Matteo Perona
[The dataset I will be using](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks) covers major outages observed in the continental U.S. These data contain geographical location, regional climatic info, land-use characteristics, electricity consumption patterns, and economic characteristis. I will be using these data to answer a fundamental question about outages in the continental United States. 


# Do states with a high number of outages per capita have higher instances of intentional attack?
Now, this question may sound a little bit contrived at first, but it attempts to distinguish a relevant issue from the umbrella-term *power outage*. It brings us one step closer to understanding why some states have disproportionately high instances of power outage for their population. I hope this incentivises you enough to read on.


### Relevant Information About these Data
>
> Shape: 1534 rows × 54 columns
>
> Relevant Columns: U.S._STATE, CAUSE.CATEGORY
>


### DF Head with Relevant Columns <br>

|   YEAR |   MONTH | U.S._STATE   | CAUSE.CATEGORY     |
|-------:|--------:|:-------------|:-------------------|
|   2011 |       7 | Minnesota    | severe weather     |
|   2014 |       5 | Minnesota    | intentional attack |
|   2010 |      10 | Minnesota    | severe weather     |
|   2012 |       6 | Minnesota    | severe weather     |
|   2015 |       7 | Minnesota    | severe weather     |


### Relevant Python Packages 
```py
import pandas as pd
import numpy as np
import os

import plotly.express as px
pd.options.plotting.backend = 'plotly'
```
>
> I opted for plotly as pandas' plotting backend over matplotlib since it has a more modern look and it's functions are better suited to this project data visualization needs.
>  


# Data Cleaning and EDA
Fortunately, data cleaning was relatively simple for this project. 

## Step 1. Pruning by Hand 
I downloaded all the data from [this link](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks) and opened it with numbers on my Mac. Then, I simply removed the first column, which contained unnecessary descriptive information about the dataset, and the first three rows, which contained no data with the first column removed. Additionally, I removed the column beneath the row containing column names (column 7 in the unchanged file); it contained units for columns with quantitative data. Once I was done pruning the dataset by hand I exported it as a CSV file and saved it to my project folder. 


## Setp 2. Reformatting Outage Start and Restoration Dates and Times
I first imported the data like so:
```python
outage_fp = os.path.join('data', 'outage.csv')
outage = pd.read_csv(outage_fp)
```

#### The relevant columns to modify were: <br>

| OUTAGE.START.DATE         | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE    | OUTAGE.RESTORATION.TIME   |
|:--------------------------|:--------------------|:---------------------------|:--------------------------|
| Friday, July 01, 2011     | 5:00:00 PM          | Sunday, July 03, 2011      | 8:00:00 PM                |
| Sunday, May 11, 2014      | 6:38:00 PM          | Sunday, May 11, 2014       | 6:39:00 PM                |
| Tuesday, October 26, 2010 | 8:00:00 PM          | Thursday, October 28, 2010 | 10:00:00 PM               |
| Tuesday, June 19, 2012    | 4:30:00 AM          | Wednesday, June 20, 2012   | 11:00:00 PM               |
| Saturday, July 18, 2015   | 2:00:00 AM          | Sunday, July 19, 2015      | 7:00:00 AM                |

I wanted to combine the outage start and restoration dates with their respective times and convert them to timestamps assigning these new series to two columns -- 'OUTAGE.START' and 'OUTAGE.RESTORATION' respectively -- while dropping the old ones. The process went as follows.<br><br>


#### First, I added the series' date and time strings together:
``` python
(outage['OUTAGE.START.DATE'] + ", " + outage['OUTAGE.START.TIME'])
```

#### Next, I applied a pd.Timestamp() to each element of the resulting series to convert each string to a timestamp:
```python
.apply(lambda x: pd.Timestamp(x))
```


#### Finally, we assign the new steries to new columns and drop the old columns from out original dataframe:
```python
outage['OUTAGE.START'] = ...
outage['OUTAGE.RESTORATION'] = ...
outage = outage.drop(columns=['OUTAGE.START.DATE', 'OUTAGE.START.TIME', 'OUTAGE.RESTORATION.DATE', 'OUTAGE.RESTORATION.TIME'])
```


#### Overall, the operations follow like so:
```python
outage['OUTAGE.START'] = (outage['OUTAGE.START.DATE'] + ", " + outage['OUTAGE.START.TIME']) \
    .apply(lambda x: pd.Timestamp(x))

outage['OUTAGE.RESTORATION'] = (outage['OUTAGE.RESTORATION.DATE'] + ", " + outage['OUTAGE.RESTORATION.TIME']) \
    .apply(lambda x: pd.Timestamp(x))

outage = outage.drop(columns=['OUTAGE.START.DATE', 'OUTAGE.START.TIME', 'OUTAGE.RESTORATION.DATE', 'OUTAGE.RESTORATION.TIME'])
```

#### Output of these operations: <br>

|   OUTAGE.RESTORATION |   OUTAGE.START |
|---------------------:|---------------:|
|          1.30972e+18 |    1.30954e+18 |
|          1.39983e+18 |    1.39983e+18 |
|          1.2883e+18  |    1.28812e+18 |
|          1.34023e+18 |    1.34008e+18 |
|          1.43729e+18 |    1.43718e+18 |

The values above are of type datetime64[ns].





## Step 3. Checking Column Dtypes and Assessing Null Values
This final step of cleaning was the most painless because the data were already formatted very cleanly. I first looked through each column's datatype using `outage.dtypes`: <br>

|                       | 0              |
|:----------------------|:---------------|
| OBS                   | int64          |
| YEAR                  | int64          |
| MONTH                 | float64        |
| U.S._STATE            | object         |
...
| PCT_WATER_TOT         | float64        |
| PCT_WATER_INLAND      | float64        |
| OUTAGE.START          | datetime64[ns] |
| OUTAGE.RESTORATION    | datetime64[ns] |

I went through each column and made sure that the data types were appropriate for the values in each. I did not have to make any changes to datatypes. Next, I looked through the unique values for each column using `outage.apply(lambda col: col.unique())` to find any nan placeholders like '-' and to, again, make sure that the data types for each column suited the values. I would include the output here, but it was far too long and unappealing (please visit the notebook if you are curious).

# EDA
The EDA I will cover here is abridged to focus on points relevant to the question posed above. If you are curious please check out the notebook in my repo. 

## Warm Up
To start our EDA let's do some univariate analysis on columns that aren't necessarily related to out question to warm up out analytical muscles.
<iframe src="assets/num-outages-each-year.html" width=800 height=600 frameBorder=0></iframe>
Here we can see the distribution of outages across years from 2000-2016. It looks like 2011 had the most outages by far. It would be interesting to look into why. Additionally, it seems as though the number of outages in the years leading up to and away from 2011 seem to follow a slight bell-like shape: growing to a climax and dying off.
<iframe src="assets/num-outages-by-month.html" width=800 height=600 frameBorder=0></iframe>
In this second image we can see the distribution of outages across months of all aforementioned years. Summer seems to have the most outages followed by winter with spring and fall seeming to have the fewest. Could this indicate that summer and winter tend to have the most extreme weather? 
<br><br>
So many interesting questions, just from two plots! Let's dive into the relevant sections of EDA now.

## Outages by State 
Here we will break down outage data for each state. 
<iframe src="assets/outage-counts-state.html" width=800 height=600 frameBorder=0></iframe>
This plot shows the number of outages that were obseved in each state. It is not very interesting as it stands, the results speak for themselves. More outages happen in more populous states. It would be interesting to see the same plot adjusted for population and another adjusted for square footage.
<iframe src="assets/outages_per_capita_state.html" width=800 height=600 frameBorder=0></iframe>
Ah! Much more interesting. Our first bivariate plot! It seems as though Delaware is has by far the most outages per person. More than double DC (the second highest). Why might Delaware's number of outages be so disproportionately high given their population? 
<br><br>
Side Note: Texas and California (the top two of the previous plot) are middle of the pack. This is a good example for the importance of bivariate analysis. Also, note that the second highest is DC which is very clse to Delaware. 


## Outages by Cause Category
<iframe src="assets/outages-cause.html" width=800 height=600 frameBorder=0></iframe>
It seems as though severe weather is to blame for most outages, but it is interesting to see that a substantial portion of outages are intentional attacks. It would be interesting to look into how what types of intentional attack and severe weather are most prevalent. Fist let's also look into the number of intentional attacks that happen per capita (mean of POPULATION of each group of observations). 
<iframe src="assets/outages-per-captia-cause.html" width=800 height=600 frameBorder=0></iframe>
Here we can see the same relationship from above, but the difference between the top two bars and the rest seems to be more extreme. Also, the intentional attack bar reaches much closer to the severe weather bar. This may indicate a relationship between per capita effects and intentional attacks, but it's difficult to say as this is an imperfect metric. The POPULATION column measures the total population in the US state where the outage was observed, not the affected population, making this metric very shaky. 
<iframe src="assets/cause-severe-weather.html" width=800 height=600 frameBorder=0></iframe>
It looks as though thunderstorms are responsible for the greatest number of power outages. Could this be because they are the most common storm with destructive potential? 
<iframe src="assets/cause-intentional-attack.html" width=800 height=600 frameBorder=0></iframe>
It seems as thought most instentional attacks are vandalism. This gives less information than I would have hoped. Next, lets look at the cause breakdowns for each state using a pivot table. 

## States and Causes Together
For this section we need to create a pivot table. 
```py
cause_by_state = pd.pivot_table(outage, columns=['CAUSE.CATEGORY'], index=['U.S._STATE'], values='OBS', aggfunc='count')
cause_by_state = cause_by_state.assign(total=cause_by_state.sum(axis=1))
cause_by_state = cause_by_state.sort_values(by='total').drop(columns=['total'])
```
#### Output dataframe head:<br>

| U.S._STATE   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:-------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Alaska       |                   1 |                       0 |                    0 |           0 |               0 |                0 |                               0 |
| South Dakota |                   0 |                       0 |                    0 |           2 |               0 |                0 |                               0 |
| North Dakota |                   0 |                       1 |                    0 |           0 |               1 |                0 |                               0 |
| Montana      |                   0 |                       0 |                    1 |           2 |               0 |                0 |                               0 |
| Mississippi  |                   0 |                       0 |                    3 |           0 |               0 |                0 |                               1 |


<iframe src="assets/cause-state.html" width=800 height=600 frameBorder=0></iframe>
This is interesting, we see the same plot from the *Outages by Location* section, but with more context. Let's look at this plot adjusted for population as we did before, but first, lets also look at the distributions of Outage Causes for each State by normalizing the rows.
<iframe src="assets/cause-state-norm.html" width=800 height=600 frameBorder=0></iframe>
This confirms our results from above; we are mostly seeing severe weather and intentional attack as the primary causes. Now, let's scale these values to understand the data per capita. 
<iframe src="assets/cause-state-per-capita.html" width=800 height=600 frameBorder=0></iframe>
Ah! This plot immediately gives us more context for Delaware's outlandish number of outages per capita. There are a disproportionate number of intentional attack related outages in Delaware. In fact, many of the higher ranking states tend to have high instances of intentional attack. Might states with high rates of intentional attack have more outages per person overall? Let's explore this possible trend.  
<iframe src="assets/intentional-attack-scatter-total-outages.html" width=800 height=600 frameBorder=0></iframe>
Correlation between total outages per capidta and severe: 0.9542374010674626 <br>
It seems like we're onto something. Hoverver, there could still be confounders. Below I created the same graph but removed Delaware since it seems to be an outlier. 
<iframe src="assets/intentional-attack-scatter-total-no-delaware.html" width=800 height=600 frameBorder=0></iframe>
Correlation between total outages per capidta and severe: 0.8246248841759625 <br>
We can see that the correlation still holds, albeit, to a lesser degree. 


# Assessment of Missingness
## Visualize Missingness
<iframe src="assets/missing-values-by-column.html" width=800 height=600 frameBorder=0></iframe>
The plot above shows the number of missing values in each column in outages which has missing values. 

## NMAR Analysis
One column which jumps out as being NMAR (not missing at random) is the HURRICANE.NAMES column since when collecting the data if the outage did not occur as a result of a hurricane there will not be a name to report. In this way thet missingness is dependent of the column itself and also on columns like CAUSE.CATEGORY.DETAILS of CAUSE.CATEGORY which both contain information about the cause of the outage. 

## Missingness Dependency
In this section we will be showing that the CAUSE.CATEGORY.DETAILS column is MAR dependent on the CAUSE.CATEGORY column using permutation tests. We will also *attempt* and fail at finding a column that it is independent from using permutation tests.
### Testing Dependent Case 
First we will attempt to show depencency between CAUSE.CATEGORY.DETAILS.

#### Hypothesis
Null Hypothesis: The distribution of CAUSE.CATEGORY.DETAIL is the same when CAUSE.CATEGORY name is missing and when it is not missing<br>
Alt Hypothesis: The distribution of CAUSE.CATEGORY.DETAIL is different when CAUSE.CATEGORY name is missing as opposed to when it is not missing

#### Code for generating the observed distributions 
```py
# Variables for columns we are testing to enable hot switching
indep_var = 'CAUSE.CATEGORY'
q_var = 'CAUSE.CATEGORY.DETAIL'

# generate pivot table to display missingness of q_var with relation to indep_var
df_indep = (
    outage
    .assign(missing = outage[q_var].isna())
    .pivot_table(index=indep_var, columns='missing', aggfunc='size')
).fillna(0)

# normalize the df
df_indep = df_indep / df_indep.sum()
# plot the distribution
df_indep.plot(
    kind='barh', 
    title='Observed Distribution of Hurricane Names Conditional on Cause Category Detail',
    barmode='group',
    height=1100  
    )
```
#### Output DF<br>

| CAUSE.CATEGORY                |     False |      True |
|:------------------------------|----------:|----------:|
| equipment failure             | 0.0451552 | 0.0254777 |
| fuel supply emergency         | 0.0301035 | 0.0403397 |
| intentional attack            | 0.348071  | 0.101911  |
| islanding                     | 0         | 0.0976645 |
| public appeal                 | 0         | 0.146497  |
| severe weather                | 0.541863  | 0.397028  |
| system operability disruption | 0.0348071 | 0.191083  |


#### Output Plot
<iframe src="assets/observed-dist-dependent.html" width=800 height=500 frameBorder=0></iframe>

#### Calculate Observed TVD
```py
observed_tvd = df_indep.diff(axis=1).iloc[:, -1].abs().sum() / 2
observed_tvd
```
**Observed TVD**: 0.41067323382726845

#### Simulation
```py
# number of times to repeat permutations and calculate TVD
n_repetitions = 500
shuffled = outage.copy()

tvds = []
for _ in range(n_repetitions):
    # Create permutation
    shuffled[indep_var] = np.random.permutation(shuffled[indep_var])
    
    # Computing and storing the TVD.
    pivoted = (
        shuffled
        .assign(missing = shuffled[q_var].isna())
        .pivot_table(index=indep_var, columns='missing', aggfunc='size')
        .apply(lambda x: x / x.sum())
    )
    
    tvd = pivoted.diff(axis=1).iloc[:, -1].abs().sum() / 2
    tvds.append(tvd)
```
#### Plotting Empirical Distribution of Calculated TVDs
<iframe src="assets/empirical-distribution-TVD-dependent.html" width=800 height=600 frameBorder=0></iframe>

#### Calculating p-value
```py
np.mean(np.array(tvds) >= observed_tvd)
```
**Out:** 0.0

#### Conclusions
P-val is less than our significance level of 0.05, so we reject the null hypothesis. The CAUSE.CATEGORY.DETAILS column is likely to be dependent on CAUSE.CATEGORY. 

### Testing Independent Case
>
>I could not find a single independent case for CAUSE.CATEGORY.DETAILS with any other column in the outage dataframe.
> 

# Hypothesis Testing
## Hypothesis 
Null: number of outages by state per capita comes from the same distribution as <br>
Alt: States whose most prevalent cause of outages is intentional attack have more outages per capita <br>
Significance level: 0.05

## Generate Observed Distributions
### Code 
```py
# number of outages by state per capita
X = outage.groupby(by='U.S._STATE').count().OBS / outage.groupby(by='U.S._STATE').mean().POPULATION

# proportion of outages that are caused by intentional attack by state 
cause_by_state = pd.pivot_table(outage, columns=['CAUSE.CATEGORY'], index=['U.S._STATE'], values='OBS', aggfunc='count').fillna(0)
Y = cause_by_state['severe weather'] / cause_by_state.sum(axis=1)

# put X and Y into a df
df = pd.DataFrame().assign(out_per_cap=X, prop_attack=Y)
# normalize the df
df = df / df.sum(axis=0)

# plot the distributions
fig = df.plot(kind='barh', height=1000, barmode='group', title='Observed Distributions of Outages by State Per Capita and Prop Outages Caused by Intentional Attack')
fig.write_html('./assets/hyp-test-observed.html', include_plotlyjs='cdn')
fig 
```
### Out Graph
<iframe src="assets/hyp-test-observed.html" width=800 height=600 frameBorder=0></iframe>

### Out Table<br>

| U.S._STATE   |   out_per_cap |   prop_attack |
|:-------------|--------------:|--------------:|
| Alabama      |    0.00414147 |    0.0367439  |
| Alaska       |    0.0051098  |    0          |
| Arizona      |    0.0142132  |    0.00629895 |
| Arkansas     |    0.0274285  |    0.0176371  |
| California   |    0.0181408  |    0.0146976  |

### Observed TVD
```py
observed_tvd = df.diff(axis=1).iloc[:, -1].abs().sum() / 2
```
**Out:**: 0.48123557430777736

## Permutations
### Code
```py
n_repetitions = 500
shuffled = outage.copy()

tvds = []
for _ in range(n_repetitions):
    shuffled['U.S._STATE'] = np.random.permutation(shuffled['U.S._STATE'])

    # number of outages by state per capita
    X = shuffled.groupby(by='U.S._STATE').count().OBS / shuffled.groupby(by='U.S._STATE').mean().POPULATION

    # proportion of outages that are caused by intentional attack by state 
    cause_by_state = pd.pivot_table(shuffled, columns=['CAUSE.CATEGORY'], index=['U.S._STATE'], values='OBS', aggfunc='count').fillna(0)
    Y = cause_by_state['severe weather'] / cause_by_state.sum(axis=1)

    # put X and Y into a df
    df = pd.DataFrame().assign(out_per_cap=X, prop_attack=Y)
    # normalize the df
    df = df / df.sum(axis=0)
    
    tvd = df.diff(axis=1).iloc[:, -1].abs().sum() / 2
    tvds.append(tvd)
```

### Empirical Distribution of TVD
<iframe src="assets/empirical-dist-tvd-hyp-test.html" width=800 height=600 frameBorder=0></iframe>

### p-value 
```py
np.mean(np.array(tvds) >= observed_tvd)
```
**Out: **0.0


## Conclusion 
Our p-value 0.0 is less than our significance level of 0.05, so we reject the null hypothesis. We can conclude that the number of outages per capita and the proportion of outages caused by intentional attack in each state are likely drawn from different distributions. What does this tell us about out original question?
>
>Do states with a high number of outages per capita have higher instances of intentional attack?
>

While we can say nothing for certain, the permutation tests above indicate that there is likely a relationship between the number of outages per capita and seeing higher instances of intentional attack. 

