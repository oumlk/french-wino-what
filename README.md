# WinoQuoi: A Replication of WinoWhat for Evaluating Commonsense Reasoning in French

This repository contains the data, code and experimental results for my Master's thesis at the University of Antwerp (Academic Year 2025-2026), which replicates and extends the WinoWhat framework (Gevers et al., 2025) for evaluating commonsense reasoning under paraphrasing in French.

The project investigates whether language models rely on commonsense reasoning or surface linguistic cues by comparing model performance on original Winograd-style sentences and meaning-preserving paraphrases in French.

---

## Project Overview

### Task

Winograd-style commonsense reasoning (fill-in-the-blank).

**Example:**
```
La coupe n'entre pas dans la valise marron, car la _ est trop grande.
```

**Options:**
1. valise
2. coupe

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
| Amsili and Seminck (2017) | 96 | French Winograd schemas, parsed from original XML |
| Gevers et al. (2025), translated | 104 | WinoWhat validation subset, translated into French via DeepL |

Each instance contains:
```
id
sentence
paraphrased_sentence
option1
option2
answer
```

**Amsili and Seminck (2017) instances:** The original XML resource provides 107 schemas. For each schema, a single instance was constructed using the first trigger word (wordA), reformatted to match the WinoWhat fill-in-the-blank format. 11 instances were removed during construction due to conversion errors or missing fields, yielding 96 usable instances from this source.

**WinoWhat instances:** 104 instances were randomly sampled from the WinoWhat validation subset of Gevers et al. (2025), translated into French using the DeepL API, and manually verified by a native speaker of French to ensure meaning, ambiguity, and candidate balance were preserved.

Paraphrases were generated with the assistance of GPT and manually verified instance by instance to preserve meaning, grammaticality, and ambiguity. Where possible, the blank token was placed at the end of the sentence following the WinoWhat design; in 103 out of 200 paraphrased instances (51.5%) the blank is sentence-final. In the remaining 97 instances (48.5%) the blank was retained in a mid-sentence position because sentence-final placement would have produced ungrammatical French. Option ordering was randomized with a fixed seed to avoid positional bias. A source column is included in the final dataset to distinguish between the two origins.

**Corpus statistics:**

| | Original | Paraphrased |
|---|---|---|
| Mean length (words) | 19.1 | 20.6 |
| Std | 5.0 | 5.5 |
| Min / Max (words) | 10 / 34 | 9 / 45 |
| Sentence-final blank | 37 (18.5%) | 103 (51.5%) |
| Mid-sentence blank | 163 (81.5%) | 97 (48.5%) |

Answer distribution: 92 instances with correct answer in position 1 (46.0%), 108 in position 2 (54.0%).

---

## Models and Evaluation Protocol

| Model | Type | Scoring Method |
|---|---|---|
| Mistral-7B-v0.1 (Jiang et al., 2023) | Causal LM | Full-continuation causal scoring |
| CamemBERT-base (Martin et al., 2020) | French Masked LM | Pseudo-log-likelihood (PLL) |
| XLM-RoBERTa-base (Conneau et al., 2020) | Multilingual Masked LM | Pseudo-log-likelihood (PLL) |

**Causal LM scoring (Mistral-7B):** Follows the partial evaluation implementation of Gevers et al. (2025) with full-continuation scoring. The model scores the option tokens together with the text that follows the blank, giving mid-sentence blanks access to right-side context.

**Masked LM scoring (CamemBERT and XLM-RoBERTa):** Each completed sentence is scored by masking one token at a time and summing log P(token | rest of sentence) across all non-special tokens. Scores are averaged to reduce sentence-length bias.

**Statistical reporting:** Confidence intervals use the Wilson score method. Significance is assessed with McNemar's exact test (α = 0.05). Effect size is reported as Cohen's g.

No score ties and no -inf scores were observed across any model or condition.

---

## Results

### Main results

| Model | Original | 95% CI | Paraphrased | 95% CI | Drop | p |
|---|---|---|---|---|---|---|
| Mistral-7B | 0.735 | [0.670, 0.791] | 0.635 | [0.566, 0.699] | +0.100 | 0.012* |
| CamemBERT-base | 0.455 | [0.387, 0.524] | 0.470 | [0.402, 0.539] | -0.015 | 0.755 |
| XLM-RoBERTa-base | 0.560 | [0.491, 0.627] | 0.545 | [0.476, 0.613] | +0.015 | 0.788 |

Random baseline: 0.500 | Always-option-1 baseline: 0.460 | * p < 0.05

Drop = original minus paraphrased accuracy. A positive drop means the model performed better on the original condition.

### McNemar's exact test: within-model

| Model | p-value | Cohen's g | Significant |
|---|---|---|---|
| Mistral-7B | 0.012 | 0.345 | Yes |
| CamemBERT-base | 0.755 | 0.073 | No |
| XLM-RoBERTa-base | 0.788 | 0.055 | No |

### McNemar's exact test: between-model

| Comparison | Condition | p-value | Cohen's g | Significant |
|---|---|---|---|---|
| Mistral vs CamemBERT | Original | <0.001 | 0.475 | Yes |
| Mistral vs XLM-RoBERTa | Original | <0.001 | 0.385 | Yes |
| CamemBERT vs XLM-RoBERTa | Original | 0.040 | 0.221 | Yes |
| Mistral vs CamemBERT | Paraphrased | <0.001 | 0.355 | Yes |
| Mistral vs XLM-RoBERTa | Paraphrased | 0.051 | 0.237 | No |
| CamemBERT vs XLM-RoBERTa | Paraphrased | 0.137 | 0.169 | No |

