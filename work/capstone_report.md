# Capstone Report: Refresh / Content Opportunity Scoring

**Project:** FlyRank ML Internship

**Track:** Machine Learning: Refresh / Content Opportunity Scoring

**Author:** Roselyn Koech

## 1. Title and Abstract

### Refresh / Content Opportunity Scoring: A Machine Learning Approach to Content Review Prioritization

This capstone addresses the question: **Which content pages should be reviewed first for refresh based on observed performance signals?** The analysis uses a public safe, anonymized starter dataset containing 30,000 prepared rows, with an observed declining label rate of 54.2%. A transparent rule based baseline is compared with Logistic Regression, Decision Tree, and Random Forest models using a client grouped holdout consisting of 27,675 training rows and 2,325 test rows, with six held out clients and zero client overlap. Random Forest achieved the strongest ranking results, including ROC AUC of 0.754, Average Precision of 0.636, Precision@20 of 0.90, and Precision@50 of 0.74, compared with baseline Precision@20 of 0.30 and Precision@50 of 0.46. The resulting workflow is intended as decision support for prioritizing human review, not as a causal model of content recovery or a prediction of Google's ranking algorithm.

## 2. Introduction and Problem

Content and SEO teams may have many pages that could potentially require attention, while the time available for detailed review is limited. Treating every page equally can make it difficult to identify which pages deserve attention first.

This capstone focuses on prioritization rather than automatic content editing.

The practical problem is that a content team needs a defensible way to identify a small set of pages that should be reviewed first.

The machine learning workflow addresses this by ranking pages according to their estimated likelihood of belonging to the observed declining class.

The purpose is not to predict Google's search algorithm, guarantee future traffic recovery, or automatically decide whether a page should be rewritten or removed.

The intended outcome is a ranked queue that helps a human reviewer decide where to start.

### Research Question

**Which content pages should be reviewed first for refresh based on observed performance signals?**

### Decision Supported

The workflow supports the decision to prioritize a small ranked queue of content pages for human review rather than treating every page equally.

The primary operational metric is **Precision@20** because the intended use case is a small review queue. Secondary metrics are Precision@50, Average Precision, and ROC AUC.

## 3. Data

### 3.1 Dataset

The capstone uses the public safe anonymized starter dataset:

`data/raw/content_refresh_anonymized.csv`

The prepared working dataset contains **30,000 rows** and 44 original columns before additional derived variables are created.

The dataset is anonymized and does not expose client names, URLs, titles, or private search queries.

### 3.2 Eligibility and Preparation

The analysis retains pages with positive 90 day impressions and content age of at least 90 days.

Duplicate `content_id` values are removed after these filters.

Relevant numeric fields are converted to numeric values, invalid values are handled, and logarithmic transformations are created for selected traffic variables.

Categorical variables are converted to string values, with missing categories represented as `"unknown"`.

After preparation, the notebook records:

**Prepared rows: 30,000**

**Declining label rate: 54.2%**

### 3.3 Target Definition

The target variable is:

`is_declining_label`

It is defined from the observed condition:

`trend_direction == "down"`

This means that the model predicts the observed declining label defined in the dataset. It is not predicting future traffic recovery or a causal need for a content refresh.

### 3.4 Predictive Features

The modelling features include observed signals covering search demand, competition, cost per click, content size, impressions, clicks, sessions, AI sessions, content age, time since last update, CTR, average position, engagement, scroll behaviour, AI traffic percentage, and content and tier categories.

Identifiers are not used as predictive variables.

`client_id` is retained for validation grouping only.

## 4. Methodology

### 4.1 Transparent Baseline

The first benchmark is a transparent, human readable scoring rule.

The baseline combines three observable signals:

| Signal                               | Weight |
| ------------------------------------ | -----: |
| Low CTR                              |    40% |
| Weak average position                |    35% |
| Low 90 day visibility or impressions |    25% |

The baseline provides a simple benchmark that can be inspected without a machine learning model.

This is important because the learned models should demonstrate improvement over a reasonable transparent approach rather than being evaluated in isolation.

### 4.2 Learned Models

Three supervised learning approaches are compared:

1. Logistic Regression
2. Shallow Decision Tree
3. Random Forest

The models generate positive class probabilities, which are then used as ranking scores.

Random Forest is selected as the final ranking component because it provides the strongest overall ranking performance under the final grouped validation.

### 4.3 Validation Design

The primary evaluation uses a **client grouped holdout**.

Entire clients are held out from model training so that the evaluation measures performance on clients the model did not see during fitting.

The final validation configuration is:

| Validation Item  |                 Result |
| ---------------- | ---------------------: |
| Split strategy   | Client grouped holdout |
| Training rows    |                 27,675 |
| Test rows        |                  2,325 |
| Held out clients |                      6 |
| Client overlap   |                      0 |
| Random seed      |                     42 |

The notebook explicitly checks the client overlap and confirms that it is zero.

This approach is more conservative than a simple random row split because pages from the same client are not intentionally placed on both sides of the final client grouped evaluation.

## 5. Leakage and Integrity Controls

