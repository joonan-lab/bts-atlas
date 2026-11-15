# BTS atlas releases

This file lists the releases of the BTS atlas that lab projects use. Each release has an ID. Write the ID (for example `BTS-v2`) in the input table of an analysis or a manuscript, and do not mix two releases in one analysis. When a new release is made, add a row here first.

| Release ID | File | Cells, features | Gene names | Checksum | Status |
|---|---|---|---|---|---|
| `BTS-v1` | `BTS_atlas.h5ad` | 393,060 cells, 49,133 features | Original symbols in `var/new_gene` (HGNC 2023-03-28, NCBI `gene_info`) | MD5 `a79cacfcbe34f99dadf59a1b693178e8` (Zenodo) | Published. Zenodo [10939706](https://doi.org/10.5281/zenodo.10939706): v1.0.0 (2024-06-17), v1.0.1 (2024-11-18, adds `BTS_atlas_seurat.rds`). The h5ad file is the same in both versions |
| `BTS-v2` | `BTS_atlas.modern_gencode_hgnc.scvi_hvg5000.h5ad` | 393,060 cells, 58,219 features | Rebuilt from the source datasets with GENCODE v49 and HGNC approved symbols; `var` has `gene_symbol`, `stable_gene_id`, `hgnc_id` | SHA-256 `bd9dfe98d1824a59f55dfab338e22459917c772aab5ef4743a01d746318f87e1` | Lab release (2026-06-14), not on Zenodo |
| `BTS-v2-compat` | `BTS_atlas.v2.scvi_hvg5000.h5ad` | 393,060 cells, 49,133 features | `BTS-v1` features with 371 names corrected in `var/new_gene` | SHA-256 `7f0ebb804af0544f41d245b763163504b2632d071055db205a8848afb0a51551` | Lab release (2026-06-14), not on Zenodo |

## BTS-v1

The atlas of Kim et al. 2024. The notebooks in this repository build it. See the [README](../README.md).

Known issue: when the gene symbols of the source datasets were harmonised, some alias and previous symbols collapsed onto one approved symbol, and AnnData made the names unique with a `-1` suffix. In `BTS-v1`, 565 features (535 symbols) carry such a suffix. Most are true duplicates, but some are other genes. For example, the feature `TCF7L2-1` is *TCF4* and the feature `TCF7L1-1` is *TCF3*; *TCF4* and *TCF3* have no feature of their own in `BTS-v1`. The source data are not the cause: the Herring et al. 2022 matrix has *TCF4* and *TCF7L2* as separate genes. If you use a gene with a `-1` feature in `BTS-v1`, check its name in `BTS-v2`.

## BTS-v2

Same cells and the same cell annotations (`obs`) as `BTS-v1`.

- `X` is the raw count matrix, rebuilt from the processed matrices of the eight source datasets.
- Features are keyed by gene (`gene_key`) and collapsed when several source features map to one gene (`collapsed_from_n_features`).
- *TCF3*, *TCF4*, *TCF7L1* and *TCF7L2* are present as separate genes. No `TCF7L1-1` or `TCF7L2-1` feature remains.
- A new scVI integration was added: 5,000 highly variable genes (`cell_ranger` flavour, batch key `Dataset`), scVI on `counts` with batch key `Dataset` and categorical covariate `Assay`, 10 latent dimensions, 20 epochs, seed 0 (scanpy 1.10.4, anndata 0.12.16, scvi-tools 1.2.2). Results are in `obsm/X_scVI_hvg5000_corrected`, `obsm/X_umap_hvg5000_corrected` and `obs/leiden_0.5_scvi_hvg5000` (43 clusters). The `BTS-v1` embedding is kept in `obsm/X_scVI_sampleID` and `obsm/X_umap`.

Used by (2026-11-15): `human-brain-geometry-dev` (BTSdiff), `asd-oligogenic-geometry-dev`.

## BTS-v2-compat

Use this release when an analysis must keep the `BTS-v1` feature set and matrix (log counts) but needs the corrected names.

- Only `var/new_gene` changed. A name was corrected when the expression of the `BTS-v1` feature matched the rebuilt gene with correlation >= 0.8 (high confidence) or >= 0.6 (medium confidence), and with a margin >= 0.2 over the next candidate when two candidates were comparable.
- 371 features were renamed. 86 candidates with low or ambiguous evidence kept their `BTS-v1` name.
- The differential-expression results computed with the old names (`uns/wilcoxon`, `uns/ttest_pts`) were removed.
- It has the same scVI embedding and Leiden clusters as `BTS-v2`.

Used by (2026-11-15): no lab repository found.

## Other atlases

The second-generation brain atlas (`BTS2-2026`) is a separate atlas built from other datasets. It is not a release of this repository.
