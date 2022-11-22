# Single-Cell Genomics Analysis with RAPIDS

This repository contains a primary jupyter notebook along with supplemental files analyse single-cell ATAC sequencing data.

We will use RAPIDS pepeline to demonstrate on how to use analyse single-cell ATAC sequencing data. RAPIDS is a suite of open-source Python libraries that can speed up data science workflows. Starting from a single-cell count matrix, RAPIDS libraries can be used to perform data processing, dimensionality reduction, clustering, visualization, and comparison of cell clusters.

Dataset sizes for single-cell genomics studies are increasing, presently reaching millions of cells. With RAPIDS pipeline, it becomes easy to analyze large datasets interactively and in real time, enabling faster scientific discoveries.

## This analysis involves using Jupyter Notebook along with Terminal.
Please give attention to notes before the begining of the each step to understand where the code should be executed.

## Installation
Download the Jupyter Notebook (Jupyter notebook Raw.ipynb) file from the github repository https://github.com/gudalab/scATACseq/find/main along with all the supplemental files and upload this file in your Google Cloud Workbench. Open the Jupyter Notebook (Jupyter notebook Raw.ipynb) file and begin executing codes in each cell by carefully following the instructions.

### conda
All dependencies for these examples can be installed with conda.

After installing the necessary dependencies, you can just run `jupyter lab notebook`.

## Configuration
Unified Virtual Memory (UVM) can be used to [oversubscribe](https://developer.nvidia.com/blog/beyond-gpu-memory-limits-unified-memory-pascal/) your GPU memory so that chunks of data will be automatically offloaded to main memory when necessary. This is a great way to explore data without having to worry about out of memory errors, but it does degrade performance in proportion to the amount of oversubscription. UVM is enabled by default in these examples and can be enabled/disabled in any RAPIDS workflow with the following:
```python
import cupy as cp
import rmm
rmm.reinitialize(managed_memory=True)
cp.cuda.set_allocator(rmm.rmm_cupy_allocator)
```

## Example: Single-cell ATAC-seq of 60K Bone Marrow Cells
<img align="left" width="240" height="200" src="https://github.com/clara-parabricks/rapids-single-cell-examples/blob/master/images/60k_bmmc_dsciATAC.png?raw=true">

We demonstrate the use of RAPIDS pipeline to accelerate the analysis of single-cell ATAC-seq data from 60,495 cells. We start with the peak-cell matrix, then perform peak selection, normalization, dimensionality reduction, clustering, and visualization. We also visualize regulatory activity at marker genes and compute differential peaks.

### Example Dataset
The dataset is taken from [Lareau et al., Nat Biotech 2019](https://www.nature.com/articles/s41587-019-0147-6). We processed the dataset to include only cells in the 'Resting' condition and peaks with nonzero coverage. In this tutorial, you will download (1) the processed peak-cell count matrix for this dataset (.h5ad), (2) the set of nonzero peak names (.npy), and (3) the cell metadata (.csv)

```bash
wget -P <path to this repository>/data https://rapids-single-cell-examples.s3.us-east-2.amazonaws.com/dsci_resting_nonzeropeaks.h5ad; \
wget -P <path to this repository>/data https://rapids-single-cell-examples.s3.us-east-2.amazonaws.com/dsci_resting_peaknames_nonzero.npy; \
wget -P <path to this repository>/data https://rapids-single-cell-examples.s3.us-east-2.amazonaws.com/dsci_resting_cell_metadata.csv
```

And begin processing peak-cell matrix, then perform peak selection, normalization, dimensionality reduction, clustering, and visualization. We also visualize regulatory activity at marker genes and compute differential peaks.

### Example Code
Follow this Jupyter Notebook (Jupyter notebook Raw.ipynb) file from the github repository https://github.com/gudalab/scATACseq/find/main for RAPIDS analysis of this dataset. In order for the notebook to run, the files [rapids_scanpy_funcs.py](notebooks/rapids_scanpy_funcs.py) and [utils.py](notebooks/utils.py) need to be in the same folder as the notebook.

### Acceleration
We report the runtime of these notebooks on various GCP instances below. All runtimes are given in seconds. Acceleration is given in parentheses. Benchmarking was performed on Dec 16, 2020.

| Step                         | CPU <br> n1-standard-16 <br> 16 vCPUs | GPU <br> n1-standard-16 <br> T4 16 GB GPU <br> (Acceleration)  | GPU <br> n1-highmem-8 <br> Tesla V100 16 GB GPU <br> (Acceleration) | GPU <br> a2-highgpu-1g <br> Tesla A100 40GB GPU <br> (Acceleration) |
|------------------------------|-------------------------|----------------------------|-------------------|-------------|
| PCA                          | 149                     |  146 (1x)                  |  68  (2.2x)       |  54  (2.8x) |
| KNN                          | 19.7                    | 19.3 (1x)                  | 5.8 (3.4x)        |  5.3 (3.7x) |
| UMAP                         | 69                      |  1.1 (63x)                 | 0.71 (97x)        | 0.69 (100x) |
| Louvain clustering           | 13.1                    | 0.13 (100x)                | 0.11 (119x)       | 0.11 (119x) |
| Leiden clustering            | 15.7                    | 0.08 (196x)                | 0.07 (224x)       | 0.06 (262x) |
| t-SNE                        | 258                     |  3.2 (81x)                 | 1.5  (172x)       |  2.2 (117x) |
| Differential Peak Analysis   | 135                     |   59 (2.3x)                | 14.8 (9x)         | 10.4  (13x) |
| End-to-end notebook run      | 682                     |    263                     | 107               |   92        |
| Price ($/hr)                 | 0.760                   | 1.110                      | 2.953             | 3.673       |
| Total cost ($)               | 0.144                   |  0.081                     | 0.110             |    0.094    |
