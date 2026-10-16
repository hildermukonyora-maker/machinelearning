# Investor Behaviour Analysis - Project Report

*Generated automatically by `run_project.py`. Re-run the project and this file updates.*

## 1. Question
Which investment avenues do people prefer, can we predict their choices from their
profile, and are there natural investor segments?

## 2. Data
| Check | Result |
|---|---|
| Rows | 40 |
| Columns | 31 |
| Missing values | 0 |
| Duplicate rows | 0 |
| Rows with a valid 1-7 ranking | 40 of 40 |

Rankings run from 1 (most preferred) to 7 (least), so **a lower mean rank means a more
popular avenue**.

## 3. What the data shows (descriptive)
- **Most popular avenue by ranking:** PPF (mean rank 2.02);
  **least popular:** Gold (mean rank 5.98).
- **Stated preferred avenue:** Mutual Fund (18 of 40),
  then Equity (10). The ranking winner and the most common stated choice differ, which is worth flagging when presenting results.
- **Main factor when investing:** Returns (25 of 40).
- **Most common horizon:** 3-5 years (19 of 40).
- **Top information source:** Financial Consultants (16 of 40).
- **Gender/age effects on rankings:** 1 of 14 simple tests
  reached p < 0.05. With this many tests, one or two can appear by chance, so treat
  them as hints only.

| Avenue | Mean_Rank | Median_Rank | Times_1st | In_Top3 |
|---|---|---|---|---|
| PPF | 2.02 | 1.0 | 24 | 33 |
| Mutual_Funds | 2.55 | 2.0 | 4 | 34 |
| Equity_Market | 3.48 | 4.0 | 2 | 19 |
| Fixed_Deposits | 3.58 | 3.5 | 8 | 20 |
| Government_Bonds | 4.65 | 5.0 | 1 | 6 |
| Debentures | 5.75 | 6.5 | 1 | 6 |
| Gold | 5.98 | 6.0 | 0 | 2 |

### Exploratory Visualizations
![Age and Gender Distribution](figures/eda_01_age_gender.png)
![Categorical Variables Breakdown](figures/eda_02_categoricals.png)
![Avenue Popularity and Rank Distribution](figures/eda_03_avenue_popularity.png)
![Spearman Rank Correlations](figures/eda_04_rank_correlation.png)
![Stated Avenue by Gender](figures/eda_05_avenue_by_gender.png)

## 4. Can machine learning predict the choices?
Four targets were tried. Every model was scored with repeated stratified
cross-validation and compared with a "always guess the majority" baseline. The best
untuned model per target was then tested against shuffled labels (permutation test)
and re-run across several random seeds.

| Target | Best_Model | Accuracy | F1_macro | Baseline_Acc | Lift_vs_Baseline | Perm_p_value |
|---|---|---|---|---|---|---|
| Avenue (4 classes) | Random Forest | 0.464 | 0.354 | 0.451 | 0.014 | 0.095 |
| Avenue grouped (3 classes) | Random Forest | 0.537 | 0.458 | 0.451 | 0.086 | 0.095 |
| Monitoring: frequent vs monthly | Logistic Regression | 0.737 | 0.565 | 0.725 | 0.012 | 0.048 |
| Horizon: 3+ years vs shorter | Logistic Regression | 0.662 | 0.658 | 0.474 | 0.188 | 0.095 |

*How to read this:* `Lift_vs_Baseline` is accuracy minus the majority-guess baseline.
`Perm_p_value` compares the model with models trained on shuffled labels. A result is
only convincing when the lift is clearly positive **and** the p-value is small.

**Verdict:** No target was predicted clearly better than the baseline with statistical support. Promising but inconclusive (clear lift, p between 0.05 and 0.15): **Avenue grouped (3 classes)** (54% vs 45% baseline, p = 0.095); **Horizon: 3+ years vs shorter** (66% vs 47% baseline, p = 0.095). More responses would settle it.

