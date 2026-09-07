# Cooking Time and Recipe Ratings

**Author:** Sahil Jain

## Introduction

I looked at recipes and reviews from food.com. There are 83,782 recipes and 731,927 reviews.

My question: **do recipes that take less time get better ratings?**

Main columns I used: `minutes` (cooking time), `n_steps`, `n_ingredients`, `nutrition`, and `avg_rating` (the average rating for each recipe).

## Data Cleaning and Exploratory Data Analysis

I merged the two datasets, then changed all ratings of 0 to missing. Food.com only lets you rate 1 to 5, so a 0 means the person left a review without a rating. Counting those as zeros would make recipes look worse than they are.

I then took the average rating for each recipe, and split the `nutrition` column into separate number columns for calories, fat, sugar, and so on.

Cooking time is very skewed. Half of recipes take under 35 minutes, but the longest one says 1,051,200 minutes. Ratings are also skewed — most recipes are close to 5 stars.

Here is the average rating by cooking time and number of steps:

| time | 1-5 steps | 6-10 steps | 11-20 steps | 20+ steps |
|---|---|---|---|---|
| under 15 min | 4.685 | 4.654 | 4.634 | 4.635 |
| 15-30 min | 4.596 | 4.624 | 4.634 | 4.685 |
| 30-60 min | 4.587 | 4.599 | 4.612 | 4.647 |
| 2-24 hr | 4.530 | 4.562 | 4.626 | 4.660 |

For quick recipes, more steps means a lower rating. For longer recipes, more steps means a higher rating.

## Assessment of Missingness

I think `description` is NMAR. People leave it blank when they have nothing interesting to say, so whether it is missing depends on what they would have written.

I tested whether missing ratings depend on other columns. Recipes with no rating have 1.49 fewer steps on average, with a p-value below 0.001. So missing ratings do depend on `n_steps`. Simple recipes get fewer reviews.

## Hypothesis Testing

**Null:** cooking time does not affect rating.
**Alternative:** short recipes (30 minutes or less) get higher ratings.
**Test statistic:** difference in average rating between the two groups.
**Significance level:** 0.05.

The observed difference was 0.035 and the p-value was below 0.0001. **I reject the null.**

But 0.035 on a 1 to 5 scale is very small, so this does not really matter in practice. And since this is not an experiment, it does not show that cooking time causes better ratings.

## Framing a Prediction Problem

I predict `avg_rating`. This is a number, so it is a **regression** problem. I use **RMSE** because it is in the same units as the rating.

All my features are known as soon as a recipe is posted: `minutes`, `n_steps`, `n_ingredients`, and the nutrition columns. I do not use anything from the reviews, because those do not exist yet when a recipe is new.

## Baseline Model

I used linear regression with `minutes`, `n_steps`, and `time_bin` (one-hot encoded).

RMSE was **0.6380**. Just guessing the average rating every time gives **0.6385**.

So the model is barely better than guessing. This makes sense, because almost every recipe is rated close to 5.

## Final Model

I added `n_ingredients`, `calories`, `protein`, `sugar`, and `total fat`.

I took the log of `minutes` and `calories` because both have huge outliers, and a log keeps those from taking over the model. I scaled the other numbers so they are all on a similar range.

I switched to a random forest because the table above showed the pattern flips direction — more steps helps long recipes but hurts short ones. A straight line cannot do that, but a tree can.

I used `GridSearchCV` to pick the settings. The best were `max_depth=5`, `min_samples_leaf=20`, `n_estimators=50`.

RMSE was **0.6374**, which is 0.0006 better than the baseline.

That is a tiny improvement. The search also picked the simplest settings I gave it, which usually means the features are not telling the model much.

## Fairness Analysis

**Group X:** short recipes (30 minutes or less). **Group Y:** long recipes.
**Metric:** RMSE.

**Null:** the model works equally well for both groups.
**Alternative:** the model works better for one group.

RMSE was 0.6097 for short recipes and 0.6591 for long ones. The difference was 0.0493 and the p-value was 0.001.

**I reject the null.** The model is worse at predicting long recipes, probably because their ratings vary more.
