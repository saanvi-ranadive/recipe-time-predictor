# Investigating the Relationship Between Nutrition and Cooking Time of Recipes
Final Project for UCSD's DSC80 Course
Names: Saanvi Ranadive and Ritvik Chand

## Introduction (Ritvik)

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

Before completing our analysis, we merged and cleaned the raw data through the following steps:

1. Left merged the recipes DataFrame on the 'id' column with the interactions DataFrame on the 'recipe_id' column, dropping ['Unnamed: 0', 'recipe_id'] columns.

2. Replaced all ratings of 0 with np.nan because 0 is not a valid rating. Additionally, when we cross-referenced ratings of 0 with their corresponding review, a rating of 0 did not match the sentiment of most reviews.

3. Left merged the resulting DataFrame with a new DataFrame grouped by 'id' and aggregated by taking the mean of the rating column. This gave us the recipe's average ratings alongside the individual user rating for each recipe.

4. Created new columns 'calories' (float), '%DV_fat' (float), and '%DV_sugar' (float) with extracted values from the original 'nutrition' column. These are the features we were most interested in exploring with regards to cooking time.

5. Standardized each of these three columns using the z-score, resulting in 'standardized_calories' (float), 'standardized_fat' (float), and 'standardized_sugar' (float) columns. We decided to standardize each column at this point because they are measured on different scales, and we wanted to use them to create a composite health score without having any one column dominate the health score.

6. Calculated a health score for each recipe by adding up standardized calories, fat, and sugar and negating the result (so that a higher health score would correspond to a lower amount of calories, fat, and sugar). Stored this new variable in a column called 'health_score' (float).

7. Created 'is_healthy' (bool) column corresponding to whether or not the recipe has a 'health_score' >= 0.5. We chose 0.5 as the threshold between healthy (>= 0.5) and unhealthy (<0.5) recipes because 0.5 is approximately the median 'health_score' and thus would divide the data points evenly between the two groups. The 'is_healthy' column will be helpful for interesting visualizations, aggregates, and permutation tests.

8. Dropped 'calories', '%DV_fat', and '%DV_sugar' columns.

9. Dropped outliers in terms of the 'minutes' column (i.e. recipes that took less than 0 minutes and more than 600 minutes, because any cooking time outside of these bounds does not seem viable.) We dropped these values so that in later steps like hypothesis testing and predictive modeling, these outliers would not skew our results.

First 5 rows of resulting DataFrame (2231766 rows x 22 columns):

| name | id | minutes | contributor_id | submitted | tags             | nutrition       | n_steps | steps                   | description         | ingredients        |  n_ingredients | user_id | date | rating | review             |   average_rating | standardized_calories | standardized_fat | standardized_sugar |   health_score | is_healthy |
|:-----|---:|-------:|---------------:|:----------|:------------------|:-----------------|--------:|:-----------------------|:--------------------|:--------------------|---------------:|--------:|:----|-------:|:------------------|--------------:|---------------------:|-----------------:|---------------------:|-------------:|
| 1 brownies in the world... | 333281 |40 | 985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts'...] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] | 10 | ['heat the oven to 350f and arrange'...] | these are the most; chocolatey... | ['bittersweet chocolate', 'unsalted butter'...] |  9 | 386585           | 2008-11-19 |   4 | These were pretty...   |  4 |  -0.482026 |  -0.395726 | -0.0657603 | 0.943512 | True |
| 1 in canada chocolate chip...| 453467 | 45 | 1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american'...] | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] | 12 | ['pre-heat oven the 350 degrees f'...] | this is the recipe...| ['white sugar', 'brown sugar' ...] | 11 | 424680 | 2012-01-26 | 5 | Originally I was gonna cut the ...| 5 | 0.301036 | 0.254186 | 0.699211  | -1.25443  | False |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient' ...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a '...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 |  29782 | 2008-12-31 | 5 | This was one of the best broccoli... | 5 |  -0.385322 |  -0.215195 | -0.274821  | 0.875337 | True |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a' ...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 1.19628e+06 | 2009-04-13 | 5 | I made this for my son's first...   | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 | True |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a'... ] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 768828 | 2013-08-02 | 5 | Loved this.  Be sure to completely... | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 | True |


