# Mixed-Effects Modelling of Brain Morphology for Alzheimer's Disease Progression

## 📌 Overview

This repository accompanies the research work **"Mixed-Effects Modelling of Brain Morphology for Alzheimer's Disease Progression"**, published at the **2026 6th International Conference on Machine Learning and Intelligent Systems Engineering (MLISE)**, Naples, Italy.

The project focuses on longitudinal modelling of Alzheimer's Disease (AD) progression using linear mixed-effects (LME) models to characterise region-wise temporal brain trajectories from structural MRI. Building on this, we introduce **Trajectory-based Apparent Brain Features (t-ABF)** — interpretable biomarkers obtained by combining LME-derived trajectory modelling with an ensemble of ridge regression and logistic regression classifiers. Longitudinal feature selection (trajectory-aware Group Lasso) together with Biased Forward Feature Selection (BFFS) is used to identify the most informative region-of-interest (ROI) trajectories for predicting **MCI-to-AD conversion (sMCI vs. cAD)**.

Our goal is to bridge the gap between high-performing predictive models and clinical interpretability by relying exclusively on transparent linear models, avoiding the opacity of deep learning approaches, while remaining competitive with black-box baselines.

---

## 🎯 Objectives

* Model region-wise longitudinal brain trajectories using **linear mixed-effects (LME) models**, with AIC-based selection between linear and quadratic fixed-effect dynamics per ROI
* Derive **trajectory-based deviations (gaps)** between predicted and observed morphology as interpretable biomarkers of pathological progression
* Predict **MCI-to-AD conversion** using an ensemble of ROI-specific t-ABF linear classifiers
* Preserve **anatomical interpretability** while remaining competitive with SVM and DNN baselines
* Provide **reproducible pipelines** for longitudinal neuroimaging research

---

## 🧬 Dataset

* **ADNI, AIBL, OASIS-2** — T1-weighted structural MRI, 1,578 subjects / 6,679 images across cohorts
* **Modality:** Structural MRI (longitudinal, 2–14 timepoints per subject)
* **Type:** Longitudinal / repeated-measures data
* Preprocessing includes:
  * FreeSurfer 7.1.1 feature extraction (486 regional features → 439 after cleaning)
  * Outlier removal via Isolation Forest and Tukey's method
  * Intracranial Volume (ICV) normalisation via linear regression
  * Subjects with a single timepoint excluded (final: 564 subjects, 2,710 images for the sMCI→cAD transition task)

**Class definitions**

| Class | Description |
|---|---|
| sMCI | MCI at all follow-up visits from baseline; no conversion to AD observed (≥2 timepoints) |
| cAD | MCI at baseline with confirmed conversion to AD at any subsequent visit |

> ⚠️ **Note:** Due to data privacy restrictions, raw data is not included. ADNI, AIBL, and OASIS-2 are publicly available upon registration with their respective consortia.

---

### 🧠 Methodology Overview

<p align="center">
  <img src="workflow_diagram.png" alt="Pipeline Diagram" width="800"/>
</p>

**Step 1 — Preprocessing** (fit on training set, applied with fixed parameters on test set)
* Data cleaning, outlier detection and removal (iForest + Tukey)
* ICV volume normalisation
* Min-max scaling

**Step 2 — Linear Mixed-Effects (LME) Modelling**
* Per-ROI linear and quadratic LME models fit with age as the time covariate
* Model selection via ΔAIC (quadratic term retained only if it significantly improves fit; linear preferred otherwise, for parsimony/interpretability)
* Trajectory-features extracted per ROI: baseline deviation, last-visit deviation, and per-year rate of change (`max_change`)

