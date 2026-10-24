# Methodology

## 1. Refined research question

The overarching question links economic shocks to cardiometabolic disease, catastrophic health spending and integrated care. The project-wide feasibility audit (Stage 10) found that the Global Findex 2025 file has no biomarkers, no out-of-pocket amounts and no care-seeking items. Those parts therefore cannot be tested with this dataset. The question was refined to the **financial-resilience arm** of the pathway, which Findex measures directly:

> **RQ (primary).
-Among adults in the 98 economies that fielded the Findex 2025 resilience module, how accurately can a person's socio-economic position, economic shocks, indebtedness and cost-of-living pressure, financial worry, and formal and informal protection identify those who are **financially fragile**? Here *financially fragile* means unable to raise 1/20 of GNI per capita within 30 days, or able to do so only with great difficulty.
> **RQ-a.
-Does a flexible learner (gradient-boosted trees) beat a penalised logistic regression by at least 0.02 AUROC (the pre-set minimal important difference)?
> **RQ-b.
-Do both models beat a simple age–sex–income baseline?
> **RQ-c.
-Does the model transfer to world regions it was not trained on, and is its error similar across sex, income, residence, education and region?
>
> **RQ (secondary).
-Can the same predictors, excluding all borrowing items, identify adults who **borrowed for health or medical purposes** in the past year?

The models are **cross-sectional screens for current fragility**, not forecasts or causal estimates.

## 2. Dataset description

Source: World Bank, *Global Findex Database 2025*, catalogue 7860, `WLD_2024_FINDEX_v02_M`, DOI 10.48529/bk9n-8r43 
Raw Data size:144,090 adults are (15+) × 199 variables, 140 economies; Gallup World Poll fieldwork May–Dec 2024 |
Analysis population: 102,954 adults in the **98 economies** that asked the resilience module (fin24). The module was skipped in 42 mostly high-income economies. After removing *don't know/refused* answers on the target: **98,815 adults** |
| Unit / structure | One row per respondent; within-economy survey weight `wgt`; no PSU or sub-national geography in the public file |

### Primary target: `FRAGILE` (binary)
fin24 asks what the *main* source of emergency money would be. Code 7 means *"I could not come up with the money"*. fin24a (asked if a source was named) asks how difficult it would be: 1 very, 2 somewhat, 3 not at all.