### Univariate Analysis

<iframe
  src="figures/univariate1.html"
  width="800"
  height="600"
  frameborder="0"
  style="margin-bottom: 0; padding: 0;"
></iframe>

This histogram displays the distribution of cooking times for all recipes in minutes. The figure indicates that majority of recipes have a cooking time that falls between 0 and 100 minutes. Despite our removal of outlier values, the plot is still heavily right skewed. This could potentially cause issues when exploring trends with recipes across the 0-600 minute range. Specifically, the trends could become less reliable as cooking time exceeds 100 minutes due to the limited data availability and potential variability in those longer cooking times.

<iframe
  src="figures/univariate2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure shows an overlayed histogram comparison of cooking times for recipes classified as "healthy" and "unhealthy". The vertical dashed lines in this figure show that recipes that are healthy take less time to prepare on average than unhealthy recipes. This is an interesting discovery as we initially predicted that healthier recipes would generally take longer to make.

### Bivariate Analysis

<iframe
  src="figures/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This scatter plot displays the trend between 'health_score' of recipes and their cooking time in 'minutes'. Interestingly, despite our findings from the last plot (that healthy recipes have a lower average cooking time), this plot shows that as 'health_score' increases, so does cooking time.

Together, these two plots indicate an example of Simpson's Paradox, because when we look at the overall trend of the entire dataset, we observe a positive relationship between 'health_score' and 'minutes' but when we split the data at the median 'health_score', we find that healthy recipes have a lower average cooking time.

Looking at the bivariate graph, we can hypothesize that the reason for this paradox is that the green (healthy) points are highly concentrated in a vertical band that is evenly distributed across the y-axis, resulting in a moderate average cooking time. Meanwhile the red (unhealthy) points are more spread out, with a significant number of long recipes skewing the average cooking time of unnhealthy recipes to be higher.


### Interesting Aggregates

In this section, we will further investigate the reason for the Simpson's Paradox described above, using a grouped table. We are interested in finding out how the average cooking time ('minutes') varies across different 'health_scores' and this trend translates to the labels of 'healthy' and 'unhealthy'.

First, we created a new column called 'health_score_bin' by segmenting the 'health_score' column into bins of [-40, -30, -20, -10, 0, 0.5, 2]. Then, we grouped by 'health_score_bin' and 'is_healthy', aggregating by both mean and count. We obtained the resulting DataFrame:

| ('health_score_bin', '')   | ('is_healthy', '')   |   ('minutes', 'mean') |   ('minutes', 'count') |
|:---------------------------|:---------------------|----------------------:|-----------------------:|
| -40 to -30                 | False                |               81.3636 |                    176 |
| -30 to -20                 | False                |               67.1911 |                    293 |
| -20 to -10                 | False                |               65.1838 |                   1398 |
| -10 to 0                   | False                |               69.8318 |                  69178 |
| 0 to 0.5                   | False                |               63.5126 |                  41927 |
| 0.5 to 2                   | True                 |               48.7758 |                 118684 |

This grouped table shows the increasing trend of mean cooking time across the 2nd, 3rd, and 4th bins. With a 'health_score' range of -30 to 0, these bins dominated the scatter plot, giving the sense that as 'health_score' increases, so does 'minutes'. However, this entire trend was within the unhealthy category, and we see that the healthy category (contained within the bin 0.5 to 2), has the lowest average cooking time.

## Assessment of Missingness

We were interested in exploring the missingness of various columns in our DataFrame. The first step we took was to count the number of missing values in each column. We received the following counts:

name 1

id 0

minutes 0

contributor_id 0

submitted 0

tags 0

nutrition 0

n_steps 0

steps 0

description 113

ingredients 0

n_ingredients 0

user_id 1

date 1

rating 14740

review 56

average_rating 2706

standardized_calories 0

standardized_fat 0

standardized_sugar 0

health_score 0

is_healthy 0


The only columns with a significant number of missing values are 'description', 'rating', 'review', and 'average_rating'.

