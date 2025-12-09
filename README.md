# CPBS 7602 Assignment 02 — Linear & Logistic Modeling with GTEx Expression Data

## Course: Introduction to Big Data in the Biomedical Sciences (CPBS 7602)  
**Instructors:** Milton Pividori  
**Institution:** University of Colorado Anschutz Medical Campus  
**Student:** Saloni Dixon  

---

## Overview

This assignment extends the GTEx-based analysis from Assignment 01 by applying **multiple linear regression** and **logistic regression** to gene expression data. The goals are to:

1. Determine which genes differ most strongly between **Brain** and **Blood**, adjusting for age and sex.
2. Evaluate whether **blood gene expression can predict sex** using logistic regression.

All modeling is done **gene-by-gene**, using the **top 5,000 most variable genes** identified from GTEx v8 TPM expression data.

This assignment builds on preprocessing steps, standardization, and metadata integration introduced in previous lectures and in Assignment 01.

---

## Dataset

**GTEx v8 Bulk Tissue Expression (TPM Normalized)**  
Downloaded from: https://gtexportal.org/home/datasets

Files used:
- `GTEx_Analysis_2017-06-05_v8_RNASeQCv1.1.9_gene_tpm.gct.gz`
- `GTEx_Analysis_v8_Annotations_SampleAttributesDS.txt`
- `GTEx_Analysis_v8_Annotations_SubjectPhenotypesDS.txt`

### Metadata fields used
- **Tissue (SMTS)** — filtered to Brain and Blood  
- **SUBJID** — used to merge sample and subject-level metadata  
- **AGE** — converted to numeric midpoints  
- **SEX** — recoded into binary (1 = male, 0 = female)

---

## Methods Summary

### 1. Data Import & Preprocessing
- Loaded GTEx TPM expression data (GCT format; skipped first two metadata rows).  
- Filtered to the **top 5,000 most variable genes** across all samples.  
- Transposed matrix so rows = samples, columns = genes.  
- **Standardized** expression using `StandardScaler`.  
- Loaded sample metadata and subject phenotype files.  
- Merged sample-level and subject-level metadata using `SUBJID`.  
- Created unified metadata frame with:
  - Tissue (Brain vs Blood)
  - Age (numeric midpoint)
  - Sex (binary)

---

## 2. Multiple Linear Regression  
_Assessing expression differences between Brain and Blood_

For each of 5,000 genes, I fit the model:

$$
\text{Expression}_{g,i} = \beta_0 + \beta_1 \cdot \text{Tissue}_i + \beta_2 \cdot \text{Age}_i + \beta_3 \cdot \text{Sex}_i + \epsilon_i
$$

Where:
- **Tissue** = 1 for Brain, 0 for Blood  
- **Age** = numeric age  
- **Sex** = 1 for male, 0 for female  
- **Expression** = standardized gene expression (mean 0, SD 1)

### Interpretation
- **β₁** reflects the standardized difference in expression between Brain and Blood **after adjusting for age and sex**.  
- Positive β₁ → gene expression is higher in Brain.  
- Negative β₁ → gene expression is higher in Blood.  

### Reported:
- **Top 5 most significant genes** by p-value  
- **Top 5 genes with largest effect size** |β₁|  

Results show extremely strong Brain–Blood differences, consistent with known tissue-specific biology.

---

## 3. Logistic Regression  
_Predicting sex from Blood gene expression_

For each gene, I fit the logistic regression model:

$$
\log \left( \frac{P(\text{Male})}{P(\text{Female})} \right)
= \beta_0 + \beta_1 \cdot \text{Expression}_{g}
$$

Where:
- **Expression** = standardized blood expression  
- **Sex** = 1 male, 0 female  

### Interpretation
- **β₁** represents the log-odds change in probability of being male per 1 SD increase in expression.  
- **exp(β₁)** = odds ratio  
- Positive β₁ → gene is **male-biased**  
- Negative β₁ → gene is **female-biased**

### Reported:
- **Top 5 most significant genes**  
- **Top 5 genes with largest |β₁|**  

Many models showed extremely large β values or convergence warnings, indicating **near-perfect sex separation**, which is expected for Y-linked and sex-biased genes in GTEx blood samples.

---

## 4. Results Summary

### Linear Regression (Brain vs Blood)
- Dozens of genes showed highly significant tissue effects (very small p-values).  
- Tissue-related β₁ effects were large and biologically interpretable:
  - Brain-enriched genes: neuronal, synaptic, developmental.
  - Blood-enriched genes: immune/hematopoiesis pathways.
- Tissue identity remained the **dominant driver of expression**, far stronger than age or sex.

### Logistic Regression (Predicting Sex)
- Several genes nearly perfectly distinguish males from females.  
- Large β₁ values and separation reflect:
  - Y-linked genes (male-only)
  - X-linked dosage effects
  - Hormone-regulated transcriptional programs
  
Overall, logistic regression confirms that **sex is highly predictable from blood expression**.

---

## 5. Discussion

This analysis demonstrates that regression-based modeling effectively identifies biological signals in high-dimensional gene expression data.

- **Brain vs Blood differences** are extremely strong and consistent with known tissue-specific transcriptional programs.
- **Sex differences in Blood** are so pronounced that many genes exhibit quasi-complete separation in logistic regression.
- Including **age and sex as covariates** in the linear model ensured confounding was controlled.
- Standardization allowed direct comparison of β values across genes.

These findings match established GTEx analyses and reinforce the power of linear and logistic models in transcriptomics.

---

## Reproducibility

### Random Seed
All operations involving randomness use:
`numpy.random.seed(42)`

### Environment Setup 
You can recreate the environment using `conda`:

`conda create -n cpbs7602 python=3.10`
`conda activate cpbs7602`
`pip install -r requirements.txt`

Alternatively, install the key dependencies manually:
`pip install pandas numpy matplotlib scikit-learn statsmodels seaborn jupyter`

### How to Run
From inside the cloned repo or folder:
'jupyter notebook assignment_02.ipynb'
Rull all cells sequentially for full analysis.
