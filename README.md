# Hypertension – Ovarian Cancer: network analysis

Tests whether ovarian cancer (OC) and hypertension share disease genes and sit close together on the human protein–protein interactome. It follows the disease-module approach of Menche et al. 2015 (*Uncovering disease-disease relationships through the incomplete interactome*). Systolic blood pressure (SBP) is included as a positive control for HTN.

This project uses the same pipeline as `../hypertention - breast cancer`, with ovarian cancer in place of breast cancer.

## Inputs (`inputs/`)
- `DataS1_interactome.tsv`: Menche interactome, with genes given as Entrez IDs
- `hgnc_complete_set.txt`: HGNC table, used to map Entrez IDs to gene symbols
- `diseases/*.tsv`: Open Targets gene–disease associations for ovarian cancer (MONDO_0008170), hypertensive disorder (MONDO_0005044) and SBP (EFO_0006335)
- `menche_sourcecode/`, `menche_paper/`: reference code and paper

## Pipeline (run the notebooks in this order)
1. **`data_process_entrez.ipynb`** maps the interactome to gene symbols, removes self-interactions and reports unmapped genes.
   Writes `outputs/interactome/` (`interactome_both_symbols.csv` is the network used downstream).
2. **`disease_gene_overlap.ipynb`**
   - Picks the `globalScore` cutoff: the lowest score above every gene supported only by weak, down-weighted sources. This comes out at 0.132 here.
   - Gene overlap per disease pair: Jaccard index, overlap coefficient, fold enrichment and hypergeometric p-value, plus a cutoff sensitivity check.
   - Menche network measures: **localization** (LCC size and mean shortest distance d_s) and **separation** s_AB, each with a z-score against random gene sets. Set `RANDOM_MODEL` to `"uniform"` or `"degree"`.
   - Writes `outputs/cutoff_<value>/` with `gene_overlap.csv`, `localization_*.csv`, `separation_*.csv` and `gene_lists/*.txt`.
3. **`visualize_network.ipynb`** builds interactive HTML views (vis-network) of the OC + HTN network, the OC + HTN + SBP network and the RAS genes. Writes `outputs/cutoff_0.4/network_*/`.

## Requirements
Python 3 with `pandas`, `numpy`, `networkx`, `openpyxl`, Jupyter.
