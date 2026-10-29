# Dataset Splitting and Disease-Aware Sampling

## 1. Overview

This document describes the methodology used to prepare the final 20,000-image subset from the NIH ChestX-ray14 dataset for the Reliable Chest X-Ray AI project.

The objective was to create a dataset suitable for binary chest X-ray classification into **Normal** and **Abnormal** classes while addressing three important requirements:

1. Preventing patient-level data leakage between training, validation, and test sets.
2. Maintaining an equal number of Normal and Abnormal images in each split.
3. Improving the representation of less frequent disease labels among the selected Abnormal images.

Instead of selecting 20,000 images randomly from the original dataset, the pipeline first assigned patients to independent splits and then selected images within each split. Disease-aware sampling was applied to the Abnormal image pool to improve the representation of less frequent findings while preserving the original multi-label annotations.

The final dataset contains 20,000 images from 10,567 unique patients.

## 2. Dataset Overview

The source dataset is NIH ChestX-ray14, which contains chest X-ray images associated with radiological findings extracted automatically from medical reports.

The metadata used in the preparation pipeline includes fields such as:

- `image_name`: The image filename.
- `finding_labels`: The original finding label or labels associated with the image.
- `patient_id`: The identifier used to group images belonging to the same patient.
- Demographic and image-acquisition metadata, where available.

### Original dataset statistics

| Metric | Count |
|---|---:|
| Total images | 112,120 |
| Unique patients | 30,805 |
| Images labeled Normal | 60,361 |
| Images labeled Abnormal | 51,759 |
| Disease labels | 14 |

The original dataset is not perfectly balanced between the two binary classes. In addition, disease findings have different frequencies, and an image may contain more than one disease label.

These properties motivated the sampling and splitting strategy described below.

## 3. Binary Label Construction

The original `finding_labels` field contains the original finding annotations. These labels were used to derive a binary target called `binary_label`.

The mapping rules are:

- **Normal:** The image has `No Finding` as its only finding label.
- **Abnormal:** The image has at least one disease label.

If an image contains multiple disease labels, it is classified as Abnormal, and all its original disease labels are retained.

For example:

| Original finding labels | Binary label |
|---|---|
| `No Finding` | Normal |
| `Effusion` | Abnormal |
| `Pneumonia` | Abnormal |
| `Mass|Nodule` | Abnormal |
| `Atelectasis|Effusion` | Abnormal |

The binary label supports the project's main classification objective, while the original findings remain available for disease-distribution analysis and future experiments.

The binary classification task does not imply that all Abnormal images represent the same disease. The Abnormal class combines multiple possible findings.

## 4. Patient-Level Data Splitting

### 4.1 Why patient-level splitting was necessary

A patient may contribute multiple chest X-ray images to the dataset. Images from the same patient can share patient-specific characteristics, acquisition conditions, or recurring medical findings.

If images belonging to the same patient appear in both the training and test sets, the model may benefit from patient-specific information that would not be available for a genuinely unseen patient. This can lead to an overly optimistic evaluation.

To reduce this risk, **the split was performed at the patient level before selecting individual images**.

Every patient was assigned to exactly one of the three splits. All selected images from that patient were restricted to the assigned split.

### 4.2 Splitting procedure

The patient assignment process used `MultilabelStratifiedShuffleSplit`.

The process consisted of two stages:

1. **Training versus temporary pool:** Approximately 70% of patients were assigned to the training pool, and the remaining 30% were assigned to a temporary pool.
2. **Validation versus test:** The temporary pool was divided approximately equally into validation and test groups, producing the intended 70% / 15% / 15% split proportions.

Patient-level stratification was used to help preserve the distribution of the class and disease-label indicators across the patient groups.

A fixed random seed of `42` was used for reproducibility.

The resulting patient counts are not expected to be exactly 70%, 15%, and 15% of the original patient population after image selection. Patients contribute different numbers of images, and the final sampling procedure selects only a subset of their available images.

### 4.3 Important ordering decision

The pipeline follows this order:

1. Load and inspect the metadata.
2. Construct the binary classification labels.
3. Build the patient-level split assignments.
4. Select the required Normal and Abnormal images independently within each split.
5. Apply disease-aware sampling to the Abnormal image pool.
6. Validate the final subset.
7. Export the split metadata and patient assignments.

This ordering prevents disease-aware sampling from moving a patient or an image from one split to another.

## 5. Disease-Aware Abnormal Sampling

### 5.1 The problem we wanted to solve

The original dataset contains 14 disease labels with different frequencies. Some findings occur in thousands of images, whereas others are substantially less common.

A simple random sample of Abnormal images may reproduce the original imbalance, leaving relatively few examples of rare findings in the selected dataset.

