# Investigating the Relationship Between Nutrition and Cooking Time of Recipes
Final Project for UCSD's DSC80 Course
Names: Saanvi Ranadive and Ritvik Chand

## Introduction (Ritvik)

## Data Cleaning and Exploratory Data Analysis (Saanvi)

### Data Cleaning

Before completing our analysis, we merged and cleaned the raw data through the following steps:

1. Left merged the recipes DataFrame on the 'id' column with the interactions DataFrame on the 'recipe_id' column, dropping ['Unnamed: 0', 'recipe_id'] columns.

2. Replaced all ratings of 0 with np.nan because 0 is not a valid rating. Additionally, when we cross-referenced ratings of 0 with their corresponding review, a rating of 0 did not match the sentiment of most reviews.

3. Left merged the resulting DataFrame with a new DataFrame grouped by 'id' and aggregated by taking the mean of the rating column. This gave us the recipe's average ratings alongside the individual user rating for each recipe.

4. Created new columns 'calories', '%DV_fat', and '%DV_sugar' with extracted values from the original 'nutrition' column. These are the features we were most interested in exploring with regards to cooking time.

5. Standardized each of these three columns using the z-score, resulting in 'standardized_calories', 'standardized_sugar', and 'standardized_fat' columns. We decided to standardize each column at this point because they are measured on different scales, and we wanted to use them to create a composite health score without having any one column dominate the health score.

6. Calculated a health score for each recipe by adding up standardized calories, fat, and sugar and negating the result (so that a higher health score would correspond to a lower amount of calories, fat, and sugar). Stored this new variable in a column called 'health_score'.

7. Dropped 'calories', '%DV_fat', and '%DV_sugar' columns.

8. Dropped outliers in terms of the 'minutes' column (i.e. recipes that took less than 0 minutes and more than 600 minutes, because any cooking time outside of these bounds does not seem viable.) We dropped these values because in later steps such as hypothesis testing and predictive modeling, we found that these outliers skewed our results.

Resulting DataFrame (2231766 rows x 21 columns):

| name | id | minutes | contributor_id | submitted | tags             | nutrition       | n_steps | steps                   | description         | ingredients        |  n_ingredients | user_id | date | rating | review             |   average_rating | standardized_calories | standardized_fat | standardized_sugar |   health_score |
|:-----|---:|-------:|---------------:|:----------|:------------------|:-----------------|--------:|:-----------------------|:--------------------|:--------------------|---------------:|--------:|:----|-------:|:------------------|--------------:|---------------------:|-----------------:|---------------------:|-------------:|
| 1 brownies in the world... | 333281 |40 | 985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts'...] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] | 10 | ['heat the oven to 350f and arrange'...] | these are the most; chocolatey... | ['bittersweet chocolate', 'unsalted butter'...] |  9 | 386585           | 2008-11-19 |   4 | These were pretty...   |  4 |  -0.482026 |  -0.395726 | -0.0657603 | 0.943512 |
| 1 in canada chocolate chip...| 453467 | 45 | 1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american'...] | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] | 12 | ['pre-heat oven the 350 degrees f'...] | this is the recipe...| ['white sugar', 'brown sugar' ...] | 11 | 424680 | 2012-01-26 | 5 | Originally I was gonna cut the ...| 5 | 0.301036 | 0.254186 | 0.699211  | -1.25443  |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient' ...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a '...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 |  29782 | 2008-12-31 | 5 | This was one of the best broccoli... | 5 |  -0.385322 |  -0.215195 | -0.274821  | 0.875337 |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a' ...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 1.19628e+06 | 2009-04-13 | 5 | I made this for my son's first...   | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a'... ] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 768828 | 2013-08-02 | 5 | Loved this.  Be sure to completely... | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 |


### Univariate Analysis

## Assessment of Missingness (Saanvi)

## Hypothesis Testing (Saanvi)

## Framing a Prediction Problem (Saanvi)

## Baseline Model (Ritvik)

## Final Model (Ritvik)

## Fairness Analysis (Ritvik)