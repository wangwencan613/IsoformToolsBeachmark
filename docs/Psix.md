# Psix Single-Cell Alternative Splicing Analysis Pipeline

This document records the complete workflow for single-cell alternative splicing analysis using the Psix tool, including environment preparation, input file requirements, and step-by-step running commands for both **droplet-based (e.g., 10X Genomics)** and **well-based (e.g., Smart-seq2)** platforms.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the Psix conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate Psix
```

## 2. Official Documentation Reference

Detailed Psix user manual and source code: https://github.com/lareaulab/psix

## 3. For Droplet-based Platforms (e.g., 10X Genomics)

### 3.1 Input File Preparation

The following files need to be prepared in advance to ensure the analysis runs smoothly. **Note**: The authors recommend mapping raw scRNA-seq reads using **STAR version ≥ 2.5.3a**.

#### Mandatory Inputs
- Splice junction files: Either `SJ.out.tab` files (named as `cellID.SJ.out.tab` and stored in the same folder) or STARsolo SJ features
- Cassette exon annotation: A table specifying the genomic location (chromosome, start, end) of splice junctions
- Low-dimensional cell space: A metric to quantify phenotypic similarity between single cells (generated via PCA on standardized TPM data, see code below)

#### Generate Low-Dimensional Cell Space (PCA on TPM Data)
```python
# Import required libraries
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

# Load TPM matrix
tpm = pd.read_csv('/home/isofrom/psix/10x/KD6.tpm', sep='\t', index_col=0).T

# Standardize TPM data
scaler = StandardScaler()
standardized_tpm = scaler.fit_transform(tpm)

# Perform PCA to reduce to 2 dimensions
pca = PCA(n_components=2)
principal_components = pca.fit_transform(standardized_tpm)

# Format and save low-dimensional embeddings
rd = pd.DataFrame()
for i in range(2):
    rd['PC_' + str(i+1)] = principal_components.T[i]
rd[['PC_1', 'PC_2']].to_csv('/home/isofrom/psix/10x/pc2_rd.tab.gz', sep='\t', index=True, header=True)
```

### 3.2 Running Commands

#### 3.2.1 Create a Psix Object
Psix supports two input modes for droplet-based data: **`SJ.out.tab` files** and **STARsolo SJ features**. Run only one mode based on your data format.

##### Mode 1: From `cellID.SJ.out.tab` Files
```python
import psix

# Initialize Psix object
psix_object = psix.Psix()

# Convert junctions to PSI (Percent Spliced In) values
psix_object.junctions2psi(
        sj_dir='/home/isofrom/psix/10x/KD6.SJ',
        intron_file='/home/isofrom/psix/hg38_introns.tab.gz',
        save_files_in='/home/isofrom/psix/10x/psix_output/',
        tenX=True
)

# Load the generated PSI and mRNA tables into the Psix object
psix_object = psix.Psix(
    psi_table='/home/isofrom/psix/10x/psix_output/psi.tab.gz',
    mrna_table='/home/isofrom/psix/10x/psix_output/mrna.tab.gz'
)
```

##### Mode 2: From STARsolo SJ Features
```python
import psix

# Initialize Psix object
psix_object = psix.Psix()

# Convert STARsolo SJ features to PSI values
psix_object.junctions2psi(
        sj_dir='/path/to/STARsolo_output/Solo.out/SJ/raw/',
        intron_file='/path/to/cassette_exon_annotation.tab',
        save_files_in='psix_output/',
        tenX=True,
        solo=True
    )

