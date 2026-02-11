
# French WinoWhat

This repository contains the data and code for a Master’s thesis project that replicates and extends the **WinoWhat** framework for evaluating commonsense reasoning under paraphrasing with a focus on **French**.

The project will investigate whether language models rely on genuine commonsense reasoning or on surface-level linguistic cues by comparing performance on original and paraphrased Winograd-style sentences.

---

## Project Overview

- **Task**: Winograd-style commonsense reasoning (fill-in-the-blank)
- **Language**: French
- **Methodology**:
  - Evaluate models on original sentences
  - Evaluate the same models on meaning-preserving paraphrases
  - Compare performance to assess robustness to paraphrasing

This follows the experimental setup introduced in *WinoWhat* and applies it to a French-language setting.

---

## Data

- **Source dataset**: French Winograd Schemas  
  (Amsili & Seminck, 2017)
- **Format**:
  - Each instance contains:
    - a sentence with a blank (`_`)
    - two candidate options
    - a gold answer label (1 or 2)
  - A parallel column stores a paraphrased version of each sentence

### Data files

- `data/raw/`
  - Original XML dataset
- `data/processed/`
  - `french_wino_unrandomized.csv`
  - `french_wino_randomized.csv`
  - `french_wino_with_paraphrases.csv`
- `french_wino_data.ipynb`

Paraphrases are generated using a large language model and manually verified to preserve meaning and ambiguity.

---

## Repository Structure

french-wino-what/
├── data/
│ ├── raw/
│ ├── processed/
│ └── logs/
├── scripts/
├── notebooks/
├── results/
├── README.md
└── requirements.txt

## Status

This repository is under active development as part of a Master’s thesis.  
Code and data will be updated as experiments progress.

---

## License

This repository is intended for academic and research purposes.
