# BTS Atlas: Brain Transcriptome Single-cell Atlas

Analysis code for the Brain Transcriptome Single-cell (BTS) atlas, an integrated single-cell atlas of the human brain from fetal development to adulthood.

> Kim S\*, Lee J\*, Koh IG, Ji J, Kim HJ, Kim E, Park J, Park J-E, An J-Y. An integrative single-cell atlas for exploring the cellular and temporal specificity of genes related to neurological disorders during human brain development. *Experimental & Molecular Medicine* 56, 2271–2282 (2024). https://doi.org/10.1038/s12276-024-01328-6
>
> \* These authors contributed equally.

## Data

The processed atlas is on Zenodo under a CC BY 4.0 license: https://doi.org/10.5281/zenodo.10939706 (all versions).

| File | Content |
|---|---|
| `BTS_atlas.h5ad` | AnnData object of the integrated atlas |
| `BTS_atlas_seurat.rds` | Seurat object of the same atlas (from v1.0.1) |
| `BTS_atlas_CellTypist_model.pkl` | CellTypist model for label transfer |
| `RiskGene_multipanel.zip` | Expression plots of neurological disorder risk genes |

The raw data come from the public studies listed under [Input datasets](#input-datasets). This repository does not contain expression data.

## Repository layout

The notebooks are Jupyter notebooks (Python, scanpy) unless noted. Run the folders in numeric order.

| Folder | Step |
|---|---|
| `1_Preprocessing/` | One notebook per public dataset. Each reads the downloaded matrices and metadata, harmonises the metadata columns, and writes one `.h5ad` file. |
| `2_Integration/` | Merges the datasets, applies quality control (minimum genes, mitochondrial fraction, Scrublet doublets) and cell-number balancing, integrates the samples with scVI (`sampleID` as the batch key), and does the first clustering. |
| `3_Refinement/` | Leiden clustering, cell-type annotation and metadata corrections. The notebooks are named by date (MMDD) and were run in this order: `clustering_analysis_1215`, `_0215`, `_0306`, `_final_0327`. `_0216`, `_0307`, `_0310` and `Leiden_testing` are checks and side analyses. |
| `4_Downstream/` | Analyses of the annotated atlas: pseudobulk profiles, pathway module scores and heatmaps, transcription factor activity (decoupler with CollecTRI/DoRothEA; SEACells with pySCENIC), a GLM test of *ESR1* expression, and trajectory analysis with scVI and Palantir in `trajectory_analysis/`. `Dorothea_TF_Target.R` links significant TF activities to cluster marker genes. |
| `Archived/` | Earlier analyses that are not part of the published results (CellTypist modelling, neural crest re-annotation, sex-hormone gene checks). Kept for reference. |

`4_Downstream/trajectory_analysis/RG_neurob_neuron_20_under_obs_data/` contains the per-cell lineage and pseudotime tables of the radial glia to neuron trajectories (fetal to adolescent samples), which `4_Downstream/Pathway_Heatmap_0324.ipynb` reads.

## Input datasets

| Notebook | Study | Source |
|---|---|---|
| `preprocess_AM1` | Allen Institute, human primary motor cortex (10x) | https://portal.brain-map.org/atlases-and-data/rnaseq/human-m1-10x |
| `preprocess_braun` | Braun et al., *Science* 2023 | https://www.biorxiv.org/content/10.1101/2022.10.24.513487v1 |
| `preprocess_cameron` | Cameron et al. 2023 | https://figshare.com/articles/dataset/11629311 |
| `preprocess_hardwick` | Hardwick et al., *Nat Biotechnol* 2022 | GEO GSE178175 |
| `preprocess_herring` | Herring et al., *Cell* 2022 | GEO GSE168408; HCA project 77dedd59-1376-4887-9bca-dc42b56d5b7a |
| `preprocess_morabito` | Morabito et al., *Nat Genet* 2021 | GEO GSE174367 |
| `preprocess_turecki` | Nagy et al., *Nat Neurosci* 2020 | GEO GSE144136 |
| `preprocess_zhangPD` | Zhu et al. (Zhang lab, Parkinson's disease controls) | GEO GSE202210; HCA project 9a23ac2d-93dd-4bac-9bb8-040e6426db9d |

Only samples from donors without a neurological diagnosis are kept. `2_Integration/1.create_input_file_v1` also reads further datasets (Allen multiple cortical areas, Cao 2020, Eze, ROSMAP, Trevino 2021) whose preprocessing is not in this repository; the filters in `2_Integration/2.Dataset_subsetting` remove them from the final atlas. The gene-symbol harmonisation uses the NCBI `Homo_sapiens.gene_info.gz` file and the HGNC `non_alt_loci_set` table (release 2023-03-28).

## Running the notebooks

The notebooks were run on a Linux server. Set the input locations with environment variables before you start Jupyter; the first code cell of each notebook reads them.

| Variable | Default | Content |
|---|---|---|
| `BTS_RAW_DIR` | `data/raw` | Downloaded public datasets and the per-dataset `.h5ad` files (`anndata/`) |
| `BTS_WORK_DIR` | `.` | Working directory of the integrated atlas |
| `BTS_SCENIC_DIR` | `data/scenic` | pySCENIC ranking databases, motif table and TF list |
| `BTS_TRAJ_DIR` | `data/trajectory` | Trajectory subsets and results |

Other notebooks read and write paths relative to the folder they run in (for example `Data/Output_240308_adata_scvi_random_sampleID.h5ad`).

`envs/BrainAtlas.yml` is the conda environment of most notebooks (Python 3.8, scanpy 1.8.2, anndata 0.8.0, pandas 1.1.4). The scVI, Palantir, decoupler, pySCENIC and SEACells notebooks ran in separate environments whose exact versions were not recorded.

## Reproducibility notes

- The notebooks are the analysis records of the study and were run interactively. Some cells were run out of order, and some steps between saved files (for example between `Output_240216`, `Output_240306` and `Output_240308`) were done in sessions that are not in this repository.
- Large intermediate `.h5ad` files are not included. To start from the final atlas, download `BTS_atlas.h5ad` from Zenodo.
- Some metadata tables that the notebooks read (`samplemeta.csv`, `Table/metadata_adata_edit.csv`, files under `Tables/` and `Resources/`) are not in the repository. The sample metadata are in Supplementary Table 1 of the paper.

## Known issues

A code review before this release found the issues below. The notebooks are kept as they were run for the paper; the fixes are given here so that a re-run can apply them.

| Notebook | Issue | Fix |
|---|---|---|
| `4_Downstream/Pathway_Heatmap_0324` (cells 6–7) | `adata_sub.obs = meta` replaces obs by position. `meta` is sorted by pseudotime and `adata_sub` keeps the order of the atlas, so the lineage and pseudotime-tercile labels can be attached to other cells. | `adata_sub = adata[meta.index].copy(); adata_sub.obs = meta.loc[adata_sub.obs_names]` |
| `4_Downstream/trajectory_analysis/RG_neurob_neuron_20_under_forshare` (cells 51–53, 62, 64, 66); `Archived/Figures_For_Share` (cell 26) | The `pts` and `pts_rest` columns of `rank_genes_groups` are indexed by gene in var order, but `names` is in rank order. Building one DataFrame from both attaches the fractions to other genes. | `sc.get.rank_genes_groups_df(adata, group=g)` |
| `4_Downstream/Dorothea_TF_Target.R` | `read.csv` reads the index column of the DEG table as column `X`, so the four-column blocks start one column early. | `read.csv(..., row.names = 1)` and select columns by name |
| `1_Preprocessing/preprocess_herring` (cell 3), `preprocess_braun` (cell 8) | The code adds two weeks to Herring gestational ages and does not convert Braun ages to days. The ages in the atlas come from a corrected metadata table (`Table/metadata_adata_edit.csv`, read in `3_Refinement/clustering_analysis_1215`) and do not follow from this code. | Use the `Age` column of `BTS_atlas.h5ad` |
| `4_Downstream/TF_doRothEA`, `TF_SEACells_by_Cluster`, `TF_pySCENIC`, `EffectTest_GLM_forshare` | These notebooks read an earlier object (`Output_231123`/`231215`) whose `Leiden` column numbers clusters differently from the final atlas (`Output_240308`/`240327`). | Map clusters through `cluster_annotated`, not the cluster number |
| `4_Downstream/EffectTest_GLM_forshare` (cells 4, 25) | The Poisson GLM is fitted to log-normalised values with a library-size offset. | Fit `layers["counts"]` |
| `2_Integration/2.Dataset_subsetting` (cells 18–32) | Cells 26–32 build the balanced object from `adata`, not from the quality-controlled `adata2`; the saved outputs show that `adata = adata2` was run in the session. Cell 28 draws the 50 Braun samples with replacement. | Add `adata = adata2` before cell 24; use `random.sample` |
| `3_Refinement/clustering_analysis_final_0327` (cell 20) | The original `pd.merge(..., on='sampleID', left_index=True)` fails in pandas 1.2 and later. | Replaced in this release by a `map` on `sampleID` |
| `1_Preprocessing/preprocess_hardwick`, `preprocess_zhangPD` (cell 2) | The folder-creation cell took the sample name from a fixed path depth. | Replaced in this release by `os.path.basename` |

## License

The code is released under the MIT License (see `LICENSE`). The data on Zenodo are released under CC BY 4.0.

## Contact

Please open an issue for questions about the code. For other questions, contact the corresponding author of the paper (Joon-Yong An, Korea University).
