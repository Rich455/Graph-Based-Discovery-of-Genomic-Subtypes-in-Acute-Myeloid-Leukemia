# AML-Genomic-Subtyping-with-Graphs-Causality

This project explores acute myeloid leukemia (AML) genomic subtypes using a graph-based machine learning approach, inspired by the paper:

“Machine learning integrates genomic signatures for subclassification beyond primary and secondary acute myeloid leukemia”
Blood (2021) https://ashpublications.org/blood/article-abstract/138/19/1885/476049

Instead of using traditional supervised classifiers alone, this project takes a network-driven perspective to understand how mutations co-occur, cluster, and define biological AML subtypes.



 Core Idea

AML is not driven by single mutations, but by combinations of cooperating mutations that define distinct disease entities.

This project answers three key questions:

Which mutations tend to appear together in the same patients?

Can we automatically identify mutation communities (subtypes)?

Which mutations show strong positive or negative (exclusive) relationships?


 Data

Source: Simulated AML mutation matrix derived from TCGA-LAML (Genomic Data Commons, NCI)

Patients: ~3000

Genes analyzed (binary: mutated / not mutated):

NPM1, FLT3-ITD, DNMT3A

RUNX1, ASXL1, SRSF2, U2AF1

TP53, Complex Karyotype

IDH1, IDH2

CEBPA (biallelic)


Data are simulated for methodological demonstration, not direct clinical inference.


🔗 1. Co-occurrence Network (Graph-Based View)
What does the graph represent?

Nodes (dots) → Gene mutations

Edges (lines) → Two mutations appear together in many patients

Edge thickness → Number of patients sharing both mutations

Threshold applied → Only draw an edge if >80 patients have both mutations

co_occurrence = aml_df[GENES].T.dot(aml_df[GENES])

This matrix counts how many patients carry both mutation A and mutation B.




Why use a threshold (e.g. >80 patients)?

Small co-occurrence counts can occur by chance

Larger counts are more biologically meaningful

Thresholding reduces noise and improves interpretability

Trade-off: rare but real relationships may be missed — this is intentional to focus on dominant AML biology.



2. Automatic Mutation Community Detection

Which mutations tend to cluster together as a group across the entire dataset?

To identify groups of mutations that tend to appear together, we applied modularity-based community detection:

communities = greedy_modularity_communities(G)

Resulting mutation communities:

Module 1
Core AML signaling / epigenetic mutations
(NPM1, FLT3-ITD, DNMT3A, IDH2, RUNX1, splicing genes)

Module 2
Adverse-risk AML
(TP53, Complex Karyotype, ASXL1)

Module 3
Isolated metabolic subtype
(IDH1)

These communities emerge automatically, without predefined labels — showing that graph structure alone recovers known AML subtypes.


 3. Raw Co-occurrence Counts (Transparency)

To maintain interpretability, the project also prints:

The full co-occurrence matrix

A ranked table of significant mutation pairs

Example:

Gene 1	Gene 2	Patients
NPM1	DNMT3A	472
NPM1	FLT3-ITD	458
TP53	Complex Karyotype	367
RUNX1	SRSF2	385

These numbers align strongly with established AML biology.


### AML Mutation Co-occurrence Network

This directed graph shows **strong and statistically significant positive co-occurrences** among recurrent mutations in Acute Myeloid Leukemia (AML), based on logistic regression and Fisher's exact test (Bonferroni-corrected).

**Key findings** (only the strongest links are displayed):

- **NPM1-centered cluster** (classic normal-karyotype AML):  
  NPM1 ↔ FLT3-ITD ↔ DNMT3A form a tightly cooperating trio (highest co-occurrence rates).

- **Splicing factor / myeloid neoplasia cluster**:  
  SRSF2 ↔ U2AF1 ↔ ASXL1 ↔ RUNX1 frequently co-occur, defining a distinct subgroup often with myelodysplastic features.

- **Adverse-risk group**:  
  TP53 ↔ Complex_Karyotype — almost inseparable, reflecting genomic catastrophe driven by loss of p53 function.

- **CEBPA / IDH2**:  
  Strong bidirectional link between biallelic CEBPA mutations and IDH2.

Green arrows indicate that the presence of one mutation significantly increases the likelihood of the other. Arrow thickness and edge labels reflect association strength (log-odds coefficient) and p-value.

This network accurately recapitulates the major molecular subclasses of AML as defined in current clinical guidelines (ELN 2022, ICC).