For example, less frequent labels such as `Hernia`, `Pneumonia`, and `Fibrosis` may have much lower representation than more frequent labels such as `Infiltration` and `Effusion`.

This matters because a subset of only 10,000 Abnormal images may not represent every disease equally well. If rare findings are underrepresented, the model has fewer examples from which to learn their associated visual patterns.

However, the project's primary task is binary classification, not 14-class disease classification. Therefore, the objective was **not to create equal numbers of images for all 14 diseases**. Instead, we aimed to improve rare-disease representation while maintaining the required binary class counts and preserving the original annotations.

### 5.2 Why disease-aware sampling was selected

Disease-aware sampling was introduced to address the imbalance among disease labels without changing the binary classification objective.

The approach has four main goals:

1. **Improve rare-disease representation:** Increase the relative representation of less frequent findings compared with ordinary random sampling.
2. **Preserve the binary class balance:** Select exactly the required number of Abnormal images for each split.
3. **Preserve multi-label information:** Keep every original finding associated with a selected image.
4. **Prevent duplicate selections and patient leakage:** Select images without replacement and operate only within the patient's previously assigned split.

This strategy improves the composition of the selected Abnormal subset without pretending that every disease has the same natural prevalence.

### 5.3 Why we did not simply use random sampling

Random sampling is straightforward, but it does not actively account for rare disease labels.

When only a limited number of Abnormal images can be retained, random selection may leave some less frequent findings poorly represented. Disease-aware sampling instead considers which disease labels are still below their desired representation targets when choosing the next image.

This gives the sampling process a specific purpose beyond merely reaching 10,000 Abnormal images.

The strategy does not guarantee that every rare disease will be represented equally, and it does not guarantee an optimal combination of labels. It is a heuristic designed to improve representation under a fixed image budget.

### 5.4 Balancing parameter: `BALANCE_ALPHA = 0.75`

The sampling procedure used the following configuration:

`BALANCE_ALPHA = 0.75`

The desired representation target for each disease was calculated using:

\[
T_d = \min\left(A_d,\left\lceil Np_d^\alpha\right\rceil\right)
\]

Where:

- \(T_d\) is the desired target count for disease \(d\).
- \(A_d\) is the number of available images containing disease \(d\) in the relevant Abnormal pool.
- \(N\) is the number of Abnormal images requested for that split.
- \(p_d\) is the prevalence of disease \(d\) in that Abnormal pool.
- \(\alpha\) is the balancing parameter, set to 0.75.
- \(\lceil \cdot \rceil\) rounds the result upward to an integer.

Because the exponent is below 1, \(p_d^{0.75}\) is relatively larger than \(p_d\) for prevalences between zero and one. Consequently, the desired target for a less frequent disease becomes larger relative to a purely proportional target.

The availability cap, \(A_d\), prevents the target from exceeding the number of available images containing that disease.

This parameter controls the target calculation. It does not force the final disease counts to match the targets exactly, because selected images can contain multiple findings and contribute to several disease counts simultaneously.

### 5.5 How the sampling algorithm works

The disease-aware selection process operates on the Abnormal images available within each previously assigned split.

The procedure is:

**Step 1 — Represent the findings.**

For each Abnormal image, construct an indicator representation of its disease labels. Each disease is marked as present or absent for that image.

**Step 2 — Calculate disease prevalence.**

Calculate the prevalence of each disease within the relevant Abnormal image pool. These prevalence values are used to establish the desired representation targets.

**Step 3 — Calculate target counts.**

Use the balancing formula and `BALANCE_ALPHA = 0.75` to determine a target count for each disease, subject to the number of available images.

**Step 4 — Identify the largest relative shortfall.**

At each iteration, prioritize a disease whose current representation is furthest below its desired target relative to that target.

**Step 5 — Select an image containing that disease.**

Randomly select an unused image that contains the prioritized disease.

**Step 6 — Update all disease counts.**

After selecting an image, update the representation count for every disease label associated with that image, not only the disease that triggered the selection.

**Step 7 — Continue until the required count is reached.**

Repeat the process until the split contains the required number of Abnormal images.

The algorithm does not select an image more than once. If an image contains multiple findings, all of those findings remain attached to it in the final metadata.

### 5.6 Why multi-label preservation matters

An important property of this dataset is that the disease labels are not mutually exclusive.

For example, consider an image with the original labels:

`Atelectasis|Effusion`

This image contributes:

- One image to the binary Abnormal class.
- One occurrence of `Atelectasis`.
- One occurrence of `Effusion`.

It must not be converted into a single disease category simply to make sampling easier.

Preserving these labels is important because:

- It retains the original annotation information.
- It avoids discarding clinically relevant findings.
- It makes the disease-representation analysis meaningful.
- It allows the metadata to support future multi-label experiments, if required.

