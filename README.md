# Does Nutrition Affect Cooking Time? A Data-Driven Analysis of Recipes
Final Project for UC San Diego's DSC80 Course
Names: Saanvi Ranadive and Ritvik Chand


## Introduction

This project was conducted at UC San Diego as a final project for the DSC80 Course.

Throughout this investigation, we explore the relationship between healthiness and cooking time of recipes. We aim to answer the question: **Is a recipe's nutritional information (including number of calories, amount of fat, and amount of sugar) a strong predictor of how long it takes to cook the recipe?** In our personal experience, we've found that healthier meals often take longer to prepare, and were wondering if this was reflective of a broader trend. Understanding whether nutritional factors impact cooking time can provide valuable insights for people cooking at home, meal preparation, and the food industry. The existence of strong trends between nutrition and cooking time could help people make more informed decisions about meal planning, assist in designing recipes, and ultimately make it easier for individuals to prioritize nutrition without sacrificing too much time in the kitchen.

We will analyze two datasets from [food.com](https://www.food.com/): **recipes**, which focuses on information about each recipe as uploaded by the contributer; and **interactions**, which details the ratings and reviews from users who used each recipe.

**Recipes Dataset**

(83782 rows x 13 columns)

| Column Name            | Description                          |
|------------------------|-------------------------------------:|
| name                  | (object) name of recipe              |
| id                    | (int64) unique recipe ID             |
| minutes               | (int64) number of minutes to prepare |
| contributor_id        | (int64) unique contributer ID        |
| submitted             | (object) date submitted              |
| tags                  | (object) relevant recipe tags        |
| nutrition             | (object) [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]|
| n_steps               | (int64) number of steps in the recipe|
| steps                 | (object) text for recipe steps       |
| description           | (object) description of recipe       |
| ingredients           | (object) text for recipe ingredients |
| n_ingredients         | (int64) number of ingredients        |

**Interactions Dataset**

(731927 rows x 5 columns)

| Column Name            | Description                            |
|------------------------|---------------------------------------:|
| user_id               | (int64) unique user ID                 |
| recipe_id             | (int64) unique recipe ID               |
| date                  | (object) date rating/ review submitted |
| rating                | (int64) rating from 1-5 submitted      |
| review                | (object) review submitted              |

The most relevant columns to our question are the 'nutrition' and 'minutes' columns from the Recipes Dataset. In the next step, we will extract calories, total fat, and sugar from the nutrition column in order to obtain the necessary information to answer our question. We will use these columns throughout our investigation, creating a composite 'health_score' and boolean 'is_healthy' column to further aid our analysis.


## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

As part of the data preparation process, we merged and cleaned the raw data through the following steps:

1. Left merged the recipes DataFrame on the 'id' column with the interactions DataFrame on the 'recipe_id' column, dropping ['Unnamed: 0', 'recipe_id'] columns.

2. Replaced all ratings of 0 with np.nan because 0 is not a valid rating. Additionally, when we cross-referenced ratings of 0 with their corresponding review, a rating of 0 did not match the sentiment of most reviews.

3. Left merged the resulting DataFrame with a new DataFrame grouped by 'id' and aggregated by taking the mean of the rating column. This gave us the recipe's average ratings alongside the individual user rating for each recipe.

4. Created new columns 'calories' (float), '%DV_fat' (float), and '%DV_sugar' (float) with extracted values from the original 'nutrition' column. These are the features we were most interested in exploring with regards to cooking time.

5. Standardized each of these three columns using the z-score, resulting in 'standardized_calories' (float), 'standardized_fat' (float), and 'standardized_sugar' (float) columns. We decided to standardize each column at this point because they are measured on different scales, and we wanted to use them to create a composite health score without having any one column dominate the health score.

6. Calculated a health score for each recipe by adding up standardized calories, fat, and sugar and negating the result (so that a higher health score would correspond to a lower amount of calories, fat, and sugar). Stored this new variable in a column called 'health_score' (float).

7. Created 'is_healthy' (bool) column corresponding to whether or not the recipe has a 'health_score' >= 0.5. We chose 0.5 as the threshold between healthy (>= 0.5) and unhealthy (<0.5) recipes because 0.5 is approximately the median 'health_score' and thus would divide the data points evenly between the two groups. The 'is_healthy' column will be helpful for interesting visualizations, aggregates, and permutation tests.

8. Dropped 'calories', '%DV_fat', and '%DV_sugar' columns.

9. Dropped outliers in terms of the 'minutes' column (i.e. recipes that took less than 0 minutes and more than 600 minutes, because any cooking time outside of these bounds does not seem viable.) We dropped these values so that in later steps like hypothesis testing and predictive modeling, these outliers would not skew our results.

**Resulting DataFrame**:

(2231766 rows x 22 columns)

| Column Name            | Data Type      |
|------------------------|---------------:|
| name                  | object         |
| id                    | int64          |
| minutes               | int64          |
| contributor_id        | int64          |
| submitted             | object         |
| tags                  | object         |
| nutrition             | object         |
| n_steps               | int64          |
| steps                 | object         |
| description           | object         |
| ingredients           | object         |
| n_ingredients         | int64          |
| user_id               | int64          |
| date                  | object         |
| rating                | int64          |
| review                | object         |
| average_rating        | float64        |
| standardized_calories | float64        |
| standardized_fat      | float64        |
| standardized_sugar    | float64        |
| health_score          | float64        |
| is_healthy            | bool           |

First 5 rows of resulting DataFrame:

| name | id      |minutes|contributor_id| submitted | tags             | nutrition                  |n_steps| steps                   | description         | ingredients        |n_ingredients| user_id       | date   |rating| review             |average_rating|standardized_calories|standardized_fat|standardized_sugar|health_score|is_healthy|
|:-----|--------:|-------:|------------:|:----------|:------------------|:--------------------------|-------:|:-------------------------|:-------------------|:------------------|-----------:|-------------:|:--------|------:|:-------------------|-------------:|--------------------:|---------------:|-----------------:|-----------:|---------:|
| 1 brownies in the world... | 333281 |40 | 985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make'...] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] | 10 | ['heat the oven to 350f and arrange'...] | these are the most; chocolatey... | ['bittersweet chocolate', 'unsalted butter'...] |  9 | 386585           | 2008-11-19 |   4 | These were pretty...   |  4 |  -0.482026 |  -0.395726 | -0.0657603 | 0.943512 | True |
| 1 in canada chocolate chip...| 453467 | 45 | 1848091 | 2011-04-11  | ['60-minutes-or-less', 'time-to-make'...] | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] | 12 | ['pre-heat oven the 350 degrees f'...] | this is the recipe...| ['white sugar', 'brown sugar' ...] | 11 | 424680 | 2012-01-26 | 5 | Originally I was gonna cut the ...| 5 | 0.301036 | 0.254186 | 0.699211  | -1.25443  | False |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30 | ['60-minutes-or-less', 'time-to-make' ...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a '...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 |  29782 | 2008-12-31 | 5 | This was one of the best broccoli... | 5 |  -0.385322 |  -0.215195 | -0.274821  | 0.875337 | True |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a' ...] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 1.19628e+06 | 2009-04-13 | 5 | I made this for my son's first...   | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 | True |
| 412 broccoli casserole | 306168 | 40 | 50969 | 2008-05-30  | ['60-minutes-or-less', 'time-to-make'...] | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0] | 6 | ['preheat oven to 350 degrees', 'spray a'... ] | since there are already 411 recipes for broccoli casserole... | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp'...] | 9 | 768828 | 2013-08-02 | 5 | Loved this.  Be sure to completely... | 5 |  -0.385322 | -0.215195 | -0.274821  | 0.875337 | True |

