# SpliZ Splicing Analysis Pipeline (Nextflow & Snakemake)

This document records the complete workflow for splicing analysis using the SpliZ tool, including environment preparation, input file requirements, step-by-step running commands (Nextflow recommended, Snakemake not recommended), code debugging tips, and key notes to ensure smooth execution.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the SpliZ conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate SpliZ
```

## 2. Official Documentation Reference

- SpliZ official GitHub repository: https://github.com/salzman-lab/SpliZ/

- SpliZ pipeline guide: https://github.com/juliaolivieri/SpliZ_pipeline

- Troubleshooting reference: https://github.com/salzman-lab/SpliZ/issues/9

## 3. Code Debugging Tip

When encountering the error *"The truth value of a DataFrame is ambiguous"*, resolve it by adding `.to_dict()` to the relevant error line in the code.

## 4. Input File Preparation

Input requirements vary based on whether the data is from SICILIAN or non-SICILIAN sources. Detailed requirements for each scenario are provided below:

### 4.1 Inputs from SICILIAN

#### 4.1.1 Input Format Requirements

- Use the input file from SICILIAN and add three additional columns: `tissue`, `compartment`, and `free_annotation`.

- Rename the column `gene_count_per_cell_no_filt` to `called` (critical for pipeline compatibility).

- Warning: The `tissue` and `compartment` columns must each contain at least 2 distinct values; otherwise, a "divided by zero" error will occur during processing.

#### 4.1.2 Input Examples

- **Example 1**: Processed SICILIAN input file with the three required additional columns and renamed `called` column.

- **Example 2**: Use published data from the SpliZ manuscript, e.g., HLCA_P3 located at `/home/jy/SICILIAN/SpliZ_pipeline/data/HLCA_P3.pq`.

### 4.2 Non-SICILIAN Inputs (SICILIAN = false, BAM Format)

#### 4.2.1 Samplesheet Requirements

The samplesheet must be in comma-separated value (CSV) format **without a header**. The `sampleID` must be a unique identifier for each BAM file entry. Format varies by platform:

- **Droplet-based platform (e.g., 10X Genomics)**: Samplesheet must have 2 columns: `sampleID` and path to the BAM file.

- **Well-based platform (e.g., Smart-seq2)**: Samplesheet must have 3 columns: `sampleID`, path to read 1 BAM file, and path to read 2 BAM file.

#### 4.2.2 Generate BAM Files (Same as SICILIAN)

First, build the STAR genome index, then generate the annotator. Use the following commands:

```bash
# Build STAR genome index
STAR --runThreadN 4 \
--runMode genomeGenerate \
--genomeDir /data/bioinf/ref/gencode/hg38_index \
--genomeFastaFiles /data/bioinf/ref/gencode/GRCh38.p14.genome.fa \
--sjdbGTFfile /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf \
--sjdbOverhang [adjust to match read length]

