# WinoQuoi? Replicating the WinoWhat Framework for Commonsense Reasoning Evaluation in French

This repository contains the data, code, and experimental results for my Master's thesis at the University of Antwerp (Academic Year 2025-2026), which replicates and extends the WinoWhat framework (Gevers et al., 2025) for evaluating commonsense reasoning under paraphrasing in French.

The project investigates whether language models rely on true commonsense reasoning or surface linguistic cues by comparing model performance on original Winograd-style sentences and meaning-preserving paraphrases in French.

---

## Project Overview

### Task

Winograd-style commonsense reasoning (fill-in-the-blank).

**Example:**
```
La coupe n'entre pas dans la valise marron, car _ est trop grande.
```

**Options:**
1. la valise
2. la coupe

The model must select the correct antecedent based on commonsense reasoning.

### Method

1. Insert each candidate option into the sentence
2. Compute the log-likelihood of each completed sentence using the appropriate scoring method for each model type
3. Select the option with the higher score
4. Compute model accuracy over the full dataset

Models are evaluated on both **original sentences** and **paraphrased sentences**.

---

## Dataset

The dataset combines two sources for a total of **200 instances**.

| Source | Instances | Description |
|---|---|---|
| Amsili and Seminck (2017) | 100 | French Winograd schemas, parsed from original XML |
| Gevers et al. (2025), translated | 100 | WinoWhat validation subset, translated into French via DeepL |

Each instance contains:
```
id
sentence
paraphrased_sentence
option1
option2
answer
```

Paraphrases were generated using a large language model and manually verified to preserve meaning, grammaticality, and ambiguity. Each paraphrase places the blank token at the end of the sentence, following the WinoWhat design. Option ordering was randomized with a fixed seed to avoid positional bias.

---

## Models and Evaluation Protocol

| Model | Type | Scoring Method |
|---|---|---|
| Mistral-7B-v0.1 (Jiang et al., 2023) | Causal LM | Partial evaluation (log-likelihood on option tokens) |
| CamemBERT-base (Martin et al., 2020) | French Masked LM | Pseudo-log-likelihood (PLL) |
| XLM-RoBERTa-base (Conneau et al., 2020) | Multilingual Masked LM | Pseudo-log-likelihood (PLL) |

**Causal LM scoring (Mistral-7B):** Follows the partial evaluation implementation of Gevers et al. (2025). The summed log-likelihood of the option tokens is computed given the sentence prefix, using `add_special_tokens=False` and `torch.gather` on the shifted logits.

**Masked LM scoring (CamemBERT and XLM-RoBERTa):** Each completed sentence is scored by masking one token at a time and summing log P(token | rest of sentence) across all non-special tokens.

**Statistical reporting:** Confidence intervals use the Wilson score interval. Significance is assessed with McNemar's exact test. Effect size is reported as Cohen's g.

---

## Results

| Model | Original | 95% CI | Paraphrased | 95% CI | Drop |
|---|---|---|---|---|---|
| Mistral-7B | 0.480 | [0.412, 0.549] | 0.510 | [0.441, 0.578] | -0.030 |
| CamemBERT-base | 0.465 | [0.397, 0.534] | 0.470 | [0.402, 0.539] | -0.005 |
| XLM-RoBERTa-base | 0.550 | [0.481, 0.617] | 0.535 | [0.466, 0.603] | +0.015 |

Random baseline: 0.500 | Always-option1 baseline: 0.460

**McNemar's exact test: original vs. paraphrased**

| Model | p-value | Significant |
|---|---|---|
| Mistral-7B | 0.3075 | No |
| CamemBERT-base | 1.0000 | No |
| XLM-RoBERTa-base | 0.7948 | No |

**McNemar's exact test: between-model comparisons**