Leakage prevention is a central part of the capstone.

The following controls are applied:

* `content_id` is treated as an identifier rather than a predictive feature.
* `client_id` is used for grouped validation and is not used for prediction.
* `trend_direction` is used to define the observed target and is excluded from predictive features.
* `trend_pct` is excluded because it is directly related to the outcome definition.
* The grouped validation split is checked for client overlap.
* The final client grouped split records zero overlap.
* The dataset used for the project is anonymized and public safe.

These controls reduce the risk that the model receives direct access to the outcome or that the evaluation is artificially strengthened by client overlap.

## 6. Results

### 6.1 Model Comparison

The final capstone evaluation produced the following results:

| Method              |   ROC AUC | Average Precision | Precision@20 | Precision@50 |
| ------------------- | --------: | ----------------: | -----------: | -----------: |
| Random Forest       | **0.754** |         **0.636** |     **0.90** |     **0.74** |
| Decision Tree       |     0.740 |             0.570 |         0.65 |         0.62 |
| Logistic Regression |     0.703 |             0.521 |         0.35 |         0.34 |
| Baseline Rules      |     0.528 |             0.443 |         0.30 |         0.46 |

Random Forest is the strongest model across the main ranking metrics.

For the primary operational metric, Random Forest achieved **Precision@20 of 0.90**.

This means that **18 of the top 20 ranked test pages** were positive under the observed declining label.

At the top 50, Random Forest achieved **Precision@50 of 0.74**, corresponding to **37 of the top 50 ranked test pages** being positive under the observed declining label.

### 6.2 Comparison with the Baseline

Random Forest improved Precision@20 from **0.30 to 0.90**.

It also improved Precision@50 from **0.46 to 0.74**.

ROC AUC increased from **0.528 to 0.754**.

These results indicate that the learned model produced a substantially stronger ranking of observed declining pages than the transparent baseline on the held out clients.

### 6.3 Interpretation

The most relevant result for the intended business workflow is Precision@20 because the objective is to identify a small set of pages for review.

The evaluation therefore supports retaining Random Forest as the ranking component of this prototype.

The result should be interpreted narrowly:

> The Random Forest ranked pages with the observed declining label more effectively than the transparent baseline on the evaluated held out clients.

It should not be interpreted as proof that the model can predict Google's ranking algorithm or guarantee that editing a page will improve its future performance.

## 7. Decision Workflow

The intended operational workflow is:

**Prepare data → Score pages → Rank pages → Review highest priority candidates → Decide action → Monitor**

More specifically:

1. Prepare eligible content and performance signals.
2. Apply the trained Random Forest model.
3. Generate a positive class probability for each page.
4. Rank pages by the model score.
5. Use the ranked output to create a small review queue.
6. Examine the page's available performance context.
7. Have a human reviewer determine the appropriate action.
8. Monitor the outcome after any action.

The model is therefore a prioritization system, not an automated content editing system.

A high model score means **review earlier**.

It does not mean **automatically rewrite, expand, protect, or prune the page**.

## 8. Ranked Recommendations

The model results support the following ranked operational recommendations.

### Recommendation 1: Review the highest ranked pages first

Use the Random Forest ranking to create the initial small review queue.

This is supported by the held out **Precision@20 of 0.90**, where 18 of the top 20 ranked test pages were positive under the observed declining label.

These pages should receive the earliest human review.

### Recommendation 2: Investigate before editing

A high score should trigger investigation rather than an automatic content change.

Review available context such as CTR, average position, traffic, impressions, content age, freshness, and engagement signals.

The objective is to determine whether the observed decline represents a meaningful content opportunity.

### Recommendation 3: Match the action to the page evidence

After human review, a surfaced page may be appropriate for refresh, expansion, monitoring, protection, or no immediate change.

The model does not independently determine which action is correct.

### Recommendation 4: Use the queue to allocate limited review time

The strongest operational value of the model is prioritization.

Instead of reviewing every page equally, the team can begin with the highest ranked candidates and expand the review queue if capacity allows.

### Recommendation 5: Keep human review as the final decision point

The model should remain a decision support tool.

A high score is evidence for **earlier review**, not evidence of guaranteed recovery.

### Recommended Operating Rule

**Random Forest ranking → Highest priority queue → Contextual page review → Human action decision → Monitoring**

The notebook supports the ranking component and its evaluation. Individual page level actions should only be made after inspecting the relevant page evidence.

## 9. What the Results Do and Do Not Show

### What the Results Show

The results show that, on the 30,000 row anonymized starter dataset and under the client grouped holdout, Random Forest produced a substantially stronger ranking of the observed declining class than the transparent baseline.

The strongest evidence is:

* Precision@20: **0.90**
* Precision@50: **0.74**
* ROC AUC: **0.754**
* Average Precision: **0.636**

### What the Results Do Not Show

The results do not establish that:

* changing a page will cause traffic or rankings to recover;
* the model predicts Google's ranking algorithm;
* the model will perform identically on future production data;
* the model will generalize unchanged to the full warehouse;
* every high ranked page is necessarily a good refresh candidate;
* the observed decline has a single cause;
* the model has demonstrated a causal relationship between content changes and performance recovery.

