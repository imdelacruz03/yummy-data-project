# Introduction: Recipes and Their Ratings: What makes a "Good" Recipe
Hi! My name is Isabela de la Cruz, for this project I chose to explore the "Recipes and Ratings" dataset. The Recipes dataset has 83,782 rows and stores the following columns: name of the recipe, its id, time it takes to make in minutes, contributor id, date it was submitted, nutrition, steps, description, ingredients, and number of igredients. The Ratings dataset has 731,927 and stores the following columns: user id, recipe id, date, rating, and review. 

With these datasets I want to answer this question, "What factors influence how well a recipe is rated? More specifically, what characteristics are associated with high ratings and what characteristics tend to lower ratings" 

Online recipe platforms are home to thousands of user submitted recipes each with various prepreation time, ingredients and difficult levels. Understanding what makes a recipe highly rated can help home cooks identify better recipes, and help cooks submit popular recipes. 

# Data Cleaning and Exploratory Data Analysis
Cleaning Steps:
- The recipes dataset required several cleaning steps:
- Converted the minutes column to numeric and removed non-positive values, since a cook time of zero or negative is not realistic.
- Created a log10_minutes column so that extremely long recipes would not dominate visualizations.
- Ensured recipe IDs aligned with the ratings dataset during merging.

The ratings dataset required:
- Converting the rating column to numeric and dropping missing values.
- Converting the date column to datetime.
- Removing invalid dates.
- Creating a new season feature based on the month of rating submission:
    - Winter: Dec, Jan, Feb,
    - Spring: Mar, Apr, May
    - Summer: Jun, Jul, Aug
    - Fall: Sep, Oct, Nov

Below is a sample of the cleaned ratings dataset:
```
recipe_id   rating   date          season
40893       5        2011-12-21    Winter
85009       5        2010-02-27    Winter
85009       5        2011-10-01    Fall
120345      2        2011-08-06    Summer
120345      2        2015-05-10    Spring
```

# Distribution of Recipe Ratings
<iframe src="assets/rating_hist.html" width="800" height="600" frameborder="0"></iframe>

Most ratings cluster around 4–5 stars, showing a strong positive skew typical of online reviews. Very few ratings fall below 3 stars, suggesting users may only leave ratings when satisfied.

# Distribution of Cook Times (log scale)
<iframe src="assets/cooktime_hist.html" width="800" height="600" frameborder="0"></iframe>

Cook times, on a log scale, form a unimodal distribution. Most recipes take between 10 and 100 minutes, but there is a long tail of very long recipes. This motivates comparing the ratings of “short” and “long” recipes.

# Average Rating by Season Submitted
<iframe src="assets/season_bar.html" width="800" height="600" frameborder="0"></iframe>

Average ratings vary slightly across seasons. Spring and Summer recipes have the highest average ratings, while Winter and Fall recipes are slightly lower. The differences are small, suggesting seasonality has limited impact but is worth investigating.

# Ratings for Short vs Long Cook Times
<iframe src="assets/cookgroup_box.html" width="800" height="600" frameborder="0"></iframe>

I divide recipes into two groups based on the median cook time. Short recipes show a slightly higher median rating compared to long recipes, though the distributions overlap heavily. This motivates hypothesis testing.

# Interesting Aggregates
Below is a grouped table showing average rating and number of ratings by season and cook-time group:

| season | cook_group | avg_rating | num_ratings |
|--------|------------|------------|-------------|
| Fall   | long       | 4.2869996  | 27453       |
| Fall   | short      | 4.4319321  | 26738       |
| Spring | long       | 4.3425963  | 30012       |
| Spring | short      | 4.4537915  | 33641       |
| Summer | long       | 4.3621699  | 26968       |
| Summer | short      | 4.4904724  | 35056       |
| Winter | long       | 4.2553680  | 27384       |
| Winter | short      | 4.3711405  | 27173       |


Across all seasons, short recipes consistently receive slightly higher ratings than long recipes. Short Summer recipes, in particular, show the highest average rating and highest engagement. This supports the hypothesis that both season and cook time may relate to user ratings.

