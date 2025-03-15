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

Here are the first few rows of the cleaned DataFrame:

{% include_relative assets/cleaned_rows.md %}

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

{% include_relative assets/pivot_table.md %}

Interestingly enough, a hurricane-prone state like Florida on the east coast had an average of 375,000 affected people, while non-hurricane-prone state like Oregon on the west coast only has an average of 75,000 people affected. The average power outage duration is almost five times longer in Florida than in Oregon: 4084 minutes on average for Florida, 833 minutes on average for Oregon. But these are averages. And there are a lot of outliers in the dataset with respect to the number of customers affected.

## Assessment of Missingness
### NMAR Analysis
`DEMAND.LOSS.MW` is a good candidate for a feature that is Not Missing at Random (NMAR). One likely explanation of this is because huge, widespread loss of demands are difficult to estimate. For example, a hurricane knocks out the entire grid in a city, making it difficult to calculate the exact MW loss because all monitoring systems fail. The more severe the outage, the more likely that outage is to go missing. The systems used to measure such demand losses might not be able to handle huge losses so they don't show up in the dataset.

### Missing Dependency
To test missing dependency, I focused on the missigness of `OUTAGE.DURATION`, given that I found a lot of missing values from it in my exploratory data analysis (EDA). More specifically, I tested the missingness of `OUTAGE.DURATION` against `NERC.REGION`. This is because I wanted to see if the durations are missing depending on where on the continental United States the outage was from. I figured maybe there was a case to be made that some regions tended to underreport or overrport outage durations.

- **Null Hypothesis**: The distribution of `NERC.REGION` regardless of wheter or not `OUTAGE.DURATION` is missing.
- **Alternative Hypothesis**: The distribution of `NERC.REGION` is different when `OUTAGE.DURATION` is missing versus when it is not missing.
- **Test Statistic**: The distribution of `NERC.REGION`s are categorical, so we use total variation distance (TVD) as the test statistic.



<iframe src="assets/nerc_region_missigness.html" width="800" height="600" frameborder="0" ></iframe>

For this test, I got an observed TVD of 0.142 with a p-value of 0.044. With 0.05 as the significance level, we are able to reject the null hypothesis and cautiously say that `OUTAGE.DURATION` is dependent on `NERC.REGION`.

This shows that `OUTAGE.DURATION` is likely missing at random (MAR) with respect to `NERC.REGION`. It suggests that certain North American regions are more likely to have missing outage durations than others.


## Hypothesis Testing
I want to test whether power outages in the rest of the United States tend to last longer than those in Florida. This is because based on our data, Florida has experienced a lot of hurricanes. However, note that the dataset doesn't only contain hurricane outages. I am curious about the overall power outage durations in Florida compared to the rest of the United States. Perhaps hurricanes contribute a lot to those outage durations?

Since outage durations can have extreme outliers, I’ll use the difference in medians instead of the mean. The mean is sensitive to large outliers, and I’ve already noticed some unusually long outages in the data that could skew the results (see the Bivariate Analysis section). Using the median gives us a more reliable comparison.

Hypotheses:
- **Null Hypothesis**: The typical (median) outage duration in Florida is about the same as in the rest of the U.S.
- **Alternative Hypothesis**: The typical outage duration in Florida is shorter than in the rest of the U.S.

Test Statistic:
- We will compare the median outage duration in Florida to the median outage duration in the rest of the U.S.
- A large difference in medians will suggest that outages in one region last significantly longer than in the other.

To do this, I used a hypothesis test with 1000 iterations and calculated how many values where less than the observed difference in medians I had. I got an observed value of 210 with a p-value of 0.663. 

We therefore must tentatively accept the null hypothesis that Florida has about the same median outage duration as the rest of the United States.


<iframe src="assets/florida_hyp_test.html" width="800" height="600" frameborder="0" ></iframe>

Here we see the dashed red line pointing to where the observed difference in medians is in the sampling distribution of difference in medians. It falls squarely in the distribution, which indicates that we must tentatively accept the null hypothesis.