FRAGILE = 1 if fin24 == 7 **or** fin24a == 1 → 40,711 adults (41.2%).
* FRAGILE = 0 if fin24a ∈ {2, 3} → 58,104 adults.
* fin24 codes 8–9 and fin24a codes 4–5 (*don't know/refused*) → excluded (4,139 adults).

### Secondary target: `MEDBORROW` (binary)
fin22d, borrowed for health or medical purposes in the past 12 months: 1 = yes (20,414; 19.9%), 0 = no (82,259).

### Features (26 engineered predictors, grouped by the pathway they represent)

athway block | Features (Findex source) |

| Socio-economic position | age, female, education (educ), within-economy income quintile (inc_q), in workforce (emp_in), rural (urbanicity), World Bank region |
| Economic shock | experienced a natural disaster (fin24c); lost income because of it (fin24d1) |
| Indebtedness and cost-of-living pressure | borrowed any (borrowed), from family or friends (fin22b), bought food on credit (fin22f), borrowed for medical purposes (fin22d, primary model only) |
| Psychological stress proxy | greatest financial worry (fin45: old age, business, **medical costs**, monthly expenses, school fees, other) |
| Formal financial and social protection | pays for insurance (fin19), government financial support (fin37), public pension (fin38) |
| Informal support | received domestic (fh2) or international (fh2a) remittance |
| Financial and digital inclusion | financial-institution account, mobile-money account, saved in past year, any digital payment, internet use, owns a phone (con1), owns foundational ID (fin46) |

### Limitations of the data
1. **No health measures.** There are no biomarkers, diagnoses, OOP amounts or care pathways, so the cardiometabolic and integrated-care parts of the overarching question are out of scope.
2. **Cross-sectional design.** Features and label come from one interview, so reverse causation is possible (for example, fragility may cause borrowing).
3. **Self-report.** The answers are hypothetical (could you raise the money?) and are subject to recall and social-desirability bias.
4. **Coarse geography.** Only economy and urban/rural are released. There are no PSUs, so the test split cannot be clustered below economy level, and Dhaka-level analysis is impossible. Bangladesh has 1,000 respondents.
5. **Item missingness by design.** Some modules were not fielded in about 11 economies, and mobile-money items were skipped where mobile money is rare.
6. **Relative emergency amount.** The sum is 1/20 of GNI per capita, so fragility is relative to each economy's income level.

## 3. Data-cleaning plan

1. **Provenance:** record the file name, row and column counts and SHA-256 checksum (results/data_manifest.json`).
2. **Population filter:** keep economies where fin24 was asked. Log every exclusion step (results/participant_flow.csv).
3. **Recode survey codes:** yes/no items become 1 → 1 and 2 → 0. *Don't know* (3) and *refused* (4) become missing; they are not treated as "no". Skip-logic blanks stay missing. The one exception: disaster income loss is set to 0 when no disaster was reported.
4. **Targets** are rebuilt from raw items. Rows with a *don't know/refused* target are dropped.
5. **Predictor missingness** is kept and handled inside the model pipeline (Section 4), not by dropping rows.
6. **Leakage control:** the target items (fin24, fin24a, fin24b, fin22d for the secondary model), IDs and weights are placed on a forbidden list that an assertion enforces. For `MEDBORROW`, **all** borrowing items are removed.

## 4. Feature-engineering plan

All learned steps sit inside a scikit-learn `Pipeline`/`ColumnTransformer` and are refitted on each training fold only.

| Step | Logistic models | Gradient boosting |
|---|---|---|
| Age | median imputation → cubic B-spline (4 knots) → standardise | raw (trees find non-linearity) |
| Education, income quintile | mode imputation → one-hot (first level dropped) | ordinal numeric |
| Region, financial worry | constant "missing" level → one-hot (levels < 1% pooled) | native categorical |
| Binary items | mode imputation **plus missing-indicator columns** | native missing-value handling |
| Class imbalance | none: the 41% prevalence is not rare, and re-weighting distorts calibration | none |

## 5. Models and why

| ID | Model | Why |
|---|---|---|
| B0 | Predict the training prevalence | Floor for Brier score and net benefit |
| B1 | Logistic regression on age (spline), sex, income quintile | A simple demographic targeting rule that a programme could use without a survey. Complex models must beat it |
| **M1** | **L2-penalised logistic regression**, `C` tuned over {0.001…10} by 5-fold CV on log-loss | Transparent (odds ratios), well calibrated, and a strong benchmark for tabular survey data |
| **M2** | **Histogram gradient-boosted trees**, 20-draw random search over learning rate, leaves, minimum leaf size, L2 penalty and feature subsampling, with early stopping | Captures non-linearities and interactions (for example, borrowing × income) and handles missing values and categories natively |

**Selection rule (fixed before testing):** keep M1 unless M2's out-of-fold AUROC is at least 0.02 higher.

## 6. Validation design

1. **Locked test set:** 20% of respondents, stratified by economy × label, fixed seed 42, drawn before any tuning and scored once.
2. **Development set (80%):** 5-fold stratified CV for tuning (log-loss) and for out-of-fold (OOF) predictions.
3. **Threshold:** fixed on OOF predictions so that OOF sensitivity is 0.80 (a programme that wants to reach 4 in 5 fragile adults).
4. **Transfer test:** leave-one-region-out on the development set, 7 folds.
5. **Uncertainty:** cluster bootstrap that resamples **economies** (1,000 draws) for test CIs, plus a paired bootstrap for differences in AUROC.
6. **Subgroup audit:** sex, age band, income quintile, residence, education, region, and Bangladesh (descriptive). A subgroup estimate is reported as a finding only if the subgroup has at least 30 positives.

## 7. Evaluation metrics and why

| Metric | Why it is used |
|---|---|
| **AUROC** (co-primary) | Threshold-free ranking: how well the model orders fragile above non-fragile adults |
| **Calibration intercept and slope, calibration curve** (co-primary) | A screening or targeting tool must give *correct probabilities*, not only a correct ranking |
| AUPRC | Precision–recall summary against the 41% prevalence line |
| Brier score and scaled Brier | Overall probabilistic accuracy compared with B0 |
| Net benefit (decision curve) | Clinical and programmatic usefulness across threshold probabilities compared with "target all" and "target none" |
| Sensitivity, specificity, PPV, NPV, F1, share flagged | Operating characteristics at the pre-set threshold |
| Survey-weighted AUROC | Sensitivity analysis for population representativeness |

## 8. Ethics and reproducibility

The analysis uses an anonymised public file and cites it as the publisher requires. No re-identification or linkage is attempted. A single seed, pinned package versions and a split-assignment file make every number reproducible from `notebooks/analysis.ipynb`.
