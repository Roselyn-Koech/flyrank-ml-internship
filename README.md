# FlyRank ML Internship — Final Project Index

**Author:** Roselyn Koech  
**Track:** Machine Learning — Refresh / Content Opportunity Scoring  
**Project:** Applied Search Intelligence: Google Search Ranking & Discoverability

## Final Submission

This repository is the working and submission repository for my FlyRank ML Internship project. The project focuses on using machine learning to prioritize content pages for human review based on observed performance signals.

### Start here

| Deliverable | Location |
|---|---|
| Capstone notebook | [`work/notebooks/capstone.ipynb`](work/notebooks/capstone.ipynb) |
| Capstone report | [`work/capstone_report.md`](work/capstone_report.md) |
| Weekly assignment notebooks | [`work/notebooks/`](work/notebooks/) |
| Work index | [`work/README.md`](work/README.md) |
| Validation evidence | [`work/validation_evidence.jpg`](work/validation_evidence.jpg) |
| Model comparison | [`work/model_comparison.jpg`](work/model_comparison.jpg) |
| Deployed paper URL | [`submission/paper_url.txt`](submission/paper_url.txt) |
| Retrospective | To be added for FL-10 |
| Build-in-public post | To be added for FL-10 |
| Personal site | To be verified for FL-10 |
| Demo | To be recorded later; FL-09 demo is intentionally not claimed as complete yet |

## Project Summary

The capstone asks: **Which content pages should be reviewed first for refresh based on observed performance signals?**

The workflow prepares an anonymized dataset, establishes a transparent baseline, compares supervised learning models, evaluates them using a client grouped holdout, and produces a ranked review queue.

The final evaluation uses 30,000 prepared rows with an observed declining label rate of 54.2%. The client grouped holdout contains 27,675 training rows and 2,325 test rows, with 6 held out clients and zero client overlap.

Random Forest was the strongest model in the verified evaluation:

| Method | ROC AUC | Average Precision | Precision@20 | Precision@50 |
|---|---:|---:|---:|---:|
| **Random Forest** | **0.754** | **0.636** | **0.90** | **0.74** |
| Decision Tree | 0.740 | 0.570 | 0.65 | 0.62 |
| Logistic Regression | 0.703 | 0.521 | 0.35 | 0.34 |
| Baseline Rules | 0.528 | 0.443 | 0.30 | 0.46 |

The primary operational result is Precision@20 of 0.90, meaning 18 of the top 20 ranked test pages were positive under the observed declining label.

## What the System Does

**Prepare data → Score pages → Rank pages → Review highest priority candidates → Decide action → Monitor**

The model is a decision-support and prioritization tool. A high score means **review earlier**. It does not automatically determine whether a page should be rewritten, expanded, protected, pruned, or left unchanged.

## Assignment Map

| Assignment | Deliverable | Status |
|---|---|---|
| ML-02 | Research question | In repository |
| ML-03 | ML task framing | In repository |
| ML-04 | Data contract | In repository |
| ML-05 | Feature leakage check | Optional stretch / retired card |
| ML-06 | Signal audit | Optional stretch / retired card |
| ML-07 | Baseline score | In repository |
| ML-08 | Model | In repository |
| ML-09 | Validation audit | In repository |
| ML-10 | Action playbook | In repository |
| ML-11 | Capstone | In repository |
| ML-12 | Demo outline, social-post cut, employer-facing summary | In capstone notebook closing section |
| FL-09 | Documentation and demo | Written documentation present; demo recording to be completed later |
| FL-10 | Final package and retrospective | This final checkpoint |

## Evidence and Reproducibility

The capstone uses a fixed random seed of 42 and documents data preparation, target definition, feature preparation, baseline construction, model comparison, grouped validation, evaluation metrics, and model selection.

The final report includes the verified validation and model-comparison evidence, limitations, and AI transparency notes.

See [`work/capstone_report.md`](work/capstone_report.md) for the complete technical report.

## Limitations

The results are based on the anonymized 30,000-row starter dataset and one client grouped holdout. The evaluation is not a full production backtest or a causal experiment. The model identifies pages associated with the observed declining label; it does not prove that changing a page will improve future performance or predict Google's ranking algorithm.

Additional time-based validation, broader client evaluation, monitoring, and production infrastructure would be needed before operational deployment.

## AI Transparency

AI assistance was used during development to help structure, review, and document parts of the project. The data preparation, leakage exclusions, validation design, evaluation metrics, and final claims were checked against the capstone notebook. Unsupported feature-importance, causal, production-performance, and page-level claims were intentionally excluded.

## FL-10 Items Still Being Finalized

The following items are intentionally left visible rather than being represented as complete before they exist:

1. **500–800 word retrospective** — next deliverable to add.
2. **Build-in-public post** — to be published after the final story is finalized.
3. **Personal site** — live URL to be verified and added.
4. **Hours log** — to be completed in the FlyRank portal.
5. **FL-09 demo recording** — to be recorded separately after the written package is complete.
6. **Final review checkpoint** — to be submitted after the package is complete.

## Repository Structure

```text
.
├── README.md                         # Final project index
├── work/
│   ├── notebooks/                    # Weekly assignment notebooks + capstone
│   ├── capstone_report.md            # Final capstone report
│   ├── validation_evidence.jpg       # Validation evidence
│   ├── model_comparison.jpg          # Model comparison evidence
│   └── README.md                     # Detailed work and assignment map
├── submission/
│   ├── README.md
│   └── paper_url.txt                 # Deployed paper URL
├── notebooks/                        # Starter/reference notebooks
├── scripts/                          # Reference pipeline
└── docs/                             # Dataset and ML guidance
```

## Original Project Documentation

The original setup, pipeline, data-safety, and reference documentation remain available in the repository for reproducibility.

## Quickstart

The fastest path is Google Colab. For local execution:

```bash
git clone https://github.com/Roselyn-Koech/flyrank-ml-internship.git
cd flyrank-ml-internship
pip install -r requirements.txt
python scripts/run_all.py
```

Only the anonymized starter dataset should be used. Do not add private client data, credentials, client names, URLs, titles, or private search queries to this repository.

## License and Credit

The project was completed as part of the FlyRank ML Internship using the public safe anonymized starter dataset and the internship's technical foundation.