## Framing a Prediction Problem
While `OUTAGE.DURATION` is a good indicator of how impactful an outage is, I am more interested in how it affects people. So instead of just looking at duration, we’ll use the data to predict how many people (customers) are affected. This will help organizations spot events that are likely to impact a lot of people, allowing them to take action ahead of time. Prediction is an important part of disaster preparedness.

Looking at the initial models, I noticed that the data is pretty skewed. A lot of really low values that show up often, along with a few extreme outliers. This has led to very high RMSE values, making it a poor predictor for how many customers are actually affected. To fix this, I’ll switch to a classification model and create a new variable, "High_Risk_Customers," which will be a simple yes/no label for whether an event affects more than a certain number of people. This way, the model focuses on identifying high-risk areas and key risk factors instead of trying to predict exact numbers.

To evaluate the model, we’ll use precision score to make sure we’re correctly identifying the communities most likely to be impacted. That way, stakeholder organizations can prioritize resources for the events that actually matter the most, thus allowing for a more efficient response to such outages.

## Baseline Model
My baseline model is a Decision Tree classifier using features like region, month, climate data, outage cause, customer percentages, and population stats. We one-hot encode categorical variables and leave numerical and ordinal ones as they are since month is already numeric.

We limit the tree’s depth to 4 as a basic hyperparameter. For classification, we define our target as whether an outage affected more than 150,000 people, labeling it as 1 if above the threshold, 0 otherwise.

Result:
1. The baseline model's accuracy is 66% 
2. The baseline model's precision is 39%. Not really that good. The model is not very confident in its positive predictions. 

## Final Model
For this final model, we're going to try using an [SVC](https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html) (Support Vector Classifier) to see if it performs better than our base model. After all, the data is much more complex and has a lot of nonlinear interactions. To improve its accuracy, we’ll tweak some of the columns by applying transformations.

One change we’re making is squaring the `ANOMALY.LEVEL`. The reason for this is that the extreme values of this feature don’t vary much, so squaring it spreads out the middle values more, making it easier for the model to find a clear separation.

We’re also applying an exponential transformation to another feature, `RES.PERCEN` (percentage of affected customers). This is because the higher percentage values are where most of the big outages happen, but there are still a lot of smaller cases mixed in. By stretching out the higher values, we give the model more room to separate major outages from minor ones. The goal is to increase True Positives (correctly identified big outages) while reducing False Positives (misclassified minor outages).

Finally, we’re going to test different tolerance levels in the model to see how they affect its performance. The `tolerance` (tol) controls when the model stops training—lower values make it train longer, while higher values might make it stop too soon. We’ll experiment with this to find the best balance.

The final model results are:
1. Accuracy: 74%
2. Precision: 60%

A modest improvement from the baseline model. 

## Fairness Analysis
I wanted to see if my model predicts outages equally well across different regions in the U.S. Specifically, I was curious about whether its precision (how often the model correctly predicts a high-impact outage) is similar for West Coast versus East Coast states. We define a high-impact outage as an outage that affected at least 150,000 customers.
- Does East Coast vs West Coast matter for our model's predictions?

I ran a permutation test with 100 trials to see if the precision difference between these two groups is statistically significant.
Hypotheses:
- **Null Hypothesis** (Fair Model): The model treats both regions fairly. The precision scores for Eastern and Western states are about the same, and any small differences we see are just due to random variation.
- **Alternative Hypothesis** (Unfair Model): The model does not treat both regions the same. The precision score for Western states is meaningfully different from the precision score for Eastern states.

Significance level: 0.05

Here is the distribution of the differences in precision:

<iframe src="assets/fairness_precision_diffs2.html" width="800" height="600" frameborder="0" ></iframe>


Here are the results:
- Observed difference: 0.0375
- p-value: 0.42

Therefore, we fail reject the null hypothesis. The likely culprit of our observed difference in precision is random chance. This suggests that the model treats both regions (East Coast vs West Coast) fairly. There is no strong evidence that the model predicts outages significantly better in one region over the other. 