# Load the generated PSI and mRNA tables into the Psix object
psix_object = psix.Psix(
    psi_table='psix_output/psi.tab.gz',
    mrna_table='psix_output/mrna.tab.gz'
)
```

#### 3.2.2 Get Cell-State Associated Exons
Identify exons associated with cell state using the precomputed low-dimensional cell space:
```python
# Run Psix to associate exons with cell state
psix_object.run_psix(
    latent='/home/isofrom/psix/10x/pc2_rd.tab.gz',
    n_neighbors=1
)
# Speed up by adding `n_jobs=t` (e.g., n_jobs=8) to run in parallel
```

#### 3.2.3 Compute Modules of Correlated Exons
Cluster exons into modules based on their expression correlation:
```python
# Compute correlated exon modules and generate visualization
psix_object.compute_modules(plot=True)
```

## 4. For Well-based Platforms (e.g., Smart-seq2)

### 4.1 Input File Preparation

The following files need to be prepared in advance for well-based platform analysis:
- Splice junction files: Either `SJ.out.tab` files or STARsolo SJ features
- Cassette exon annotation: A table specifying the genomic location of splice junctions
- Low-dimensional cell space: A metric to quantify phenotypic similarity between single cells (generated via PCA on TPM data, same as droplet-based)
- TPM matrix: Gene expression matrix (**only required for Smart-seq2**)

### 4.2 Running Commands

#### 4.2.1 Create a Psix Object
```python
import pandas as pd
import psix

# Load list of neurons to analyze
neuron_cells = pd.read_csv(
    '/home/isofrom/psix/demo/sample2_pc3_rd.tab.gz',
    sep='\t', index_col=0
).index

# Initialize Psix object
psix_object = psix.Psix()

# Convert junctions to PSI values (includes TPM matrix for Smart-seq2)
psix_object.junctions2psi(
        sj_dir='/home/isofrom/psix/demo/SJ/',
        intron_file='/home/isofrom/psix/demo/mm10_introns.tab.gz',
        tpm_file='/home/isofrom/psix/demo/sample2_tpm.tab.gz',
        cell_list=neuron_cells,
        save_files_in='/home/isofrom/psix/demo/psix_output'
)

# Load the generated PSI and mRNA tables into the Psix object
psix_object = psix.Psix(
    psi_table='/home/isofrom/psix/demo/psix_output/psi.tab.gz',
    mrna_table='/home/isofrom/psix/demo/psix_output/mrna.tab.gz'
)
```

#### 4.2.2 Get Cell-State Associated Exons
```python
# Run Psix to associate exons with cell state
psix_object.run_psix(
    latent='/home/isofrom/psix/demo/sample2_pc3_rd.tab.gz',
    n_neighbors=1
)
# Speed up by adding `n_jobs=t` (e.g., n_jobs=8) to run in parallel
```

#### 4.2.3 Compute Modules of Correlated Exons
```python
# Compute correlated exon modules and generate visualization
psix_object.compute_modules(plot=True)
```

## Notes

1. **Path Replacement**: All paths in this document are example paths. Replace them with your actual file paths to avoid execution errors.
2. **Environment Confirmation**: Ensure the `Psix` conda environment is successfully activated before running any commands.
3. **STAR Version Requirement**: For droplet-based data, use **STAR ≥ 2.5.3a** for read mapping to generate valid SJ files/STARsolo outputs.
4. **Input Format**:
   - `SJ.out.tab` files must be named `cellID.SJ.out.tab` and stored in a single directory.
   - The low-dimensional cell space file must be a tab-separated table with PCA embeddings (PC_1, PC_2, etc.).
   - For Smart-seq2, the TPM matrix must be a tab-separated table with genes as rows and cells as columns.
5. **Parallelization**: Speed up `run_psix` by specifying `n_jobs=t` (e.g., `n_jobs=16`) to utilize multiple CPU cores.
6. **Parameter Tuning**: Adjust `n_neighbors` (in `run_psix`) and `plot` (in `compute_modules`) based on your data characteristics and visualization needs.
7. **Platform Distinction**: Use the dedicated `tenX=True` flag for droplet-based data and include the `tpm_file` argument only for Smart-seq2 (well-based) data.
8. **Result Saving**: The Psix object and intermediate files (PSI/mRNA tables) are automatically saved to the specified `save_files_in` directory for downstream analysis.