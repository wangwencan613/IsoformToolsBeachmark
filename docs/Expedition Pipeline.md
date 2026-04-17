# Expedition Single-Cell Alternative Splicing Analysis Pipeline

This document records the complete workflow for alternative splicing analysis using the Expedition tool, including environment preparation, input file requirements and running commands.

## 1. Environment Preparation

First, load the Outrigger runtime environment via Singularity to ensure a clean environment with complete dependencies:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate outrigger-env
```

## 2. Official Documentation Reference

Detailed Outrigger user manual: http://yeolab.github.io/outrigger/

## 3. Input File Preparation

The following files need to be prepared in advance to ensure the analysis runs smoothly:

- Sorted BAM files (.sortbam files)

- Genome sequence file (genome.fasta)

- Gene annotation file (gene.gtf)

- Chromosome size file (hg38.chrom.sizes)

## 4. Running Commands

### 4.1 Index Construction (outrigger index)

Construct the index file using sorted BAM files and gene annotation GTF file:

```bash
outrigger index --bam /data/bioinf/scRNaseq_NSCLC/tbam/*.sortedByCoord.out.bam \
--gtf /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf
```

### 4.2 Validation (outrigger validate)

Validate the index and related files using chromosome size file and genome FASTA file:

```bash
outrigger validate -g /data/bioinf/ref/gencode/hg38.chrom.sizes \
--fasta /data/bioinf/ref/gencode/GRCh38.p14.genome.fa
```

### 4.3 PSI Calculation (outrigger psi)

Calculate the Percent Spliced In (PSI) values for alternative splicing events:

```bash
outrigger psi
```

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths when using to avoid running errors.

2. Environment Confirmation: Ensure the "outrigger-env" conda environment is successfully activated before running the commands, otherwise the tool may not be found.

3. Input File Requirements: Make sure all input files are in the correct format (e.g., BAM files are sorted, GTF file matches the genome version) to ensure the analysis is valid.

4. Command Dependence: The three commands need to be run in sequence (index → validate → psi), and the next command can only be executed after the previous one is completed successfully.