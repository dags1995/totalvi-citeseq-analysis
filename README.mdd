# totalVI CITE-seq analysis

This repository documents a reproducible workflow to analyze CITE-seq data using totalVI from scvi-tools, following the tutorial: https://docs.scvi-tools.org/en/1.3.3/tutorials/notebooks/multimodal/totalVI.html

## Objective

The goal is to build a clean and reproducible workflow for integrating RNA and ADT protein measurements from CITE-seq data using totalVI.

## Workflow

1. Create Conda environment.
2. Load CITE-seq data.
3. Separate RNA and ADT protein features.
4. Prepare AnnData object.
5. Train totalVI model.
6. Extract integrated latent representation.
7. Generate UMAP and clustering.
8. Visualize protein expression on the integrated embedding.


---

## Repository structure

```text
totalvi-citeseq-analysis/
├── README.md
├── environment.yml
├── notebooks/
│   └── 01_totalVI_MALT_test.ipynb
├── scripts/
├── data/
│   ├── raw/
│   └── processed/
├── results/
│   ├── figures/
│   └── models/
└── .gitignore
```


---

## 1. Clone the repository

```bash
git clone https://github.com/dags1995/totalvi-citeseq-analysis.git
cd totalvi-citeseq-analysis
```


---

## 2. Create the Conda environment

Create the environment from the provided `environment.yml` file:

```bash
conda env create -f environment.yml
```

Activate it:

```bash
conda activate totalvi_env
```

Register the environment as a Jupyter kernel:

```bash
python -m ipykernel install --user --name=totalvi_env --display-name "totalvi_env"
```

---

## 3. Check the installation

Run:

```bash
python -c "import scipy; import scanpy as sc; import scvi; import muon; import torch; print('scipy', scipy.__version__); print('scanpy', sc.__version__); print('scvi-tools', scvi.__version__); print('torch', torch.__version__)"
```

If this command runs without errors, the environment is ready.

---
