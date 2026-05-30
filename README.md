---
---

<header class="hero" markdown="1">

# Do Higher-Rated Recipes Have Different Nutritional Profiles?

<p class="authors" markdown="1">**By Varun Sinha and Rishab Kolan**</p>

<p class="badges"><span class="badge">UCSD DSC 80</span> <span class="badge">Final Project</span> <span class="badge badge-accent">Recipes &amp; Ratings</span></p>

Ratings shape what millions of people decide to cook — but does a five-star recipe actually look different *under the hood*? This project pairs the **Recipes &amp; Ratings** dataset from Food.com with a full data-science workflow to ask whether a recipe's **nutrition** and **complexity** are related to how users rate it.

</header>

<div class="note" markdown="1">

📝 **Checkpoint build.** The structure, narrative, and styling on this page are final. Every value marked <span class="todo-tag">TODO</span> is a deliberate placeholder that will be filled in directly from the project notebook. No p-values, model scores, table values, or conclusions have been fabricated.

</div>

<section class="section-card" markdown="1">

## Introduction

This project uses the **Recipes &amp; Ratings** dataset from [Food.com](https://www.food.com), which combines recipe metadata — ingredients, steps, preparation time, and nutrition — with user-submitted star ratings.

Our central question:

> **Are recipe characteristics — calories, nutrition, cooking time, number of ingredients, and number of steps — related to the ratings users give?**

**Why this matters.** Ratings strongly influence what people choose to cook, yet a high rating may reflect more than taste alone — it can also track how indulgent, quick, or simple a recipe is. Understanding these relationships helps home cooks read between the stars and helps recipe platforms understand what a rating really signals.

The merged dataset contains <span class="todo-tag">TODO</span> rows after joining recipes with their ratings.

<div class="todo" markdown="1">

**🚧 TODO** — Insert the final row count and note the unit of a row (e.g., *one row per recipe* after aggregation, or *one row per rating* before it). Add any extra columns the analysis ends up using.

</div>

The columns most relevant to our question are described below.

| Column | Description |
|---|---|
| `rating` | A single user's star rating for a recipe, on a 1–5 scale. |
| `average_rating` | The mean of all ratings a recipe received — our primary signal of how well-liked it is. |
| `calories` | Calories (kcal) per serving, parsed from the first entry of the `nutrition` field. |
| `nutrition` | A list `[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]`; every value except calories is a percent of daily value (PDV). |
| `minutes` | Reported preparation time, in minutes. |
| `n_ingredients` | Number of ingredients the recipe requires. |
| `n_steps` | Number of steps in the recipe's instructions. |

</section>

<section class="section-card" markdown="1">

## Data Cleaning and Exploratory Data Analysis

We cleaned the raw data so that each row reflects a single recipe with trustworthy, analysis-ready features. Our planned steps:

- **Handle missing ratings.** Replace ratings of `0` with `NaN`, since a `0` indicates the absence of a rating rather than a true score of zero.
- **Extract nutrition.** Parse the `nutrition` string into its seven components and pull **`calories`** (and other nutrition values) into their own numeric columns.
- **Create an average rating per recipe.** Group all ratings belonging to each recipe and take the mean to form **`average_rating`**.
- **Build complexity features.** Confirm/derive **`n_ingredients`** (number of ingredients) and **`n_steps`** (number of steps) as integer counts.
- **Define the prediction target.** Engineer a binary **`high_rating`** column: `True` when a recipe's `average_rating` is **≥ 4**, otherwise `False`.

<div class="todo" markdown="1">

**🚧 TODO** — Confirm the exact cleaning decisions actually used (e.g., outlier thresholds on `minutes`/`calories`, how recipes with no ratings are handled) and describe their effect on the data.

</div>

### Cleaned DataFrame (head)

<div class="todo" markdown="1">

**🚧 TODO** — Replace the skeleton below with the real `df.head()`. Tip: in the notebook run `print(df.head().to_markdown(index=False))` and paste the output here.

</div>

| `name` | `minutes` | `calories` | `n_ingredients` | `n_steps` | `average_rating` | `high_rating` |
|---|---|---|---|---|---|---|
| … | … | … | … | … | … | … |
| … | … | … | … | … | … | … |
| … | … | … | … | … | … | … |

---

### Univariate analysis

<div class="todo" markdown="1">

**🚧 TODO** — Replace the placeholder figure with a real Plotly univariate plot (e.g., the distribution of `calories`) exported from the notebook via `fig.write_html("assets/univariate-calories.html")`. Then write one or two sentences interpreting the trend.

</div>

<iframe src="assets/univariate-calories.html" width="800" height="600" frameborder="0"></iframe>

### Bivariate analysis

<div class="todo" markdown="1">

**🚧 TODO** — Replace with a real Plotly bivariate plot (e.g., `calories` vs. `average_rating`, or calorie distributions split by high- vs. low-rated). Export with `fig.write_html("assets/bivariate-calories-rating.html")` and add a short interpretation.

</div>

<iframe src="assets/bivariate-calories-rating.html" width="800" height="600" frameborder="0"></iframe>

---

### Interesting aggregates

We will compare the average nutrition and complexity of high- and low-rated recipes.

<div class="todo" markdown="1">

**🚧 TODO** — Replace the skeleton below with a real grouped or pivot table (e.g., `df.groupby("high_rating")[["calories", "n_ingredients", "n_steps", "minutes"]].mean()`), then comment on any visible pattern.

</div>

| `high_rating` | mean `calories` | mean `n_ingredients` | mean `n_steps` | mean `minutes` |
|---|---|---|---|---|
| `False` (avg &lt; 4) | … | … | … | … |
| `True` (avg ≥ 4) | … | … | … | … |

</section>

<section class="section-card" markdown="1">

## Assessment of Missingness

### NMAR analysis

We believe the missingness of `rating` is plausibly **NMAR** (Not Missing At Random). A user who strongly dislikes a recipe may simply never come back to leave a rating, which means the probability that a rating is missing depends on the **value of the rating itself** — exactly the condition that defines NMAR. Because the reason for missingness is tied to the unobserved value, no other column in the dataset can fully account for it.

<div class="note" markdown="1">

**Could it become MAR?** If we could collect additional behavioral data — for example, whether a user *viewed*, *saved*, or *abandoned* a recipe page without rating it — the missingness might be explained by those observed columns instead, which would make it **MAR** rather than NMAR.

</div>

<div class="todo" markdown="1">

**🚧 TODO** — Finalize which column's missingness you analyze and refine the NMAR argument once you've inspected the data.

</div>

### Missingness dependency (permutation tests)

We will run permutation tests to see whether the missingness of one column depends on other columns.

**Dependency found (likely MAR on that column):**

<div class="todo" markdown="1">

**🚧 TODO** — Report a column whose value the missingness *does* depend on (e.g., the missingness of `rating` vs. `n_steps`). Insert the test statistic, p-value, significance level, and the conclusion that missingness **does depend** on this column.

</div>

**No dependency found:**

<div class="todo" markdown="1">

**🚧 TODO** — Report a column whose value the missingness *does not* depend on. Insert the test statistic, p-value, and the conclusion that missingness **does not depend** on this column.

</div>

<div class="todo" markdown="1">

**🚧 TODO** — Replace the placeholder figure with a real Plotly visualization related to missingness (e.g., the empirical distribution of the permuted test statistic with the observed value marked). Export it to `assets/missingness-permutation.html`.

</div>

<iframe src="assets/missingness-permutation.html" width="800" height="600" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Hypothesis Testing

We test whether calorie content is associated with how recipes are rated. Here, **high-rated** means `average_rating` **≥ 4** and **low-rated** means `average_rating` **&lt; 4**.

<div class="note" markdown="1">

**Null hypothesis (H₀).** There is no association between recipe calorie content and rating. High-rated and low-rated recipes have roughly the same mean calories, and any observed difference is due to random chance.

**Alternative hypothesis (H₁).** There is an association between recipe calorie content and rating. High-rated and low-rated recipes have different mean calorie counts.

</div>

- **Test statistic:** the difference in mean calories between high-rated and low-rated recipes.
- **Observed statistic:** <span class="todo-tag">TODO</span>
- **Significance level (α):** <span class="todo-tag">TODO</span>
- **P-value:** <span class="todo-tag">TODO</span>

<div class="todo" markdown="1">

**🚧 TODO** — State the conclusion using careful statistical language: based on the p-value relative to α, *we reject* or *we fail to reject* the null hypothesis. Note that a hypothesis test does **not** "prove" the alternative — it only provides evidence for or against the null.

</div>

</section>

<section class="section-card" markdown="1">

## Framing a Prediction Problem

- **Prediction problem:** predict whether a recipe is **high-rated**.
- **Type:** binary classification.
- **Response variable:** `high_rating`, created from whether `average_rating` is at least 4. We chose this because it directly answers our question — whether a recipe's measurable characteristics anticipate strong reception.
- **Evaluation metrics:** **accuracy** and **F1-score**. We report F1 alongside accuracy because the high-rated and low-rated classes are likely **imbalanced**; with skewed classes, accuracy can look strong simply by predicting the majority class, whereas F1 balances precision and recall.

**Information available at prediction time.** To avoid leakage, we use only features that would be known **before** any rating exists — recipe properties like `calories`, `minutes`, `n_ingredients`, and `n_steps`. We do **not** use `rating` or `average_rating` (or anything derived from them) as inputs, since those are unknown at the moment a recipe is published.

<div class="todo" markdown="1">

**🚧 TODO** — List the exact feature set used as model inputs once it is finalized.

</div>

</section>

<section class="section-card" markdown="1">

## Baseline Model

Our baseline uses at least two simple, original features — for example **`calories`** and **`n_ingredients`** — both of which are **quantitative**. The whole workflow (any preprocessing plus the estimator) is implemented inside a single scikit-learn **`Pipeline`** so it can be reproduced and fairly compared against the final model. The baseline is intentionally simple so the final model has a meaningful comparison point.

The baseline will use **Logistic Regression** or a **Decision Tree Classifier**.

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

**🚧 TODO** — Assess whether the baseline is "good." Comment on its accuracy and F1 relative to a naive majority-class predictor, and explain what its errors suggest before moving to the final model.

</div>

</section>

<section class="section-card" markdown="1">

## Final Model

The final model improves on the baseline by adding at least **two engineered features** and (likely) a stronger estimator. Candidate engineered features include:

- **Calories per ingredient** (`calories / n_ingredients`)
- **Recipe complexity score** (e.g., combining `n_steps` and `n_ingredients`)
- **Minutes per step** (`minutes / n_steps`)
- **Number of steps** (`n_steps`)
- **Nutrition-based features** (e.g., sugar or fat as a share of calories)

**Why these features (not just accuracy-chasing).** Each one reflects something real about the *data-generating process* — how a recipe is actually written and cooked. Calories per ingredient separates rich, indulgent recipes from light ones independent of length; minutes per step captures how demanding each instruction is; a complexity score reflects the effort a cook must invest. These are mechanisms that could plausibly shape a cook's experience — and therefore their rating — rather than coincidental correlations.

We will try a **Random Forest Classifier** (or another stronger model) and tune it with **`GridSearchCV`** or a careful manual hyperparameter search.

- **Final selected model:** <span class="todo-tag">TODO</span>
- **Engineered features added:** <span class="todo-tag">TODO</span>
- **Hyperparameters tuned:** <span class="todo-tag">TODO</span>
- **Best hyperparameters:** <span class="todo-tag">TODO</span>
- **Final model accuracy:** <span class="todo-tag">TODO</span>
- **Final model F1-score:** <span class="todo-tag">TODO</span>
- **Comparison to baseline:** <span class="todo-tag">TODO</span>

<div class="note" markdown="1">

**Fair comparison.** The final model and the baseline must be evaluated on the **same test set** so the improvement reflects modeling choices rather than a different split.

</div>

</section>

<section class="section-card" markdown="1">

## Fairness Analysis

We ask whether the final model performs equally well for simple and complex recipes.

- **Group X:** simpler recipes (e.g., recipes with **fewer** ingredients).
- **Group Y:** more complex recipes (e.g., recipes with **more** ingredients).
- **Evaluation metric:** F1-score (or accuracy).
- **Test statistic:** the difference in model performance between the two groups.

<div class="note" markdown="1">

**Null hypothesis (H₀).** The model is fair. Its performance for simpler recipes and more complex recipes is roughly the same, and any observed difference is due to random chance.

**Alternative hypothesis (H₁).** The model is unfair. Its performance is worse for one of the two recipe-complexity groups.

</div>

- **Significance level (α):** <span class="todo-tag">TODO</span>
- **P-value:** <span class="todo-tag">TODO</span>

<div class="todo" markdown="1">

**🚧 TODO** — State the conclusion with careful language (*we reject* / *we fail to reject* the null), and optionally replace the figure below with a real permutation-test visualization exported to `assets/fairness-permutation.html`.

</div>

<iframe src="assets/fairness-permutation.html" width="800" height="600" frameborder="0"></iframe>

</section>

<section class="section-card" markdown="1">

## Next Steps

<div class="checklist" markdown="1">

- [ ] Complete hypothesis test and insert observed statistic / p-value
- [ ] Train baseline model and insert performance
- [ ] Add cleaned DataFrame table
- [ ] Add Plotly univariate visualization
- [ ] Add Plotly bivariate visualization
- [ ] Add grouped or pivot table
- [ ] Complete missingness permutation tests
- [ ] Complete final model and hyperparameter tuning
- [ ] Complete fairness analysis

</div>

</section>

<p class="footnote-tip"><em>All values marked TODO are placeholders and will be replaced with real results once the project notebook is finalized. Built with Jekyll and the Cayman theme.</em></p>
