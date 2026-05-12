# Weighted Individual Deviation Score (WIDS)
### Identifying outlier patients in microbiome networks using network topology

---

## Objective of the project 

Most microbiome studies ask "which bacteria are present and how many?" — but that misses something important. Two patients can have nearly identical bacterial abundance profiles and still have very different diseases, because what matters is not just *what* bacteria are there, but *how they interact*.

This project builds on that idea. I implemented **WIDS** — a scoring method that looks at the network of co-abundance relationships between bacteria for each patient, and asks: how much does this patient's network deviate from everyone else's?

No clinical data. No labels fed in during scoring. Just the network structure.

I tested it on two completely different diseases — gestational diabetes (gut microbiome) and tuberculosis with COVID co-infection (sputum microbiome) — to see whether the same framework holds across biological contexts.

---

## The algorithm implemented 

WIDS gives each patient a deviation score based on two things:

- **J_direct** — how different is this patient's microbial network from the rest of the group (measured by leave-one-out Jaccard dissimilarity)
- **J_indirect** — how far is this patient's network from a healthy reference network

These combine as:
WIDS(k) = α · J_direct(k) + (1 − α) · J_indirect(k)


The balance parameter **α** is not guessed — it is optimised per dataset using 5-fold stratified cross-validation with AUPR as the objective, chosen specifically because dysbiotic patients are the minority class and standard accuracy metrics would be misleading.

---

## Datasets

| Dataset | Type | Samples | Disease | Source |
|---|---|---|---|---|
| Liu et al. 2023 | Gut (fecal) | 103 | Gestational diabetes | [PLOS Comp Bio](https://doi.org/10.1371/journal.pcbi.1011193) |
| NIRT 2025 | Sputum | 48 | TB / TB-COVID co-infection | NIRT Consortium (https://pmc.ncbi.nlm.nih.gov/articles/PMC13055300/) |

---

## Results

| Dataset | Method | Optimal α | AUPR |
|---|---|---|---|
| GDM | Spearman | 0.15 | 0.452 |
| GDM | Pearson | 0.35 | **0.903** |
| TB-alone | Spearman | 0.95 | 0.396 |
| TB-COVID | Spearman | 1.00 | 0.477 |

One finding that wasn't expected going in: the optimal α for GDM (0.35) produced near-random performance when applied to the TB dataset. Each disease needs its own calibration — the algorithm encodes something disease-specific in α that doesn't transfer. This turned out to be one of the more interesting results of the project.

Negative control validation confirmed that healthy controls (n=72) showed a random WIDS rank distribution (permutation test p > 0.05), meaning the algorithm isn't just flagging everyone.

---

## Repository structure
├── notebooks/
│   └── WIDS_analysis.ipynb       # Full analysis — preprocessing to results
├── figures/                      # All figures from the report
├── requirements.txt
└── README.md


---

## Running the code

```bash
# 1. Clone
git clone https://github.com/DeepikkaSenthil27/Weighted-Individual-Deviation-Score-Algorithm.git
cd Weighted-Individual-Deviation-Score-Algorithm

# 2. Install dependencies
pip install -r requirements.txt

# 3. Get the data
# GDM dataset: download from https://doi.org/10.1371/journal.pcbi.1011193
# Place OTU table at: data/gdm_otu_table.csv

# 4. Run
jupyter notebook notebooks/WIDS_analysis.ipynb
```

---

## Dependencies
numpy, scipy, scikit-learn, pandas, matplotlib, seaborn, jupyter
Full versions in `requirements.txt`.

---

## References

1. Liu Y et al. (2023) PLOS Computational Biology — *primary GDM dataset*
2. Bolyen E et al. (2019) Nature Biotechnology — *QIIME2*
3. Aitchison J. (1982) J Royal Statistical Society B — *CLR transformation*
4. Davis J & Goadrich M. (2006) ICML — *AUPR vs AUROC*
5. Hasain Z et al. (2020) Frontiers in Cellular and Infection Microbiology — *GDM microbiome review*
6. Ma ZS et al. (2020) npj Biofilms and Microbiomes — *dysbiosis networks*
7. Namasivayam S & Sher A. (2018) mBio — *TB microbiome*
8. Zhu T et al. (2022) Virulence — *COVID respiratory microbiome*
9. NIRT Consortium (2025) Clinical Infectious Diseases — *TB-COVID dataset*
