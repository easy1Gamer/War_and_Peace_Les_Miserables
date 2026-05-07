# Codebook — `les_miserables.csv`

## Overview

This dataset contains character-level linguistic annotations extracted from Victor Hugo's *Les Misérables*. Each row represents one character entity (co-reference chain) within a specific chapter of the text. The data captures each character's frequency of occurrence, inferred gender and number, and their syntactic roles (agent, patient, modifier, possessor) within that chapter.

- **Rows:** 2,676
- **Columns:** 13
- **Granularity:** One row per character per chapter

---

## Column Descriptions

### `Unnamed: 0`
Row index inherited from the original export. No analytical meaning; can be ignored or dropped.

---

### `volume`
The volume of the novel, identified by a slug combining the author's name and the volume title.

---

### `livre`
The book (part) within the volume, encoded as a descriptive string label derived from its title.

---

### `chapitre`
The chapter within the book, encoded as a descriptive string label derived from its title.

---

### `id`
The identifier of the character (co-reference chain) within its chapter. IDs are local to each chapter, with `0` assigned to the most frequently mentioned character. Values restart from `0` in each new chapter.

---

### `count`
A serialized dictionary containing the character's raw mention statistics within the chapter.
- `occurrence`: Total number of times the character is mentioned.
- `mention_ratio`: Share of this character's mentions out of all character mentions in the chapter.

---

### `gender`
A serialized dictionary representing the inferred gender of the character based on their mentions in the chapter.
- `ratio`: Proportion of mentions that carried gendered information.
- `inference`: Probability distribution over `Male` and `Female`.
- `max`: Highest probability in the distribution.
- `argmax`: Predicted gender — `"Male"` or `"Female"`. Less reliable when `ratio` is low (i.e., most mentions are gender-neutral).

---

### `number`
A serialized dictionary representing the inferred grammatical number of the character entity.
- `ratio`: Proportion of mentions that carried number information.
- `inference`: Probability distribution over `Singular` and `Plural`.
- `max`: Highest probability in the distribution.
- `argmax`: Predicted number — `"Singular"` or `"Plural"`.

---

### `mentions`
A serialized dictionary listing all surface forms used to refer to the character in the chapter, organized by mention type.
- `proper`: Proper name mentions, each with a surface form (`n`) and occurrence count (`c`).
- `common`: Common noun mentions (descriptive phrases, titles, relational nouns), same structure.
- `pronoun`: Pronominal mentions, same structure.

---

### `agent`
A serialized list of verbs for which this character is the grammatical subject. Captures what the character *does* in the chapter. Each entry contains the verb lemma (`w`) and its token index within the chapter (`i`).

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

## Notes on Parsing

All columns from `count` to `poss` are stored as strings representing Python/JSON-like structures and must be parsed before use, for instance with `ast.literal_eval()` in Python. The token index (`i`) values in `agent`, `patient`, `mod`, and `poss` refer to positions within the chapter's full token sequence, and can be used to retrieve the original textual context of each annotation.
