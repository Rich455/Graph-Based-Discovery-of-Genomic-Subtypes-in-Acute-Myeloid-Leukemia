# AML-Genomic-Subtyping-with-Graphs-Causality

This project explores acute myeloid leukemia (AML) genomic subtypes using a graph-based machine learning approach, inspired by the paper:

“Machine learning integrates genomic signatures for subclassification beyond primary and secondary acute myeloid leukemia”
Blood (2021) https://ashpublications.org/blood/article-abstract/138/19/1885/476049

Unlike traditional supervised classifiers, this project takes a network-driven perspective, focusing on co-occurrence of mutations and mutation communities to better understand AML heterogeneity.

---




 **Core Idea**

AML is not driven by single mutations, but by combinations of cooperating mutations that define distinct disease entities.

This project answers three key questions:

Which pair mutations (co-occurence)tend to appear together in the same patients? 

Can we automatically identify mutation communities (can be one gene mutation or more than one)?


---



 **Data**

Source: Simulated AML mutation matrix derived from TCGA-LAML (Genomic Data Commons, NCI)

Patients: ~3000

Genes analyzed (binary: mutated / not mutated):

NPM1, FLT3-ITD, DNMT3A

RUNX1, ASXL1, SRSF2, U2AF1

TP53, Complex Karyotype(−5/del(5q), −7/del(7q),−17/del(17p))

IDH1, IDH2

CEBPA (biallelic)


Data are simulated for methodological demonstration, not direct clinical inference.



Image 01



![Alt text describing the image](https://github.com/Rich455/AML-Genomic-Subtyping-with-Graphs-Causality/blob/main/Dataset%20from%20excel.png)



---


**1. Co-occurrence Network (Graph-Based View)**

What does the graph represent?

Nodes (dots) → Gene mutations

Edges (lines) → Two mutations appear together in many patients

Edge thickness → Number of patients sharing both mutations

Threshold applied → Only draw an edge if >80 patients have both mutations

co_occurrence = aml_df[GENES].T.dot(aml_df[GENES])

This matrix counts how many patients carry both mutation A and mutation B.

---


Why use a threshold (e.g. >80 patients)?

Small co-occurrence counts can occur by chance

Larger counts are more biologically meaningful

Thresholding reduces noise and improves interpretability

Trade-off: rare but real relationships may be missed-this is intentional to focus on dominant AML biology.



Image 02




![Alt text describing the image](https://raw.githubusercontent.com/Rich455/AML-Genomic-Subtyping-with-Graphs-Causality/main/Co-occurrence%20table%20.png)







---

**2. Automatic Mutation Community Detection**

Which mutations tend to cluster together as a group across the entire dataset?

To identify groups of mutations that tend to appear together, we applied modularity-based community detection:

Louvain community algorthim which much better cluster and produce more stable community. 

Resulting mutation communities:

Module 0
('NPM1', 'FLT3_ITD', 'DNMT3A', 'IDH2', 'CEBPA_biallelic')

Module 1
('RUNX1', 'ASXL1', 'SRSF2', 'U2AF1')

Module 2
('TP53', 'Complex_Karyotype')

Module 3
('IDH1')

---


Image  03




![Alt text describing the image](https://raw.githubusercontent.com/Rich455/AML-Genomic-Subtyping-with-Graphs-Causality/main/Modules.png)




---




 **4. AML Mutation Co-occurrence Network**


This directed graph shows **strong and statistically significant positive co-occurrences** among recurrent mutations in Acute Myeloid L

eukemia (AML), based on logistic regression and Fisher's exact test (Bonferroni-corrected).




**Key findings** (only the strongest links are displayed):

- **NPM1-centered cluster** (classic normal-karyotype AML):  
  NPM1 ↔ FLT3-ITD ↔ DNMT3A - CEBPA_biallelic form a tightly cooperating trio (highest co-occurrence rates).

- **Splicing factor / myeloid neoplasia cluster**:  
  SRSF2 ↔ U2AF1 ↔ ASXL1 ↔ RUNX1 frequently co-occur, defining a distinct subgroup often with myelodysplastic features.

- **Adverse-risk group**:  
  TP53 ↔ Complex_Karyotype — almost inseparable, reflecting genomic catastrophe driven by loss of p53 function.

- **CEBPA / IDH2**: 
  Strong bidirectional link between biallelic CEBPA mutations and IDH2.


This network accurately recapitulates the major molecular subclasses of AML as defined in current clinical guidelines (ELN 2022, ICC).



Image 04



![Alt text describing the image](https://raw.githubusercontent.com/Rich455/AML-Genomic-Subtyping-with-Graphs-Causality/main/Graph%20of%20Co-occurence%20and%20Communities.png)
---


**Strengths of the Approach**

Graph-based view: Highlights cooperating mutations rather than analyzing single mutations independently.

Community detection: Automatically identifies clusters of mutations that co-occur across patients.

Biologically interpretable: Strong edges represent dominant AML biology, reflecting clinically relevant subtypes.

Flexible: Can be applied to any mutation matrix for exploratory analysis.


**Weaknesses / Limitations**

Not predictive: This method clusters mutations and identifies relationships, but cannot predict patient outcomes or classify new patients.

Threshold dependency: Low-frequency but biologically important mutations may be excluded due to edge thresholding.

Data limitations: Simulated dataset is used here for demonstration; real clinical inference would require larger, fully curated datasets.



**References**

Papaemmanuil E, et al. Blood (2021). Machine learning integrates genomic signatures for subclassification beyond primary and secondary AML

ELN 2022 Clinical Guidelines for AML

ICC 2022 AML Classification




