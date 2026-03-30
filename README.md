# French WinoWhat: Replicating Commonsense Reasoning Evaluation in French

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
| Amsili & Seminck (2017) | 100 | French Winograd schemas, parsed from original XML |
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

| Model | Type | Scoring Method | Notes |
|---|---|---|---|
| Mistral-7B-v0.1 (Jiang et al., 2023) | Causal LM | Partial evaluation (log-likelihood) | Follows Gevers et al. (2025) run_llm.py |
| CamemBERT-base (Martin et al., 2020) | Masked LM | Pseudo-log-likelihood (PLL) | Standard protocol for encoder-only models |

### Causal LM scoring (Mistral-7B)

Scoring follows the partial evaluation implementation of Gevers et al. (2025). For each instance and each option, the text before the blank is treated as context. The summed log-likelihood of the option tokens is computed using `tokenizer.encode` with `add_special_tokens=False`, and log-probabilities are extracted with `torch.gather` on the shifted logits. The `evaluate_on_full_continuation` flag is set to `False`, matching the WinoWhat setting.

### Masked LM scoring (CamemBERT)

Each completed sentence is scored by masking one token at a time and summing log P(token | rest of sentence) across all non-special tokens. This is the standard pseudo-log-likelihood protocol for encoder-only models.

### Statistical reporting

Confidence intervals are computed using the Wilson score interval. Significance of the original vs. paraphrased performance difference is assessed with McNemar's exact test. Effect size is reported as |b - c| / (b + c) (Cohen's g). Between-model comparisons use the same test applied to predictions on the same instances, following the approach of Gevers et al. (2025).

---

## Results

| Model | Original | 95% CI | Paraphrased | 95% CI | Drop |
|---|---|---|---|---|---|
| Mistral-7B | 0.480 | [0.412, 0.549] | 0.510 | [0.441, 0.578] | -0.030 |
| CamemBERT-base | 0.490 | [0.422, 0.559] | 0.485 | [0.417, 0.554] | +0.005 |

Random baseline: 0.500 | Always-option1 baseline: 0.460

**McNemar's exact test: original vs. paraphrased**

| Model | b | c | p-value | Effect size | Significant |
|---|---|---|---|---|---|
| Mistral-7B | 9 | 15 | 0.3075 | 0.250 | No |
| CamemBERT-base | 16 | 15 | 1.0000 | 0.032 | No |

**McNemar's exact test: between-model comparisons**

| Condition | p-value | Effect size | Significant |
|---|---|---|---|
| Mistral vs. CamemBERT (original) | 0.9234 | 0.019 | No |
| Mistral vs. CamemBERT (paraphrased) | 0.6849 | 0.052 | No |

Both models perform at approximately chance level (0.50) on both the original and paraphrased sentences. No significant performance drop under paraphrasing is observed for either model, and no significant difference between models is found. These results diverge from the English findings of Gevers et al. (2025) and are discussed in the thesis.

---

## Repository Structure
```
.
├── data/
│   ├── French_Wino_Schemas.xml          # Original XML dataset (Amsili & Seminck, 2017)
│   ├── french_wino_unrandomized.csv     # Structured dataset extracted from XML
│   ├── french_wino_randomized.csv       # Dataset with randomized option ordering
│   └── french_wino_with_paraphrases.csv # Final dataset used in experiments
│
├── notebooks/
│   ├── french_wino_data.ipynb                          # Data processing pipeline
│   └── french_winowhat_replication_experiment.ipynb    # Main experiment notebook
│
├── results/
│   ├── results_original_mistral.csv       # Mistral predictions on original sentences
│   ├── results_paraphrased_mistral.csv    # Mistral predictions on paraphrased sentences
│   ├── results_original_camembert.csv     # CamemBERT predictions on original sentences
│   ├── results_paraphrased_camembert.csv  # CamemBERT predictions on paraphrased sentences
│   ├── wrong_original_mistral.csv         # Mistral errors on original sentences
│   ├── wrong_paraphrased_mistral.csv      # Mistral errors on paraphrased sentences
│   ├── wrong_original_camembert.csv       # CamemBERT errors on original sentences
│   ├── wrong_paraphrased_camembert.csv    # CamemBERT errors on paraphrased sentences
│   └── summary_results.csv               # Full results summary with confidence intervals
│
└── README.md
```

---

## Reproducing the Experiments

### Requirements
```bash
pip install transformers torch pandas numpy tqdm scikit-learn statsmodels
```

### Steps

1. Run `french_wino_data.ipynb` to parse the XML dataset, apply option randomization, and construct the final dataset with paraphrases.
2. Run `french_winowhat_replication_experiment.ipynb` to evaluate both models and generate all result files and figures. The notebook is designed to run on Google Colab with an A100 GPU.

---

## References

Amsili, P., & Seminck, O. (2017). A Google-proof collection of French Winograd schemas. *Proceedings of the 2nd Workshop on Coreference Resolution Beyond OntoNotes (CORBON 2017)*, 24-29.

Gevers, I., De Marez, V., De Bruyne, L., & Daelemans, W. (2025). WinoWhat: A parallel corpus of paraphrased WinoGrande sentences with common sense categorization. *Proceedings of the 29th Conference on Computational Natural Language Learning (CoNLL 2025)*, 68-80.

Jiang, A. Q., et al. (2023). Mistral 7B. *arXiv preprint arXiv:2310.06825*.

Martin, L., et al. (2020). CamemBERT: A tasty French language model. *Proceedings of ACL 2020*, 7203-7219.

Sakaguchi, K., Le Bras, R., Bhagavatula, C., & Choi, Y. (2021). WinoGrande: An adversarial Winograd schema challenge at scale. *Communications of the ACM*, 64(9), 99-106.

Trinh, T. H., & Le, Q. V. (2018). A simple method for commonsense reasoning. *arXiv preprint arXiv:1806.02847*.

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