A causal claim about content changes would require an experiment or another appropriate causal design.

This capstone is therefore appropriately described as decision support and prioritization.

## 10. Limitations and Next Steps

### 10.1 Dataset Scope

The evaluation uses the anonymized 30,000 row starter dataset rather than the full warehouse.

The reported results should therefore be treated as evaluation evidence for this dataset rather than guaranteed production performance.

### 10.2 Validation Scope

The client grouped holdout provides a stronger evaluation design than a simple random row split, but it remains one held out evaluation.

Additional client groups and future data should be evaluated before production use.

### 10.3 Time Generalization

The current evaluation is not a full time based production backtest.

A future validation stage should include an appropriate future looking time split where the available data supports it.

### 10.4 Observational Signals

CTR, position, traffic, freshness, and related variables are observational signals.

They may help identify pages associated with decline, but they should not be interpreted as causes of that decline.

### 10.5 Human Review

The ranked queue still requires human interpretation.

A high score means **review earlier**, not automatic rewriting or pruning.

### 10.6 Production Monitoring

The current notebook is a research and decision support artifact rather than a fully deployed production service with scheduled retraining, monitoring, and model registry infrastructure.

A production version should monitor ranking quality over time and periodically revalidate the model.

### 10.7 Feature Importance and Error Analysis

No feature importance analysis or detailed error pattern analysis is claimed in this report because those results are not part of the verified capstone evidence used here.

## 11. Reproducibility

The project repository contains the code and notebook used for the capstone.

The main notebook is:

`work/notebooks/capstone.ipynb`

The project uses a fixed random seed of **42**.

The notebook records data preparation, target definition, feature preparation, baseline construction, model comparison, client grouped validation, train and test sizes, held out client count, client overlap check, evaluation metrics, and final model selection.

The repository is intended to allow a reviewer to inspect and rerun the work.

### Public Repository

The complete project is available in the public FlyRank repository:

**https://github.com/Roselyn-Koech/flyrank-ml-internship**

The paper's deployed public URL will be recorded separately in:

`submission/paper_url.txt`

as required by the ML 11 submission specification.

## 12. Artifacts Embedded in the Paper

The paper should be accompanied by concise evidence artifacts from the capstone notebook.

### Artifact 1: Validation Evidence

The notebook records:

* Client grouped holdout
* 27,675 training rows
* 2,325 test rows
* 6 held out clients
* 0 client overlap

This artifact demonstrates that the final evaluation was performed using the intended grouped validation design.

### Artifact 2: Model Comparison

The notebook records the final comparison:

| Method              | ROC AUC | Average Precision | Precision@20 | Precision@50 |
| ------------------- | ------: | ----------------: | -----------: | -----------: |
| Random Forest       |   0.754 |             0.636 |         0.90 |         0.74 |
| Decision Tree       |   0.740 |             0.570 |         0.65 |         0.62 |
| Logistic Regression |   0.703 |             0.521 |         0.35 |         0.34 |
| Baseline Rules      |   0.528 |             0.443 |         0.30 |         0.46 |

These artifacts provide direct evidence for the model selection and primary ranking result.

## 13. Acknowledgments and Data Credit

This project was completed as part of the **FlyRank ML Internship**.

The analysis uses the public safe anonymized starter dataset provided for the internship project.

The FlyRank project repository and associated materials are credited below:

**FlyRank:** https://github.com/flyrank-bih/flyrank-ml-internship-starter

The project does not expose private client information, private queries, client names, or other confidential data.

## 14. AI Transparency

AI assistance was used during development to help structure, review, and document parts of the capstone.

The data preparation, leakage exclusions, validation design, evaluation metrics, and final claims were checked against the capstone notebook.

The report intentionally avoids unsupported claims. In particular, it does not claim feature importance, detailed error patterns, causal effects, production performance, or individual page recommendations where those results were not verified in the capstone evidence.

## 15. Conclusion

This capstone demonstrates a practical machine learning workflow for prioritizing content pages for human review.

The analysis compares a transparent baseline with Logistic Regression, Decision Tree, and Random Forest models using a client grouped holdout. The final evaluation contains **27,675 training rows and 2,325 test rows**, with **six held out clients and zero client overlap**.

Random Forest produced the strongest ranking performance, including:

**ROC AUC = 0.754**

**Average Precision = 0.636**

**Precision@20 = 0.90**

**Precision@50 = 0.74**

The transparent baseline achieved Precision@20 of 0.30 and Precision@50 of 0.46.

The strongest practical conclusion is:

> **The learned Random Forest ranking provides a more useful small review queue than the fixed baseline on this held out evaluation set.**

The model should be used to focus human attention on the pages that appear most relevant for review. It should not be treated as a guarantee of recovery, a causal model of content performance, or a prediction of Google's ranking algorithm.

The appropriate workflow is:

**Rank → Review → Decide → Act → Monitor**

This keeps the machine learning model in its appropriate role as decision support for content prioritization.
