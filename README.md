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




