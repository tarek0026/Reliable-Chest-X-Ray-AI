# Embedding Analysis Report (Phase 1, task 1)

**Author:** Mostafa · **Notebook:** `notebooks/04_Embedding_Analysis.ipynb` · **Date:** 24 Nov 2026

## 1. Goal

We check whether pretrained image embeddings separate **Normal** from **Abnormal** chest X-rays, and whether they also carry non-disease information that a model could use as a shortcut, such as view position, scanner, sex or age.

## 2. Data and method

| Item | Value |
|---|---|
| Embeddings | TorchXRayVision DenseNet121 (`densenet121-res224-chex`), frozen, 1024 dimensions per image |
| Images | 20,000 (train 14,000 / validation 3,000 / test 3,000), balanced 50 % Normal / 50 % Abnormal in every split |
| Checks passed | All images matched to metadata, no missing values, no patient in more than one split |
| Fitting rule | StandardScaler, PCA and UMAP are **fitted on train only**. Validation and test are only transformed |
| PCA | 50 components (used as input for UMAP, t-SNE and the metrics) |
| UMAP | n_neighbors = 30, min_dist = 0.1, metric = cosine (robustness check with n_neighbors = 15 and 50) |
| t-SNE | perplexity = 30, init = PCA (train only, as a cross-check) |
| Overlap metrics | silhouette score (5,000-image sample), kNN class purity (k = 15), logistic-regression AUROC |
| Cross-validation | 5-fold, **grouped by patient** (StratifiedGroupKFold), train split only |
| Seed | 42 |
| Versions | Python 3.13.18, numpy 2.4.6, pandas 2.2.3, scikit-learn 1.6.1, umap-learn 0.5.12 |

**PCA variance:** PC1 explains 18.8 % and PC2 11.7 % (30.5 % together). The first 50 components explain 79.5 %. 148 components are needed for 90 % and 248 for 95 %.

## 3. Results

### 3.1 Class overlap (train)

| Space | Silhouette | kNN purity (all) | kNN purity Normal | kNN purity Abnormal | CV AUROC (mean ± std) |
|---|---|---|---|---|---|
| PC1–2 | 0.057 | 0.570 | 0.570 | 0.570 | 0.697 ± 0.010 |
| PCA-50 | 0.028 | 0.596 | 0.612 | 0.580 | **0.750 ± 0.008** |
| UMAP-2D | 0.036 | 0.577 | 0.579 | 0.575 | 0.646 ± 0.014 |
| Raw 1024 | – | – | – | – | 0.723 ± 0.005 |

Silhouette near 0 means the two classes do not form separate clusters. A kNN purity of 0.5 would be chance level for a balanced set.

### 3.2 How strongly the embeddings encode non-disease variables

| Variable | Target | All train | Normal images only |
|---|---|---|---|
| View position | AP vs PA (AUROC) | **0.990** | **0.989** |
| Pixel spacing | 0.143 vs rest (AUROC) | **0.999** | **0.999** |
| Image size | 2500×2048 vs rest (AUROC) | **0.964** | **0.971** |
| Sex | M vs F (AUROC) | 0.850 | 0.876 |
| Age | R² (regression) | 0.311 | 0.327 |

For comparison, Normal vs Abnormal reaches an AUROC of only **0.750** in the same space.

### 3.3 How these variables relate to the label (train, % Abnormal)

| Variable | Groups (% Abnormal) |
|---|---|
| View position | AP 56.9 % (n = 5,674) · PA 45.3 % (n = 8,326) |
| Pixel spacing | 0.139: 57.3 % · 0.168: 52.2 % · 0.143: 48.0 % · 0.194: 45.6 % · 0.171: 37.9 % |
| Image size | 3056×2544: 59.3 % · 2500×2048: 51.7 % · 2992×2991: 49.2 % · 2048×2500: 38.5 % |
| Follow-up number | first visit 33.3 % · 1–5: 47.8 % · 6–20: 61.6 % · >20: 67.8 % |
| Age group | <18: 44.3 % · 18–39: 45.5 % · 40–59: 49.8 % · 60–79: 56.2 % · 80+: 65.0 % (n = 157) |
| Sex | F 49.1 % · M 50.7 % |

### 3.4 Normal vs Abnormal AUROC inside each subgroup (train, PCA-50)