### NMAR Analysis
We believe that the 'review' column may be Not Missing At Random (NMAR). For features that are NMAR, the probability of a value being missing depends on the missing value itself. We think the 'review' column is NMAR because people who felt strongly about the recipe (people who either loved it or hated it) would be more likely to express their opinion and give feedback than someone who felt neutral about the recipe.

If a user constantly revisit the same recipe, it would be an indication that they really enjoyed it, but if a user only visited the recipe once, it could indicate that they did not like it very much, or that they felt neutral about it. We think it would be interesting to collect data on how many times the user cooked this recipe, as it could indicate how much they interacted with the recipe and thus their likeliness to leave a review. 

### Missingness Dependency
We will analyze the missingness of the 'description' column in relation to the 'health_score' and 'minutes' columns. Specifically, we will run two permutation tests to investigate if the missingness of 'description' is dependent on the 'health_score' and 'minutes' columns.

**Investigating the Missingness Dependency of 'Description' on 'Health_Score'**

**Null Hypothesis**: The distribution of 'health_score' when 'description' is missing is the same as the distribution when 'description' is not missing.

**Alternative Hypothesis**: The distribution of 'health_score' when when 'description' is missing is not the same as the distribution when 'description' is not missing.

**Test Statistic**: total variation distance

**Significance Level**: 0.05

**Number of Permutations**: 1000

<iframe
  src="figures/missingness1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We found an observed tvd of 0.153 and a p-value 0.002 (represented by the vertical red line), which leads us to 
*reject the null hypothesis*. 

This test indicates that the missingness of 'description' does depend on 'health_score'.

**Investigating the Missingness Dependency of 'Description' on 'Minutes'**

**Null Hypothesis**: The distribution of 'minutes' when 'description' is missing is the same as the distribution when 'description' is not missing.

**Alternative Hypothesis**: The distribution of 'minutes' when when 'description' is missing is not the same as the distribution when 'description' is not missing.

**Test Statistic**: total variation distance

**Significance Level**: 0.05

**Number of Permutations**: 1000

<iframe
  src="figures/missingness2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We found an observed tvd of 0.227 and a p-value of 0.225, (represented by the vertical red line), which leads us to 
*fail to reject the null hypothesis*. 

This test does not provide evidence that the missingness of 'description' depends on 'health_score'.

## Hypothesis Testing (Saanvi)

We are interested in exploring the relationship between the healthiness of recipes and their cooking time. Therefore, we decided to implement a permutation test to find out if recipes classified as "healthy" (recipes with a health_score of >= 0.5) have the same distribution of cooking times as those recipes classified as "unhealthy". We have already seen from the second univariate graph that healthy recipes in our dataset have a lower average cooking time. Now we will investigate if this difference was purely by chance or due to a difference in distributions.

**Null hypothesis**: The average number of minutes needed to prepare recipes that are classified as healthy is the same as the average number of minutes needed to prepare recipes that are classified as unhealthy.

**Alternative Hypothesis**: The average number of minutes needed to prepare recipes that are classified as healthy is less than the average number of minutes needed to prepare recipes that are classified as unhealthy.

**Test Statistic**: Difference of means (mean_time~(healthy)~ - mean_time~(unhealthy)~)

**Significance Level**: 0.05

**Number of Permutations**: 1000

The observed test statistic for this test is -18.669, meaning that for this dataset, recipes classified as healthy took 18.669 minutes less to make than recipes classified as unhealthy, on average.

The graph below displays the observed_tvd as a vertical red line, showing a significant deviation from the distribution. Consequently, the calculated p-value is 0.0.

<iframe
  src="figures/permutation_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Because our p-value 0.0 < the significance level of 0.005, we **reject the null hypothesis**. The distribution of cooking times for healthy recipes has a lower mean than the distribution of cooking times for unhealthy recipes. 

This could be due to outliers in either group: healthy outliers could include salads and sandwiches (foods that do not take long to prepare), while unhealthy outliers could be deep-fried foods or baked goods (foods that would take a long time to prepare).


## Framing a Prediction Problem (Saanvi)

## Baseline Model (Ritvik)

## Final Model (Ritvik)

## Fairness Analysis (Ritvik)