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
|     'nutrition' |      Nutrition information in this form [calries (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
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
<!-- #endregion -->