# Assessment of Missingness
Column Selected: review

The review text column in the ratings dataset is plausibly Not Missing At Random (NMAR). Users are more likely to write a review when they have a strong opinion (positive or negative). Users who feel neutral often skip writing a review. Because this missingness depends on internal user sentiment — which is not recorded — it cannot be explained using observed variables. To convert this mechanism to MAR, we would need data about users’ reviewing habits.

# Missingness Dependency Tests
I performed two permutation tests to assess dependency of review missingness:
- Does missingness in review depend on rating?
- Does missingness in review depend on minutes?
I created an indicator review_missing and compared mean values across groups.

# Missingness ∼ Rating
Observed difference: 0.3834
Permutation test p-value: 0.0
This indicates a statistically significant difference. Missingness of review depends on rating.

# Missingness ∼ Minutes
Observed difference: 35.2348
Permutation test p-value: 0.655
This suggests cook time does not meaningfully relate to whether a user leaves a written review.

# Visualization of Missingness Analysis
<iframe src="assets/missingness_rating.html" width="800" height="600" frameborder="0"></iframe>

# Hypothesis Testing
I conducted two hypothesis tests using permutation methods.

# Test 1: Are Winter and Summer Mean Ratings Different?
Null hypothesis: The average rating in Winter equals the average rating in Summer.
Alternative hypothesis: The two seasons have different mean ratings.
Test statistic: Absolute difference in means.
Observed difference: 0.0528
p-value: 0.0

Conclusion: There is a statistically significant difference between Winter and Summer ratings, though the effect size is small.

<iframe src="assets/hypothesis_winter_summer.html" width="800" height="600" frameborder="0"></iframe>

# Test 2: Are Short Cook Time and Long Cook Time Ratings Different?
Null hypothesis: Mean rating of short recipes equals that of long recipes.
Alternative hypothesis: The groups differ.
Observed difference: 0.1289
p-value: 0.0

Conclusion: Short recipes receive higher ratings on average than long recipes, with statistical significance.

<iframe src="assets/hypothesis_cooktime.html" width="800" height="600" frameborder="0"></iframe>

# Prediction Problem
The goal is to predict whether a recipe will be highly rated (rating ≥ 4). This is a binary classification problem.

Response variable: high_rating (1 if rating ≥ 4, else 0)
Evaluation metric: Accuracy

Accuracy is appropriate because classes are moderately imbalanced but not extremely so, and the cost of false positives and false negatives is relatively symmetric in this context.

To ensure valid prediction, only features available before a user leaves a rating were used, such as cook time and submission season.

# Baseline Model
The baseline model uses:
- minutes
- season (one-hot encoded)

Model: Logistic Regression
Training accuracy: 0.72378
Test accuracy: 0.72379
This establishes a reference point for improvement.

# Final Model
To improve performance, the final model added engineered features:
- log10_minutes (captures the skew in cook-time distribution)
- is_short (binary indicator for short vs long recipes)

I used a GridSearchCV over Logistic Regression with hyperparameter:
- C ∈ {0.01, 0.1, 1, 10}
Best hyperparameter: C = 0.01

# Final model performance:
Training accuracy: 0.72378
Test accuracy: 0.72379

The final model performs similarly to the baseline, suggesting that cook time and season provide limited predictive power for rating.

# Fairness Analysis
I evaluated whether the final model performs differently for short versus long recipes.
Groups:
Group X: Long cook-time recipes
Group Y: Short cook-time recipes
Evaluation metric: Accuracy

Observed accuracies:
Long recipes: 0.7136
Short recipes: 0.7359
Observed difference: −0.0223

Permutation test p-value: 0.112

Conclusion: There is no statistically significant evidence that the model is unfair with respect to cook time group.

# Final Thoughts
This project demonstrates how exploratory analysis, statistical inference, and machine learning can be combined to understand rating behavior. While some factors show small differences, predicting recipe quality remains challenging, and user behavior introduces complexity beyond the observable features.
