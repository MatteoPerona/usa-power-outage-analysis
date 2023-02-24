# USA Power Outage Analysis
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