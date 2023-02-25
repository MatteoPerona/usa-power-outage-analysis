# Introduction
[The dataset I will be using](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks) covers major outages observed in the continental U.S. These data contain geographical location, regional climatic info, land-use characteristics, electricity consumption patterns, and economic characteristis. I will be using these data to answer a fundamental question about outages in the continental United States. 


# Do states with a high number of outages per capita have higher instances of intentional attack?
Now, this question may sound a little bit contrived at first, but it attempts to distinguish a relevant issue from the umbrella-term *power outage*. It brings us one step closer to understanding why some states have disproportionately high instances of power outage for their population. I hope this incentivises you enough to read on.


### Relevant Information About these Data
>
> Shape: 1534 rows × 54 columns
>
> Relevant Columns: U.S._STATE, CAUSE.CATEGORY
>


### DF Head with Relevant Columns
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

#### Output of these operations:
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
|                       |                |
|:----------------------|:---------------|
| OBS                   | int64          |
| YEAR                  | int64          |
| MONTH                 | float64        |
| U.S._STATE            | object         |
| POSTAL.CODE           | object         |
| NERC.REGION           | object         |
| CLIMATE.REGION        | object         |
| ANOMALY.LEVEL         | float64        |
...
| POPDEN_RURAL          | float64        |
| AREAPCT_URBAN         | float64        |
| AREAPCT_UC            | float64        |
| PCT_LAND              | float64        |
| PCT_WATER_TOT         | float64        |
| PCT_WATER_INLAND      | float64        |
| OUTAGE.START          | datetime64[ns] |
| OUTAGE.RESTORATION    | datetime64[ns] |

I went through each column and made sure that the data types were appropriate for the values in each. I did not have to make any changes to datatypes. Next, I looked through the unique values for each column using `outage.apply(lambda col: col.unique())` to find any nan placeholders like '-' and to, again, make sure that the data types for each column suited the values. I would include the output here, but it was far too long and unappealing (please visit the notebook if you are curious).

# EDA
The EDA I will cover here is abridged to focus on points relevant to the question posed above. If you are curious please check out the notebook in my repo. 

## Warm Up
To start our EDA let's do some univariate analysis on columns that aren't necessarily related to out question to warm up out analytical muscles.
<iframe src="assets/num-outages-each-year.html" width=1100 height=600 frameBorder=0></iframe>
Here we can see the distribution of outages across years from 2000-2016. It looks like 2011 had the most outages by far. It would be interesting to look into why. Additionally, it seems as though the number of outages in the years leading up to and away from 2011 seem to follow a slight bell-like shape: growing to a climax and dying off.
<iframe src="assets/num-outages-by-month.html" width=1100 height=600 frameBorder=0></iframe>
In this second image we can see the distribution of outages across months of all aforementioned years. Summer seems to have the most outages followed by winter with spring and fall seeming to have the fewest. Could this indicate that summer and winter tend to have the most extreme weather? 
<br><br>
So many interesting questions, just from two plots! Let's dive into the relevant sections of EDA now.

## Outages by State 
Here we will break down outage data for each state. 
<iframe src="assets/outage-counts-state.html" width=1100 height=600 frameBorder=0></iframe>
This plot shows the number of outages that were obseved in each state. It is not very interesting as it stands, the results speak for themselves. More outages happen in more populous states. It would be interesting to see the same plot adjusted for population and another adjusted for square footage.
<iframe src="assets/outages_per_capita_state.html" width=1100 height=600 frameBorder=0></iframe>
Ah! Much more interesting. Our first bivariate plot! It seems as though Delaware is has by far the most outages per person. More than double DC (the second highest). Why might Delaware's number of outages be so disproportionately high given their population? 
<br><br>
Side Note: Texas and California (the top two of the previous plot) are middle of the pack. This is a good example for the importance of bivariate analysis. Also, note that the second highest is DC which is very clse to Delaware. 
## Outages by Category 

## Outages by Cause Category
<iframe src="assets/outages-cause.html" width=1100 height=600 frameBorder=0></iframe>
It seems as though severe weather is to blame for most outages, but it is interesting to see that a substantial portion of outages are intentional attacks. It would be interesting to look into how what types of intentional attack and severe weather are most prevalent. Fist let's also look into the number of intentional attacks that happen per capita (mean of POPULATION of each group of observations). 
<iframe src="assets/outages-per-captia-cause.html" width=1100 height=600 frameBorder=0></iframe>
Here we can see the same relationship from above, but the difference between the top two bars and the rest seems to be more extreme. Also, the intentional attack bar reaches much closer to the severe weather bar. This may indicate a relationship between per capita effects and intentional attacks, but it's difficult to say as this is an imperfect metric. The POPULATION column measures the total population in the US state where the outage was observed, not the affected population, making this metric very shaky. 
<iframe src="assets/cause-severe-weather.html" width=1100 height=600 frameBorder=0></iframe>
It looks as though thunderstorms are responsible for the greatest number of power outages. Could this be because they are the most common storm with destructive potential? 
<iframe src="assets/cause-intentional-attack.html" width=1100 height=600 frameBorder=0></iframe>
It seems as thought most instentional attacks are vandalism. This gives less information than I would have hoped. Next, lets look at the cause breakdowns for each state using a pivot table. 

## States and Causes Together
<iframe src="assets/cause-state.html" width=1100 height=600 frameBorder=0></iframe>
This is interesting, we see the same plot from the *Outages by Location* section, but with more context. Let's look at this plot adjusted for population as we did before, but first, lets also look at the distributions of Outage Causes for each State by normalizing the rows.
<iframe src="assets/cause-state-norm.html" width=1100 height=600 frameBorder=0></iframe>
This confirms our results from above; we are mostly seeing severe weather and intentional attack as the primary causes. Now, let's scale these values to understand the data per capita. 
<iframe src="assets/cause-state-per-capita.html" width=1100 height=600 frameBorder=0></iframe>
Ah! This plot immediately gives us more context for Delaware's outlandish number of outages per capita. There are a disproportionate number of intentional attack related outages in Delaware. In fact, many of the higher ranking states tend to have high instances of intentional attack. Might states with high rates of intentional attack have more outages per person overall? Let's explore this possible trend.  
<iframe src="assets/intentional-attack-scatter-total-outages.html" width=1100 height=600 frameBorder=0></iframe>
It seems like we're onto something. Hoverver, there could still be confounders. Below I created the same graph but removed Delaware since it seems to be an outlier. 
<iframe src="assets/intentional-attack-scatter-total-no-delaware.html" width=1100 height=600 frameBorder=0></iframe>
Correlation between total outages per capidta and severe: 0.8246248841759625
We can see that the correlation still holds, albeit, to a lesser degree. 