### Univariate Analysis

**Univariate Plot #1**

<iframe
  src="figures/univariate1.html"
  width="800"
  height="450"
  frameborder="0"
  style="margin-bottom: 0; padding: 0;"
></iframe>

This histogram displays the distribution of cooking times for all recipes in minutes. The figure indicates that majority of recipes have a cooking time that falls between 0 and 100 minutes. Despite our removal of outlier values, the plot is still heavily right skewed. This could potentially cause issues when exploring trends with recipes across the 0-600 minute range. Specifically, the trends could become less reliable as cooking time exceeds 100 minutes due to the limited data availability and potential variability in those longer cooking times.

**Univariate Plot #2**

<iframe
  src="figures/univariate2.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

This figure shows an overlayed histogram comparison of cooking times for recipes classified as "healthy" and "unhealthy". The vertical dashed lines in this figure show that recipes that are healthy take less time to prepare on average than unhealthy recipes. This is an interesting discovery as our initial thinking was that healthier recipes would generally take longer to make.

### Bivariate Analysis

**Bivariate Plot**

<iframe
  src="figures/bivariate.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

This scatter plot displays the trend between 'health_score' of recipes and their cooking time in 'minutes'. Interestingly, despite our findings from Univariate Plot #2 (that healthy recipes have a lower average cooking time), this plot shows that as 'health_score' increases, so does cooking time.

Together, these two plots indicate an example of Simpson's Paradox, because when we look at the overall trend of the entire dataset, we observe a positive relationship between 'health_score' and 'minutes' but when we split the data at the median 'health_score', we find that healthy recipes have a lower average cooking time.

Looking at the Bivariate Plot, we can hypothesize that the reason for this paradox is that the blue (healthy) points are highly concentrated in a vertical band that is evenly distributed across the y-axis, resulting in a moderate average cooking time. Meanwhile the orange (unhealthy) points are more spread out, with a significant number of long recipes skewing the average cooking time of unnhealthy recipes to be higher.

