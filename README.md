# Predicting Antibody Aggregation from Sequence using Protein Language Models

Small experiment exploring whether **protein language model embeddings capture developability signal related to antibody aggregation**.

The goal was to test whether **antibody sequence alone contains information correlated with SEC monomer percentage**, a common developability metric used to assess aggregation propensity.

---

# Overview

Antibody aggregation is influenced by multiple factors:

- sequence composition
- framework regions
- antibody isotype
- expression system
- purification process
- formulation conditions

Because of this complexity, predicting aggregation directly from sequence is challenging.

This project investigates whether **embeddings from a protein language model (ESM2)** contain signal that correlates with **SEC monomer percentage**.

---

# Dataset

Dataset used:

**GDPa1 antibody developability dataset**

Approximately **242 antibodies** with experimental developability measurements including:

- SEC % Monomer (aggregation)
- Hydrophobic interaction chromatography (HIC)
- Thermostability (Tm1, Tm2)
- AC-SINS self-association
- Antibody subtype annotations

The dataset provides:

- heavy chain sequences
- light chain sequences
- antibody subtype metadata
- cluster-based cross-validation folds

These folds prevent **sequence similarity leakage**, which is critical when evaluating protein ML models.

---

# Method

## Feature extraction

Sequence embeddings were generated using a **protein language model**:

Model:

ESM2-t6-8M

This is a transformer-based protein language model trained on large protein sequence datasets.

Embeddings were extracted by:

1. Tokenizing each protein sequence
2. Passing the sequence through the transformer
3. Mean-pooling residue embeddings

For antibodies, embeddings were created by concatenating:

heavy chain embedding + light chain embedding

---

## Model

A simple linear model was used:

Ridge regression

Linear models are often strong baselines for small biological datasets.

---

## Evaluation

Evaluation used **cluster-based cross-validation** provided in the dataset:

hierarchical_cluster_IgG_isotype_stratified_fold

This ensures that **similar antibodies are not present in both training and test sets**, which prevents overly optimistic performance estimates.

Metrics used:

- Spearman correlation
- RMSE

Spearman correlation measures the model's ability to **rank antibodies by aggregation propensity**.

---

# Experiments

Three feature configurations were evaluated.

---

## 1. Variable regions only

Embeddings created from:

VH + VL sequences

Performance:

Spearman ≈ 0.13

This indicates that **variable regions alone contain limited signal for aggregation prediction**.

---

## 2. Full antibody chains

Embeddings created from:

HC + LC sequences

Performance:

Spearman ≈ 0.39

Including framework regions significantly improves predictive performance.

This suggests that **aggregation-related sequence features extend beyond CDR regions**.

---

## 3. Full chains + antibody subtype features

Additional categorical features were added:

- hc_subtype
- lc_subtype

These were encoded using one-hot encoding and concatenated with sequence embeddings.

Final feature vector:

ESM embeddings (640) + subtype features (5)

Performance:

Cluster CV Spearman ≈ 0.417  
RMSE ≈ 2.74

---

# Key Observations

### Developability signal exists in sequence embeddings

Even without fine-tuning the protein language model, embeddings contain measurable signal related to antibody aggregation.

---

### Framework regions contribute to aggregation behavior

Prediction performance improved substantially when using **full heavy and light chain sequences** instead of only variable regions.

Aggregation-related properties likely depend on **global structural context**, not just CDRs.

---

### Antibody subtype contributes additional signal

Including antibody subtype information further improved prediction performance.

Isotype context likely affects:

- structural stability
- aggregation behavior

---

### Simple models perform well on small biological datasets

A linear model performed better than neural networks in this setting.

This is common when:

N (samples) ≪ feature dimension

---

# Repository Structure

```
antibody-sec-prediction/

README.md

notebooks/
    sec_prediction_experiment.ipynb

data/
    GDPa1_dataset.csv

embeddings/
    cached_ESM_embeddings.npy

requirements.txt
```

---

# Requirements

Example environment:

```
python >= 3.10
torch
transformers
scikit-learn
pandas
numpy
scipy
```

---

# Running the experiment

1. Clone the repository

```
git clone https://github.com/YOUR_USERNAME/antibody-sec-prediction
```

2. Install dependencies

```
pip install -r requirements.txt
```

3. Run the notebook

```
notebooks/sec_prediction_experiment.ipynb
```

The notebook will:

- load the dataset
- compute ESM embeddings
- train regression models
- evaluate cluster-based cross validation

Embeddings are cached to avoid recomputation.

---

# Future Work

Potential extensions include:

- fine-tuning ESM embeddings for developability prediction
- multi-task learning across developability assays
- predicting multiple developability metrics simultaneously
- incorporating structural features
- testing larger protein language models

---

# Takeaway

This experiment shows that **protein language model embeddings capture meaningful sequence-level signal related to antibody aggregation**, even when used with simple downstream models.

---

# Citation

If using the dataset or model in research, please cite:

ESM Protein Language Model  
Rives et al.

GDPa1 antibody developability dataset  
Jain et al.
