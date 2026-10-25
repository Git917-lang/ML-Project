## Abstract
Financial fragility, the inability to cope with an unexpected expense, is one channel through which economic shocks reach health. Using Global Findex 2025 microdata (98,815 adults, 98 economies), I trained two classifiers to identify adults who could not raise an emergency sum equal to 1/20 of gross national income per capita within 30 days, or could do so only with great difficulty (41.2% of adults). Predictors covered socio-economic position, shocks, indebtedness, financial worry, and formal and informal protection. An L2-penalised logistic regression and a histogram gradient-boosting model were tuned by cross-validation and evaluated once on a locked 20% test set with economy-clustered bootstrap intervals. Both outperformed an age–sex–income baseline (AUROC 0.646). Logistic regression reached AUROC 0.769 (95% CI 0.750–0.788) and boosting 0.779 (0.760–0.797), and both were well calibrated. Boosting's gain (0.010) was below the pre-specified 0.02 margin, so the interpretable logistic model was selected. Lower income, no savings, food bought on credit, borrowing for medical costs and income lost to disasters marked higher risk; insurance and internet use marked lower risk. Discrimination transferred to unseen regions (AUROC 0.70–0.80) but calibration did not, so local recalibration is needed before use in settings such as Bangladesh.
## 2. Methodology
# 2.1 Data:
The Global Findex Database 2025 (World Bank, 2025; Klapper et al., 2025) interviewed about 145,000 adults aged 15+ in 141 economies between May and December 2024. The public file used here has 144,090 respondents and 199 variables. The financial-resilience module was fielded in 98 economies (102,954 adults); after excluding don't know/refused answers on the target, the analysis sample was 98,815 adults, including 1,000 from Bangladesh. The file carries a within-economy survey weight but no cluster identifier or sub-national geography.
# 2.2 Outcomes
The primary target, FRAGILE, was rebuilt from raw items. It equals 1 if the respondent said they could not come up with an emergency sum equal to 1/20 of GNI per capita within 30 days (fin24 = 7), or that doing so would be very difficult (fin24a = 1). It equals 0 if doing so would be somewhat or not at all difficult. Prevalence was 41.2%. The secondary target, MEDBORROW, is borrowing for health or medical purposes in the past year (fin22d; 19.9%).
# 2.3 Cleaning and feature engineering
Yes/no items were recoded 1 → 1 and 2 → 0, with don't know and refused set to missing rather than to no. Twenty-six predictors were grouped by pathway:
•
socio-economic position: age, sex, education, within-economy income quintile, workforce status, rural residence and region;
•
economic shocks: natural disaster, and income lost because of it;
•
indebtedness and cost-of-living pressure: any borrowing, borrowing from family, food bought on credit, and borrowing for medical costs (primary model only);
•
psychological stress proxy: greatest financial worry;
•
formal protection: insurance payments, government support, public pension;
•
informal support: domestic and international remittances;
•
financial and digital inclusion: accounts, saving, digital payments, internet, phone and ID ownership.
All learned steps sat inside scikit-learn pipelines (Pedregosa et al., 2011) fitted on training folds only. For the logistic model these were a cubic B-spline for age, one-hot encoding, and mode imputation with missing-indicator columns. Gradient boosting used native handling of categories and missing values. Outcome items, identifiers and weights were blocked by an assertion. For MEDBORROW every borrowing item was removed, because each contains or follows the label.

# 2.4 Models, validation and metrics
Four models were compared:
2
•
B0: the training prevalence, used as a floor;
•
B1: a logistic baseline on age, sex and income quintile;
•
M1: L2-penalised logistic regression, with C tuned over 0.001–10;
•
M2: histogram gradient-boosted trees, with a 20-draw random search over learning rate, tree size, leaf size, L2 penalty and feature subsampling, plus early stopping.
No class re-weighting was used, because the outcome is common and re-weighting distorts calibration. A 20% test set, stratified by economy and label, was locked before tuning and scored once. Tuning used 5-fold cross-validation on log-loss. Out-of-fold (OOF) predictions fixed a screening threshold giving 80% sensitivity. A pre-set rule kept M1 unless M2's OOF AUROC was at least 0.02 higher. Test metrics were AUROC and AUPRC (discrimination), calibration intercept and slope (calibration), Brier score, net benefit (Vickers & Elkin, 2006), and sensitivity, specificity and PPV at the threshold. 95% confidence intervals came from a bootstrap that resampled economies (1,000 draws). Generalisation was tested by leave-one-region-out refitting, and a subgroup audit reported a subgroup as a finding only if it had at least 30 positives. Reporting follows TRIPOD+AI (Collins et al., 2024).