**Step 3 — Longitudinal Feature Selection**
* Trajectory-Aware Group Lasso (TAGL) reduces the feature space from 439 → 54 relevant ROI biomarkers (treating each ROI's trajectory-features as a structured block)
* Biased Forward Feature Selection (BFFS) further selects a compact, ROI-specific predictor subset

**Step 4 — Trajectory-based Apparent Brain Feature (t-ABF) Models**
* For each target ROI: multi-output ridge regression estimates the expected trajectory from the remaining ROIs; logistic regression classifies sMCI vs. cAD from actual + predicted trajectory-features
* Produces an interpretable **ABF contribution score** per ROI, quantifying how much that region's trajectory gap contributes to conversion risk

**Ensemble Methods**
* Ensemble 1: Majority voting across ROI-specific t-ABF classifiers
* Ensemble 2: Individually optimised per-model thresholds, aggregated for a final subject-level prediction

---

## 📊 Results

10-fold cross-validation (training) and repeated hold-out testing (10 runs), relative SD < 2%:

| Model | Feature Selection | Train (CV) | Test (HO) |
|---|---|---|---|
| DNN (E1) | TAGL + BFFS | 0.82 | 0.77 |
| DNN (E2) | TAGL + BFFS | 0.80 | 0.74 |
| SVM (E1) | TAGL + BFFS | 0.76 | 0.76 |
| SVM (E2) | TAGL + BFFS | 0.89 | 0.73 |
| LR+Log. R (E1) | — | 0.76 | 0.71 |
| LR+Log. R (E2) | — | 0.78 | 0.69 |
| **LR+Log. R (E1)** | **TAGL + BFFS** | **0.78** | **0.74** |
| **LR+Log. R (E2)** | **TAGL + BFFS** | **0.80** | **0.74** |

*E1: Ensemble 1 (majority voting), E2: Ensemble 2 (optimised thresholds)*

**Key findings:**
* The t-ABF ensemble reaches performance competitive with SVM/DNN baselines (within 2–4 points of accuracy) while remaining fully interpretable — no post-hoc explainability (SHAP/LIME) required
* Individual t-ABF models such as *Cortical-nucleus-left* (84% train / 76% test), *presubiculum-body-left* (81% / 74%), and *right amygdala* (84% / 77%) capture distinct, clinically meaningful neuroanatomical patterns associated with AD progression
* ABF contribution scores identify which ROI trajectories drive each subject's prediction — e.g. `lh_rostralmiddlefrontal_meancurv`, `lh_precuneus_thickness`, `lh_caudalmiddlefrontal_thickness`, `cortical-nucleus_right`

---

## 🛠️ Repository Structure

```
├── data/                # Data handling scripts (no raw data included)
├── preprocessing/       # MRI preprocessing pipelines (FreeSurfer, outlier removal, ICV normalisation)
├── lme_models/          # Linear mixed-effects trajectory modelling and AIC-based model selection
├── feature_selection/   # TAGL (Group Lasso) and BFFS implementations
├── t_abf/                # t-ABF model (ridge regression + logistic regression per ROI)
├── ensemble/             # Majority voting and optimised-threshold ensemble methods
├── results/              # Outputs, figures, confusion matrices, evaluation metrics
├── code/                 # Jupyter notebooks for experiments
├── knime_workflows/      # KNIME pipelines (for ensemble method)
└── README.md
```

---

## ⚙️ Installation

```bash
git clone https://github.com/AliBhatti21/Interpretable-MRI-Based-Biomarkers-for-Alzheimer-s-Disease-Classification.git
cd Interpretable-MRI-Based-Biomarkers-for-Alzheimer-s-Disease-Classification
pip install -r requirements.txt
```

---

## 🔍 Reproducibility

* Fixed random seeds for experiments
* Strict train/test separation: preprocessing, LME trajectory fitting, feature selection, and t-ABF model fitting are all learned on the training fold only and applied with fixed parameters to the held-out set (no leakage)
* Modular pipeline design (preprocessing → LME → feature selection → t-ABF → ensemble) for easy experimentation

---

## 📄 Publication

**Title:** Mixed-Effects Modelling of Brain Morphology for Alzheimer's Disease Progression
**Authors:** H. M. A. Bhatti, A. Rosani, M. Faheem, U. Ali, U. Ramzan, G. Di Fatta
**Conference:** 2026 6th International Conference on Machine Learning and Intelligent Systems Engineering (MLISE), Naples, Italy
**Year:** 2026
**Pages:** 464–470
**DOI:** [10.1109/MLISE70044.2026.11607592](https://doi.org/10.1109/MLISE70044.2026.11607592)

**Citation:**
```
H. M. A. Bhatti, A. Rosani, M. Faheem, U. Ali, U. Ramzan and G. Di Fatta,
"Mixed-Effects Modelling of Brain Morphology for Alzheimer's Disease Progression,"
2026 6th International Conference on Machine Learning and Intelligent Systems
Engineering (MLISE), Naples, Italy, 2026, pp. 464-470,
doi: 10.1109/MLISE70044.2026.11607592.
```

This work builds on prior work introducing the Apparent Brain Features (ABF) framework:
> H. M. A. Bhatti, T. Borsani, A. Rosani, and G. Di Fatta, "Interpretable MRI-based biomarkers for Alzheimer's disease classification," in *Proceedings of the 18th International Conference on Brain Informatics (BI 2025)*, LNCS vol. 16347, Bari, Italy: Springer, 2026.

---

## 🤝 Contributions

Contributions are welcome! Please open an issue or submit a pull request for improvements or discussions.

---

## 📬 Contact

* **Name:** Hafiz Muhammad Ali Bhatti
* **Email:** Bhatti.hafizali@gmail.com / HafizMuhammadAli.Bhatti@student.unibz.it
* **LinkedIn:** [www.linkedin.com/in/muhammad-ali-bhatti-281409314](https://www.linkedin.com/in/muhammad-ali-bhatti-281409314)
* **Affiliation:** Faculty of Engineering, Free University of Bozen-Bolzano, Bolzano, Italy

---

## ⭐ Acknowledgements

This work was carried out within the framework of the National Recovery and Resilience Plan (PNRR), Mission 4 "Education and Research", Component 1, funded by the European Union – NextGenerationEU (CUP: I52B23000590005).

* Free University of Bozen-Bolzano
* GIFT University, Gujranwala, Pakistan
* Open datasets: ADNI, AIBL, OASIS-2, and the broader neuroimaging research community