| Comparison | Condition | p-value | Significant |
|---|---|---|---|
| Mistral vs CamemBERT | Original | 0.8376 | No |
| Mistral vs XLM-RoBERTa | Original | 0.1843 | No |
| CamemBERT vs XLM-RoBERTa | Original | 0.0966 | No |
| Mistral vs CamemBERT | Paraphrased | 0.4397 | No |
| Mistral vs XLM-RoBERTa | Paraphrased | 0.6646 | No |
| CamemBERT vs XLM-RoBERTa | Paraphrased | 0.1875 | No |

All models perform near chance level. No significant paraphrasing drop is observed for any model, and no significant difference between models is found. XLM-RoBERTa is the only model to perform above chance on original sentences (0.55), but the drop after paraphrasing is not significant. An English sanity check confirmed that the Mistral scorer produces above-chance results on English (0.60 on 50 instances), verifying the pipeline is functioning correctly.

---

## Repository Structure
```
.
├── data/
│   ├── French_Wino_Schemas.xml          # Original XML dataset (Amsili and Seminck, 2017)
│   ├── french_wino_unrandomized.csv     # Structured dataset extracted from XML
│   ├── french_wino_randomized.csv       # Dataset with randomized option ordering
│   └── french_wino_with_paraphrases.csv # Final dataset used in experiments
│
├── notebooks/
│   ├── french_wino_data.ipynb                          # Data processing pipeline
│   └── french_winowhat_replication_experiment.ipynb    # Main experiment notebook
│
├── results/
│   ├── results_original_mistral.csv
│   ├── results_paraphrased_mistral.csv
│   ├── wrong_original_mistral.csv
│   ├── wrong_paraphrased_mistral.csv
│   ├── results_original_camembert.csv
│   ├── results_paraphrased_camembert.csv
│   ├── wrong_original_camembert.csv
│   ├── wrong_paraphrased_camembert.csv
│   ├── results_original_xlmr.csv
│   ├── results_paraphrased_xlmr.csv
│   ├── wrong_original_xlmr.csv
│   ├── wrong_paraphrased_xlmr.csv
│   └── summary_results.csv
│
└── README.md
```

---

## Reproducing the Experiments

### Requirements
```bash
pip install transformers==4.44.0 tokenizers==0.19.1 accelerate sentencepiece pandas tqdm scikit-learn statsmodels
```

### Steps

1. Run `french_wino_data.ipynb` to parse the XML dataset, apply option randomization, and construct the final dataset with paraphrases.
2. Run `french_winowhat_replication_experiment.ipynb` to evaluate all three models and generate result files and figures. The notebook is designed to run on Google Colab with an A100 GPU.

Note: `transformers==4.44.0` and `tokenizers==0.19.1` are required to avoid a tokenizer compatibility bug with CamemBERT's SentencePiece vocabulary loading in newer library versions.

---

## References

Amsili, P., and Seminck, O. (2017). A Google-proof collection of French Winograd schemas. *Proceedings of CORBON 2017*, 24-29.

Conneau, A., et al. (2020). Unsupervised cross-lingual representation learning at scale. *Proceedings of ACL 2020*, 8440-8451.

Gevers, I., De Marez, V., De Bruyne, L., and Daelemans, W. (2025). WinoWhat: A parallel corpus of paraphrased WinoGrande sentences with common sense categorization. *Proceedings of CoNLL 2025*, 68-80.

Jiang, A. Q., et al. (2023). Mistral 7B. *arXiv preprint arXiv:2310.06825*.

Martin, L., et al. (2020). CamemBERT: A tasty French language model. *Proceedings of ACL 2020*, 7203-7219.

Sakaguchi, K., Le Bras, R., Bhagavatula, C., and Choi, Y. (2021). WinoGrande: An adversarial Winograd schema challenge at scale. *Communications of the ACM*, 64(9), 99-106.

Trinh, T. H., and Le, Q. V. (2018). A simple method for commonsense reasoning. *arXiv preprint arXiv:1806.02847*.

---

## Author

**Oum Loukili**
Master of Digital Text Analysis
University of Antwerp
Academic Year 2025-2026

Supervisor: Jens Lemmens
Promotor: Ine Gevers

---

*This repository is intended for academic and research purposes.*
