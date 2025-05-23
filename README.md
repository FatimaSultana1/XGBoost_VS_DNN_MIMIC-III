# Comparative Analysis of XGBoost vs Deep Neural Networks for ICU Diagnosis (MIMIC‑III)

A fully reproducible pipeline that **downloads**, **cleans**, and **models** the open‑access MIMIC‑III database to predict critical‑care mortality, then **explains** each model’s behaviour.  
This README aligns with the Milestone 3 research plan and includes hypothesis, literature context, tools, and a week‑by‑week timeline.

---

## 1  Problem Statement
Early detection of adverse ICU outcomes saves lives, yet classical scores under‑capture non‑linear interactions present in EHR data.  
We compare two supervised learners:

| Model | Strengths | Weaknesses |
|-------|-----------|------------|
| **XGBoost** (tree ensemble) | Fast; handles missingness; interpretable via SHAP/gain | Limited depth of interactions |
| **Feed‑Forward DNN** | Learns arbitrary patterns; mixes continuous & one‑hot | Needs more compute; opaque (needs IG/SHAP) |

**Goal:** decide which model is *practically superior* for real‑time ICU use, balancing AUROC, recall, interpretability, and compute.

---

## 2  Hypothesis & Expected Outcomes
*H0 :* No meaningful AUROC difference between XGBoost and DNN  
*HA :* DNN attains higher AUROC, but XGBoost wins on interpretability & latency.

Expected results
* DNN AUROC ≈ 0.02–0.05 higher than XGBoost  
* SHAP (XGBoost) easier for clinicians; DNN uses Integrated Gradients  
* Inference: XGBoost < 50 ms vs DNN ≈ 200 ms per batch on CPU

---

## 3  Literature Snapshot
* **Hou 2020** – XGBoost AUROC 0.857 on sepsis‑3 mortality  
* **Purushotham 2017** – Deep nets beat classic ML on MIMIC‑III mortality  
* **Lee 2020** – Feature‑engineered XGBoost > raw‑signal DNN for hypotension  
* **Harutyunyan 2019** – Gradient‑boosted trees competitive with deep nets

Full citations in `/docs/references.bib`.

---

## 4  Dataset & Feature Scope
* **MIMIC‑III v1.4 10 k subset** – 10 376 adult ICU stays  
* Tables: `ADMISSIONS`, `PATIENTS`, `ICUSTAYS`, `DIAGNOSES_ICD`  
* Target: `HOSPITAL_EXPIRE_FLAG` (binary mortality)  
* Features (24 total)  
  * Continuous — `Age`, `LOS_hours`  
  * Categorical — `GENDER`, `ETHNICITY`, `ADMISSION_TYPE` (one‑hot)  
  * Count — `num_diagnoses`

---

## 5  Environment & Tools
| Layer | Stack |
|-------|-------|
| Python 3.10 | Conda env `mimic` |
| Data | pandas 2.1, NumPy 1.26 |
| Modelling | scikit‑learn 1.3, XGBoost 1.7, TensorFlow 2.13, SciKeras 0.12 |
| Explainability | SHAP 0.44, TensorFlow Integrated Gradients |
| Visuals | matplotlib 3.8, seaborn 0.13 |

Google Colab. GPU optional; CPU run < 15 min.

---

## 6  Pipeline Overview

1. **Download** MIMIC‑III CSVs with KaggleHub  
2. **Pre‑process** (`dataset_part_1‑6.ipynb`)  
   * Merge tables; compute `Age`, `LOS_hours`  
   * One‑hot encode categoricals; fill missing with 0  
3. **Train/test split** 80‑20 (stratified)  
4. **Model training**  
   * **XGBoost** GridSearchCV over depth, learning‑rate, estimators, subsample  
   * **DNN** GridSearchCV via SciKeras over units, dropout, lr, activation  
   * Class imbalance mitigated with `class_weight`  
5. **Evaluation** – ROC, PR, confusion matrix on hold‑out test set  
6. **Explainability**  
   * XGBoost gain + SHAP  
   * DNN Integrated Gradients (custom TF implementation)  
7. **Visualisation** – side‑by‑side bar charts, scatter agreement, cumulative curves  

---

## 7  Metrics & Tests
* **Primary** – AUROC  
* **Secondary** – precision, recall, F1 (class‑imbalance aware)  
* **Latency** – mean inference time per 128‑row batch  
* **Interpretability** – # features covering ≥ 80 % cumulative importance

---

## 8  Academic Timeline (Spring 2025)

| Week | Deliverable |
|------|-------------|
| 7‑8 | Data acquisition & cleaning scripts |
| 9‑10 | Feature engineering & XGBoost tuning |
| 11‑12 | DNN tuning & metric dashboards |
| 13 | SHAP & Integrated Gradients visualisation |
| 14 | Comparative analysis report & final README |

---

## 9  Feasibility & Risk Mitigation
* **Over‑fitting** – early‑stopping, dropout, L2  
* **Class imbalance** – `class_weight`, AUROC focus  
* **Compute budget** – small grids; GPU only for DNN  
* **Interpretability gap** – IG + cumulative coverage charts

---

## 10  Reproduce Locally
```bash
git clone https://github.com/FatimaSultana1/XGBoost_VS_DNN_MIMIC-III.git
cd XGBoost_VS_DNN_MIMIC-III
conda create -n mimic python=3.10 -y
conda activate mimic
pip install -r requirements.txt
jupyter notebook     # run notebooks sequentially
