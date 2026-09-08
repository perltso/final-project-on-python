# Predicting Blood Transfusion Need in Dengue Fever Patients

Applied Data Science (83901) — Bar-Ilan University, Faculty of Engineering
Final project · Nechama Ehrman, Avia Shevach, Tsofiya Perel

## Abstract

Dengue fever can deteriorate quickly toward plasma leakage and bleeding, and a
blood transfusion may be needed during the hospital stay. We build a model that
predicts, **at admission** and from **routine laboratory tests only**, whether a
patient will require a transfusion — giving the clinical team an early warning
and helping the blood bank plan its stock. The clinical priority throughout is
**recall on the transfusion class** (a missed transfusion is far worse than a
false alarm).

The notebook goes through the full model-building process: EDA, data cleaning and
leakage removal, feature engineering, a Logistic Regression baseline, an ablation
/ robustness study, threshold tuning for clinical safety, an error analysis, a
Random Forest comparison, and formal statistical tests.

## Research questions

1. **Which admission biomarkers best predict the need for a transfusion?**
2. **Can admission-time routine lab tests alone reach high predictive accuracy?**
3. **How robust is the model in a low-resource field clinic** without an
   abdominal ultrasound or a fast platelet count?
4. **How does removing the dominant features change performance**, and does the
   choice of model matter?

## Key results

| # | Question | Answer |
|---|---|---|
| RQ1 | Top biomarkers | Abdominal free fluid, low albumin, low platelet count (then the liver enzymes). Agreed by the model coefficients, the ablation curve, and a Mann-Whitney test with Benjamini-Hochberg correction (8 of 13 markers significant). |
| RQ2 | Routine labs alone | Yes. Dropping the ultrasound costs essentially nothing; a basic lab with only a blood count reaches CV ROC-AUC ≈ 0.98. |
| RQ3 | Field-clinic robustness | Robust down to a clear capability threshold: the model needs a reliable **platelet count**, but not a chemistry panel or an ultrasound. With only point-of-care tests it collapses (ROC-AUC ≈ 0.5). |
| RQ4 | Removing dominant features | The predictive signal is concentrated in ~5 markers (`Abdominal_Free_Fluid`, `ALB`, `PLT`, `AST`, `ALT`); `PLT` alone carries most of it. Below ~7 features the model drops toward chance. On the reduced set a Random Forest is slightly better than the (tuned) Logistic Regression, but the linear model stays preferable for interpretability. |

**Deployment recommendation:** a Logistic Regression on six basic-lab features
(`Age`, `Gender`, `Hb`, `PLT`, `Lymphocytes%`, `Neutrophils%`) with a decision
threshold tuned for high recall. It needs only a blood count and patient history.

## Repository structure

| Path | Contents |
|---|---|
| `final_project_unified.ipynb` | Main notebook. **Part 1** — EDA (4 segments). **Part 2** — modelling workflow, Sections 1–6. |
| `PROJECT.md` | Internal project log: decisions, section-by-section status. |
| `סיכומי_סעיפים.{md,docx,html}` | Section-by-section plain-language summary (Hebrew). |
| `מצגת_חלק_מודלים.{md,docx,html}` | Presentation plan for the modelling part — per slide: content, narration, and the figure. |
| `slide_figures/` | The individual figures (PNG) used in the presentation. |

The dataset itself is **not** part of this repository.

## Notebook outline (Part 2)

| Section | What it does |
|---|---|
| 1 — Data cleaning & leakage closure | Drop `LOS` and row-order leakage, clean the target, fix impossible values, impute missing values (median / missing-indicator / observed distribution). |
| 2 — Feature engineering | Spearman correlation + within-class check; drop one redundant marker on clinical grounds (13 features). |
| 3 — Baseline model | Logistic Regression in a pipeline (impute → scale → fit); repeated 5-fold CV + held-out test; standardised coefficients (RQ1). |
| 4 — Ablation / robustness | Iterative feature removal; clinical capability-tier scenarios; defines the working feature sets (RQ2–RQ4). |
| 5 — Improved model, error analysis, statistics | Hyper-parameter tuning, decision-threshold tuning for recall ≥ 0.95, error analysis on the missed cases, Random Forest comparison, Mann-Whitney tests. |
| 6 — Final comparison & answers | Side-by-side comparison of all model configurations; reasoned answers to every research question; limitations and recommendation. |

## How to run

Python 3.8+ with:

```
pip install pandas numpy scikit-learn scipy statsmodels seaborn matplotlib jupyter
```

1. Place the dataset CSV in the project root.
2. Set the path: `file_path` in **Part 1 / EDA 1**, and `DATA_PATH` in the
   **Section 1.0 — Setup** cell (on Google Colab use the `/content/...` path).
3. Run the notebook top to bottom.

The random seed (`RANDOM_STATE = 42`) is fixed, so the results are reproducible.

## Limitations

- Small sample (172 usable patients); the held-out test set is only ~35, so
  cross-validation is the primary evidence throughout.
- A few markers separate the classes almost perfectly in this dataset, which
  makes the full-feature model saturated; the improvement work was done on a
  deliberately reduced feature set.
- Single dataset — external validation would be required before any clinical use.