| Subgroup | n | AUROC ± std |
|---|---|---|
| PA | 8,326 | 0.755 ± 0.005 |
| AP | 5,674 | 0.718 ± 0.019 |
| Female / Male | 6,138 / 7,862 | 0.741 ± 0.014 / 0.758 ± 0.012 |
| Age <18 | 646 | 0.798 ± 0.048 |
| Age 18–39 | 3,876 | 0.783 ± 0.024 |
| Age 40–59 | 6,101 | 0.737 ± 0.010 |
| Age 60–79 | 3,220 | 0.695 ± 0.014 |
| Age 80+ | 157 | 0.508 ± 0.098 (very small group, unreliable) |

### 3.5 Which Abnormal findings look most like Normal

The table shows the share of each Abnormal image's 15 nearest neighbours that are also Abnormal, in PCA-50 space. A value below 0.5 means most of its neighbours are Normal.

| Finding (single label) | n | Share of Abnormal neighbours |
|---|---|---|
| Hernia | 43 | 0.397 |
| Nodule | 308 | 0.445 |
| Infiltration | 640 | 0.459 |
| Fibrosis | 134 | 0.466 |
| Pleural thickening | 132 | 0.483 |
| Atelectasis | 340 | 0.506 |
| … | | |
| Edema | 100 | 0.615 |
| Consolidation | 151 | 0.624 |
| Effusion | 176 | 0.625 |
| Multiple findings | 4,133 | 0.629 |

### 3.6 Figures

| File | Content |
|---|---|
| `01_pca_variance.png` | Explained variance per component and cumulative |
| `02_pca_by_class.png`, `03_pca_marginals.png` | PC1 vs PC2 by class, PC1–3 distributions |
| `04_umap_by_class_all_splits.png` | UMAP for train / validation / test, by class |
| `05_umap_sensitivity.png` | UMAP with n_neighbors 15 / 30 / 50 |
| `06_tsne_by_class.png` | t-SNE by class |
| `07_umap_by_metadata.png` | UMAP coloured by view, sex, age, image size, pixel spacing, follow-up, number of findings, finding |
| `08_umap_class_within_view.png` | Class pattern inside AP-only and PA-only images |
| `09_umap_by_split.png` | Train / validation / test on the same map |

## 4. Findings

### Finding 1: Normal and Abnormal overlap heavily, with only a weak signal between them
- **Observation:** The two classes do not form separate clusters in PCA, UMAP or t-SNE.
- **Evidence:** The silhouette score is 0.028–0.057 in every space. kNN purity is 0.57–0.60, where 0.5 is chance. A linear model reaches an AUROC of 0.697 on 2 PCs and 0.750 on 50 PCs. Using all 1,024 raw dimensions does not help (0.723).
- **Implication:** The task is not easy for a simple linear model on frozen features. Part of the reason may be label noise, because the labels come from report text mining. Baselines on these embeddings, such as Logistic Regression and SVM, should be expected to land near an AUROC of 0.75.
- **Recommendation:** Use about 0.75 AUROC as the realistic reference for the embedding baselines. Report AUROC and AUPRC rather than accuracy. Expect fine-tuned CNNs to be needed to beat this clearly.

### Finding 2: The embeddings encode acquisition settings much more strongly than disease (shortcut risk)
- **Observation:** View position, pixel spacing and image size can be predicted almost perfectly from the embeddings. This still holds when only Normal images are used, so it is not explained by disease. In the UMAP map (Figure 07), AP/PA and scanner settings form the main regions. One device group (2021×2021, pixel spacing 0.194) forms its own island. The Normal-rich region is mainly PA with pixel spacing 0.143, and the Abnormal-rich side is mainly AP.
- **Evidence:** AP vs PA AUROC is 0.990 (0.989 on Normal only). Pixel spacing reaches 0.999 and image size 0.964, compared with 0.750 for Normal vs Abnormal. These variables are also linked to the label: 56.9 % of AP images are Abnormal against 45.3 % of PA images. Pixel spacing 0.139 is 57.3 % Abnormal against 37.9 % for 0.171, and image size 3056×2544 is 59.3 % Abnormal against 38.5 % for 2048×2500.
- **Implication:** A model can raise its score by learning "AP view / this scanner setting means Abnormal" instead of learning lung findings. Overall metrics could then look better than the model's true medical ability, and the model may fail on data from other hospitals or a different AP/PA mix.
- **Recommendation:** In Phase 2, report every metric separately for AP and PA, and for the main pixel-spacing groups. Add a "metadata-only" baseline (view + pixel spacing + image size + age + sex) to measure how much of the score a shortcut alone can reach. Consider a view-balanced evaluation subset.