### Interesting Aggregates

In this section, we will further investigate the reason for the Simpson's Paradox described above, using a grouped table. We are interested in finding out how the average cooking time ('minutes') varies across different 'health_scores,' and how this trend translates to the labels of 'healthy' and 'unhealthy'.

First, we created a new column called 'health_score_bin' by segmenting the 'health_score' column into bins of [-40, -30, -20, -10, 0, 0.5, 2]. Then, we grouped by 'health_score_bin' and 'is_healthy', aggregating by both mean and count. We obtained the resulting DataFrame:

| ('health_score_bin')   | ('is_healthy')   |   ('minutes', 'mean') |   ('minutes', 'count') |
|:---------------------------|:---------------------|----------------------:|-----------------------:|
| -40 to -30                 | False                |               81.3636 |                    176 |
| -30 to -20                 | False                |               67.1911 |                    293 |
| -20 to -10                 | False                |               65.1838 |                   1398 |
| -10 to 0                   | False                |               69.8318 |                  69178 |
| 0 to 0.5                   | False                |               63.5126 |                  41927 |
| 0.5 to 2                   | True                 |               48.7758 |                 118684 |

This grouped table shows an increasing trend of mean cooking time across the 2nd, 3rd, and 4th bins. With a 'health_score' range of -30 to 0, these bins dominated the scatter plot, giving the sense that as 'health_score' increases, so does 'minutes'. However, this entire trend was within the unhealthy category, and we see that the healthy category (contained within the 'health_score_bin' 0.5 to 2), actually has the lowest average cooking time.


## Assessment of Missingness

We were interested in exploring the missingness of various columns in our DataFrame. The first step we took was to count the number of missing values in each column. We received the following counts:

| Column Name            | Missing Values |
|------------------------|---------------:|
| name                  | 1              |
| id                    | 0              |
| minutes               | 0              |
| contributor_id        | 0              |
| submitted             | 0              |
| tags                  | 0              |
| nutrition             | 0              |
| n_steps               | 0              |
| steps                 | 0              |
| description           | 113            |
| ingredients           | 0              |
| n_ingredients         | 0              |
| user_id               | 1              |
| date                  | 1              |
| rating                | 14,740         |
| review                | 56             |
| average_rating        | 2,706          |
| standardized_calories | 0              |
| standardized_fat      | 0              |
| standardized_sugar    | 0              |
| health_score          | 0              |
| is_healthy            | 0              |

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
  height="450"
  frameborder="0"
></iframe>

We found an observed tvd of 0.153 (represented by the vertical red line) and a p-value 0.002, which leads us to *reject the null hypothesis*. 

This test indicates that the missingness of 'description' depends on 'health_score'.

**Investigating the Missingness Dependency of 'Description' on 'Minutes'**

**Null Hypothesis**: The distribution of 'minutes' when 'description' is missing is the same as the distribution when 'description' is not missing.

**Alternative Hypothesis**: The distribution of 'minutes' when when 'description' is missing is not the same as the distribution when 'description' is not missing.

**Test Statistic**: total variation distance

**Significance Level**: 0.05

**Number of Permutations**: 1000

<iframe
  src="figures/missingness2.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

We found an observed tvd of 0.227 (represented by the vertical red line) and a p-value of 0.225, which leads us to *fail to reject the null hypothesis*. 

This test does not provide evidence that the missingness of 'description' depends on 'health_score'.


## Hypothesis Testing

We are interested in exploring the relationship between the healthiness of recipes and their cooking time. Therefore, we decided to implement a permutation test to find out if recipes classified as "healthy" (recipes with a 'health_score' of >= 0.5) have the same distribution of cooking times as those recipes classified as "unhealthy" (recipes with a 'health_score' of < 0.5). We have already seen from Univariate Plot #2 that healthy recipes in our dataset have a lower average cooking time. Now we will investigate if this difference was purely by chance or due to a difference in distributions of cooking times.

**Null hypothesis**: The average number of minutes needed to prepare recipes that are classified as healthy is the same as the average number of minutes needed to prepare recipes that are classified as unhealthy.

**Alternative Hypothesis**: The average number of minutes needed to prepare recipes that are classified as healthy is less than the average number of minutes needed to prepare recipes that are classified as unhealthy.

**Test Statistic**: Difference of means (mean_time<sub>healthy</sub> - mean_time<sub>unhealthy</sub>)

**Significance Level**: 0.05

**Number of Permutations**: 1000

The observed test statistic for this test is -18.669, meaning that for this dataset, recipes classified as healthy took 18.669 minutes less to make than recipes classified as unhealthy, on average.

