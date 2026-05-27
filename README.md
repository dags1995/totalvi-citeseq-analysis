# Longitudinal totalVI CITE-seq analysis

This repository documents a reproducible workflow to analyze longitudinal CITE-seq data using totalVI from scvi-tools.

The workflow is based on the official totalVI tutorials from scvi-tools:
https://docs.scvi-tools.org/en/stable/tutorials/notebooks/use_cases/preprocessing.html#cite-seq
https://docs.scvi-tools.org/en/1.3.3/tutorials/notebooks/multimodal/totalVI.html

## Objective

The goal of this project is to adapt a totalVI-based CITE-seq workflow to a longitudinal experimental design with four timepoints and one sample per timepoint.

At the current development stage, the workflow is tested using the two tutorial CITE-seq samples. The code structure is prepared for four timepoints, but the sections corresponding to T3 and T4 are kept as placeholders until the real data are available.


---

## Workflow

1. Create Conda environment.
2. Load CITE-seq samples.
3. Assign sample-level metadata:
   - `sample_id`
   - `timepoint`
   - `batch`
4. Pre-process and merge the samples into a single multi-sample object.
5. Format the integrated object for totalVI.
6. Train the totalVI model using batch information.
7. Extract the integrated RNA + ADT latent representation.
8. Generate UMAP and clustering.
9. Visualize the integrated embedding by:
   - batch
   - timepoint
   - cluster
   - protein expression
10. Compare cellular populations and molecular profiles across timepoints.

---

## Repository structure

```text
totalvi-citeseq-analysis/
├── README.md
├── environment.yml
├── notebooks/
│   ├── 01_totalVI_CITEseq_preprocessing.ipynb
│   ├── 02_totalVI_model_training.ipynb
│   └── 04_totalVI_latent_visualization.ipynb
├── scripts/
├── data/
│   ├── raw/
│   │   ├── T1/
│   │   ├── T2/
│   │   ├── T3/        # placeholder
│   │   └── T4/        # placeholder
│   ├── processed/
│   │   ├── T1/
│   │   ├── T2/
│   │   ├── T3/        # placeholder
│   │   └── T4/        # placeholder
│   └── integrated/
└── results/
    ├── figures/
    ├── models/
    ├── qc/
    └── longitudinal_comparison/

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

To check the installation, run:

```bash
python -c "import scipy; import scanpy as sc; import scvi; import muon; import torch; print('scipy', scipy.__version__); print('scanpy', sc.__version__); print('scvi-tools', scvi.__version__); print('torch', torch.__version__)"
```

If this command runs without errors, the environment is ready.

---

## 2. Data

This project is designed for a longitudinal CITE-seq study with four timepoints and one sample per timepoint.

The final expected structure is:

```text
data/raw/
├── T1/
├── T2/
├── T3/
└── T4/
```

At the current development stage, only two tutorial samples are used:

```text
T1 = PBMC 10k CITE-seq sample
T2 = PBMC 5k CITE-seq sample
```

The `T3` and `T4` folders are kept as placeholders for future real longitudinal samples.

The tutorial data are downloaded from 10x Genomics using `pooch`, following the same logic used in the original totalVI/scvi-tools preprocessing workflow. The downloaded files correspond to the original filtered 10x Genomics feature-barcode matrices, not to a preprocessed `.h5mu` object.

The active input files are stored as:

```text
data/raw/
├── T1/
│   └── CITE-seq_pbmc_10k.untar/
│       └── filtered_feature_bc_matrix/
│           ├── barcodes.tsv.gz
│           ├── features.tsv.gz
│           └── matrix.mtx.gz
├── T2/
│   └── CITE-seq_pbmc_5k.untar/
│       └── filtered_feature_bc_matrix/
│           ├── barcodes.tsv.gz
│           ├── features.tsv.gz
│           └── matrix.mtx.gz
├── T3/
│   └── .gitkeep
└── T4/
    └── .gitkeep
```

The corresponding source datasets are:

```text
T1: pbmc_10k_protein_v3_filtered_feature_bc_matrix.tar.gz
T2: 5k_pbmc_protein_v3_filtered_feature_bc_matrix.tar.gz
```

The files are not tracked by Git. Only the folder structure is kept using `.gitkeep` files.

---

## 3. Sample metadata

Each sample is assigned metadata before integration.

The minimum required metadata are:

```text
sample_id
timepoint
batch
```

For the tutorial test run, the active samples are:

```text
sample_id     timepoint     batch
sample_T1     T1            batch_1
sample_T2     T2            batch_2
```

The future samples are prepared but not executed:

```text
# sample_T3   T3            batch_3
# sample_T4   T4            batch_4
```

This structure allows the same workflow to be used first with the tutorial data and later with the full longitudinal dataset.

---

## 4. First notebook: CITE-seq preprocessing

The first notebook of the workflow is:

```text
notebooks/01_totalVI_CITEseq_preprocessing.ipynb
```

The objective of this notebook is to load the original 10x Genomics CITE-seq matrices for each active timepoint and perform the preprocessing steps required before totalVI model training.

The notebook checks that each dataset contains both RNA and ADT protein features:

- `Gene Expression`: RNA counts
- `Antibody Capture`: ADT protein counts

Each sample is processed individually before integration.

For each active sample, the notebook:

1. loads the 10x Genomics matrix using `scanpy.read_10x_mtx`;
2. separates RNA and ADT protein features;
3. stores raw RNA counts;
4. assigns sample-level metadata;
5. saves the preprocessed object.

The expected output files are:

```text
data/processed/T1/t1_citeseq_mudata.h5mu
data/processed/T2/t2_citeseq_mudata.h5mu
# data/processed/T3/t3_citeseq_mudata.h5mu
# data/processed/T4/t4_citeseq_mudata.h5mu
```

T3 and T4 are included as placeholders but are not executed in the current tutorial-based test.

---

## 5. Second notebook: totalVI model training

The second notebook of the workflow is:

```text
notebooks/02_totalVI_model_training.ipynb
```

This notebook loads the preprocessed MuData object generated in the first notebook:

```text
data/processed/citeseq_longitudinal_preprocessed.h5mu
```

The object already contains the active tutorial samples T1 and T2 merged into a single MuData object, with the following modalities:

- `rna`: complete RNA modality
- `prot`: shared ADT protein modality
- `rna_subset`: RNA modality restricted to highly variable genes

The goal of this notebook is to train a totalVI model using:

- RNA counts from `mdata.mod["rna_subset"].layers["counts"]`
- ADT protein data from `mdata.mod["prot"].X`
- batch information from `mdata.mod["rna_subset"].obs["batch"]`

The notebook follows the `setup_mudata()` workflow from scvi-tools and trains a totalVI model directly from the MuData object.

After training, the notebook generates a training history plot based on the negative ELBO values for the training and validation sets.

The main outputs are saved as:

```text
data/processed/citeseq_longitudinal_totalvi_trained.h5mu
results/models/totalvi_longitudinal_model/
results/figures/totalvi_training_elbo.png
```

The trained MuData object includes the totalVI latent representation stored in:

```text
mdata.mod["rna_subset"].obsm["X_totalVI"]
```

At the current stage, the trained model uses only T1 and T2 from the tutorial dataset. The same structure is intended to support future inclusion of T3 and T4 samples.

---


