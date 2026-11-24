# Phase 1 — Team Task Distribution

## General Instructions for Everyone

1. Read the doctor's project pipeline first, then review the repository README before starting work.
2. Keep all Phase 1 work in the `data-pipeline` branch. Do not merge it into another branch yet; we will merge everything at the end of the project.
3. Everyone must understand the full Phase 1 workflow, not only their assigned tasks, because any team member may be asked about any part of the project.
4. Coordinate changes and keep your code, notebooks, decisions, and results documented and reproducible.

---

# 1. Mostafa — Embedding Analysis

## 1.1 PCA + UMAP / t-SNE

The **pretrained image embeddings have already been generated** and are available here:

[Pretrained Image Embeddings](https://drive.google.com/drive/folders/1ZZmQl_Bizu0KJR9TmoYZDjth4Xrnf16K?usp=sharing)

Use these existing embeddings directly; there is no need to generate the embeddings again.

### Requirements

- Apply **PCA** to the existing embeddings.
- Apply **UMAP or t-SNE** for 2D visualization.
- Visualize the resulting embeddings and distinguish between **Normal** and **Abnormal** classes.
- Document the selected dimensionality-reduction methods and parameters.

## 1.2 Clustering / Overlap Interpretation

Analyze the PCA and UMAP/t-SNE visualizations.

### Requirements

- Determine whether Normal and Abnormal images form separate clusters or show significant overlap.
- Identify possible clustering patterns related to metadata or acquisition characteristics.
- Investigate potential **shortcuts or confounders** visible in the embedding space.
- Explain how the observed clustering or overlap may affect model training and evaluation.

## 1.3 Expected Deliverables

- PCA visualization.
- UMAP or t-SNE visualization.
- Analysis of clustering and class overlap.
- A short report describing the findings and their implications.

---

# 2. Ahmed — Dataset Visualization, Duplicate Detection and Final Report

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

## 2.2 Exact & Near Duplicate Detection

### 1. Exact Duplicates

Detect images that are identical using a reliable image hashing method.

### 2. Near-Duplicates

Detect images that are visually very similar using an appropriate image-similarity method.

### 3. Data Leakage Analysis

Check whether duplicates occur:

- Within the same split.
- Across Train/Validation/Test.
- Between images belonging to the same patient.

### 4. Expected Results

Report:

- Number of exact duplicate groups.
- Number of near-duplicate pairs/groups.
- Patient IDs associated with duplicates.
- Dataset splits containing duplicates.
- Examples of detected duplicates.
- The method and threshold used for near-duplicate detection.

Conclude with the appropriate **cleaning or splitting decision** to prevent data leakage.

---

## 2.3 Final IEEE-Format Report

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