![Alt text describing the image](https://github.com/Rich455/AML-Genomic-Subtyping-with-Graphs-Causality/blob/main/Significant%20Co-occurrence%20Network%20of%20Gene%20Mutations.png)


⚠️ Limitations

Cross-sectional data → associations, not true temporal causality

Thresholding may miss rare subtypes

Simulated dataset used for demonstration



###Causal**

Project applies causal inference–inspired statistical modeling to large-scale AML mutation data to reconstruct a directed mutation co-occurrence network. The goal is to identify strong, statistically significant mutation dependencies that may reflect mutation ordering, pathway coupling, or shared clonal evolution.

⚠️ Important:
The inferred directions represent statistical dependencies, not definitive mechanistic causality.

🎯 Objectives

Identify strong positive mutation co-occurrence relationships in AML

Infer directional dependencies using logistic regression

Enforce statistical rigor via Fisher’s exact test with Bonferroni correction

Visualize results as a directed gene mutation network

Generate biologically interpretable hypotheses for AML evolution


Directional Dependency Modeling

For every pair of genes (A, B):

A logistic regression model is fitted:

𝑃
(
𝐵
=
1
∣
𝐴
)
=
𝜎
(
𝛽
⋅
𝐴
)
P(B=1∣A)=σ(β⋅A)

The regression coefficient (β) measures how strongly mutation A increases the odds of mutation B

Only relationships satisfying all criteria below are retained:




Criterion	Threshold
Minimum mutation count	≥ 20 samples
Effect size	β > 0.6
Statistical significance	Fisher’s Exact Test
Multiple testing	Bonferroni correction

Statistical Validation

To ensure robustness:

Each gene pair is tested using Fisher’s exact test

P-values are corrected for multiple comparisons using Bonferroni adjustment

Only positive and statistically significant relationships are reported


Key Results
Significant Directed Co-occurrence Relationships
NPM1 → FLT3_ITD        (β=2.17, p=6.56e-126)
NPM1 → DNMT3A         (β=2.19, p=7.05e-130)
FLT3_ITD → DNMT3A     (β=2.08, p=1.60e-113)
RUNX1 → ASXL1         (β=1.58, p=8.57e-64)
RUNX1 → SRSF2         (β=2.20, p=6.74e-116)
RUNX1 → U2AF1         (β=1.83, p=1.53e-78)
ASXL1 → SRSF2         (β=1.84, p=3.35e-80)
ASXL1 → U2AF1         (β=1.60, p=3.37e-58)
SRSF2 → U2AF1         (β=2.17, p=6.35e-102)
TP53 → Complex_Karyotype (β=2.35, p=1.55e-124)
IDH2 → CEBPA_biallelic   (β=2.29, p=1.15e-132)



🧬 Network Visualization

Nodes represent genes or cytogenetic features

Green arrows indicate strong, statistically significant positive dependencies

Edge thickness corresponds to effect size

Edge labels display regression strength and p-value

Interpretation:

Arrows indicate that mutation A increases the probability of mutation B

Bidirectional patterns reflect co-selection or pathway coupling, not reciprocal causation

🧠 Biological Interpretation

This framework recapitulates known AML biology:

NPM1 → FLT3_ITD
Consistent with early founder mutations followed by proliferative signaling hits

Spliceosome gene clustering (SRSF2, U2AF1, ASXL1)
Reflects pathway-level selection and shared clonal evolution

TP53 → Complex Karyotype
Matches established associations between TP53 loss and genomic instability

⚠️ Causal Disclaimer

While arrows suggest mutation ordering, this model does not establish direct mechanistic causality. Observed dependencies may reflect shared clonal origin, functional pathway coupling, or selective pressure during leukemogenesis.

This project is intended as a hypothesis-generating causal scaffold.


# 🧬 AML-Genomic-Subtyping-with-Graphs-Causality

**Graph-based discovery of AML genomic subtypes and mutation dependencies**

---

## 📖 Background & Motivation

This project explores **Acute Myeloid Leukemia (AML)** genomic subtypes using a **graph-based machine learning and causal-inference–inspired approach**, inspired by the landmark paper:

> **Machine learning integrates genomic signatures for subclassification beyond primary and secondary acute myeloid leukemia**  
> *Blood* (2021)  
> https://ashpublications.org/blood/article-abstract/138/19/1885/476049

Rather than relying solely on supervised classifiers, this project adopts a **network-driven perspective**, allowing AML subtypes and mutation relationships to **emerge directly from the data**.

> **Core premise:**  
> AML is driven by **combinations of cooperating mutations**, not isolated genetic events.

---

## 💡 Core Questions

1. Which mutations tend to co-occur in the same patients?
2. Can mutation communities (AML subtypes) be discovered automatically?
3. Which mutations show strong positive or exclusive (negative) relationships?

---

## 📊 Data

**Source:** Simulated AML mutation matrix derived from **TCGA-LAML**  
**Patients:** ~3000  

**Genes analyzed (binary: mutated / not mutated):**

- **Signaling / Epigenetic:**  
  `NPM1`, `FLT3-ITD`, `DNMT3A`, `IDH1`, `IDH2`
- **Transcription / Splicing:**  
  `RUNX1`, `ASXL1`, `SRSF2`, `U2AF1`
- **Chromosomal instability:**  
  `TP53`, `Complex Karyotype`
- **Favorable-risk AML:**  
  `CEBPA (biallelic)`

⚠️ Data are simulated for methodological demonstration only.

---

## 🔗 1. Mutation Co-occurrence Network

### Graph Representation

- **Nodes:** Gene mutations  
- **Edges:** Frequent co-occurrence in patients  
- **Edge thickness:** Number of shared patients  

Co-occurrence matrix:

```python
co_occurrence = aml_df[GENES].T.dot(aml_df[GENES])

