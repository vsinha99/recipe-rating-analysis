---
---

<header class="hero" markdown="1">

# Do Higher-Rated Recipes Have Different Nutritional Profiles?

<p class="authors" markdown="1">**By Varun Sinha and Rishab Kolan**</p>

<p class="badges"><span class="badge">UCSD DSC 80</span> <span class="badge">Final Project</span> <span class="badge badge-accent">Recipes &amp; Ratings</span></p>

This project looks at recipes from Food.com and the ratings people gave them. We want to find out whether a recipe's nutrition and complexity are related to how highly it is rated.

Live site: [vsinha99.github.io/recipe-rating-analysis](https://vsinha99.github.io/recipe-rating-analysis/)

</header>

<section class="section-card" markdown="1">

## Introduction

This project uses the Recipes & Ratings dataset from [Food.com](https://www.food.com). It comes in two files: a recipes file with one row per recipe (`RAW_recipes.csv`, 83,782 rows) holding ingredients, steps, prep time, and nutrition, and a reviews file with one row per user review (`interactions.csv`, 731,927 rows) holding the star ratings people left.

Our main question:

> Are recipe features like calories, nutrition, cooking time, number of ingredients, and number of steps related to the ratings users give?

Ratings affect what people decide to cook, but a high rating may depend on more than taste. It could also be linked to how simple, quick, or rich a recipe is. Looking at these features helps show what a rating might reflect.

We average the ratings for each recipe and join them onto the recipe table. After dropping recipes that never received a rating and the few whose nutrition could not be parsed, we are left with **81,173 recipes**. From here on, **one row is one recipe** together with its average rating.

The columns most relevant to our question are listed below.

| Column | Description |
|---|---|
| `rating` | A single user's star rating for a recipe, from 1 to 5. A value of 0 means the user reviewed without leaving a star, so we treat it as missing. |
| `average_rating` | The mean of all non-missing ratings a recipe received. This is our main measure of how well liked it is. |
| `calories` | Calories per serving, taken from the first value in the `nutrition` field. |
| `nutrition` | A list `[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]`. Every value except calories is a percent of daily value (PDV). |
| `minutes` | Reported preparation time in minutes. |
| `n_ingredients` | Number of ingredients in the recipe. |
| `n_steps` | Number of steps in the recipe. |

</section>

<section class="section-card" markdown="1">

## Data Cleaning and Exploratory Data Analysis

We cleaned the data so each row represents one recipe:

- **Handled missing ratings.** Replaced ratings of `0` with `NaN`, since a `0` means there was no rating rather than a true score of zero.
- **Averaged ratings per recipe.** Grouped the ratings for each recipe and took the mean to create `average_rating`, then merged it onto the recipe table.
- **Extracted nutrition.** Split the `nutrition` field into its seven parts and pulled `calories` (and the other PDV values) into their own numeric columns.
- **Complexity features.** Kept `n_ingredients` and `n_steps` as integer counts.
- **Prediction target.** Created a binary `high_rating` column that is `True` when a recipe's average rating is 4 or higher.

We only dropped rows that had no usable rating or unparseable nutrition. We left the extreme values in `minutes` and `calories` alone, since there is no clean rule to separate a real (if unusual) recipe from a typo, and removing them could bias the results.

### Cleaned DataFrame (head)

| `name` | `minutes` | `calories` | `n_ingredients` | `n_steps` | `average_rating` | `high_rating` |
|---|---|---|---|---|---|---|
| 1 brownies in the world best ev… | 40 | 138.4 | 9 | 10 | 4.0 | True |
| 1 in canada chocolate chip cookies | 45 | 595.1 | 11 | 12 | 5.0 | True |
| 412 broccoli casserole | 40 | 194.8 | 9 | 6 | 5.0 | True |
| millionaire pound cake | 120 | 878.3 | 7 | 7 | 5.0 | True |
| 2000 meatloaf | 90 | 267.0 | 13 | 17 | 5.0 | True |

---

### Univariate analysis

The distribution of calories is heavily right-skewed: most recipes fall under about 600 calories, with a long tail of very rich recipes. We clip the axis at the 99th percentile so the shape stays readable.

<iframe src="assets/univariate-calories.html" width="800" height="400" frameborder="0"></iframe>

### Bivariate analysis

Comparing calories across the two rating groups, the boxes overlap a lot, but low-rated recipes sit a little higher in calories than high-rated ones. The gap is small, which is what we test formally further down.

<iframe src="assets/bivariate-calories-rating.html" width="800" height="400" frameborder="0"></iframe>

---

### Interesting aggregates

Grouping by `high_rating` shows the average nutrition and complexity of each group.

| `high_rating` | mean `calories` | mean `n_ingredients` | mean `n_steps` | mean `minutes` |
|---|---|---|---|---|
| `False` (below 4) | 444.61 | 9.19 | 10.17 | 96.56 |
| `True` (4 or higher) | 425.96 | 9.21 | 10.05 | 112.42 |

High-rated recipes average slightly fewer calories and almost the same number of ingredients and steps. The `minutes` average looks higher for high-rated recipes, but that mean is dragged around by a few extreme outliers, so calories is the difference we trust most.

</section>

<section class="section-card" markdown="1">

## Assessment of Missingness

### NMAR analysis

We think the missingness of `rating` is likely NMAR (Not Missing At Random). When a rating is missing, it usually means a user left a review but skipped the star score. Whether someone bothers to leave a star can depend on how they actually felt about the recipe, so the missingness can depend on the unobserved rating itself, which is what NMAR means.

If we had more data, such as whether a user only viewed or saved a recipe, how long their review was, or whether they were a first-time reviewer, the missingness might be explained by those observed columns instead. That would make it MAR rather than NMAR.

### Missingness dependency (permutation tests)

Working at the review level, we built an indicator for whether a rating is missing and ran permutation tests against recipe features. The test statistic was the absolute difference in a feature's mean between the missing and non-missing groups.

**Dependency found — `calories`.** The observed difference in mean calories was about 69, with a p-value of about 0.000. At a 0.05 significance level we reject the null hypothesis: reviews with a missing rating tend to sit on higher-calorie recipes, so the missingness of `rating` does depend on `calories`.

**No dependency — `minutes`.** The p-value here was about 0.12, so at the same significance level we fail to reject the null hypothesis. There is no evidence that the missingness of `rating` depends on prep time.

The figure below shows the permutation distribution for the calories test, with the observed statistic marked.

<iframe src="assets/missingness-permutation.html" width="800" height="400" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Hypothesis Testing

We test whether calorie content is related to how recipes are rated. Here, high-rated means an average rating of 4 or higher, and low-rated means an average rating below 4.

<div class="note" markdown="1">

**Null hypothesis (H₀).** There is no association between recipe calorie content and rating. High-rated and low-rated recipes have roughly the same mean calories, and any observed difference is due to random chance.

**Alternative hypothesis (H₁).** High-rated and low-rated recipes have different mean calorie counts.

</div>

- **Test statistic:** mean calories of high-rated recipes minus mean calories of low-rated recipes.
- **Method:** two-sided permutation test, 5,000 shuffles of the high/low labels, seed 42.
- **Observed statistic:** −18.65 calories (high-rated recipes average fewer calories).
- **Significance level (α):** 0.05.
- **P-value:** 0.0362.

Since the p-value is below 0.05, we reject the null hypothesis. This is evidence that calorie content is associated with rating, with higher-rated recipes tending to be a little lower in calories. This is an observed association, not proof that calories cause the rating.

</section>

<section class="section-card" markdown="1">

## Framing a Prediction Problem

- **Prediction problem:** predict whether a recipe is high-rated.
- **Type:** binary classification.
- **Response variable:** `high_rating`, which is `True` when the average rating is 4 or higher. This matches the question we have been asking about whether recipe features relate to ratings.
- **Features available at prediction time:** values known the moment a recipe is posted, before any ratings exist — `calories` and the other nutrition PDVs, `minutes`, `n_ingredients`, and `n_steps`. We do not use `rating` or `average_rating`, since those only exist after users rate the recipe.

The classes are very imbalanced: about **93.4%** of recipes are high-rated. A model that always guesses "high" would already score about 93% accuracy while never identifying a single low-rated recipe. Because of that, our **primary metric is macro-averaged F1**, which weighs the high and low classes equally and only improves if the model does well on both. We still report accuracy, but treat it as secondary for exactly this reason.

</section>

<section class="section-card" markdown="1">

## Baseline Model

The baseline is a Logistic Regression that uses two features from the original data, `calories` and `n_ingredients`. Both are quantitative (2 quantitative, 0 ordinal, 0 nominal), so no categorical encoding is needed. The whole process — median imputation, scaling, and the classifier — runs inside a single scikit-learn `Pipeline`, and we evaluate on a held-out test set.

- **Features used:** `calories`, `n_ingredients`
- **Model type:** Logistic Regression
- **Accuracy:** 0.934
- **Macro F1 (primary):** 0.483
- **Recall on the low-rated class:** 0.000

The baseline's accuracy looks high, but it is essentially the same as the majority-class baseline (also about 0.934). The per-class numbers explain why: the model collapses to the majority class, predicting "high" for nearly every recipe, and its recall on the low-rated class is 0. So despite the high accuracy, it never actually identifies a low-rated recipe, and its macro F1 is poor. This gives the final model a clear goal — raise macro F1 by handling the minority class.

</section>

<section class="section-card" markdown="1">

## Final Model

The final model is a tuned, class-balanced Random Forest. On top of the baseline features we engineered new ones, all known before any rating exists:

- `calories_per_ingredient` (`calories / n_ingredients`) — separates rich recipes from lighter ones.
- `minutes_per_step` (`minutes / n_steps`) — how involved each step is.
- `complexity_score` (`n_steps × n_ingredients`) — overall effort.
- the remaining nutrition PDVs (sugar, sodium, protein, total fat, saturated fat, carbohydrates).

These describe real properties of a recipe rather than being added just to nudge the score. We tuned `n_estimators`, `max_depth`, and `min_samples_leaf` with `GridSearchCV` (3-fold, scored on macro F1) and used `class_weight="balanced"` so the model stops ignoring the minority class. The final model and the baseline use the same train/test split, so the comparison is fair.

- **Final model:** Random Forest (balanced)
- **Best hyperparameters:** `max_depth=10`, `min_samples_leaf=5`, `n_estimators=200`
- **Accuracy:** 0.785
- **Macro F1 (primary):** 0.498
- **Recall on the low-rated class:** 0.221

<iframe src="assets/model-performance.html" width="800" height="400" frameborder="0"></iframe>

The final model improves on the baseline where it matters: macro F1 rises from 0.483 to 0.498, and recall on the low-rated class rises from 0 to about 0.22, meaning it now catches some low-rated recipes instead of ignoring them entirely. Accuracy drops from 0.934 to 0.785, but that drop is expected and is actually a sign of progress — the baseline only looked accurate because it always guessed the majority class. Under this kind of class imbalance, the model that improves macro F1 and handles the minority class is the better model, even though its raw accuracy is lower. The overall signal is still modest, which fits the small calorie effect we found in the hypothesis test.

</section>

<section class="section-card" markdown="1">

## Fairness Analysis

We checked whether the final model performs about equally well for simple and complex recipes, split at the median number of ingredients (9). We did not retrain the model for this.

- **Group X:** simple recipes (`n_ingredients` ≤ 9).
- **Group Y:** complex recipes (`n_ingredients` > 9).
- **Evaluation metric:** F1-score.
- **Test statistic:** F1 for simple recipes minus F1 for complex recipes.

<div class="note" markdown="1">

**Null hypothesis (H₀).** The model is fair. Its F1 for simple and complex recipes is roughly the same, and any difference is due to random chance.

**Alternative hypothesis (H₁).** The model is unfair. Its F1 differs between the two groups.

</div>

- **F1, simple recipes:** 0.873
- **F1, complex recipes:** 0.884
- **Observed difference:** −0.0104
- **Significance level (α):** 0.05
- **P-value:** 0.016

Since the p-value is below 0.05, we reject the null hypothesis: the model's F1 differs between the two groups, performing a little better on complex recipes. The gap itself is small (about 0.01) and is detectable mostly because the test set is large, so in practical terms the difference is minor.

<iframe src="assets/fairness-permutation.html" width="800" height="400" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Summary

Across 81,173 recipes, higher-rated recipes are associated with slightly fewer calories (permutation test: −18.65 calories, p = 0.0362). The missingness of `rating` depends on `calories` but not on `minutes`, and `rating` is most likely NMAR. For prediction, the heavy class imbalance (93.4% high-rated) makes macro F1 the metric to watch: a balanced Random Forest with engineered features lifts macro F1 from 0.483 to 0.498 and low-rated recall from 0 to 0.22 over the baseline, even though its headline accuracy is lower. A fairness check finds a small but statistically detectable performance gap between simple and complex recipes.

</section>