# Generate annotator
python3 create_annotator.py -a annotation_name -g /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf
```

Note: Replace `[adjust to match read length]` with a value suitable for your sequencing read length (typically read length - 1).

## 5. Running Commands

SpliZ supports two running modes: Nextflow (recommended) and Snakemake (not recommended). Run the commands in sequence according to your chosen mode.

### 5.1 Usage of SpliZ (Nextflow, Recommended)

#### 5.1.1 For SICILIAN Inputs (--SICILIAN true)

Run the Nextflow pipeline with the prepared SICILIAN input file. Key parameters are explained below the command:

```bash
cd /home/SICILIAN/SpliZ/
nextflow run salzmanlab/spliz -r main \
-latest \
--dataname "HLCA" \
--input_file /home/SICILIAN/SpliZ_pipeline/data/HLCA_P3.pq \
--SICILIAN true \
--grouping_level_2 "compartment" \
--grouping_level_1 "tissue" \
--libraryType "10X" \
--run_analysis true \
--pin_S 0.1 \
--pin_z 0.0 \
--bounds 5 \
--light false \
--n_perms 100 \
--svd_type "normdonor"
```

#### 5.1.2 For Non-SICILIAN Inputs (--SICILIAN false)

Ensure the samplesheet is prepared according to platform requirements (Section 4.2.1), then run the Nextflow pipeline with `--SICILIAN false` (adjust other parameters as needed):

```bash
cd /home/SICILIAN/SpliZ/
nextflow run salzmanlab/spliz -r main \
-latest \
--dataname "SampleName" \
--input_file /path/to/samplesheet.csv \
--SICILIAN false \
--grouping_level_2 "compartment" \
--grouping_level_1 "tissue" \
--libraryType "[10X/Smart-seq2]" \
--run_analysis true \
--pin_S 0.1 \
--pin_z 0.0 \
--bounds 5 \
--light false \
--n_perms 100 \
--svd_type "normdonor"
```

### 5.2 Usage of SpliZ_pipeline (Snakemake, Not Recommended)

#### 5.2.1 Input Preparation

- Input files are the same as those for SpliZ (Section 4).

- Place the data file in`/home/SICILIAN/SpliZ_pipeline/data/`.

- Place the configuration file in `/home/SICILIAN/SpliZ_pipeline/`.

#### 5.2.2 Running Command

```bash
cd /home/SICILIAN/SpliZ_pipeline
snakemake -p --config datasets="HLCA_P3" --restart-times 0
```

Note: Replace `HLCA_P3` with your actual dataset name. The `--restart-times 0` parameter means the pipeline will not restart if it fails.

### Key Parameter Explanations (Nextflow)

- -r main: Use the main branch of the SpliZ Nextflow pipeline.

- --latest: Use the latest version of the pipeline.

- --dataname: Name of the dataset (for output file labeling).

- --input_file: Path to the input file (SICILIAN .pq file or non-SICILIAN CSV samplesheet).

- --SICILIAN: Boolean (true/false) indicating whether the input is from SICILIAN.

- --grouping_level_1/2: Grouping variables (e.g., `tissue` and `compartment`) for analysis.

- --libraryType: Sequencing platform (e.g., "10X" for droplet-based, "Smart-seq2" for well-based).

- --run_analysis: Boolean (true/false) indicating whether to run downstream analysis.

- --pin_S: Threshold for SpliZ score filtering (set to 0.1 in this example).

- --pin_z: Threshold for z-score filtering (set to 0.0 in this example).

- --bounds: Bounds parameter for SpliZ calculation (set to 5 in this example).

- --light: Boolean (true/false) indicating whether to use light mode (faster, less detailed).

- --n_perms: Number of permutations for statistical testing (set to 100 in this example).

- --svd_type: Type of SVD normalization (set to "normdonor" in this example).

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths (e.g., input files, output directories, STAR index) when using to avoid running errors.

2. Environment Confirmation: Ensure the "SpliZ" conda environment is successfully activated before running the commands. Also, confirm Nextflow/Snakemake is installed in the environment.

3. SICILIAN Input Requirements: Strictly follow the column addition and renaming rules; otherwise, the pipeline will fail. Ensure `tissue` and `compartment` have at least 2 distinct values to avoid division-by-zero errors.

4. Samplesheet Format (Non-SICILIAN): Ensure the CSV samplesheet has no header and the correct number of columns for your platform (2 for 10X, 3 for Smart-seq2).

5. STAR Index: The `--sjdbOverhang` parameter must be adjusted to match your sequencing read length (typically read length - 1) for accurate alignment.

6. Pipeline Preference: Nextflow is the recommended running mode; Snakemake is not recommended and should be used only if necessary.

7. Debugging: If encountering the "DataFrame truth value ambiguous" error, add `.to_dict()` to the relevant line in the code as specified in Section 3.

8. Parameter Tuning: Adjust parameters like `--pin_S`, `--pin_z`, `--bounds`, and `--n_perms` based on your data characteristics and research needs.

9. Troubleshooting: Refer to the SpliZ GitHub issues page (https://github.com/salzman-lab/SpliZ/issues/9) for additional troubleshooting tips.

10. Output Files: Analysis results will be saved in the default output directory of the pipeline (or specified via additional parameters) for further downstream analysis.