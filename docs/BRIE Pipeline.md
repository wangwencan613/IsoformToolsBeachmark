# BRIE Single-Cell Alternative Splicing Analysis Pipeline

This document records the complete workflow for alternative splicing analysis of different single-cell RNA sequencing platforms (droplet-based/well-based) using the BRIE tool.

## 1. Environment Preparation

First, load the BRIE runtime environment via Singularity to ensure a clean environment with complete dependencies:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_brie.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate BRIE
```

## 2. Official Documentation Reference

Detailed BRIE user manual: https://brie.readthedocs.io/en/latest/

Resources for splicing events: https://github.com/huangyh09/briekit/wiki

## 3. Analysis for Droplet-based Platforms (e.g., 10X Genomics)

### 3.1 Input File Preparation

The following files need to be prepared in advance:

- Sorted BAM files (.sortbam)

- Cell barcode file (barcodes.tsv.gz)

- Genome sequence file (genome.fasta)

- Gene annotation file (gene.gtf)

- Cell information file (cell_info.tsv, optional)

- Splicing event annotation file (refer to the wiki link above)

- Sequence features file (optional)

### 3.2 Running Commands

#### 3.2.1 Data Counting (brie-count)

```bash
brie-count \
-a /data/bioinf/brie-tutorials/tests/mouse_SE.lenient_50events.gff3 \
-s /data/bioinf/brie-tutorials/tests/10xData/neuron_1k_v3_possorted_genome_bam.50events.bam \
-b /data/bioinf/brie-tutorials/tests/10xData/barcodes.tsv.gz \
-o data/bioinf/10xtest/brie2_10X_test/out_dir \
-p 15
```

#### 3.2.2 Quantification Analysis (brie-quant) - Different Models

##### Model 0: No Intercept

```bash
brie-quant\ 
-i data/bioinf/10xtest/brie2_10X_test/out_dir/brie_count.h5ad \
-o data/bioinf/10xtest/brie2_10X_test/out_dir/brie_quant_pure.h5ad\
--interceptMode None
```

##### Model 1: Cell-level Intercept

```bash
brie-quant\ 
-i data/bioinf/10xtest/brie2_10X_test/out_dir/brie_count.h5ad \
-o data/bioinf/10xtest/brie2_10X_test/out_dir/brie_quant_gene.h5ad \
-g mouse_features.csv \
--interceptMode cell
```

##### Model 2-Quantification: Gene-level Intercept

```bash
brie-quant \
-i /data/bioinf/10xtest/brie2_10X_test/out_dir/brie_count.h5ad \
-o /data/bioinf/10xtest/brie2_10X_test/out_dir/brie_quant_aggr.h5ad \
--interceptMode gene
```

##### Model 2-Differential Analysis: Gene-level Intercept + Likelihood Ratio Test

```bash
brie-quant \
-i data/bioinf/10xtest/brie2_10X_test/out_dir/brie_count.h5ad \
-o data/bioinf/10xtest/brie2_10X_test/out_dir/brie_quant_cell.h5ad \
-c cell_info.tsv \
--interceptMode gene \
--LRTindex=All
```

## 4. Analysis for Well-based Platforms (e.g., Smart-seq2)

### 4.1 Input File Preparation

The following files need to be prepared in advance:

- Sorted BAM files (.sortbam)

- Cell table file (cell_table.tsv)

- Genome sequence file (genome.fasta)

- Gene annotation file (gene.gtf)

- Cell information file (cell_info.tsv, optional)

- Splicing event annotation file (refer to the wiki link above)

- Sequence features file (optional)

### 4.2 Running Commands

#### 4.2.1 Data Counting (brie-count)

```bash
brie-count \
-S /data/bioinf/scRNaseq_NSCLC/tbam/cell_table.tsv \
-a /data/bioinf/AS_events/SE.filtered.gff3.gz \
-o /data/bioinf/scRNaseq_NSCLC/BRIE2 \
-p 10
```

#### 4.2.2 Quantification Analysis (brie-quant) - Different Models

##### Model 0: No Intercept

```bash
brie-quant \
-i /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_count.h5ad \
-o /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_quant_pure.h5ad \
--interceptMode None
```

##### Model 1: Cell-level Intercept

```bash
brie-quant \
-i /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_count.h5ad \
-o /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_quant_gene.h5ad \
-g /data/bioinf/AS_events/human_features.csv \
--interceptMode cell
```

##### Model 2-Quantification: Gene-level Intercept

```bash
brie-quant \
-i /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_count.h5ad \
-o /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_quant_aggr.h5ad \
--interceptMode gene
```

##### Model 2-Differential Analysis: Gene-level Intercept + Likelihood Ratio Test

```bash
brie-quant \
-i /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_count.h5ad \
-o /data/bioinf/scRNaseq_NSCLC/BRIE2/brie_quant_cell.h5ad \
-c /data/bioinf/scRNaseq_NSCLC/tbam/cell_info.tsv \
--interceptMode gene \
--LRTindex=All
```

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths when using.

2. Path Correction: There were path errors in the original commands (e.g., `$/data/bioinf/jyuan`), which have been corrected to reasonable paths.

3. Thread Adjustment: The number of threads (parameter -p) can be adjusted according to server resources to avoid excessive resource occupation.

4. Optional Files: Optional files (such as cell_info.tsv) are only required for differential analysis and can be omitted for basic quantification.