Stability across seeds (does the conclusion hold when the randomness changes?):

| Target | Model | Mean_Acc | Min_Acc | Max_Acc | Baseline_Acc |
|---|---|---|---|---|---|
| Avenue (4 classes) | Random Forest | 0.42 | 0.397 | 0.443 | 0.451 |
| Avenue grouped (3 classes) | Random Forest | 0.464 | 0.427 | 0.501 | 0.451 |
| Monitoring: frequent vs monthly | Logistic Regression | 0.701 | 0.676 | 0.725 | 0.725 |
| Horizon: 3+ years vs shorter | Logistic Regression | 0.591 | 0.584 | 0.598 | 0.474 |

### Model Visualizations
![Model Comparison Across Targets](figures/model_01_comparison.png)
![Out-of-Fold Confusion Matrices](figures/model_02_confusion_matrices.png)
![Permutation Feature Importance](figures/model_03_feature_importance.png)

## 5. Investor segments
K-means on the seven rankings, k chosen by silhouette score, cross-checked with
hierarchical clustering (agreement ARI = 1.00; 1.0 = identical groups).
Best k = **3**.

- **Segment 1: Mutual_Funds + Equity_Market** (n = 4): average age 25.2, 25% male, most common stated avenue Equity, main factor Returns.
- **Segment 2: PPF + Mutual_Funds** (n = 31): average age 28.1, 71% male, most common stated avenue Mutual Fund, main factor Returns.
- **Segment 3: Government_Bonds + PPF** (n = 5): average age 28.2, 40% male, most common stated avenue Mutual Fund, main factor Returns.

Segments vs stated avenue: Cramer's V = 0.27 (0 = no link, 1 = perfect).
Predicting grouped avenue: demographics only 45%,
segment only 40%, demographics + segment
34%, baseline 45%.
Adding the segment changes accuracy by -12%, so the segments do not help predict the stated avenue. They describe how people rank avenues, not what they say they prefer. (Segments are built from the rankings, which overlap with the
stated avenue, so any gain here would be optimistic.)

| Label | Size | Mutual_Funds | Equity_Market | Debentures | Government_Bonds | Fixed_Deposits | PPF | Gold |
|---|---|---|---|---|---|---|---|---|
| Segment 1: Mutual_Funds + Equity_Market | 4 | 1.5 | 1.5 | 3.5 | 5.0 | 5.5 | 5.25 | 5.75 |
| Segment 2: PPF + Mutual_Funds | 31 | 2.48 | 3.52 | 6.52 | 5.06 | 2.97 | 1.55 | 5.9 |
| Segment 3: Government_Bonds + PPF | 5 | 3.8 | 4.8 | 2.8 | 1.8 | 5.8 | 2.4 | 6.6 |

### Segmentation Visualizations
![Investor Segment Profiles Heatmap](figures/segment_01_profiles.png)
![Investor Segments in 2-D PCA Space](figures/segment_02_pca.png)
![Cluster Evaluation - Silhouette Scores](figures/segment_03_choose_k.png)

## 6. Recommendations
1. Lead product messaging with the top-ranked avenues (PPF, Mutual_Funds), and position the
   lowest-ranked avenues as niche or diversification options.
2. Treat the largest segment as the mainstream audience and the smaller ones as
   targeted campaigns (see section 5 for what each segment favours).
3. Use the main information sources above as the channels for reaching investors.

## 7. Limitations and next steps
- Only 40 respondents and self-reported answers: results are a proof of
  concept, not a production model.
- Rare classes (for example PPF as a stated avenue, 3 people) cannot be learned reliably.
- Next: collect more responses, re-run `python run_project.py`, and revisit the
  targets that came close to beating the baseline.

## 8. Where everything is
- `outputs/tables/` - all result tables (CSV)
- `outputs/figures/` - all charts (PNG)
- `outputs/finance_with_segments.csv` - the dataset with each person's segment
