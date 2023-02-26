# Introduction
This project consists of analysis on a dataset containing the major power outages accross the U.S. throughout the time period January 2000 - July 2016. In total, it contains information on 1534 outages.

This analysis will mainly be focused around the following question: 

## *Where do power outages tend to occur, and how does this affect the overall circumstances surrounding these outages?*

To answer this question, the following columns will be used:

- `POSTAL.CODE`
    - State abbreviations for the outage
- `OUTAGE.START.DATE` and `OUTAGE.START.TIME`
    - Date and time at the start of the outage
- `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME`
    - Date and time at the end of the outage
- `CAUSE.CATEGORY`
    - Cause of the outage from the following possibilities:
        - severe weather
        - intentional attack
        - system operability disruption
        - public appeal
        - equipment failure
        - fuel supply emergency
        - islanding
- `OUTAGE.DURATION`
    - Total duration of the outage in minutes
- `CUSTOMERS.AFFECTED`
    - Approximate customers affected by the outage
- `RES.PRICE`
    - Residential price at the time of the outage in cents / kilowatt-hour

By answering this question, this dataset can provide us with evaluations on how various states experience and restore power outages.

---

# Cleaning and EDA

## Data Cleaning

To clean the data, we begin by loading the relevant columns into pandas, giving us the following dataset:

| POSTAL.CODE   | OUTAGE.START.DATE   | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE   | OUTAGE.RESTORATION.TIME   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |   RES.PRICE |
|:--------------|:--------------------|:--------------------|:--------------------------|:--------------------------|:-------------------|------------------:|---------------------:|------------:|
| MN            | 2011-07-01 00:00:00 | 17:00:00            | 2011-07-03 00:00:00       | 20:00:00                  | severe weather     |              3060 |                70000 |       11.6  |
| MN            | 2014-05-11 00:00:00 | 18:38:00            | 2014-05-11 00:00:00       | 18:39:00                  | intentional attack |                 1 |                  nan |       12.12 |
| MN            | 2010-10-26 00:00:00 | 20:00:00            | 2010-10-28 00:00:00       | 22:00:00                  | severe weather     |              3000 |                70000 |       10.87 |
| MN            | 2012-06-19 00:00:00 | 04:30:00            | 2012-06-20 00:00:00       | 23:00:00                  | severe weather     |              2550 |                68200 |       11.79 |
| MN            | 2015-07-18 00:00:00 | 02:00:00            | 2015-07-19 00:00:00       | 07:00:00                  | severe weather     |              1740 |               250000 |       13.07 |

Looking at the data, the pairs of columns `OUTAGE.START.DATE` and `OUTAGE.START.TIME` as well as `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME` are redundant. Therfore, we merge the two columns so that the date contains the timestamp for both start and restoration. This gives us the resulting cleaned dataset:

| POSTAL.CODE   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |   RES.PRICE | OUTAGE.START        | OUTAGE.RESTORATION   |
|:--------------|:-------------------|------------------:|---------------------:|------------:|:--------------------|:---------------------|
| MN            | severe weather     |              3060 |                70000 |       11.6  | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |
| MN            | intentional attack |                 1 |                  nan |       12.12 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |
| MN            | severe weather     |              3000 |                70000 |       10.87 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |
| MN            | severe weather     |              2550 |                68200 |       11.79 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |
| MN            | severe weather     |              1740 |               250000 |       13.07 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |

As a result, the timestamps are now accurate, and it is easy to verify the outage duration from the difference in the start and restoration timestamps.

## Univariate Analysis

This choropleth map visualizes where the outages from the dataset came from:

<iframe src="assets/map_count.html" width=800 height=600 frameBorder=0></iframe>

As we can see, the vast majority of power outages come from California, although Texas and Washington contain relatively high amounts of outages as well.

## Bivariate Analysis

This choropleth map visualizes the average outage duration for each state:

<iframe src="assets/map_out.html" width=800 height=600 frameBorder=0></iframe>

As we can see, Wisconsin and West Virginia have a much longer average duration, although they have a lower amount of outages reported in the dataset.

## Interesting Aggregates

The following table was constructed by taking the average outage duration accross each state and category

