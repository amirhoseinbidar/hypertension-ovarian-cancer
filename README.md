# Hypertension – Ovarian Cancer: network analysis

Do ovarian cancer (OC) and hypertension (HTN) share disease genes, and do their genes sit close together on the human protein–protein interactome? The project follows the disease-module method of Menche et al. 2015 (*Uncovering disease-disease relationships through the incomplete interactome*). Systolic blood pressure (SBP) is included as a positive control for HTN.

Inputs

| File                                                        | Content                                                                      |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `../our_paper/sep_25/DataS1_interactome.tsv`              | Menche interactome (genes as Entrez IDs). A copy is also kept in `inputs/` |
| `inputs/hgnc_complete_set.txt`                            | HGNC table, used to map Entrez IDs to gene symbols                           |
| `inputs/diseases/ovarian cancer-MONDO_0008170.tsv`        | Open Targets gene–disease associations for OC                               |
| `inputs/diseases/hypertensive disorder-MONDO_0005044.tsv` | Same, for HTN                                                                |
| `inputs/diseases/systolic blood pressure-EFO_0006335.tsv` | Same, for SBP                                                                |
| `inputs/menche_sourcecode/`, `inputs/menche_paper/`     | Reference code, paper and supplementary material                             |

## Pipeline

Run the notebooks in this order from the project folder.

### 1. `data_process_entrez.ipynb`: build the interactome

- Maps Entrez IDs to gene symbols with the `entrez_id` column of HGNC.
- Lists the 126 Entrez IDs that have no HGNC symbol. An optional cell (needs internet) asks NCBI whether each ID was replaced, discontinued or is still live.
- Checks that known pairs (TP53–MDM2, BRCA1–BARD1, …) are present.
- Removes self-interactions and duplicate pairs (A–B = B–A).
- Writes `outputs/interactome/`. **`interactome_both_symbols.csv`** (edges where both genes have a symbol) is the network used by the later steps.

### 2. `disease_gene_overlap.ipynb`: overlap and network measures

- **Disease genes:** genes with Open Targets `globalScore ≥ CUTOFF` (0.1) that are on the network.
- **Gene overlap** for each disease pair: Jaccard index, overlap coefficient, fold enrichment and hypergeometric p-value. A sensitivity check repeats this at cutoffs 0.2–0.5.
- **Localization** (Menche): size of the largest connected component (LCC) and mean distance d_s to the nearest gene of the same disease.
- **Separation** (Menche): s_AB = ⟨d_AB⟩ − (⟨d_AA⟩ + ⟨d_BB⟩)/2. A negative value means the two disease modules overlap.
- Each measure gets a z-score against `N_SIMS = 10000` random gene sets. `RANDOM_MODEL` is `"uniform"` (Menche's full randomization) or `"degree"` (degree-matched).
- All gene-pair distances are computed once as a matrix (about 180 MB). The random repeats run in parallel with `multiprocessing` (`fork`, Linux only).
- Writes `outputs/cutoff_0.1/`: `gene_overlap.csv`, `localization_<model>.csv`, `separation_<model>.csv` and `gene_lists/{OC,HTN,SBP}.txt`.

### 3. `visualize_network.ipynb`: interactive network views

Writes standalone HTML pages (vis-network, loaded from a CDN). The big network is split into Louvain modules, and each module is drawn in its own circle. Genes in several diseases are drawn as pies of the disease colours. The module of every gene is saved as `<view>_modules.csv`.

| Folder in `outputs/cutoff_0.1/` | Views                                                                                                                            |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `network_OC_HTN/`               | `all_genes.html`, `shared_bridges.html` (shared genes and their neighbours), `cross_links.html` (links between OC and HTN) |
| `network_OC_HTN_SBP/`           | `all_genes.html`, `shared_bridges.html` (genes in 2+ diseases and their neighbours)                                          |
| `network_RAS/`                  | `ras_network.html` and `ras_genes_summary.csv`: the 23 renin–angiotensin system genes and the disease genes linked to them  |

## Main results (cutoff 0.1, uniform model)

| Pair                 | Shared genes | Fold enrichment | s_AB    | z-score                  |
| -------------------- | ------------ | --------------- | ------- | ------------------------ |
| OC – HTN            | 383          | 1.94            | −0.233 | −3.9                    |
| OC – SBP            | 343          | 1.48            | −0.178 | −0.25 (not significant) |
| HTN – SBP (control) | 869          | 2.92            | −0.536 | −23.2                   |

All three diseases form significant modules (LCC z-score: OC 14.5, HTN 6.3, SBP 5.1). The full write-up is in `Report.pdf`.

## Requirements

Python 3 with `pandas`, `numpy`, `networkx` (3.4 or later, for `forceatlas2_layout`), `openpyxl` and Jupyter. Run on Linux, because step 2 uses `fork`.