Consequently, disease counts overlap and their percentages are not expected to add up to 100%.

### 5.7 Why the disease targets do not produce equal disease counts

Disease-aware sampling is not the same as equal-count sampling.

The target for each disease depends on its prevalence, the balancing parameter, and the number of available examples. In addition, selecting one image can increase the counts of several diseases simultaneously.

For these reasons, the final dataset does not contain the same number of images for each disease.

This is intentional: the method aims to improve the representation of less frequent diseases while retaining the original multi-label structure, not to create an artificial 14-class balanced dataset.

## 6. Final Dataset Size and Split Distribution

### 6.1 Images and unique patients

The final subset contains 20,000 images distributed as follows:

| Split | Images | Unique patients | Percentage of images |
|---|---:|---:|---:|
| Train | 14,000 | 7,365 | 70% |
| Validation | 3,000 | 1,595 | 15% |
| Test | 3,000 | 1,607 | 15% |
| **Total** | **20,000** | **10,567** | **100%** |

The patient count represents unique patients contributing at least one selected image to each split.

### 6.2 Binary class distribution

| Split | Normal | Abnormal | Total |
|---|---:|---:|---:|
| Train | 7,000 | 7,000 | 14,000 |
| Validation | 1,500 | 1,500 | 3,000 |
| Test | 1,500 | 1,500 | 3,000 |
| **Total** | **10,000** | **10,000** | **20,000** |

Every split has a 50:50 Normal-to-Abnormal image ratio.

This balance was deliberately enforced during image selection rather than inherited from the original dataset.

### 6.3 Unique patients represented within each binary class

| Split | Binary class | Images | Unique patients |
|---|---|---:|---:|
| Train | Normal | 7,000 | 5,046 |
| Train | Abnormal | 7,000 | 3,676 |
| Validation | Normal | 1,500 | 1,090 |
| Validation | Abnormal | 1,500 | 803 |
| Test | Normal | 1,500 | 1,116 |
| Test | Abnormal | 1,500 | 760 |

The patient counts in the Normal and Abnormal rows are not mutually exclusive. A patient can contribute both Normal and Abnormal images to the same split, but cannot appear in another split.

Therefore, the number of patients in the Normal and Abnormal rows should not be added to calculate the total number of unique patients.

## 7. Final Disease Representation in Abnormal Images

The disease-aware sampling procedure selected 10,000 Abnormal images across all three splits.

The following table reports the observed disease representation in the final Abnormal subset.

| Disease label | Images | Percentage of Abnormal images |
|---|---:|---:|
| Atelectasis | 2,491 | 24.91% |
| Cardiomegaly | 856 | 8.56% |
| Consolidation | 1,264 | 12.64% |
| Edema | 744 | 7.44% |
| Effusion | 2,770 | 27.70% |
| Emphysema | 797 | 7.97% |
| Fibrosis | 590 | 5.90% |
| Hernia | 133 | 1.33% |
| Infiltration | 3,747 | 37.47% |
| Mass | 1,485 | 14.85% |
| Nodule | 1,587 | 15.87% |
| Pleural_Thickening | 993 | 9.93% |
| Pneumonia | 525 | 5.25% |
| Pneumothorax | 1,391 | 13.91% |

### 7.1 Interpreting these results

The final subset contains examples of all 14 disease labels. Their frequencies remain different, reflecting the multi-label structure and the constraints of the sampling procedure.

The most frequent findings in this subset include `Infiltration`, `Effusion`, and `Atelectasis`. Less frequent findings, such as `Hernia`, remain less common, even after disease-aware sampling.

The results should be interpreted as disease-label occurrences, not as mutually exclusive image categories.

For example, an image containing both `Effusion` and `Atelectasis` contributes to both rows. Therefore, the sum of the disease-image counts can exceed the number of Abnormal images.

### 7.2 Comparison with the earlier random sample

The disease-aware subset was compared with the earlier 10,000-image Abnormal sample.

| Disease | Earlier sample | Disease-aware sample |
|---|---:|---:|
| Atelectasis | 22.37% | 24.91% |
| Cardiomegaly | 5.33% | 8.56% |
| Consolidation | 9.03% | 12.64% |
| Edema | 4.47% | 7.44% |
| Effusion | 25.77% | 27.70% |
| Emphysema | 4.82% | 7.97% |
| Fibrosis | 3.26% | 5.90% |
| Hernia | 0.45% | 1.33% |
| Infiltration | 38.49% | 37.47% |
| Mass | 11.20% | 14.85% |
| Nodule | 12.24% | 15.87% |
| Pleural_Thickening | 6.57% | 9.93% |
| Pneumonia | 2.77% | 5.25% |
| Pneumothorax | 10.21% | 13.91% |