<iframe
  src="figures/permutation_test.html"
  width="800"
  height="520"
  frameborder="0"
></iframe>

The graph above displays the observed_tvd as a vertical red line, showing a significant deviation from the distribution. Consequently, the calculated p-value is 0.0.

Because our p-value 0.0 < the significance level of 0.005, we *reject the null hypothesis*. The distribution of cooking times for healthy recipes has a lower mean than the distribution of cooking times for unhealthy recipes. 

After discovering these results, we reasoned that this could be due to outliers in either group: healthy outliers could include salads and sandwiches (healthy foods that do not take long to prepare), while unhealthy outliers could be deep-fried foods or baked goods (unhealthy foods that would take a long time to prepare).


## Framing a Prediction Problem

Our permutation test from the previous section showed us that 'healthy' and 'unhealthy' recipes do not necessarily have the same distribution of cooking times. This indicates that the nutritional information reflected in the 'health_score' could be a meaningful feature in predicting cooking time of a recipe. In the following sections, we will address the following prediction problem:

**How many minutes will it take to cook a recipe based on its number of calories, amount of fat, and amount of sugar?**

This is a regression problem, as the column we are planning to predict, 'minutes', is a numerical variable signifying cooking time.

We will use **root mean squared error** (RMSE) as the main metric to evaluate our models. The reason for this choice is that the distribution of our response variable, 'minutes' was highly right skewed. RMSE heavily penalizes larger errors (due to the squaring step in the RMSE calculation), so it is particularly useful for minimizing error for extreme values. Additionally, it returns a value in the same units as the response variable (minutes), allowing for better interpretability and evaluation of results. We will also use the **R^2 score** to measure how much of the variance in the target variable is explained by the model. Together, both of these metrics will help us effectively compare performance across models.


## Baseline Model

### Description of Model
For the baseline model, we decided to use a Linear Regression model to predict the cooking time of food recipes based on the nutritional information of the food. Using this model gives us a basic starting point that we can use to establish a benchmark before training more complicated models such as Random Forest regression, Gradient Boosting, and KNN Regressor.
The y variable which we wanted to predict in our analysis is cooking time in minutes. We first transformed this variable using a log function because of the right-skewed distribution that we observed in cooking times (see Univariate Plot #1). Transforming helped us linearize the relationship between the features and y variable.

### Model Features
The features that we included in our baseline model are the following: 

- standardized_calories: The standardized number of calories that the recipe contains (quantitative)
- standardized_fat: The standardized amount of fat that the recipe contains (quantitative)
- standardized_sugar: The standardized sugar content of the recipe (quantitative)

All three features in our baseline model are quantitative variables that had already been standardized in the data cleaning part above. The standardization helps address scale differences between the nutritional variables and improves the model's stability. As all our features were already preprocessed and no categorical variables were included in the baseline model, we did not need to add any extra encoding techniques.

### Model Evaluation and Assessment
To evaluate our baseline model's performance, we used two metrics, RMSE and R^2. The scores that our baseline model achieved are below:

1. **RMSE**: 0.94
2. **R^2**: 0.06

While our baseline model gave us a starting point to work with, we knew that there was room for improvement. One of the reasons why the Linear Regression model did not perform optimally is because it assumes a linear relationship between the features and y variable. Since cooking times for various food recipes have more complex interactions between ingredients (features), the linear model would not be able to identify those relationships. In addition, our baseline model uses three nutritional factors, which may not include all the factors that influence cooking time. Some other factors that we posed include the method of cooking and steps of preparation. Lastly, the linear model allows us to identify how only one feature can affect cooking time but not how two features interact with each other. Thus, our baseline model using linear regression gives us satisfactory performance, but many things could be improved to better represent the relationship between cooking time and nutritional content.


## Final Model

### Feature Engineering

To improve the performance of our baseline model, we implemented 2 feature engineering techniques to help improve the precision of our baseline model. The first one was **Polynomial Features** which helped us to interpret the non-linear relationship between the nutritional features (standardized_calories, standardized_fat, and standardized_sugar). This transformation approach creates interaction values that allow the model to learn how combinations of these nutritional factors might asynchronously affect cooking time. This helps the model discover relationships that exist in cooking. The second technique is a **Quantile Transformer**, which essentially just normalizes the distributions of the features that we have selected. It helps to handle skewed data by mapping the original values to a normal distribution. There are many recipes with moderate calorie counts and fewer with extremely high values. By normalizing these distributions, we help the model treat outliers more appropriately and improve its ability to learn from the entire range of nutritional profiles.


## Fairness Analysis

After training and evaluating our model, we were curious to see how it would perform on recipes with various numbers of ingredients. We decided to analyze the fairness of our model by evaluating how it performs on **recipes with more than 9 ingredients** vs. **recipes with less than 9 ingredients**. 