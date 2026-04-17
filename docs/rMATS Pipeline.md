# rMATS Turbo Alternative Splicing Analysis Pipeline

This document records the complete workflow for differential alternative splicing analysis using the rMATS Turbo tool, including environment preparation, input file requirements, and running commands to ensure smooth execution.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the rMATS conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate rmats
```

## 2. Official Documentation Reference

Detailed rMATS Turbo user manual and source code (v4.1.2): https://github.com/Xinglab/rmats-turbo/tree/v4.1.2

## 3. Input File Preparation

The following files need to be prepared in advance to ensure the analysis runs smoothly. Note that replicate RNA-Seq data is required for differential alternative splicing analysis:

- Fastq files: Replicate RNA-Seq data (paired-end, as specified in the command)

- Gene annotation file (gtf): Used for annotating splicing events and gene structures

- STAR-index: Pre-constructed STAR genome index for efficient read alignment

Additional note: The `s1.txt` file (specified by `--s1` parameter) is a text file containing the paths of fastq files for the sample group, with each line corresponding to one replicate fastq file.

## 4. Running Commands

Run the rMATS main command to perform differential alternative splicing analysis. The command includes parameters for input files, alignment settings, output directories, and advanced options. Key parameter explanations are provided below the command:

```bash
python  /tools/miniforge3/envs/rmats/rMATS/rmats.py \
    --s1 s1.txt \
    --gtf /data/bioinf/ref/38/Homo_sapiens.GRCh38.110.gtf  \
    --bi /data/bioinf/ref/38/hg38_index  \
    -t paired  \
    --nthread 4 \
    --od /data/bioinf/rmats/smartseq \
    --tmp /data/bioinf/rmats/tmp_smartseq \
    --readLength 100  --novelSS  --variable-read-length
```

### Key Parameter Explanations

- --s1: Path to the text file (`s1.txt`) containing fastq file paths for the sample group (one path per line).

- --gtf: Path to the gene annotation GTF file.

- --bi: Path to the pre-constructed STAR genome index directory.

- -t: Library type; `paired` indicates paired-end sequencing data (adjust to `single` for single-end data).

- --nthread: Number of threads used for parallel computing (adjust according to server resources).

- --od: Output directory where all analysis results will be saved.

- --tmp: Temporary directory for storing intermediate files (avoiding disk space issues in the default temporary directory).

- --readLength: Length of sequencing reads (set to 100 in this example, adjust to match actual read length).

- --novelSS: Enable detection of novel splice sites (not annotated in the GTF file).

- --variable-read-length: Enable support for variable read lengths (useful if reads have inconsistent lengths).

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths (e.g., `s1.txt`, `--gtf`, `--bi`, `--od`, `--tmp`) when using to avoid running errors.

2. Environment Confirmation: Ensure the "rmats" conda environment is successfully activated before running the command. Also, confirm the path to `rmats.py` is correct (matches the conda environment path).

3. Replicate Requirement: rMATS requires replicate RNA-Seq data for differential analysis; ensure `s1.txt` contains paths for multiple replicate fastq files.

4. Library Type: Adjust the `-t` parameter according to the sequencing type (paired/single). Using the wrong type will cause alignment errors.

5. Resource Adjustment: Adjust the `--nthread` parameter based on server CPU resources to balance running speed and resource occupation.

6. Temporary Directory: Ensure the temporary directory (`--tmp`) has sufficient disk space, as rMATS generates large intermediate files during analysis.

7. Read Length: The `--readLength` parameter must match the actual length of the sequencing reads; incorrect settings may affect splicing event detection.

8. Novel Splice Sites: The `--novelSS` parameter is optional; enable it if you need to detect novel splice sites not annotated in the GTF file.

9. Output Files: After analysis, results (including differential splicing event tables, junction counts, etc.) will be saved in the directory specified by `--od` for further downstream analysis.