# Phase 1 --- Team Task Distribution                                    
                                                                        
                                                                        
## General Instructions for Everyone                                    
                                                                        
                                                                        
1.  Read the doctor's project pipeline first, then review the repository
    README before starting work.                                        
2.  Keep all Phase 1 work in the `data-pipeline` branch. Do not merge it
    into another branch yet; we will merge everything at the end of the 
    project.                                                            
3.  Everyone must understand the full Phase 1 workflow, not only their  
    assigned tasks, because any team member may be asked about any part 
    of the project.                                                     
4.  Coordinate changes and keep your code, notebooks, decisions, and    
    results documented and reproducible.                                



# Mostafa — Dataset Acquisition, Cleaning, and Preprocessing

## 1. Dataset Acquisition;

1. Download the Chest X-ray dataset from the official source: [https://nihcc.app.box.com/v/ChestXray-NIHCC](https://nihcc.app.box.com/v/ChestXray-NIHCC)
2. Store the downloaded dataset in the agreed Google Drive location and notify Ahmed when it is ready so he can organize the images according to the CSV splits.

## 2. Cleaning and Preprocessing

**Important:** Start the preprocessing stage after Ahmed has organized the images into `Data/C/train`, `Data/C/validation`, and `Data/C/test`. Coordinate with Ahmed to ensure that the preprocessing pipeline uses the correct split for each image.

1. **Image resizing:** Resize all images to a fixed input resolution suitable for the selected models. Document the chosen resolution and resizing method.
2. **Intensity normalization:** Define and document a consistent procedure for normalizing image pixel intensities.
3. **Data augmentation:** Apply augmentation to the **training set only**. Use clinically plausible transformations and explain why each transformation is appropriate for chest X-ray images. Do not apply augmentation to the validation or test sets.
4. **Prevent data leakage:** Fit any preprocessing step that learns parameters from the data using the training set only. Apply the learned parameters to the validation and test sets without fitting them again.
5. **Reproducibility:** Save the preprocessing configuration, selected input resolution, normalization method, augmentation settings, and fixed random seed so the pipeline can be reproduced.

## 3. Expected Deliverables

- Downloaded dataset stored in the agreed location.
- Dataset source and access conditions documented.
- A reproducible preprocessing pipeline.
- Documented resizing, normalization, and augmentation decisions.
- Saved preprocessing configuration and random seed.
- A brief report describing the preprocessing steps and confirming that augmentation is applied only to the training set.

**Dependency:** Ahmed is responsible for organizing and verifying the dataset splits first. Mostafa should coordinate with him and begin preprocessing once the folder structure and split assignments are ready.








# Ahmed — Dataset Organization and Quality Checks

## 1. Organize the Dataset by Split

1. Work on the `feature/data-pipeline` branch and navigate to `Data/`.
2. Locate `balanced_20K.csv`. Once Mostafa uploads the dataset to Google Drive, locate the actual X-ray image files in the downloaded dataset directories.
3. Create a folder named `C` inside `Data/`, with three subfolders: `train`, `validation`, and `test`.
4. Read `balanced_20K.csv` and use its split column to determine where each image belongs.
5. **Copy** each image into the appropriate subfolder, preserving its original filename. Do not move, rename, or delete the original images.

The expected folder structure is:

```text
Data/
├── balanced_20K.csv
└── C/
    ├── train/
    ├── validation/
    └── test/
```

## 2. Verify the Dataset

- Count the images in each split and report the number of images in `train`, `validation`, and `test`.
- Confirm that the total number of records in the CSV is 20,000 and compare the expected split counts with the actual number of copied images.
- Identify and report any images listed in the CSV that cannot be found in the source directories.
- Check for duplicate image files, including exact duplicates and visually similar (near-duplicate) images.
- Check for repeated examinations from the same patient, where patient identifiers or metadata are available.

## 3. Handle Unreadable Images

- Identify image files that cannot be opened or read correctly.
- Exclude unreadable images from the usable dataset and document every exclusion in a cleaning log, including the filename and reason for exclusion.
- Keep the original files unchanged and report the final counts after exclusions.

**Expected outcome:** A correctly organized dataset with verified split counts, a report of missing and duplicate images, checks for repeated patient examinations, and a cleaning log documenting all exclusions.






# Bola — Exploratory Data Analysis (EDA) and Image Quality Assessment

## 1. Visualize the Dataset

1. Create random sample grids of **Normal** and **Abnormal** chest X-ray images to visually inspect differences between the two classes.
2. Create additional sample grids for important metadata subgroups, such as age groups, sex, and image view position (AP/PA), where the required metadata is available.
3. Use a fixed random seed so the selected samples can be reproduced.

## 2. Analyze Image Characteristics and Quality

Calculate and summarize the following characteristics across the dataset:

- **Image resolution:** Width, height, and variation in image dimensions.
- **Aspect ratio:** The relationship between image width and height.
- **Intensity:** The distribution of pixel intensity values.
- **Brightness:** The overall brightness of each image.
- **Contrast:** The variation in pixel intensities within each image.
- **Blur:** A suitable quantitative measure of image sharpness.

Present the results using appropriate tables, plots, and summary statistics.

## 3. Compare Classes and Metadata Subgroups

1. Compare image-quality measurements between the **Normal** and **Abnormal** classes.
2. Compare the same measurements across available metadata subgroups, including age, sex, and AP/PA view position.
3. Investigate whether differences in image quality or acquisition characteristics are associated with class labels or specific subgroups.
4. Identify potential **shortcuts or confounders** that a model might exploit instead of learning meaningful medical features.

## 4. Evidence-Based Findings

Conclude with **at least three evidence-based findings** supported by your actual analysis results.

For each finding, include:
- **Observation:** What the tables, plots, or statistics show.
- **Evidence:** The relevant measurements, counts, comparisons, or figures.
- **Implication:** How the finding could affect model training, evaluation, or reliability.
- **Recommendation:** Any preprocessing, splitting, or evaluation decision justified by the evidence.

Do not assume that differences exist before analyzing the data. If a comparison reveals no meaningful difference, report that result as well.

## Expected Deliverables

- Random image sample grids for both classes and relevant metadata subgroups.
- Tables and plots summarizing image dimensions and quality measures.
- Comparisons between classes and metadata subgroups.
- A short report containing at least three evidence-based findings and their implications for the project.