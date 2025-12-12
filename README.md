# yummy-data-project
This is a UCSD DSC80 project. 

# Introduction
Hi! My name is Isabela de la Cruz, for this project I chose to explore the "Recipes and Ratings" dataset. The Recipes dataset has 83,782 rows and stores the following columns: name of the recipe, its id, time it takes to make in minutes, contributor id, date it was submitted, nutrition, steps, description, ingredients, and number of igredients. The Ratings dataset has 731,927 and stores the following columns: user id, recipe id, date, rating, and review. 

With these datasets I want to answer this question, "What factors influence how well a recipe is rated? More specifically, what characteristics are associated with high ratings and what characteristics tend to lower ratings" 

Online recipe platforms are home to thousands of user submitted recipes each with various prepreation time, ingredients and difficult levels. Understanding what makes a recipe highly rated can help home cooks identify better recipes, and help cooks submit popular recipes. 

# Data Cleaning and Exploratory Data Analysis

To prepare the data for analysis, I first cleaned both datasets.
For the Recipes dataset, I converted the minutes column to numeric and removed recipes with non-positive cook times, since these values are unrealistic. I also created a new log10_minutes column so that extremely long recipes would not dominate the distribution.

For the Ratings dataset, I converted rating to numeric and dropped rows with missing ratings. I also converted the date column to a datetime type and removed invalid dates. From the cleaned dates, I engineered a new season feature (Winter, Spring, Summer, Fall) based on the month in which the rating was submitted.

Below is a preview of the cleaned Ratings dataset used in later steps:

(Paste your table here — using either HTML or CSV format.)

# Distribution of Recipe Ratings
<iframe src="assets/rating_hist.html" width="800" height="600" frameborder="0"> </iframe>

The distribution of ratings is extremely skewed toward the high end. Most ratings are 4 or 5 stars, and very few recipes receive low ratings. This suggests that users tend to rate recipes positively, which is common in online rating systems.

# Distribution of Cook Times (log scale)
<iframe src="assets/cooktime_hist.html" width="800" height="600" frameborder="0"> </iframe>

Cook times (on a log scale) are roughly unimodal, with most recipes taking between 10 and 100 minutes. There are also some very quick recipes and a long tail of extremely time-consuming ones. This motivates examining whether “short” versus “long” recipes receive different ratings..

# Average Rating by Season Submitted
<iframe src="assets/season_bar.html" width="800" height="600" frameborder="0"> </iframe>

Average ratings vary only slightly by season. In this dataset, Spring and Summer recipes receive the highest average ratings, while Winter and Fall recipes receive slightly lower ratings. However, the differences are small, suggesting that season likely has only a weak relationship with rating.

# Ratings for Short vs Long Cook Times
<iframe src="assets/cookgroup_box.html" width="800" height="600" frameborder="0"> </iframe>

The boxplot compares ratings for recipes with short versus long cook times, where “short” recipes are those at or below the median cook time. The distributions overlap heavily, but short recipes have a slightly higher median rating than long recipes. This observation motivates a hypothesis test to determine whether the difference is statistically significant.

# Interesting Aggregates

(Paste the markdown or HTML table from agg_table.to_html() here.)

The table above shows the average rating and number of ratings for each combination of season and cook-time group. Across every season, short recipes consistently receive higher average ratings than long recipes, and some combinations—such as short recipes in Summer—receive particularly high ratings and a large number of reviews. This supports the idea that both season and cook time may influence how well a recipe is rated.


# Assessment of Missingness

## NMAR Column: `review`

I believe the `review` column is **Not Missing At Random (NMAR)**. A user is more likely to leave a written review when they have a particularly strong opinion about a recipe—either very positive or very negative. Users who feel neutral or unmotivated often skip writing a review entirely.

Because this missingness depends on the user’s **unobserved internal satisfaction**, which is not recorded anywhere in the dataset, the missingness mechanism cannot be explained using observed variables alone. Therefore, `review` is plausibly NMAR.

To make this mechanism MAR, we would need additional information about user behavior, such as whether a user typically writes reviews or only writes them in exceptional situations.

---

## Missingness Dependency Tests

To understand whether missingness in `review` is related to other observed variables, I performed permutation tests. The missingness indicator used was:
ratings["review_missing"] = ratings["review"].isna()



I tested whether missingness depends on:

- `rating`  
- `minutes`  

---

### **Test 1: Does missingness depend on `rating`?**

<iframe  
  src="assets/missingness_rating.html"  
  width="900"  
  height="350"  
  frameborder="0">  
</iframe>

- **Observed difference in mean rating:** 0.3834  
- **p-value:** 0.0  

Because the p-value is effectively zero, I **reject the null hypothesis**.

➡️ **Conclusion:** Missingness in `review` *does* depend on rating. Users who give very low or very high ratings are more likely to write a review.

---

### **Test 2: Does missingness depend on `minutes`?**

- **Observed difference in mean cook time:** 35.2348  
- **p-value:** 0.655  

Since the p-value is large, I **fail to reject the null hypothesis**.

➡️ **Conclusion:** Missingness in `review` does **not** depend on recipe cook time. Whether someone writes a review is unrelated to how long the recipe takes to make.

---

# Hypothesis Testing
# Hypothesis Test 1 — Do recipes submitted in Winter and Summer receive different ratings?

Research question:
Does the season in which a recipe is submitted affect its rating?

Hypotheses:

H₀: The mean rating for Winter and Summer recipes is the same.

H₁: The mean ratings differ between Winter and Summer.

Test statistic: T = |mean_winter - mean_summer|
Observed statistic: T_obs = 0.0528
p-value: p = 0.0

