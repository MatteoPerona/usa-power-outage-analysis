# USA Power Outage Analysis
[The dataset I will be using](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks) covers major outages observed in the continental U.S. These data contain geographical location, regional climatic info, land-use characteristics, electricity consumption patterns, and economic characteristis. I will be using these data to answer a fundamental question about outages in the continental United States. 
# Do states with a high number of outages per capita have higher instances of intentional attack?
Now, this question may sound a little bit contrived at first, but it attempts to distinguish a relevant issue from the umbrella-term *power outage*. It brings us one step closer to understanding why some states have disproportionately high instances of power outage for their population. I hope this incentivises you enough to read on.
### Relevant Information About these Data
Shape: 1534 rows × 54 columns <br>
Relevant Columns: U.S._STATE, CAUSE.CATEGORY <br>
### DF Head with Relevant Columns
|   YEAR |   MONTH | U.S._STATE   | CAUSE.CATEGORY     |<br>
|-------:|--------:|:-------------|:-------------------|<br>
|   2011 |       7 | Minnesota    | severe weather     |<br>
|   2014 |       5 | Minnesota    | intentional attack |<br>
|   2010 |      10 | Minnesota    | severe weather     |<br>
|   2012 |       6 | Minnesota    | severe weather     |<br>
|   2015 |       7 | Minnesota    | severe weather     |<br>
### Relevant Python Packages 
`
import pandas as pd
import numpy as np
import os

import plotly.express as px
pd.options.plotting.backend = 'plotly'
`
>
> I opted for plotly as pandas' plotting backend over matplotlib since it has a more modern look and it's functions are better suited to this project data visualization needs.
>  

# Data Cleaning
Fortunately, data cleaning was relatively simple for this project. 
## Step 1. Pruning by Hand 
I downloaded all the data from [this link](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks) and opened it with numbers on my Mac. Then, I simply removed the first column, which contained unnecessary descriptive information about the dataset, and the first three rows, which contained no data with the first column removed. Additionally, I removed the column beneath the row containing column names (column 7 in the unchanged file); it contained units for columns with quantitative data. Once I was done pruning the dataset by hand I exported it as a CSV file and saved it to my project folder. 
## Setp 2. Reformatting Outage Start and Restoration Dates and Times
I first imported the data like so:
`
outage_fp = os.path.join('data', 'outage.csv')
outage = pd.read_csv(outage_fp)
`
The relevant columns to modify were: <br>
| OUTAGE.START.DATE         | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE    | OUTAGE.RESTORATION.TIME   |<br>
|:--------------------------|:--------------------|:---------------------------|:--------------------------|<br>
| Friday, July 01, 2011     | 5:00:00 PM          | Sunday, July 03, 2011      | 8:00:00 PM                |<br>
| Sunday, May 11, 2014      | 6:38:00 PM          | Sunday, May 11, 2014       | 6:39:00 PM                |<br>
| Tuesday, October 26, 2010 | 8:00:00 PM          | Thursday, October 28, 2010 | 10:00:00 PM               |<br>
| Tuesday, June 19, 2012    | 4:30:00 AM          | Wednesday, June 20, 2012   | 11:00:00 PM               |<br>
| Saturday, July 18, 2015   | 2:00:00 AM          | Sunday, July 19, 2015      | 7:00:00 AM                |<br>

I wanted to combine the outage start and restoration dates with their respective times and convert them to timestamps assigning these new series to two columns -- 'OUTAGE.START' and 'OUTAGE.RESTORATION' respectively -- while dropping the old ones. The process went as follows.
First, I added the series' date and time strings together:
`
(outage['OUTAGE.START.DATE'] + ", " + outage['OUTAGE.START.TIME'])
`
Next, I applied a pd.Timestamp() to each element of the resulting series to convert each string to a timestamp:
`
.apply(lambda x: pd.Timestamp(x))
`
Finally, we assign the new steries to new columns and drop the old columns from out original dataframe:
`
outage['OUTAGE.START'] = ...
outage['OUTAGE.RESTORATION'] = ...
outage = outage.drop(columns=['OUTAGE.START.DATE', 'OUTAGE.START.TIME', 'OUTAGE.RESTORATION.DATE', 'OUTAGE.RESTORATION.TIME'])
`
Overall, the operations follow like so:
`
outage['OUTAGE.START'] = (outage['OUTAGE.START.DATE'] + ", " + outage['OUTAGE.START.TIME']) \
    .apply(lambda x: pd.Timestamp(x))

outage['OUTAGE.RESTORATION'] = (outage['OUTAGE.RESTORATION.DATE'] + ", " + outage['OUTAGE.RESTORATION.TIME']) \
    .apply(lambda x: pd.Timestamp(x))

outage = outage.drop(columns=['OUTAGE.START.DATE', 'OUTAGE.START.TIME', 'OUTAGE.RESTORATION.DATE', 'OUTAGE.RESTORATION.TIME'])
`
