# Lights Out in the Eye of the Storm
A Data Analysis of Power Outages and Hurricanes

by Russel Luber ([rluber@ucsd.edu](mailto:rluber@ucsd.edu))


## Introduction
In this project, I worked on a dataset about major power outage events in the continental United States. This dataset can be accessed online [here](https://engineering.purdue.edu/LASCI/research-data/outages), courtesy of Purdue University's Laboratory for Advancing Sustainable Critical Infrastructure. An article detailing the data dictionary for the dataset can be found [here](https://www.sciencedirect.com/science/article/pii/S2352340918307182).

The dataset contains information about major power outage events, as well as geographical data, electricity consumption patterns, and economic characteristics related to said outages.


### Motivation
I experienced many power outages growing up in the Philippines. Sometimes due to [typhoons](https://en.wikipedia.org/wiki/Typhoon). Other times due to insufficient electrical supply. We would always call them "[brownouts](https://en.wikipedia.org/wiki/Brownout_(electricity))" no matter the cause. 

I chose this dataset because I have experienced power outages myself - from outages that lasted a couple hours to blackouts that spanned a couple months at a time. When I hear the phrase "power outage," it brings back memories of my childhood, distinct stretches of time tinged with boredom and inconvenience but also marked with death and destruction.

In my experience, power outages were almost always caused by typhoons. Or as North America likes to call them: hurricanes. Perhaps this dataset can change my mind about how diverse the causes of power outages are and give me insight about the toll and damage these outages cause on people's lives.


## Data Cleaning and Exploratory Data Analysis

### Cleaning
Cleaning the data is always the first step for any analysis. I pulled the dataset from the website and put it into a [Pandas](https://pandas.pydata.org/) DataFrame called `outages`.

Here's what I did to clean the `outages`:
1. I converted `OUTAGE.START.DATE` and `OUTAGE.RESTORATION.DATE` to full datetime timestamps, integrating information from both `OUTAGE.START.TIME` and `OUTAGE.RESTORATION.TIME`.
2. There were duplicate rows, so I dropped duplicate rows. 
3. I did data quality checks on relevant columns like `CUSTOMERS.AFFECTED`, `OUTAGE.DURATION`, and `DEMAND.LOSS.MW`. It turns out that a lot of these columns contained zeroes, so I decided to replace the zeroes with `np.nan`s because those are probably missing values instead of the absence of customers affected or 0 minutes of power outage duration. This is a key assumption I'm making. 

### Exploratory Data Analysis
#### Univariate Analysis
At the beginning of this investigation, I was curious about hurricanes because of my experiences with them. I decided to look at the number of power outages caused by hurricanes across the dataset.

<iframe src="assets/univariate_1.html" width="800" height="600" frameborder="0" ></iframe>
This is the distribution of hurricane-caused power outages across the dataset. Florida has the most outages due to hurricanes, which makes sense due to its tropical climate and weather. 

I found that there were about 72 hurricane-caused power outages in the dataset, a drop in the bucket compared to the overall dataset.

That's why I decided to shift my analysis and look at the bigger picture. I took a step back and looked at the distribution of power outages across `CAUSE.CATEGORY`.

<iframe src="assets/univariate_2.html" width="800" height="600" frameborder="0" ></iframe>

This bar chart shows the number of power outages categorized by reported cause. About 50% of the outages were due to "severe weather," only a subset of which are hurricanes. So I decided to pivot my analysis towards damage and people affected.

<iframe src="assets/univariate_3.html" width="800" height="600" frameborder="0" ></iframe>

This chart shows how many millions of U.S. resident customers have been affected by power outages over time. In 2008, about 20 million people experienced at least one power outage!

While not much can be gleaned from it, it's worth noting the spikes over time, some of which may be attributed to hurricanes, over "severe weather" more broadly.

#### Bivariate Analysis
To analyze the relationship between features in the dataset, I did a few bivariate analyses. In particular, I was interested in damage and the toll on people's lives caused by these outages, so I focused on `CUSTOMERS.AFFECTED`. 

<iframe src="assets/bivariate_1.html" width="800" height="600" frameborder="0" ></iframe>

In this scatterplot, I looked at the relationship between `CUSTOMERS.AFFECTED` and `OUTAGE.DURATION`. Judging from the scatterplot, the variability is low save for a bunch of outliers. Most power outages affected a lot of people (in the hundreds of thousands) and lasted more than a few weeks at worst.


<iframe src="assets/bivariate_2.html" width="800" height="600" frameborder="0" ></iframe>

I also looked at the distributions of `CUSTOMERS.AFFECTED` by `CAUSE.CATEGORY` to get a better grasp on the outliers. It seems that "severe weather" outages had a lot of outliers with respect to the number of people affected. Needless to say, these outlier events are power outages that affected millions of people.

#### Interesting Aggregates
I wanted to get a sense of averages when it came to my features of interest: `CUSTOMERS.AFFECTED`, `DEMAND.LOSS.MW`, and `OUTAGE.DURATION`. I created a pivot table indexed by state.

{% include assets/pivot_table.md %}


## Assessment of Missingness


## Hypothesis Testing



## Framing a Prediction Problem


## Baseline Model


## Final Model



## Fairness Analysis