Interpretation
The observed difference (≈0.053 stars) is extremely unlikely under the null hypothesis.
Because p = 0, we reject H₀, meaning:

Recipes submitted in Winter and Summer do have statistically different average ratings.
The difference is small in magnitude, but statistically meaningful due to the large dataset.

# Hypothesis Test 2 — Do short and long cook-time recipes receive different ratings?

Research question:
Do recipes that take longer to cook get rated differently?

Hypotheses:

H₀: The mean rating of short-cook-time and long-cook-time recipes is the same.

H₁: The mean ratings differ between short and long recipes.

Test statistic: T = |mean_short​ − mean_long​|
Observed statistic: T_obs​=0.1289
p-value: p = 0.40

Interpretation

The observed difference (~0.13 stars) is not unusual under the null hypothesis.
Because p = 0.40, we fail to reject H₀, meaning:
There is no statistically significant difference in ratings between short and long cook-time recipes.
Even though short recipes appear slightly higher-rated on average, the difference is not statistically meaningful.

# Framing a Prediction Problem
To explore whether recipe characteristics can help predict user ratings, I built a simple classification model. I defined a high rating as a recipe receiving 4 or 5 stars, and created a binary column:

1 = high rating

0 = rating 0–3

Across the entire dataset, 72.38% of all reviews are high ratings, confirming the strong skew observed earlier.

Model Setup

I used only one predictor:

log10 cook time

After cleaning the data and merging recipe information with ratings, I split the dataset into:

Training set: 187,540 rows

Test set: 46,885 rows

I trained a logistic regression classifier to predict whether a review was a high rating.

Model Performance

Training accuracy: 0.72378

Test accuracy: 0.72379

These values are almost exactly equal to the marginal probability of a high rating (72.38%). This means:

The model does not meaningfully improve upon a trivial baseline that always predicts “high rating.”
Cook time alone has extremely weak predictive power, which matches the findings of the hypothesis test in Step 4 (p = 0.40).

Interpretation

This modeling exercise confirms that:

Most recipes receive high ratings regardless of cook time

Cook time is not a useful predictor of rating behavior

Additional features (ingredients, cuisine, number of steps, user metadata, etc.) would likely be required to build a meaningful predictive model

# Baseline Model
The goal of this project was to explore what factors influence whether a recipe receives a high rating. Using over 80,000 recipes and more than 730,000 user interactions, I examined how rating patterns relate to cook time, seasonality, and missingness, and evaluated whether simple models could predict recipe ratings. Across my analyses, several clear themes emerged.

More than 72% of all ratings in the dataset are 4 or 5 stars.
The distribution is extremely right-skewed, indicating that users tend to rate recipes positively regardless of other characteristics. This strong positivity bias is important to keep in mind when interpreting later results.

Although shorter recipes had slightly higher average ratings in exploratory plots, the difference was small.
A hypothesis test comparing short vs. long cook times found:

Observed difference: 0.1289

p-value: 0.40

Since the p-value is large, we fail to reject the null hypothesis.

Cook time does not meaningfully affect recipe ratings.

This was reinforced by the prediction model in Step 5:
A logistic regression using cook time alone performed no better than a naive baseline.

When comparing ratings from Winter vs. Summer:

Observed difference: 0.0528

p-value: 0.0

This indicates a statistically detectable difference in average ratings between seasons.

However, the effect size is extremely small (around 0.05 stars) and not practically important.
Season matters statistically, but not in a meaningful way.

The written review column is most likely Not Missing At Random (NMAR) because:

Users with strong opinions (positive or negative) are more likely to write a review.

Neutral or unmotivated users tend to skip writing text.

This missingness depends on unobserved internal user sentiment, not on recipe features.

Permutation tests showed:

Missingness does depend on rating (p ≈ 0.0)

Missingness does not depend on cook time (p ≈ 0.655)

This supports the NMAR conclusion.

The model in Step 5 used only cook time as a predictor. Its performance:

Training accuracy: 0.72378

Test accuracy: 0.72379

These match the overall high-rating proportion (72.38%), meaning the model does not improve upon blindly predicting “high rating” every time.

Cook time alone is not a useful predictor.

To build a more accurate model, richer features (ingredients, steps, cuisine, review text, etc.) would be necessary.

# Final Model
For the final model, I aimed to improve upon the baseline classifier by adding new engineered features, performing appropriate preprocessing, and tuning hyperparameters within a single sklearn pipeline.

In addition to the baseline features (log10_minutes, n_ingredients, and season), I engineered:
- n_steps – the number of steps in the recipe, representing complexity.
- One-hot encoded season – capturing the small but statistically significant seasonal variation observed earlier.

These features reflect reasonable aspects of the data-generating process: more complex recipes or seasonal user behavior could plausibly influence ratings.

I used a Logistic Regression classifier wrapped in a pipeline consisting of:
- StandardScaler for numeric features
- OneHotEncoder for season
- Logistic regression as the estimator

All transformations and model fitting occurred within a single pipeline to avoid data leakage.

I tuned the regularization strength C using GridSearchCV over: C ∈ {0.01, 0.1, 1, 10}
Best hyperparameter found: C = 0.01

Final Model Performance:
Traning Accuracy = 0.72378
Testing Accuracy = 0.72379

The final model did not improve predictive performance. This result aligns with earlier analysis:
- Cook time and season have very weak relationships with rating.
- Most recipes receive high ratings regardless of structure or complexity.
- Simple recipe metadata is not sufficient to predict rating outcomes.

A more sophisticated model would require richer features (e.g., NLP on ingredients or review text, user-level behavior, cuisine categories).














# Fairness Analysis





