### Summary

Mistral-7B performs well above chance on the original sentences and shows a statistically significant drop after paraphrasing (p = 0.012, Cohen's g = 0.345), consistent with the paraphrasing sensitivity observed in English by Gevers et al. (2025). The two masked language models perform at or near chance level in both conditions with no significant paraphrasing effect. Mistral-7B significantly outperforms both masked models on the original condition. The difference between CamemBERT-base and XLM-RoBERTa-base is significant on the original condition (p = 0.040) but not on the paraphrased condition.

### English sanity checks

Three sanity checks were run on 50 English WinoWhat instances (random seed 42):

| Model | English accuracy | 95% CI | Interpretation |
|---|---|---|---|
| XLM-RoBERTa-base | 0.480 | [0.35, 0.61] | At chance — confirms PLL limitation, not a pipeline error |
| Mistral-7B | 0.780 | [0.65, 0.87] | Above chance — confirms causal scorer works correctly |
| Llama-2-7B | 0.720 | [0.58, 0.83] | Above chance — confirms scorer generalises across causal models |

---

## Repository Structure

```
.
├── data/
│   ├── French_Wino_Schemas.xml                # Original XML dataset (Amsili and Seminck, 2017)
│   ├── french_wino_unrandomized.csv           # Structured dataset extracted from XML
│   ├── french_wino_randomized.csv             # Dataset with randomized option ordering
│   ├── french_wino_final_with_source.csv      # Final dataset with source labels
│   ├── french_wino_with_paraphrases.csv       # Final dataset with paraphrases used in experiments
│   └── WinoWhat.csv                           # Original WinoWhat data from Gevers et al. 
│
├── notebooks/
│   ├── corpus_construction.ipynb              # Corpus construction pipeline
│   └── WinoQuoi_Experiment.ipynb              # Main experiment notebook
│
├── results/
│   ├── results_original_mistral.csv
│   ├── results_paraphrased_mistral.csv
│   ├── results_original_camembert.csv
│   ├── results_paraphrased_camembert.csv
│   ├── results_original_xlmr.csv
│   ├── results_paraphrased_xlmr.csv
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

Note: `transformers==4.44.0` and `tokenizers==0.19.1` are required to avoid a tokenizer compatibility issue with CamemBERT's SentencePiece vocabulary loading in newer library versions.

### Steps

1. Run `corpus_construction.ipynb` to parse the XML dataset, apply option randomization, and construct the final dataset with paraphrases.
2. Run `WinoQuoi_Experiment.ipynb` to evaluate all three models and generate result files and figures. The notebook is designed to run on Google Colab with an A100 GPU.

---

## References

Amsili, P. and Seminck, O. (2017). A Google-proof collection of French Winograd schemas. *Proceedings of CORBON 2017*, 24-29.

Conneau, A., Khandelwal, K., Goyal, N., Chaudhary, V., Wenzek, G., Guzman, F., Grave, E., Ott, M., Zettlemoyer, L. and Stoyanov, V. (2020). Unsupervised cross-lingual representation learning at scale. *Proceedings of ACL 2020*, 8440-8451.

Gevers, I., De Marez, V., De Bruyne, L. and Daelemans, W. (2025). WinoWhat: A parallel corpus of paraphrased WinoGrande sentences with common sense categorization. *Proceedings of CoNLL 2025*, 68-80.

Jiang, A. Q., Sablayrolles, A., Mensch, A., Bamford, C., Chaplot, D. S., Casas, D., Bressand, F., Lengyel, G., Lample, G., Saulnier, L., Lavaud, L. R., Lachaux, M.-A., Stock, P., Scao, T. L., Lavril, T., Wang, T., Lacroix, T. and Sayed, W. E. (2023). Mistral 7B. *arXiv preprint arXiv:2310.06825*.

Martin, L., Muller, B., Suarez, P. J. O., Dupont, Y., Romary, L., de la Clergerie, E., Seddah, D. and Sagot, B. (2020). CamemBERT: A tasty French language model. *Proceedings of ACL 2020*, 7203-7219.

Sakaguchi, K., Le Bras, R., Bhagavatula, C. and Choi, Y. (2021). WinoGrande: An adversarial Winograd schema challenge at scale. *Communications of the ACM*, 64(9), 99-106.

Touvron, H., Martin, L., Stone, K., Albert, P., Almahairi, A., Babaei, Y., Bashlykov, N., Batra, S., Bhargava, P. and Bhosale, S. (2023). Llama 2: Open foundation and fine-tuned chat models. *arXiv preprint arXiv:2307.09288*.

Trinh, T. H. and Le, Q. V. (2018). A simple method for commonsense reasoning. *arXiv preprint arXiv:1806.02847*.

---

## Author

**Oum Loukili**
Master of Digital Text Analysis
University of Antwerp
Academic Year 2025-2026

Supervisor: Jens Lemmens
Co-promotor: Ine Gevers

---

*This repository is intended for academic and research purposes.*
## Author
