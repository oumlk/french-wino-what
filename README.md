
# French WinoWhat

This repository contains the **data, code, and experimental results** for my thesis project that **replicates and extends the WinoWhat framework** for evaluating commonsense reasoning under paraphrasing in **French**.

The project investigates whether language models rely on **true commonsense reasoning** or **surface linguistic cues** by comparing performance on **original Winograd sentences** and **meaning-preserving paraphrases**.

The evaluation follows the methodology introduced in:

**Gevers et al. (2023). _WinoWhat: A Dataset for Evaluating Commonsense Reasoning under Paraphrasing._**


---

# Project Overview

## Task

Winograd-style commonsense reasoning (fill-in-the-blank).

Example:

```text
La coupe n'entre pas dans la valise marron, car _ est trop grande.
```

Options:

```
1. la valise
2. la coupe
```

The model must select the correct antecedent.

## Method

1. Insert each candidate option into the sentence  
2. Compute the **log-likelihood** of each completed sentence  
3. Select the option with the higher probability  
4. Compute model accuracy  

Models are evaluated on:

- **Original sentences**
- **Paraphrased sentences**

# Dataset

The dataset combines two sources.

## French Winograd Schemas

Amsili & Seminck (2017)  
*A Google-proof collection of French Winograd schemas.*

## WinoWhat Validation Subset

Gevers et al. (2023)  
Translated into French using the **DeepL API**.

## Dataset Size

**201 total instances**

- 100 French Winograd schemas  
- 101 translated WinoWhat instances  

Each instance contains:

```
id
sentence
paraphrased_sentence
option1
option2
answer
```

Paraphrases were generated using a **large language model** and manually verified to preserve meaning and ambiguity.

# Repository Files

## Dataset

### French_Wino_Schemas.xml

Original XML dataset containing French Winograd schemas.

Source: Amsili & Seminck (2017)

---

### french_wino_unrandomized.csv

Structured dataset extracted from the XML file.

Contains:

- sentence
- candidate options
- correct answer

---

### french_wino_randomized.csv

Same dataset with randomized option ordering to avoid positional bias.

---

### french_wino_with_paraphrases.csv

Final dataset used for experiments.

Contains:

- original sentence
- paraphrased sentence
- two candidate options
- gold answer label

---

# Data Processing Notebook

### french_wino_data.ipynb

Notebook used to:

- parse the XML dataset
- convert the dataset to CSV format
- generate paraphrases
- construct the final dataset used in experiments

---

# Experiment Notebook

### french_winowhat_replication_experiment.ipynb

Main experimental pipeline.

The notebook:

- loads the dataset
- evaluates models
- computes log-likelihood scores
- calculates accuracy
- performs statistical testing (McNemar test)

Models evaluated:

- **Mistral-7B**
- **CamemBERT**

---

# Model Predictions

### results_original_mistral.csv

Model predictions for **original sentences** using Mistral-7B.

### results_paraphrased_mistral.csv

Model predictions for **paraphrased sentences** using Mistral-7B.

### results_original_camembert.csv

Predictions for **original sentences** using CamemBERT.

### results_paraphrased_camembert.csv

Predictions for **paraphrased sentences** using CamemBERT.

---

# Error Analysis

These files contain incorrectly predicted examples.

### wrong_original_mistral.csv

Incorrect predictions by Mistral on original sentences.

### wrong_paraphrased_mistral.csv

Incorrect predictions by Mistral on paraphrased sentences.

### wrong_original_camembert.csv

Incorrect predictions by CamemBERT on original sentences.

### wrong_paraphrased_camembert.csv

Incorrect predictions by CamemBERT on paraphrased sentences.

These files are used for **qualitative error analysis**.

---

# Results Summary

### summary_results.csv

Summary of experimental results including:

- model name
- evaluation method
- accuracy on original sentences
- accuracy on paraphrased sentences
- performance drop
- confidence intervals

---


# Experimental Results

| Model | Original | Paraphrased | Drop |
|------|------|------|------|
| Mistral-7B | 0.739 | 0.633 | −0.106 |
| CamemBERT | 0.487 | 0.482 | −0.005 |

McNemar test:

| Model | p-value |
|------|------|
| Mistral-7B | 0.0128 |
| CamemBERT | 1.0 |

---

# Dependencies

Main libraries:

```
transformers
torch
pandas
numpy
tqdm
scikit-learn
statsmodels
```

Install with:

```bash
pip install transformers torch pandas numpy tqdm scikit-learn statsmodels
```

---

# References

Amsili, P., & Seminck, O. (2017)  
*A Google-proof collection of French Winograd schemas.*

Gevers, I. et al. (2023)  
*WinoWhat: A Dataset for Evaluating Commonsense Reasoning under Paraphrasing.*

## License

This repository is intended for academic and research purposes.