| POSTAL.CODE   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:--------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| CA            |              524.81 |                  6154.6 |              946.458 |    214.857  |         2028.11 |          2928.37 |                         363.667 |
| WA            |             1204    |                     1   |              371.871 |     73.3333 |          248    |          5473.55 |                          25     |
| TX            |              405.6  |                 13920   |              298.769 |    nan      |         1140.41 |          3854.89 |                         810.8   |
| MI            |            26435.3  |                   nan   |             3635.25  |      1      |         1078    |          4831.65 |                        2610     |
| NY            |              247    |                 16687.2 |              309.083 |    nan      |         2655    |          6034.58 |                        1176.57  |

As we can see, there is no definitive category that has a higher average duration time. All the states have a different ordering for the average duration time among the categories.

---

# Assessment of Missingness

## NMAR Analysis

A potential NMAR column in the dataset is `CUSTOMERS.AFFECTED`, which has a total of `443` missing values. This column only contains approximations of the amount of customers affected as there is no real way to find the exact number. For this reason, the process by which these numbers are approximated may affect the missingness of this column. For example, if there was not enough data on the outage region to produce a reliable estimate on the customers, this could result in the value being missing. In order to determine the missingness, more information must be collected on the data generating process.

## Missingness Dependency

We will compare the missingness of the `CUSTOMERS.AFFECTED` column against `RES.PRICE` and `OUTAGE.DURATION`

| null_customers   |   RES.PRICE |   OUTAGE.DURATION |
|:-----------------|------------:|------------------:|
| False            |     11.9389 |           2845.51 |
| True             |     12.0409 |           2071.97 |

As we are testing missingness, we will be using the TVD (Total Variation Distance) as our test statistic. The TVD for `RES.PRICE` is `0.102` and for `OUTAGE.DURATION` is `773.547`.

Running the permutation test with `1000` iterations, we get the following results:

<iframe src="assets/tvd_price.html" width=800 height=600 frameBorder=0></iframe>

The reported p-value for `RES.PRICE` was `0.567`. As we can see on the visualization, this indicates that it is unlikely for the missingness of `CUSTOMERS.AFFECTED` to depend on `RES.PRICE`.

<iframe src="assets/tvd_out.html" width=800 height=600 frameBorder=0></iframe>

The reported p-value for `OUTAGE.DURATION` was `0.024`. As we can see on the visualization, this indicates that it is likely for the missingness of `CUSTOMERS.AFFECTED` to depend on `RES.PRICE`.

---

# Hypothesis Testing

If we look at the five most represented states in the dataset and their average outage durations, we get the following table:

| POSTAL.CODE   |   OUTAGE.DURATION |   counts |
|:--------------|------------------:|---------:|
| CA            |           1666.34 |      210 |
| TX            |           2704.82 |      127 |
| WA            |           1508.15 |       97 |
| MI            |           5302.98 |       95 |
| NY            |           6034.96 |       71 |

However if we look at the five states with the highest average outage durations, we get the following table:

| POSTAL.CODE   |   OUTAGE.DURATION |   counts |
|:--------------|------------------:|---------:|
| WI            |           7904.11 |       20 |
| WV            |           6979    |        4 |
| NY            |           6034.96 |       71 |
| MI            |           5302.98 |       95 |
| KY            |           5093.92 |       13 |

In particular, California is number 23rd in this table, despite having the most recorded outages. This begs the question: is the outage duration in any way related to the state?

To answer this question, a hypothesis test was conducted with the following null and alternative hypotheses:

- **Null Hypothesis**: Outage Duration and State are not related, the lower average duration of California is due to chance
- **Alternative Hypothesis**: Outage Duration and State are related, the lower average duration of California is statistically significant

For this hypothesis test, we will be using the average duration for California (`1666.34`) and compare it against randomly drawn samples of size `210` from the dataset.

<iframe src="assets/avg_out.html" width=800 height=600 frameBorder=0></iframe>

From this test, we get a p-value of `0.001`, which along with the above graph, suggests that it is likely that the average outage duration of California is lower than other states. Going back to our initial question, despite California having a large amount of power outages, it seems likely that California also has a lower average outage duration. In order to conclude why this is, more research has to be done with how California manages power outages, in order to compare against other states.