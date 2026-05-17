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
│   ├── guerre_et_paix.csv              # 7,874 rows × 30 columns
│   ├── les_miserables.csv              # 5,463 rows × 30 columns
│   ├── Codebook_war_and_peace.md
│   └── codebook_les_miserables.md
├── other_files/                                # Technical files used to set up data_preparation 
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
├── analysis.ipynb                      
├── rapport.pdf                         
├── support_oral.pdf                    
└── requirements.txt
```

---

## Pipeline

### 1. Text extraction (`data_preparation.ipynb`)

Parses 3 EPUB files of *Guerre et Paix* and 5 EPUB files of *Les Misérables* into per-chapter `.txt` files. For each chapter, applies the `propp_fr` NLP pipeline:

1. **Tokenization** — spaCy `fr_dep_news_trf`
2. **CamemBERT embeddings** — token-level vector representations
3. **NER + mention detection** — identifies character mentions
4. **Coreference resolution** — groups mentions into character chains
5. **Syntactic attribute extraction** — extracts agents, patients, modifiers, and possessors per character
6. **Attribute classification** — MLP classifier assigns each attribute to one of 17 semantic classes

Output: `data/guerre_et_paix.csv` and `data/les_miserables.csv`.

### 2. Analysis (`analysis.ipynb`)

- Filters to character entities with ≥ 8 total attributes (75th percentile), yielding 2,424 character–chapter observations
- Defines a **psychological hyperclass**: `perception + personality_traits + affect_emotion + capability_obligation + cognition`
- Computes `psychological_attributes_ratio` = psychological attributes / total attributes per observation
- Identifies Napoleon entries via name dictionaries for each novel

**Results:**

| Test | t-statistic | p-value |
|------|-------------|---------|
| All characters (Tolstoy vs. Hugo) | 2.96 | 0.003 |
| Napoleon only (Tolstoy vs. Hugo) | −0.64 | 0.528 |

Tolstoy's characters overall have a significantly higher proportion of psychological attributes than Hugo's, but this difference does not hold specifically for Napoleon.

---

## Data

### `data/guerre_et_paix.csv`

| Field | Description |
|---|---|
| `tome` | Volume (`TOME_I` / `TOME_II` / `TOME_III`) |
| `chapitre` | Chapter |
| `section` | Sub-chapter section (Roman numeral) |
| `id` | Character rank within section (0 = most mentioned) |
| `count` | `{occurrence, mention_ratio}` |
| `gender` | `{ratio, inference, max, argmax}` |
| `number` | `{ratio, inference, max, argmax}` |
| `mentions` | `{proper: [{n, c}], common: [{n, c}], pronoun: [{n, c}]}` |
| `agent / patient / mod / poss` | `[{w: lemma, i: token_index}]` |
| 17 class columns | Integer count of attributes per semantic class |

7,874 rows × 30 columns. Full column documentation: [`data/Codebook GP.md`](data/Codebook%20GP.md)

### `data/les_miserables.csv`

Same structure, with `volume` / `livre` instead of `tome` / `section`.

5,463 rows × 30 columns. Full column documentation: [`data/codebook_les_miserables.md`](data/codebook_les_miserables.md)

> All columns from `count` to `poss` are serialized Python/JSON strings and must be parsed with `ast.literal_eval()` before use.

---

## Requirements

- Python 3.10+
- `propp-fr==0.0.93` (local NER + coreference model)
- spaCy with `fr_dep_news_trf-3.8.0`
- `torch==2.11.0` (CamemBERT embeddings + MLP classifier)
- `pandas`, `numpy`, `scikit-learn`
- `matplotlib`, `seaborn`, `scipy`, `networkx`
- `ebooklib`, `beautifulsoup4` (EPUB parsing, data preparation only)

Install pinned dependencies:

```bash
pip install -r requirements.txt
pip install scipy seaborn networkx
```
