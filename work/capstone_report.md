# Capstone Report — Refresh / Content Opportunity Scoring

**Project:** FlyRank ML Internship  
**Track:** Machine Learning — Refresh / Content Opportunity Scoring  
**Author:** Roselyn Koech

## 1. Executive Summary

This capstone builds a decision-support workflow for prioritizing content pages for review. The goal is not to predict Google's algorithm or guarantee that a page will recover after an edit. The goal is to rank pages that show stronger observed signals of decline so a content or SEO team can review a small queue first.

The analysis uses the anonymized starter dataset with 30,000 prepared rows and an observed declining base rate of 54.2%. The primary evaluation uses a client-grouped holdout: 27,675 rows for training and 2,325 rows for testing, with 6 clients held out and zero client overlap between train and test.

Under this validation design, the Random Forest produced ROC AUC of 0.754, Average Precision of 0.636, Precision@20 of 0.90, and Precision@50 of 0.74. The transparent baseline produced ROC AUC of 0.528, Precision@20 of 0.30, and Precision@50 of 0.46. The learned model therefore produced a substantially stronger ranking of the highest-priority review candidates on the held-out clients.

The result supports using the Random Forest as the ranking component for this prototype, while keeping the output explicitly decision-support rather than a causal claim about content performance.

## 2. Business / Research Question

**Research question:** Which content pages should be reviewed first for refresh based on observed performance signals?

**Decision supported:** Provide a ranked queue of pages for human review instead of treating every page equally.

The operational metric is **Precision@20**, because the intended workflow is a small review queue. Secondary metrics are Precision@50, Average Precision, and ROC AUC.

## 3. Data

The capstone uses the public-safe anonymized starter CSV:

`data/raw/content_refresh_anonymized.csv`

The prepared dataset contains 30,000 rows. Pages are retained when they have positive 90-day impressions and are at least 90 days old, and duplicate `content_id` values are removed.

The target is `is_declining_label`, defined from the observed condition `trend_direction == "down"`.

Client identifiers are used for grouped validation only and are not predictive features. The modeling features consist of numeric performance, content, search, engagement, and freshness signals together with categorical content and tier variables. Outcome-defining fields such as `trend_direction` are excluded from the predictive feature set.

The data is anonymized and does not expose client names, URLs, titles, or private queries.

## 4. Methodology

### 4.1 Transparent baseline

The baseline is a human-readable scoring approach built from observable performance signals. It combines low CTR at 40%, average position opportunity at 35%, and 90-day impressions / visibility at 25%. This creates a transparent benchmark that can be inspected without a machine-learning model.

### 4.2 Learned models

Three supervised models were compared: Logistic Regression, Decision Tree, and Random Forest. Positive-class probabilities were used as ranking scores. Random Forest was selected for the final ranking because it produced the strongest overall results under the grouped validation used for the capstone.

### 4.3 Validation design

The primary split is a **client-grouped holdout**. Entire clients are kept out of training so the model is evaluated on clients it did not see during fitting.

Validation results from the capstone notebook:

* Training rows: 27,675
* Test rows: 2,325
* Held-out clients: 6
* Client overlap: 0
* Random seed: 42

This is more conservative than a simple random row split because it reduces the risk that pages from the same client appear on both sides of the evaluation.

## 5. Results

| Method | ROC AUC | Average Precision | Precision@20 | Precision@50 |
|---|---:|---:|---:|---:|
| Random Forest | **0.754** | **0.636** | **0.90** | **0.74** |
| Decision Tree | 0.740 | 0.570 | 0.65 | 0.62 |
| Logistic Regression | 0.703 | 0.521 | 0.35 | 0.34 |
| Baseline Rules | 0.528 | 0.443 | 0.30 | 0.46 |

The Random Forest is the strongest model across the ranking metrics reported by the capstone notebook.

For the primary operational metric, Precision@20 of 0.90 means that 18 of the top 20 ranked test pages were positive under the observed declining label. Precision@50 of 0.74 means that 37 of the top 50 ranked test pages were positive under that label.

Compared with the baseline, the Random Forest improved Precision@20 from 0.30 to 0.90 and Precision@50 from 0.46 to 0.74. ROC AUC also increased from 0.528 to 0.754.

These are held-out evaluation results on the anonymized starter dataset. They should not be interpreted as evidence that the model will achieve the same performance on a different dataset, on future production traffic, or on the full warehouse without revalidation.

## 6. Decision Workflow

The intended workflow is:

1. Prepare observable content and performance signals.
2. Apply the trained model to generate a decline-risk probability.
3. Rank pages by the model score.
4. Combine the learned ranking with transparent baseline evidence when producing the final review queue.
5. Where implemented, attach human-readable reason codes to help reviewers understand why a page was surfaced.
6. Review the highest-priority pages manually before taking action.

The prototype is therefore a prioritization system, not an automated content-editing system.

A high ranking should be interpreted as **review first**, not **guaranteed recovery**.

## 7. Leakage and Integrity Controls

The capstone explicitly separates identifiers and outcome-defining information from predictive features.

* `content_id` is treated as an identifier, not a predictive feature.
* `client_id` is used for grouped validation, not prediction.
* `trend_direction` is used to define the observed target and is excluded from predictive features.
* `trend_pct` is excluded because it is directly related to the outcome definition.
* The dataset contains anonymized observable signals rather than private client information.
* The client-grouped split was checked for overlap and returned zero client overlap.

These controls are intended to keep the reported validation result honest and reproducible.

## 8. What the Results Do and Do Not Show

The results show that, on this 30,000-row anonymized starter dataset and under the client-grouped holdout, the Random Forest ranks observed declining pages substantially better than the transparent baseline.

The results do **not** establish that:

* changing a page will cause traffic or rankings to recover;
* the model predicts Google's ranking algorithm;
* the model will generalize unchanged to the full warehouse;
* every high-ranked page is a good refresh candidate after human review;
* the observed decline cannot have another explanation such as consolidation, seasonality, or noise.

A content change causing recovery would require an experiment or another causal design. This capstone is decision-support work.

## 9. Limitations and Next Steps

The main limitation is scope. The evaluation is based on the anonymized 30,000-row starter dataset rather than the full warehouse. The grouped holdout provides a stronger validation design, but it remains one held-out evaluation rather than evidence of production performance across time and clients.

The next validation stage should test the workflow on additional client groups and, where appropriate, a future-looking time split. The final production workflow should also monitor ranking quality after deployment and periodically revalidate the model as content behavior changes.

No feature-importance analysis or detailed error-pattern analysis is claimed in this report because those results are not part of the verified capstone evidence used here.

## 10. Reproducibility

The project code and capstone notebook are stored in the repository at `work/notebooks/capstone.ipynb`.

The repository also contains the required Python environment specification used to support reproducible execution.

The capstone uses a fixed random seed of 42 and records the validation strategy, train/test sizes, held-out client count, client-overlap check, and evaluation metrics in the notebook.

## 11. Conclusion

The capstone demonstrates a practical machine-learning approach to content refresh prioritization. A Random Forest substantially outperformed the transparent baseline on the held-out clients, including a Precision@20 of 0.90 versus 0.30 for the baseline.

The strongest conclusion is therefore operational: **the learned ranking provides a more useful small review queue than the fixed baseline on this evaluation set.** The output should be used to focus human review, with the model treated as decision support rather than a guarantee of recovery or a causal prediction system.
