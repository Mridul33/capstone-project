# Comparing LLMs and Fine-Tuned Transformer Models for Biomedical NER and Relation Extraction

This repository contains the implementation for my MSc Artificial Intelligence capstone project at the University of Galway.

The project compares a fine-tuned biomedical transformer model, **PubMedBERT**, with a few-shot Large Language Model, **Qwen2.5-7B-Instruct**, for two biomedical information extraction tasks:

- Named Entity Recognition (NER)
- Relation Extraction (RE)

The extracted biomedical information is also used to construct a knowledge graph in **Neo4j**.

---

## Project Objective

The main objective of this project is to compare a supervised, domain-specific transformer with a few-shot instruction-tuned LLM for biomedical information extraction.

Both approaches are evaluated using the **BioRED** dataset, which contains manually annotated biomedical entities and relations from PubMed abstracts.

The project focuses on:

- biomedical entity recognition
- biomedical relation classification
- few-shot LLM prompting
- structured JSON output validation
- class imbalance in relation extraction
- comparison of supervised and prompt-based approaches
- biomedical knowledge graph construction

---

## Dataset

The project uses the **BioRED** dataset.

The original dataset split is retained:

- Training: 400 documents
- Development: 100 documents
- Test: 100 documents

### Entity Types

BioRED contains six biomedical entity categories:

- `ChemicalEntity`
- `DiseaseOrPhenotypicFeature`
- `GeneOrGeneProduct`
- `SequenceVariant`
- `OrganismTaxon`
- `CellLine`

### Relation Types

The relation extraction task uses the eight BioRED relation categories:

- `Association`
- `Bind`
- `Comparison`
- `Conversion`
- `Cotreatment`
- `Drug_Interaction`
- `Negative_Correlation`
- `Positive_Correlation`

An additional `No_Relation` class is introduced for entity pairs without an annotated relation.

---

## Project Pipeline

The implementation is divided into the following main stages:

### Stage 1 – BioRED Data Preparation

- Load BioRED documents and annotations
- Reconstruct title and abstract text
- Validate entity offsets
- Inspect entity and relation distributions
- Prepare the dataset for downstream NER and RE tasks

### Stage 2 – NER and Relation Dataset Preparation

- Convert biomedical entities into BIO sequence labels
- Tokenise documents using PubMedBERT
- Align entity spans with tokens
- Generate relation extraction examples
- Add explicit entity markers
- Create negative `No_Relation` examples
- Validate prepared datasets

### Stage 3 – PubMedBERT Fine-Tuning

#### Named Entity Recognition

PubMedBERT is fine-tuned as a token-classification model using BIO labels.

Long documents are handled using sentence-based chunking so that sequences remain within the 512-token model limit.

#### Relation Extraction

PubMedBERT is fine-tuned as a sequence-classification model using explicitly marked entity pairs.

Class-weighted cross-entropy loss is used to reduce the effect of relation class imbalance.

---

## Stage 4 – Qwen2.5-7B-Instruct Named Entity Recognition

Qwen2.5-7B-Instruct is evaluated without task-specific fine-tuning.

The NER pipeline uses:

- 4-bit quantised model loading
- three frozen few-shot demonstrations
- structured JSON generation
- output validation
- entity surface-form matching
- mention expansion
- character-offset recovery

The same frozen prompt is used throughout evaluation.

---

## Stage 5 – Qwen2.5-7B-Instruct Relation Extraction

The relation extraction pipeline uses:

- nine frozen few-shot demonstrations
- one demonstration for each relation label, including `No_Relation`
- explicit entity markers
- deterministic generation
- structured JSON output
- relation label validation
- development and test-set evaluation

The model predicts one relation label for each predefined entity pair.

---

## Stage 6 – Knowledge Graph Construction

The final knowledge graph is constructed in **Neo4j Aura** using the biomedical entity information associated with the PubMedBERT relation extraction examples and the corresponding predicted relations.

Entities are canonicalised using their biomedical identifiers before graph construction.

Predicted `No_Relation` cases are excluded because they do not represent graph edges.

The final knowledge graph contains:

- **948 biomedical entity nodes**
- **1,126 PubMedBERT-derived relationships**

The graph can be queried using Cypher to inspect overall connectivity and individual biomedical subgraphs.

---

## Final Results

### Named Entity Recognition

| Model | Precision | Recall | F1 |
|---|---:|---:|---:|
| PubMedBERT | 0.8500 | 0.8851 | 0.8672 |
| Qwen2.5-7B-Instruct | 0.6484 | 0.5445 | 0.5768 |

PubMedBERT achieved substantially stronger NER performance across all three metrics.

Qwen produced valid JSON for **97/100** test documents.

---

### Relation Extraction

| Model | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| PubMedBERT | 0.7048 | 0.3826 | 0.7030 |
| Qwen2.5-7B-Instruct | 0.5331 | 0.3873 | 0.5431 |

PubMedBERT achieved stronger overall accuracy and weighted F1.

Qwen achieved higher macro recall and a slightly higher macro F1, indicating broader coverage of some minority relation classes but with lower precision.

For Qwen relation extraction:

- Valid JSON outputs: **2326/2326**
- Valid relation labels: **2281/2326**

---

## Knowledge Graph Example

The Neo4j knowledge graph allows extracted biomedical concepts to be explored as connected nodes and relationships.

An SCN5A-centred subgraph contains six nodes and five `Association` relationships connecting SCN5A with concepts including:

- bradycardia
- tachycardia
- arrhythmia
- tetrodotoxin
- long QT syndrome

---

## Repository Structure

```text
capstone-project/
│
├── Stage_1/
│   └── BioRED preprocessing and validation
│
├── Stage_2/
│   └── NER and relation extraction dataset preparation
│
├── Stage_3/
│   ├── PubMedBERT NER
│   └── PubMedBERT relation extraction
│
├── Stage_4/
│   └── Qwen2.5-7B-Instruct few-shot NER
│
├── Stage_5/
│   └── Qwen2.5-7B-Instruct few-shot relation extraction
│
├── Stage_6/
│   └── Neo4j knowledge graph construction
│
└── README.md
