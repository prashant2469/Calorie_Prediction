# Calorie Prediction
### Name: Prashant Potluri

# Overview
This data science project, conducted at UCSD, focuses on examining how nutritional components of recipes, such as fat content, sugar, and protein, influence the prediction of overall calorie count, while uncovering unexpected patterns and insights in recipes categorized as 'high-fat' versus 'low-fat.'

# Table of Contents
- [Introduction](#introduction)
- [Data Cleaning and Exploratory Data Analysis](#datacleaning)
- [Framing the Problem](#framingtheproblem)
- [Baseline Model](#baselinemodel)
- [Final Model](#finalmodel)
- [Fairness Analysis](#fairnessanalysis)

<!-- #region -->
# Introduction <a name="Introduction"></a>

This data science project, conducted at UCSD, investigates how calorie content and fat levels in recipes influence their popularity among users. With the increasing awareness of health and nutrition, understanding the relationship between a dish's calorie count, fat content, and user ratings is critical. Our goal is to explore whether low-calorie, low-fat recipes are more highly rated than their higher-calorie, fattier counterparts. We also aim to uncover any patterns or trends that may provide insights into user preferences for healthy versus indulgent recipes.

The dataset, sourced from food.com, includes over 83,000 recipes and nearly 732,000 user reviews collected since 2008. It provides detailed information on nutrition, including calories, fat, sugar, and preparation times. By merging the recipes and reviews data, we can analyze the average ratings for dishes while considering their nutritional profile and cooking time. This project seeks to determine whether recipes with fewer calories and lower fat content align with user preferences or whether indulgent meals still maintain a strong appeal. Ultimately, we hope to shed light on how health-conscious trends influence recipe popularity.

### This is a description of what the recipes dataframe contains (83782 rows):

|    Column  |  Desciption |
|-----------:|------------:|
|    'name' |       Recipe name |
|     'id' |       Recipe ID |
|      'minutes' |       Minutes to prepare recipe |
|     'submitted' |      User ID who submitted this recipe |
| 'tags' |      Food.com tags for recipe |
|     'nutrition' |      Nutrition information in this form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
|     'n_steps' |      Number of steps in the recipe |
|    'steps' |      Step by step instructions to follow |
|     'description' |      A description of what the recipe makes |

### This is a description of what the reviews dataframe contains (731927 rows):

|    Column  |  Desciption |
|-----------:|------------:|
|    'user_id' |       User ID |
|     'recipe_id' |       Recipe ID |
|      'date' |       Date of interaction|
|     'rating' |      Rating (1-5) |
| 'review' |      Review text |

<!-- #endregion -->
<!-- #region -->

# Data Cleaning and Exploratory Data Analysis <a name="datacleaning"></a>

1. We merged the 'reviews' and 'recipes' data frames by linking the 'recipe_id' column from the reviews data frame to the 'id' column from the recipes data frame. This step ensured that every recipe in the dataset was associated with its corresponding ratings.

2. We grouped the data by 'recipe_id' and calculated the mean of all ratings for each recipe. This provided an average rating for every recipe, which was then added as a new column in the recipes data frame.

3. The 'nutrition' column, originally stored as a string, was converted into a list format. This allowed us to extract the calorie count and fat content from the nutrition data. These were added as new columns named 'calories' and 'total_fat' for easier analysis.

4. Using the 'total_fat' column, we introduced a classification system to categorize recipes as "high-fat" or "low-fat." Recipes with a total fat content greater than 25% of the daily value were labeled as "high-fat," while those with 25% or less were labeled as "low-fat." This classification was added as a new column called 'fat_classification.'

5. We addressed missing or invalid data by removing recipes with no average rating (e.g., NA or 0 values) or missing descriptions. The cleaned data was saved in a new data frame, recipes_nona. This process removed about 2,600 rows, but it left us with a sufficiently large and comprehensive dataset for further analysis.

### Our cleaned recipes data frame now contains no 'NA' values, with certain columns removed to focus on the key variables of interest for easier analysis.

| name                                 |     id |   minutes |   contributor_id | submitted   | nutrition                                     |   average_rating |   calories |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|:----------------------------------------------|-----------------:|----------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]      |                4 |         138.4 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | 2011-04-11  | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]  |                5 |        595.1 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]     |                5 |        194.8 |
| millionaire pound cake               | 286009 |       120 |           461724 | 2008-02-12  | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0] |                5 |        878.3 |
| 2000 meatloaf                        | 475785 |        90 |          2202916 | 2012-03-06  | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]    |                5 |        267.0 |

# Univariate Analysis <a name="univariateanalysis"></a>

### Here we examine the distribution of calorie counts in the recipes, focusing on understanding how calories vary across the dataset.


<iframe src="assets/caloriesdistribution.html" width=800 height=1000 frameBorder=0></iframe>

Above, we have visualized the distribution of calorie counts after removing outliers. There were some recipes with unrealistically high calorie values, which were likely errors or extreme outliers. To address this, we excluded values beyond 2 standard deviations from the mean. As shown in the histogram, most recipes have a moderate calorie count, with the majority falling below 500 calories. Recipes with extremely high calorie counts are much less common, indicating that the dataset primarily consists of lower to moderately caloric recipes. This analysis helps us understand the overall nutritional profile of the dataset and informs how we might categorize recipes for further insights.

# Bivariate Analysis <a name="bivariateanalysis"></a>

### Here we examine the scatter plot to explore the relationship between the fat content (measured in PDV) and calorie count among the recipes.

<iframe src="assets/calories_vs_fat.html" width=800 height=600 frameBorder=0></iframe>

The scatter plot of Calories vs. Fat shows a clear positive trend, as recipes with higher fat content tend to have more calories. This is expected, as fat is calorie-dense, contributing significantly to the total calorie count. However, there are some recipes with relatively high calories and lower fat content, indicating other nutritional components like sugar or carbohydrates may also play a role. The trendline helps visualize the general relationship, but there is still noticeable variability in the data points.

# Interesting Aggregates <a name="interestingaggregates"></a>

### Here we are aggregating the recipes by calorie buckets and analyzing how the average rating and fat content vary within each group.

| Calorie Bucket   | Rating Mean   | Rating Median   | Rating Std   | Fat Mean   | Fat Median   | Fat Std   | Recipe Count   |
|:-----------------|:-------------:|:---------------:|:------------:|:----------:|:------------:|:---------:|:--------------:|
| 0-200            | 4.634017      | 5.0             | 0.642686     | 7.023122   | 6.0          | 6.286557  | 24868          |
| 200-400          | 4.621413      | 5.0             | 0.640466     | 20.126607  | 19.0         | 11.120862 | 27305          |
| 400-600          | 4.621188      | 5.0             | 0.628379     | 36.603547  | 36.0         | 16.093335 | 14771          |
| 600-800          | 4.621446      | 5.0             | 0.628640     | 55.345790  | 56.0         | 22.465911 | 6556           |
| 800-1000         | 4.631750      | 5.0             | 0.639777     | 75.289387  | 76.0         | 30.388885 | 3034           |
| 1000+            | 4.613146      | 5.0             | 0.681995     | 116.044725 | 112.0        | 59.662950 | 3242           |
<!-- #endregion -->
<!-- #region -->
# Assessment of Missingness <a name="nmaranalysis"></a>
We understand that "Not Missing at Random" (NMAR) implies a relationship between the tendency of a value to be missing and its value. In this case, we think the missingness in the description column of our dataset might be NMAR. It is plausible that recipes with missing descriptions are simpler or less significant dishes that were uploaded without much detail. Alternatively, users may omit descriptions for recipes they consider obvious or not worth elaborating on. This potential cause-and-effect relationship suggests NMAR might explain the missingness of the description column.



# Missingness dependency <a name="missingnessdependency"></a>
We wanted to test if there is a correlation between the missingness in the average_rating column and other features in the dataset. Specifically, we analyzed whether missingness in average_rating is dependent on calories. Based on the plot below and our permutation test, there appears to be a significant correlation between these variables.


<iframe src="assets/permutation_calories_plotly_2.html" width=800 height=600 frameBorder=0></iframe>

The observed difference in calorie means between recipes with and without average_rating was 87.86, with a p-value of 0.0. Since the p-value is below 0.05, we reject the null hypothesis and conclude that there is a significant correlation between the missingness in average_rating and calories. This suggests that recipes with higher calorie counts are less likely to have missing average ratings, likely due to their popularity or complexity drawing more user engagement.


We also tested whether the missingness of the description column is dependent on calories. The results are shown below:

<iframe src="assets/permutation_calories_plotly.html" width=800 height=600 frameBorder=0></iframe>

The observed difference in calorie means between recipes with and without descriptions was 75.99, with a p-value of 0.254. Since the p-value is greater than 0.05, we fail to reject the null hypothesis. This suggests there is no significant correlation between the missingness of the description column and calories. The missingness in the description column is not influenced by the calorie content of the recipes.

<!-- #endregion -->
<!-- #region -->

# Hypothesis Testing <a name="hypothesistesting"></a>
## Permutation Test for correlation between protein content and average rating:

To investigate the difference in calorie distributions between high-fat and low-fat recipes, we performed a Kolmogorov-Smirnov (KS) test. This test is designed to determine whether two samples are drawn from the same distribution.


**Null Hypothesis (H0)**: The distributions of calories for high-fat and low-fat recipes are the same.

**Alternative Hypothesis (Ha)**:  The distributions of calories for high-fat and low-fat recipes are different.

**Test Statistics**: To evaluate whether the distributions of calories differ significantly between high-fat and low-fat recipes, we used the Kolmogorov-Smirnov (KS) test. The KS test is a non-parametric test that compares the cumulative distributions of two datasets to determine if they come from the same distribution. It is well-suited for comparing distributions, as it does not rely on assumptions of normality or equal variance.

**Significance Level**: To ensure the accuracy of our findings, we selected a significance level of 5% for this test.

The plot below showcases the coorelation from permutation testing of our test statics in 10000 permutations
<iframe src="assets/ks_test_permutation_plotly.html" width=800 height=600 frameBorder=0></iframe>

KS-statistic: 0.6957
P-value: 0.0

The observed p-value is below the significance level of 0.05, indicating that we reject the null hypothesis. This means there is strong evidence that the distributions of calories for high-fat and low-fat recipes are significantly different.

<!-- #endregion -->

<!-- #region -->
# Framing a Prediction Problem <a name="framingtheproblem"></a>
In this project, our objective is to create a model that predicts the calorie count in a recipe based on its nutritional information. The dataset provides various nutritional features such as total fat, sugar, sodium, saturated fat, carbohydrates, and protein. Since the target variable, calories, is a continuous numeric value, this is a regression task. Our goal is to accurately estimate calorie counts for recipes, leveraging the given nutritional data.

- **Response Variable**: The target variable for our model is **calories**, a crucial metric for individuals monitoring their dietary intake or managing fitness goals. Predicting calorie values is practical and valuable for recipes where such information might be missing, providing users with deeper insights into their food choices.


- **Evaluation Metrics**: Since we are solving a regression problem, we will evaluate our model using metrics such as RMSE, MAE, and R² values:
RMSE (Root Mean Squared Error): Provides a measure of the average magnitude of prediction error, with larger penalties for significant deviations between predicted and actual values.
MAE (Mean Absolute Error): Measures the average magnitude of the errors in a more interpretable way, without overly penalizing larger errors like RMSE does.
R² (Coefficient of Determination): Indicates how well our model explains the variance in the calorie data. An R² value close to 1 suggests a strong fit to the data.


- **Information Known**: At the time of prediction, we have access to all other nutrition features from the dataset (**total_fat, sugar, sodium, saturated fat, carbs, and protein**) except calories. By leveraging these features, we aim to predict the caloric content for recipes that do not provide it explicitly in their nutrition label.
<!-- #endregion -->




