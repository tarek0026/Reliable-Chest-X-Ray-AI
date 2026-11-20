# Phase 1 — Team Task Distribution

## General Instructions for Everyone

1. Read the doctor's project pipeline first, then review the repository README before starting work.
2. Keep all Phase 1 work in the `data-pipeline` branch. Do not merge it into another branch yet; we will merge everything at the end of the project.
3. Everyone must understand the full Phase 1 workflow, not only their assigned tasks, because any team member may be asked about any part of the project.
4. Coordinate changes and keep your code, notebooks, decisions, and results documented and reproducible.

---

# 1. Mostafa — Cleaning and Preprocessing

## 1.1 Cleaning and Preprocessing

**Important:** Start the preprocessing stage after Ahmed has organized the images into `Data/C/train`, `Data/C/validation`, and `Data/C/test`. Coordinate with Ahmed to ensure that the preprocessing pipeline uses the correct split for each image.

### 1. Image Resizing

Resize all images to a fixed input resolution suitable for the selected models. Document the chosen resolution and resizing method.

### 2. Intensity Normalization

Define and document a consistent procedure for normalizing image pixel intensities.

### 3. Data Augmentation

Apply augmentation to the **training set only**.

Use clinically plausible transformations and explain why each transformation is appropriate for chest X-ray images.

Do not apply augmentation to the validation or test sets.

### 4. Prevent Data Leakage

Fit any preprocessing step that learns parameters from the data using the **training set only**.

Apply the learned parameters to the validation and test sets without fitting them again.

### 5. Reproducibility

Save the preprocessing configuration, selected input resolution, normalization method, augmentation settings, and fixed random seed so the pipeline can be reproduced.

## 1.2 Expected Deliverables

- Downloaded dataset stored in the agreed location.
- Dataset source and access conditions documented.
- A reproducible preprocessing pipeline.
- Documented resizing, normalization, and augmentation decisions.
- Saved preprocessing configuration and random seed.
- A brief report describing the preprocessing steps and confirming that augmentation is applied only to the training set.

**Dependency:** Ahmed is responsible for organizing and verifying the dataset splits first. Mostafa should coordinate with him and begin preprocessing once the folder structure and split assignments are ready.

---

# 2. Ahmed — Dataset Visualization and Final Report

## 2.1 Visualize the Dataset

### 1. Random Sample Grids

Create random sample grids of **Normal** and **Abnormal** chest X-ray images to visually inspect differences between the two classes.

### 2. Metadata Subgroups

Create additional sample grids for important metadata subgroups, such as:

- Age groups
- Sex
- Image view position (AP/PA)

where the required metadata is available.

### 3. Reproducibility

Use a fixed random seed so the selected samples can be reproduced.

---

## 2.2 Final IEEE-Format Report

**Important:** This report is prepared **after the entire project is completed**, not during the initial Phase 1 implementation.

Ahmed will be responsible for assembling and producing the final:

`report.pdf`

The report must be **4–6 pages excluding appendices** and follow the required IEEE format.

### Required Report Sections

The report must include:

1. Problem Definition
2. Dataset and Label Definition
3. Sample-Size Justification
4. Data-Quality Audit
5. Required EDA
6. Cleaning Decisions
7. Leakage Analysis
8. Patient-Wise Split
9. Baseline and Learning Curve
10. Selected Evaluation Metrics
11. Phase Two Plan

### Report Collaboration

Ahmed is responsible for the final writing, organization, formatting, and submission of the report.

However, each team member must provide Ahmed with the results from their assigned work.

---

# 3. Bola — Exploratory Data Analysis (EDA) and Image Quality Assessment

## 3.1 Analyze Image Characteristics and Quality

Calculate and summarize the following characteristics across the dataset:

### 1. Image Resolution

- Width
- Height
- Variation in image dimensions

### 2. Aspect Ratio

Analyze the relationship between image width and height.

### 3. Intensity

Analyze the distribution of pixel intensity values.

### 4. Brightness

Analyze the overall brightness of each image.

### 5. Contrast

Analyze the variation in pixel intensities within each image.

### 6. Blur

Use a suitable quantitative measure of image sharpness.

Present the results using appropriate tables, plots, and summary statistics.

---

## 3.2 Compare Classes and Metadata Subgroups

### 1. Class Comparison

Compare image-quality measurements between:

- **Normal**
- **Abnormal**

### 2. Metadata Comparison

Compare the same measurements across available metadata subgroups, including:

- Age
- Sex
- AP/PA view position

### 3. Investigate Potential Differences

Investigate whether differences in image quality or acquisition characteristics are associated with class labels or specific subgroups.

### 4. Identify Potential Shortcuts or Confounders

Identify potential **shortcuts or confounders** that a model might exploit instead of learning meaningful medical features.

---

## 3.3 Evidence-Based Findings

Conclude with **at least three evidence-based findings** supported by the actual analysis results.

For each finding, include:

- **Observation:** What the tables, plots, or statistics show.
- **Evidence:** The relevant measurements, counts, comparisons, or figures.
- **Implication:** How the finding could affect model training, evaluation, or reliability.
- **Recommendation:** Any preprocessing, splitting, or evaluation decision justified by the evidence.

**Important:** Do not assume that differences exist before analyzing the data.

If a comparison reveals no meaningful difference, report that result as well.

---

## 3.4 Expected Deliverables

- Tables and plots summarizing image dimensions and quality measures.
- Comparisons between classes and metadata subgroups.
- A short report containing at least three evidence-based findings and their implications.
