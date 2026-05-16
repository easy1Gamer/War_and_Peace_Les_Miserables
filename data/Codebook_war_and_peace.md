# Codebook — `guerre_et_paix.csv`

## Overview

This dataset contains character-level linguistic annotations extracted from the French translation of *War and Peace* (*Guerre et Paix*) by Leo Tolstoy. Each row represents one character entity (co-reference chain) as it appears within a specific section of the text. The data captures the character's frequency of occurrence, inferred gender and number, their syntactic roles (agent, patient, modifier, possessor), and the counts of their attributes across 17 semantic classes.

- **Rows:** 7,874
- **Columns:** 30
- **Granularity:** One row per character per section

---

## Column Descriptions

### `Unnamed: 0`
Row index inherited from the original export. No analytical meaning; can be ignored or dropped.

---

### `tome`
The internal book division (e.g., `TOME_I`, `TOME_II`, `TOME_III`).

---

### `chapitre`
The chapter within the tome, encoded as a string label (e.g., `CHAPITRE_PREMIER`).

---

### `section`
A finer sub-division within a chapter, encoded as Roman numerals. Sections represent the smallest unit of text for which annotations are computed.

---

### `id`
The identifier of the character (co-reference chain) within its section. IDs are local to each section and restart from `0` in each new section. Characters are ranked by frequency of occurrence (`0` = most mentioned character in that section).

---

### `count`
A serialized dictionary containing the character's raw mention statistics within the section.
- `occurrence` *(int)*: Total number of times the character is mentioned (across all mention types).
- `mention_ratio` *(float)*: Share of this character's mentions out of all character mentions in the section.

---

### `gender`
A serialized dictionary representing the inferred gender of the character based on their mentions in the section.
- `ratio` *(float)*: Proportion of mentions that carried gendered information.
- `inference` *(dict)*: Probability distribution over `Male` and `Female`.
- `max` *(float)*: Highest probability in the distribution.
- `argmax` *(str)*: Predicted gender — `"Male"` or `"Female"`. Less reliable when `ratio` is low (i.e., most mentions are gender-neutral).

---

### `number`
A serialized dictionary representing the inferred grammatical number of the character entity.
- `ratio` *(float)*: Proportion of mentions that carried number information.
- `inference` *(dict)*: Probability distribution over `Singular` and `Plural`.
- `max` *(float)*: Highest probability in the distribution.
- `argmax` *(str)*: Predicted number — `"Singular"` or `"Plural"`.

---

### `mentions`
A serialized dictionary listing all surface forms used to refer to the character in the section, organized by mention type.
- `proper` *(list of dicts)*: Proper name mentions. Each entry has a surface form (`n`) and occurrence count (`c`).
- `common` *(list of dicts)*: Common noun mentions (descriptive phrases, titles, relational nouns). Same `n`/`c` structure.
- `pronoun` *(list of dicts)*: Pronominal mentions. Same `n`/`c` structure.

---

### `agent`
A serialized list of verbs for which this character is the grammatical subject. Captures what the character *does* in the section. Each entry contains the verb lemma (`w`) and its token index within the section (`i`).

---

### `patient`
A serialized list of verbs for which this character is the grammatical object. Captures what is *done to* the character. Same structure as `agent`.

---

### `mod`
A serialized list of adjectives (and attributive nouns) that directly modify this character in the text. Captures how the character is *described*. Same structure as `agent`.

---

### `poss`
A serialized list of nouns appearing in a possessive or genitive relation with this character. Captures the character's associated social and physical sphere. Same structure as `agent`.

---

## Semantic Attribute Class Columns

The following 17 columns contain integer counts of character attributes classified into semantic categories by a trained MLP classifier (CamemBERT embeddings). Each value is the number of extracted attribute tokens (from `agent`, `patient`, `mod`, `poss`) assigned to that class for this character in this section.

| Column | Description |
|--------|-------------|
| `action` | Goal-oriented actions and achievements |
| `affect_emotion` | Emotional states and feelings |
| `body_parts` | References to physical body parts |
| `capability_obligation` | Abilities, duties, and requirements |
| `cognition` | Thinking, knowing, and understanding |
| `communication` | Speech acts and language use |
| `environment` | Spatial and environmental context |
| `existence_state` | States of being and existence conditions |
| `human_relations` | Social relationships and kinship |
| `motion` | Movement and travel |
| `not_a_valid_character_attribute` | Noise — tokens that are not meaningful character attributes |
| `perception` | Sensory input and observation |
| `personality_traits` | Character traits and disposition |
| `physical_objects` | Tangible items and possessions |
| `physical_traits` | Physical appearance and health |
| `possession` | Ownership and possession relations |
| `sociological` | Social status, class, and profession |

---

## Notes on Parsing

All columns from `count` to `poss` are stored as strings representing Python/JSON-like structures and must be parsed before use with `ast.literal_eval()` in Python. The token index (`i`) values in `agent`, `patient`, `mod`, and `poss` refer to positions within the section's full token sequence, and can be used to retrieve the original textual context of each annotation.
