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
│   ├── 01_totalVI_MALT_CITEseq_preprocessing.ipynb
│   └── 02_totalVI_MALT_model_training.ipynb
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

## 1. Create the Conda environment

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

To check the installation run:

```bash
python -c "import scipy; import scanpy as sc; import scvi; import muon; import torch; print('scipy', scipy.__version__); print('scanpy', sc.__version__); print('scvi-tools', scvi.__version__); print('torch', torch.__version__)"
```

If this command runs without errors, the environment is ready.

---


## 2. Data 

For this example, we will use the CITE-seq data provided in the `sciPENN_codes` repository (https://github.com/jlakkis/sciPENN_codes). The original data files are available from the Box link provided by the sciPENN authors:

https://upenn.app.box.com/s/1p1f1gblge3rqgk97ztr4daagt4fsue5

For this first `totalVI` workflow, we will use the following 10x Genomics CITE-seq file: 

`malt_10k_protein_v3_filtered_feature_bc_matrix.h5`

This file should be placed in:

`data/raw/`


---

## 3. First notebook: load CITE-seq data

The first notebook of the workflow is:

`notebooks/01_totalVI_MALT_CITEseq_preprocessing.ipynb`

The first goal is to load the 10x Genomics CITE-seq file and verify that it contains both RNA and ADT protein features.

This step will check the feature types present in the dataset:

- `Gene Expression`: RNA counts
- `Antibody Capture`: ADT protein counts


---

## 4. Preprocessed MuData object

The first notebook separates the CITE-seq data into two modalities:

- `mdata.mod["rna"]`: RNA count matrix
- `mdata.mod["prot"]`: ADT protein count matrix

Raw RNA counts are stored in:

- `mdata.mod["rna"].layers["counts"]`

The preprocessed multimodal object is saved locally as:

- `data/processed/malt_citeseq_mudata.h5mu`


---

## 5. Second notebook: totalVI model training

The second notebook of the workflow is:

`notebooks/02_totalVI_MALT_model_training.ipynb`

This notebook uses the preprocessed MuData object generated in the first notebook:

`data/processed/malt_citeseq_mudata.h5mu`

The RNA and ADT protein modalities are converted into the AnnData format expected by `totalVI`.

The model is trained for 100 epochs and an integrated RNA + ADT latent representation is extracted:

`adata.obsm["X_totalVI"]`

The resulting latent space has shape:

`8412 cells × 20 latent dimensions`

The trained outputs are saved locally as:

- `data/processed/malt_totalvi_trained.h5ad`
- `results/models/totalvi_malt_model/`


---

## 6. Third notebook: latent space visualization

The third notebook of the workflow will be:

`notebooks/03_totalVI_MALT_latent_visualization.ipynb`

This notebook will use the trained AnnData object generated in the second notebook:

`data/processed/malt_totalvi_trained.h5ad`

The objective of this notebook is to:

1. load the trained AnnData object;
2. use the integrated latent representation stored in `adata.obsm["X_totalVI"]`;
3. compute the neighborhood graph using the totalVI latent space;
4. generate a UMAP embedding;
5. perform Leiden clustering;
6. visualize the integrated latent space.

The main downstream analysis will use:

`adata.obsm["X_totalVI"]`

This representation contains the integrated RNA + ADT protein information learned by `totalVI`.