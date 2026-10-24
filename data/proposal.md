# Project Proposal
## 1. Research Question
Here
## 2. Dataset
-Name: findex_microdata_2025
-Source: World Bak
-Size & Variables: 144,090 (rows 2 to 144,091) & 199 columns
## 4. Two-source scan of existing work

**Source 1: Lusardi, Schneider & Tufano (2011), *Financially fragile households: Evidence and implications*.** This paper introduced the measure used here: can a person come up with a fixed emergency sum within 30 days? In 2009 US survey data, about one in four Americans said they certainly could not raise USD 2,000 in a month. Counting those who probably could not as well, close to half were financially fragile. Fragility was concentrated among less-educated people, families with children and people who had lost wealth, but it also reached the middle class. The paper is the conceptual basis for the target in this project.

**Source 2: Leive & Xu (2008), *Coping with out-of-pocket health payments: empirical evidence from 15 African countries*.** This paper used WHO World Health Survey data. It showed that between 23% and 69% of households paying for care financed those payments by borrowing or selling assets. High inpatient spending raised the chance of this hardship financing in 12 of the 15 countries, and the richest income quintile was protected in 9 of them. It shows that health shocks become household debt, and it frames a binary "hardship financing" outcome that fits a classification task. That is the basis for the secondary target here (`fin22d`, borrowed for medical purposes).

**Gap the project addresses.** Both studies explain fragility or hardship financing with inferential models fitted to the whole sample. Neither tests whether individual-level survey items can *predict* who is fragile in held-out data. Neither checks calibration, or whether a model transfers across regions. The project does both on the newest multi-country data.

## References

Leive, A., & Xu, K. (2008). Coping with out-of-pocket health payments: Empirical evidence from 15 African countries. *Bulletin of the World Health Organization, 86*(11), 849–856C. https://doi.org/10.2471/BLT.07.049403

Lusardi, A., Schneider, D. J., & Tufano, P. (2011). Financially fragile households: Evidence and implications. *Brookings Papers on Economic Activity, 2011*(1), 83–134. https://doi.org/10.1353/eca.2011.0002

World Bank. (2025). *The Global Findex Database 2025* (WLD_2024_FINDEX_v02_M) [Data set]. World Bank Microdata Library. https://doi.org/10.48529/bk9n-8r43
