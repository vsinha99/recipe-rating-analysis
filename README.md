---
---

<header class="hero" markdown="1">

# Do Higher-Rated Recipes Have Different Nutritional Profiles?

<p class="authors" markdown="1">**By Varun Sinha and Rishab Kolan**</p>

<p class="badges"><span class="badge">UCSD DSC 80</span> <span class="badge">Final Project</span> <span class="badge badge-accent">Recipes &amp; Ratings</span></p>

This project looks at recipes from Food.com and the ratings people gave them. We want to find out whether a recipe's nutrition and complexity are related to how highly it is rated.

</header>

<div class="note" markdown="1">

This page is a work in progress. Items marked TODO will be filled in once the analysis is complete. No results have been made up.

</div>

<section class="section-card" markdown="1">

## Introduction

This project uses the Recipes & Ratings dataset from [Food.com](https://www.food.com). It includes recipe information such as ingredients, steps, preparation time, and nutrition, along with the star ratings users left.

Our main question:

> Are recipe features like calories, nutrition, cooking time, number of ingredients, and number of steps related to the ratings users give?

Ratings affect what people decide to cook, but a high rating may depend on more than taste. It could also be linked to how simple, quick, or rich a recipe is. Looking at these features helps show what a rating might reflect.

The dataset has <span class="todo-tag">TODO</span> rows after the recipes and ratings are joined.

<div class="todo" markdown="1">

**TODO:** Confirm the number of rows and note what one row represents.

</div>

The columns most relevant to our question are listed below.

| Column | Description |
|---|---|
| `rating` | A single user's star rating for a recipe, from 1 to 5. |
| `average_rating` | The mean of all ratings a recipe received. This is our main measure of how well liked it is. |
| `calories` | Calories per serving, taken from the first value in the `nutrition` field. |
| `nutrition` | A list `[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]`. Every value except calories is a percent of daily value (PDV). |
| `minutes` | Reported preparation time in minutes. |
| `n_ingredients` | Number of ingredients in the recipe. |
| `n_steps` | Number of steps in the recipe. |

</section>

<section class="section-card" markdown="1">

## Data Cleaning and Exploratory Data Analysis

We plan to clean the data so each row represents one recipe. The main steps:

- **Handle missing ratings.** Replace ratings of `0` with `NaN`, since a `0` means there was no rating rather than a true score of zero.
- **Extract nutrition.** Split the `nutrition` field into its seven parts and pull `calories` (and other values) into their own numeric columns.
- **Average rating per recipe.** Group the ratings for each recipe and take the mean to create `average_rating`.
- **Complexity features.** Set up `n_ingredients` and `n_steps` as integer counts.
- **Prediction target.** Create a binary `high_rating` column that is `True` when a recipe's average rating is 4 or higher and `False` otherwise.

<div class="todo" markdown="1">

**TODO:** Note any other cleaning steps used, such as outlier handling, and their effect on the data.

</div>

### Cleaned DataFrame (head)

<div class="todo" markdown="1">

**TODO:** Add the first few rows of the cleaned dataset.

</div>

<!-- Skeleton rows below; fill from df.head().to_markdown(index=False) output -->

| `name` | `minutes` | `calories` | `n_ingredients` | `n_steps` | `average_rating` | `high_rating` |
|---|---|---|---|---|---|---|
| … | … | … | … | … | … | … |
| … | … | … | … | … | … | … |
| … | … | … | … | … | … | … |

---

### Univariate analysis

<div class="todo" markdown="1">

**TODO:** Add a Plotly histogram of calories and briefly describe the distribution.

</div>

<!-- Figure file: assets/univariate-calories.html -->

<iframe src="assets/univariate-calories.html" width="800" height="300" frameborder="0"></iframe>

### Bivariate analysis

<div class="todo" markdown="1">

**TODO:** Add a Plotly plot of calories versus average rating and note any relationship.

</div>

<!-- Figure file: assets/bivariate-calories-rating.html -->

<iframe src="assets/bivariate-calories-rating.html" width="800" height="300" frameborder="0"></iframe>

---

### Interesting aggregates

We will compare the average nutrition and complexity of high and low rated recipes.

<div class="todo" markdown="1">

**TODO:** Add a grouped table comparing nutrition and complexity for high and low rated recipes, and note any pattern.

</div>

<!-- Suggested grouping: df.groupby("high_rating")[["calories", "n_ingredients", "n_steps", "minutes"]].mean() -->

| `high_rating` | mean `calories` | mean `n_ingredients` | mean `n_steps` | mean `minutes` |
|---|---|---|---|---|
| `False` (below 4) | … | … | … | … |
| `True` (4 or higher) | … | … | … | … |

</section>

<section class="section-card" markdown="1">

## Assessment of Missingness

### NMAR analysis

We think the missingness of `rating` could be NMAR (Not Missing At Random). Someone who dislikes a recipe may simply not leave a rating, so whether a rating is missing can depend on the rating itself. If that is the case, no other column can fully explain why it is missing.

<div class="note" markdown="1">

If we had more data, such as whether a user viewed or saved a recipe without rating it, the missingness might be explained by those columns instead. That would make it MAR rather than NMAR.

</div>

<div class="todo" markdown="1">

**TODO:** Confirm which column's missingness is analyzed and finalize the NMAR discussion.

</div>

### Missingness dependency (permutation tests)

We will use permutation tests to check whether the missingness of one column depends on other columns.

**Dependency found:**

<div class="todo" markdown="1">

**TODO:** Report a column that the missingness of `rating` depends on, with the test statistic, p-value, and significance level.

</div>

**No dependency found:**

<div class="todo" markdown="1">

**TODO:** Report a column that the missingness of `rating` does not depend on, with the test statistic and p-value.

</div>

<div class="todo" markdown="1">

**TODO:** Add a Plotly figure for the permutation test.

</div>

<!-- Figure file: assets/missingness-permutation.html -->

<iframe src="assets/missingness-permutation.html" width="800" height="300" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Hypothesis Testing

We test whether calorie content is related to how recipes are rated. Here, high-rated means an average rating of 4 or higher, and low-rated means an average rating below 4.

<div class="note" markdown="1">

**Null hypothesis (H₀).** There is no association between recipe calorie content and rating. High-rated and low-rated recipes have roughly the same mean calories, and any observed difference is due to random chance.

**Alternative hypothesis (H₁).** There is an association between recipe calorie content and rating. High-rated and low-rated recipes have different mean calorie counts.

</div>

- **Test statistic:** the difference in mean calories between high-rated and low-rated recipes.
- **Observed statistic:** <span class="todo-tag">TODO</span>
- **Significance level (α):** <span class="todo-tag">TODO</span>
- **P-value:** <span class="todo-tag">TODO</span>

<div class="todo" markdown="1">

**TODO:** Add the conclusion. Based on the p-value and significance level, state whether we reject or fail to reject the null hypothesis.

</div>

</section>

<section class="section-card" markdown="1">

## Framing a Prediction Problem

- **Prediction problem:** predict whether a recipe is high-rated.
- **Type:** binary classification.
- **Response variable:** `high_rating`, based on whether the average rating is 4 or higher. This matches our question about whether recipe features relate to ratings.
- **Evaluation metrics:** accuracy and F1-score. We report F1 along with accuracy because the high-rated and low-rated classes are likely imbalanced. With imbalanced classes, accuracy can look high just by predicting the majority class, while F1 balances precision and recall.

To avoid leakage, we only use features that are known before any rating exists, such as `calories`, `minutes`, `n_ingredients`, and `n_steps`. We do not use `rating` or `average_rating` as inputs, since those are not known when a recipe is first posted.

<div class="todo" markdown="1">

**TODO:** List the exact features used by the model.

</div>

</section>

<section class="section-card" markdown="1">

## Baseline Model

The baseline model uses at least two simple features from the original data, such as `calories` and `n_ingredients`. Both are quantitative. The whole process, including any preprocessing and the model, is built in a single scikit-learn `Pipeline`. The baseline is kept simple so the final model has something to compare against. It will use either Logistic Regression or a Decision Tree Classifier.

- **Features used:** <span class="todo-tag">TODO</span>
- **Quantitative features:** <span class="todo-tag">TODO</span>
- **Ordinal features:** <span class="todo-tag">TODO</span>
- **Nominal features:** <span class="todo-tag">TODO</span>
- **Encoding for categorical variables:** <span class="todo-tag">TODO</span>
- **Train/test split:** <span class="todo-tag">TODO</span>
- **Model type:** <span class="todo-tag">TODO</span>
- **Accuracy:** <span class="todo-tag">TODO</span>
- **F1-score:** <span class="todo-tag">TODO</span>

<div class="todo" markdown="1">

**TODO:** Note whether the baseline performs well, compared with a simple majority-class prediction.

</div>

</section>

<section class="section-card" markdown="1">

## Final Model

The final model adds at least two engineered features and likely a stronger model. Possible engineered features:

- Calories per ingredient (`calories / n_ingredients`)
- A recipe complexity score (for example, combining `n_steps` and `n_ingredients`)
- Minutes per step (`minutes / n_steps`)
- Number of steps (`n_steps`)
- Nutrition based features (for example, sugar or fat as a share of calories)

These features describe real properties of a recipe. Calories per ingredient separates rich recipes from lighter ones. Minutes per step reflects how involved each step is. A complexity score reflects how much work a recipe takes. We choose them because they relate to the data, not only because they might raise the score.

We will try a Random Forest Classifier or another stronger model, and tune it with GridSearchCV or a manual search.

- **Final selected model:** <span class="todo-tag">TODO</span>
- **Engineered features added:** <span class="todo-tag">TODO</span>
- **Hyperparameters tuned:** <span class="todo-tag">TODO</span>
- **Best hyperparameters:** <span class="todo-tag">TODO</span>
- **Final model accuracy:** <span class="todo-tag">TODO</span>
- **Final model F1-score:** <span class="todo-tag">TODO</span>
- **Comparison to baseline:** <span class="todo-tag">TODO</span>

<div class="note" markdown="1">

The final model and the baseline are evaluated on the same test set so the comparison is fair.

</div>

</section>

<section class="section-card" markdown="1">

## Fairness Analysis

We check whether the final model performs about equally well for simple and complex recipes.

- **Group X:** simpler recipes (for example, recipes with fewer ingredients).
- **Group Y:** more complex recipes (for example, recipes with more ingredients).
- **Evaluation metric:** F1-score (or accuracy).
- **Test statistic:** the difference in model performance between the two groups.

<div class="note" markdown="1">

**Null hypothesis (H₀).** The model is fair. Its performance for simpler recipes and more complex recipes is roughly the same, and any observed difference is due to random chance.

**Alternative hypothesis (H₁).** The model is unfair. Its performance is worse for one of the two recipe-complexity groups.

</div>

- **Significance level (α):** <span class="todo-tag">TODO</span>
- **P-value:** <span class="todo-tag">TODO</span>

<div class="todo" markdown="1">

**TODO:** Add the conclusion, stating whether we reject or fail to reject the null hypothesis. Add a Plotly figure for the fairness permutation test.

</div>

<!-- Figure file: assets/fairness-permutation.html -->

<iframe src="assets/fairness-permutation.html" width="800" height="300" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Next Steps

<div class="checklist" markdown="1">

- [ ] Complete hypothesis test and add observed statistic and p-value
- [ ] Train baseline model and add performance
- [ ] Add cleaned DataFrame table
- [ ] Add Plotly univariate visualization
- [ ] Add Plotly bivariate visualization
- [ ] Add grouped or pivot table
- [ ] Complete missingness permutation tests
- [ ] Complete final model and hyperparameter tuning
- [ ] Complete fairness analysis

</div>

</section>

<p class="footnote-tip"><em>TODO items will be filled in once the analysis is complete.</em></p>
