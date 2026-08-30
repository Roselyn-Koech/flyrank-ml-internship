# FL-10 Retrospective

## Looking Back on My FlyRank ML Internship

At the beginning of the FlyRank ML Internship, I wanted to become more confident in applying machine learning to a practical problem rather than only working through isolated exercises. My project focused on **Refresh / Content Opportunity Scoring**, with the goal of identifying which content pages should be reviewed first based on observed performance signals.

At first, I was mainly focused on building and comparing machine learning models. As the project progressed, I realized that building a useful ML system involves much more than choosing an algorithm and reporting a high score. I became more focused on the decision the model was intended to support, the quality of the validation strategy, leakage prevention, and whether the evidence actually justified the claims I was making.

One of the most important changes in my approach was my understanding of validation. Rather than treating a random row split as sufficient, I used a **client grouped holdout** so that entire clients were held out from training. The final independently reconstructed validation contained 27,675 training rows from 26 clients and 2,325 test rows from 6 completely held out clients, with zero client overlap. The held out test clients also had a lower declining label rate than the training clients, which provided an additional distribution shift check.

The Random Forest remained the strongest model under this evaluation. The independently reconstructed validation produced a **ROC AUC of 0.745**, compared with the earlier Week 5 reported ROC AUC of 0.750. Average Precision was 0.611, while Precision@50 was 0.72. Precision@20 was 0.80, although I would not interpret that increase alone as evidence of general improvement because it is based on only the top 20 predictions.

This validation experience taught me an important lesson: **a model score is only meaningful when I understand how the score was produced and what question the evaluation answers**. I also learned to be careful about the difference between association and causation. My model identifies pages associated with the observed declining label, but it does not prove that refreshing a page will cause its performance to improve. It also does not predict Google's ranking algorithm.

Another major change was learning to communicate limitations honestly. Earlier in my ML learning, I might have focused mainly on the strongest metric. During this internship, I learned that a stronger technical explanation includes what the model does well, where the evaluation is limited, and what cannot reasonably be concluded from the available evidence.

The project also changed how I think about model selection. I compared a transparent baseline with Logistic Regression, Decision Tree, and Random Forest rather than assuming that the most complex model would automatically be the best choice. The goal was to establish whether the learned models provided meaningful improvement over a simple, interpretable benchmark.

If I continued the project, I would add stronger temporal validation, evaluate more client groups, monitor performance over time, and connect the ranked recommendations to actual content outcomes. I would also want to study whether pages identified by the system eventually improve after human reviewed interventions. That would require a more appropriate experimental or causal design rather than relying only on observational data.

### Three transferable lessons

**1. Start with the decision, not the model.**

A machine learning model should solve a clearly defined problem. Thinking first about the decision the output will support helped me choose ranking metrics such as Precision@20 instead of focusing only on general classification metrics.

**2. Validation design matters as much as model selection.**

A model can appear strong under an inappropriate split. Using client grouped validation made me think more carefully about generalization and whether the evaluation represented the situation in which the system might eventually be used.

**3. Honest limitations make technical work more credible.**

I learned that saying what a model cannot prove is part of good technical communication. The model can help prioritize pages for human review, but it cannot guarantee future recovery or establish causality.

Looking back to Week 1, I would now approach the same project differently. I would define the decision and evaluation strategy earlier, establish leakage controls before modelling, and treat the final metric as only one part of the evidence.

The biggest outcome of this internship is therefore not simply that I trained a Random Forest model. It is that I became more comfortable building a machine learning project that I can explain, evaluate, defend, and communicate honestly.
