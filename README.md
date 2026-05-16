# Character Attribute Analysis: Tolstoy vs. Hugo

A Digital Humanities / NLP project comparing character attribute profiles in two French-language classic novels:
- *Guerre et Paix* (*War and Peace*, French translation) — Leo Tolstoy
- *Les Misérables* — Victor Hugo

The core research question is whether the **ratio of psychological attributes** assigned to characters differs between the two authors — and whether this difference holds specifically for the shared historical figure of **Napoleon Bonaparte**.

---

## Repository Structure

```
Tolstoy_Hugo_characters/
├── data/
│   ├── guerre_et_paix.csv
│   ├── les_miserables.csv
│   ├── Codebook GP.md
│   └── codebook_les_miserables.md
├── other_files/
│   ├── 0_train_ontology_classifier.ipynb
│   ├── 1_propp_process_annotated_datasets.ipynb
│   ├── AntoineBourgois/                        # Local NER + coreference model weights
│   ├── annotated_attributes_dataset.json       # 600 manually annotated attributes
│   ├── annotated_attributes_embeddings.npy     # CamemBERT embeddings for the dataset
│   ├── attributes_classification_model.pt      # Trained MLP classifier weights
│   ├── ontology_17Classes_classification_report.md
│   ├── syntactic_role_mapping.json             # Maps syntactic roles to one-hot indices
│   └── training_label_mapping.json             # Maps class names to label indices
├── data_preparation.ipynb
├── analysis.ipynb          # Hard threshold (≥ 10 attributes)
└── analysis_v2.ipynb       # Bayesian smoothing (k = 10)
```

---

## Data

### `data/guerre_et_paix.csv`

| Field | Description |
|---|---|
| `tome` | Volume (TOME_I / TOME_II / TOME_III) |
| `chapitre` | Chapter |
| `section` | Sub-chapter section (Roman numeral) |
| `id` | Character rank within section (0 = most mentioned) |
| `count` | `{occurrence, mention_ratio}` |
| `gender` | `{ratio, inference, max, argmax}` |
| `number` | `{ratio, inference, max, argmax}` |
| `mentions` | `{proper: [{n, c}], common: [{n, c}], pronoun: [{n, c}]}` |
| `agent / patient / mod / poss` | `[{w: lemma, i: token_index}]` |
| 17 class columns | Integer count of attributes per class |

~7,874 rows. Full column documentation: [`data/Codebook GP.md`](data/Codebook%20GP.md)

### `data/les_miserables.csv`

Same structure, with `volume` / `livre` instead of `tome` / `section`.

~5,463 rows. Full column documentation: [`data/codebook_les_miserables.md`](data/codebook_les_miserables.md)

> All columns from `count` to `poss` are serialized Python/JSON strings and must be parsed with `ast.literal_eval()` before use.

---

## Requirements

- Python 3.10+
- spaCy with `fr_dep_news_trf`
- `transformers` (CamemBERT)
- `pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`, `networkx`
- propp_fr (local models: NER + coreference)