The comparison shows that the representation of most disease labels increased in the disease-aware subset. For example, `Hernia` increased from 0.45% to 1.33%, while `Pneumonia` increased from 2.77% to 5.25%.

`Infiltration` decreased slightly from 38.49% to 37.47%. This is consistent with the goal of improving the relative representation of less frequent findings rather than increasing every disease count.

These percentages describe the composition of the selected Abnormal images. They do not, by themselves, demonstrate improved model accuracy or better clinical performance.

## 8. Data Integrity and Leakage Checks

The pipeline performed validation checks after the final selection.

### 8.1 Patient leakage

The patient IDs represented in the training, validation, and test sets were checked for overlap.

**Result: PASSED**

No patient was present in more than one split.

### 8.2 Duplicate image names

The selected image names were checked for duplicates.

**Result: PASSED**

The final subset contains unique image names.

This check does not prove that no visually identical images exist under different filenames.

### 8.3 Exact dataset size and binary counts

The final dataset was checked against the requested image counts:

- Training: 14,000 images.
- Validation: 3,000 images.
- Test: 3,000 images.
- Total: 20,000 images.
- Normal: 10,000 images.
- Abnormal: 10,000 images.

The reported counts match the intended configuration.

### 8.4 Patient assignment

The split assignments were saved separately so that the patient-level separation can be inspected and reused.

The reported final subset contains 10,567 unique patients.

## 9. Exported Files

The generated CSV files are stored in `Data/splits/`.

| File | Description |
|---|---|
| `balanced_20k_metadata.csv` | Metadata for the complete 20,000-image subset |
| `train_metadata.csv` | Training-set metadata |
| `validation_metadata.csv` | Validation-set metadata |
| `test_metadata.csv` | Test-set metadata |
| `patient_split_assignments.csv` | Patient IDs and their assigned splits |

The split-specific metadata files support model training and evaluation without repeating the selection process.

The patient assignment file records the split to which each patient was assigned. It can be used to audit patient-level separation.

The metadata retains the original findings and the derived binary labels, allowing the project to perform binary classification while retaining disease-level information for analysis.

## 10. Limitations and Evaluation Considerations

Several limitations should be considered when using this dataset.

### 10.1 Altered class prevalence

The 50:50 Normal-to-Abnormal ratio was created deliberately. It does not represent the original dataset's binary class prevalence and should not be assumed to reflect prevalence in a real clinical population.

### 10.2 Altered disease representation

Disease-aware sampling changes the composition of the Abnormal subset relative to the original dataset. Model results must therefore be interpreted in the context of this sampling strategy.

### 10.3 Multi-label overlap

Disease counts overlap because individual images can have multiple findings. Disease percentages should not be interpreted as a mutually exclusive distribution.

### 10.4 Patient-level counts differ by class

The same patient can contribute images to both binary classes within one split. The number of patients associated with each class can therefore differ, even when the image counts are equal.

### 10.5 Potential duplicate images

Unique filenames do not guarantee unique image content. A separate image-level duplicate check would be needed to investigate visually identical images under different filenames.

### 10.6 Source-label limitations

The source findings were automatically extracted from radiology reports and may contain label noise. The binary labels inherit limitations from the original annotations.

### 10.7 Demographic and acquisition differences

Patient demographics and acquisition characteristics, including AP versus PA views, may be unevenly represented. These factors should be examined during exploratory data analysis.

### 10.8 Test-set interpretation

The test set must remain separate from model fitting and model-selection decisions. Repeatedly adjusting the sampling strategy based on test performance would compromise its role as an evaluation set.

Because the dataset is a sampled subset of NIH ChestX-ray14, test results alone do not establish generalization to other hospitals, populations, or clinical settings.

## 11. Summary

The final dataset was constructed to support binary Normal-versus-Abnormal chest X-ray classification using a reproducible, patient-separated sampling pipeline.

The main decisions were:

- A 20,000-image subset was selected from the original dataset.
- Patients were assigned to splits before image selection.
- The intended image proportions were 70% training, 15% validation, and 15% test.
- Every split contains equal numbers of Normal and Abnormal images.
- Disease-aware sampling was applied to the Abnormal image pool using `BALANCE_ALPHA = 0.75`.
- The sampling algorithm prioritized disease labels below their desired representation targets.
- Original multi-label annotations were preserved, and each selected image contributed to the counts of all its associated findings.
- Patient-level leakage and duplicate image names were checked.
- The final metadata and patient assignments were exported as CSV files.

This approach balances the requirements of binary classification, disease-label representation, and patient-level evaluation integrity. It provides a documented starting point for model development while retaining the limitations of the original dataset and the effects of the sampling strategy.