### Finding 3: Sicker, older and more frequently imaged patients are over-represented in Abnormal
- **Observation:** The share of Abnormal images rises strongly with the follow-up number and with age. Age and sex are also partly encoded in the embeddings.
- **Evidence:** 33.3 % of first-visit images are Abnormal, against 67.8 % of images after more than 20 follow-ups. By age group, the share rises from 45.5 % (18–39) to 56.2 % (60–79) and 65.0 % (80+). Sex is predicted with an AUROC of 0.850 and age with an R² of 0.311. Sex itself is balanced (F 49.1 % vs M 50.7 % Abnormal).
- **Implication:** Patients with many follow-ups contribute many similar images. Without a patient-wise split this would cause leakage, and our split already prevents it. Age acts as a second possible shortcut.
- **Recommendation:** Keep the patient-wise split, which is already verified. Do not use `follow_up` as a model input. Include age-group and follow-up-group results in the Phase 2 subgroup analysis.

### Finding 4: Separation is weaker for AP images and older patients
- **Observation:** Normal vs Abnormal is harder to separate in some subgroups.
- **Evidence:** AUROC is 0.755 for PA against 0.718 for AP. It is 0.783 for ages 18–39 against 0.695 for ages 60–79. The 80+ group (n = 157) is too small to judge (0.508 ± 0.098).
- **Implication:** Overall performance can hide weaker performance on AP and elderly patients, which are often the more clinically important cases.
- **Recommendation:** Report subgroup metrics with confidence intervals in Phase 2. Treat the 80+ group as "insufficient data".

### Finding 5: Subtle or small findings sit inside the Normal region
- **Observation:** Abnormal images with small or diffuse findings have mostly Normal neighbours.
- **Evidence:** The share of Abnormal neighbours is 0.397 for Hernia, 0.445 for Nodule, 0.459 for Infiltration and 0.466 for Fibrosis. It is 0.615–0.629 for Edema, Consolidation, Effusion and multi-finding images.
- **Implication:** Missed cases (false negatives) are likely to cluster in these findings. They are also natural candidates for referral to human review.
- **Recommendation:** In Phase 2, check sensitivity for each finding, and check whether uncertainty and referral catch these cases.

## 5. Limitations
- The embeddings come from one frozen model (DenseNet121 trained on CheXpert). Other models may encode the data differently.
- A 2D map (UMAP or t-SNE) distorts distances. All conclusions are therefore based on the numeric metrics, which are computed in the 50-dimensional space, not on the pictures alone.
- "Encodable" does not prove that a model *uses* the shortcut. That needs the Phase 2 subgroup and metadata-only tests.
- All metrics are computed on train with patient-grouped cross-validation. Validation and test were only projected for the plots.

## 6. Summary for the final report (for Ahmed)
- On frozen DenseNet121 embeddings, Normal and Abnormal **overlap heavily**: silhouette ≈ 0.03 and linear AUROC 0.75 on 50 PCs.
- The embeddings encode **view position (AUROC 0.99), pixel spacing (0.999) and image size (0.96)** much more strongly than disease, even within Normal images only. These variables are also linked to the label (AP 56.9 % vs PA 45.3 % Abnormal), so they are **shortcut risks**.
- The Abnormal share rises with follow-up count (33.3 % → 67.8 %) and age (45.5 % → 65.0 %). This confirms the need for the patient-wise split and for subgroup evaluation.
- Separation is weaker for AP (0.718 vs 0.755 for PA) and for ages 60–79 (0.695).
- Subtle findings (Nodule, Infiltration, Fibrosis, Hernia) lie inside the Normal region, so expect false negatives there.
- **Decisions for Phase 2:** stratify metrics by AP/PA, scanner group and age; add a metadata-only shortcut baseline; check per-finding sensitivity; expect embedding baselines around AUROC 0.75.
