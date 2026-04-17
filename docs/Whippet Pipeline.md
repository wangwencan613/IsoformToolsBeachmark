# Whippet.jl Splicing Quantification & Differential Analysis Pipeline

This document records the complete workflow for splicing quantification and differential splicing analysis using Whippet.jl, including environment preparation, dependent software requirements, input file specifications, step-by-step running commands, and key notes to ensure smooth execution.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the corresponding conda environment (no specific conda environment name provided; use the appropriate environment for Whippet.jl):

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment (adjust to your Whippet-compatible environment)
source /tools/miniforge3/bin/activate
```

## 2. Dependent Software & Official Documentation

### 2.1 Dependent Software Requirements

- **Julia**: Whippet v1.6 is compatible with the long-term support (LTS) release of Julia (v1.6.7), which can be downloaded from: https://julialang.org.
        

    - Critical Note: Whippet.jl does **not** work on Julia v1.9 or higher versions.

- **trim_galore**: Used for read quality control and trimming (required before quantification).

- **Optional**: Spliced read aligner (e.g., STAR, HISAT) for generating BAM files (used for building BAM-supplemented indexes).

### 2.2 Official Documentation Reference

Whippet.jl official GitHub repository: https://github.com/timbitz/Whippet.jl

## 3. Input File Preparation

The following files need to be prepared in advance. Note the specific requirements for the GTF file to avoid pipeline errors:

- **Reference genome FASTA file**: Genome sequence file (e.g., hg19.fa).

- **Gene GTF file**: 
        

    - Whippet.jl only uses GTF `exon` lines (all other line types are ignored).

    - The GTF file must contain both `gene_id` and `transcript_id` attributes, and these two attributes must **not** be the same.

    - The GTF file must comply with the GTF2.2 specification, and all entries for a single transcript must be in a continuous block.

    - Warning: GTF files from the UCSC Table Browser, iGenomes, or Refseq websites do **not** meet these specifications and cannot be used directly.

- **Fastq files**: Raw sequencing reads of the samples to be processed (single-end or paired-end).

- **Optional**: Sorted and indexed BAM file (from STAR/HISAT) for building BAM-supplemented indexes (for non-model organisms with poor annotation).

## 4. Running Commands

The Whippet.jl workflow consists of three key steps: build genome index (optional BAM-supplemented index), read quality control & quantification, and compare multiple PSI files. Run the commands in sequence as follows:

### 4.1 Step 1: Build Genome Index

Build a standard index using the reference FASTA and GTF files. An optional BAM-supplemented index is also provided for non-model organisms.

#### 4.1.1 Standard Index

```bash
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-index.jl \
--fasta /data/bioinf/ref/19/hg19.fa \
--gtf /data/bioinf/Whippet.jl-1.6.1/anno/gencode_hg19.v25.tsl1.gtf.gz \
-x /data/bioinf/Whippet.jl-1.6.1/bin/index
```

#### 4.1.2 Optional: BAM-Supplemented Index

For non-model organisms with poorly annotated genomes, use a BAM file (from independent spliced read aligners like STAR/HISAT) to supplement the index, which helps capture more splice sites and exons:

```bash
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-index.jl \
--bam filename.sort.rmdup.bam \
--fasta /data/bioinf/ref/19/hg19.fa \
--gtf /data/bioinf/Whippet.jl-1.6.1/anno/gencode_hg19.v25.tsl1.gtf.gz \
-x /data/bioinf/Whippet.jl-1.6.1/bin/index
```

Note: Replace `filename.sort.rmdup.bam` with the path to your sorted and indexed BAM file.

### 4.2 Step 2: Read Quality Control & Quantification

First, use `trim_galore` to perform read quality control and trimming, then run Whippet.jl quantification. Commands are provided for both single-end and paired-end reads.

#### 4.2.1 Single-End Reads

```bash
# Read quality control and trimming
trim_galore -q 20 --phred33 --stringency 3 --length 20 -e 0.1 \
--fastqc --gzip \
-o /data/bioinf/RNA-Bloom_v2.0.1/ \
/data/bioinf/Whippet.jl-1.6.1/test/SRR1199010.fastq.gz

# Splicing quantification
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/whippet-quant.jl \
/data/bioinf/RNA-Bloom_v2.0.1/SRR1199010_trimmed.fq.gz \
-x /tools/Whippet.jl/bin/index.jls \
-o /tools/Whippet.jl/SRR1199010
```

#### 4.2.2 Paired-End Reads

Process multiple paired-end samples (example with 2 samples) with quality control and quantification:

```bash
# Sample 1: Read quality control and trimming (paired-end)
trim_galore -q 20 --phred33 --stringency 3 --length 20 -e 0.1 \
--fastqc --paired --gzip \
-o /data/bioinf/RNA-Bloom_v2.0.1/ \
/data/bioinf/RNA-Bloom_v2.0.1/LADC_17_NF_1.fastq.gz /data/bioinf/RNA-Bloom_v2.0.1/LADC_17_NF_2.fastq.gz

# Sample 2: Read quality control and trimming (paired-end)
trim_galore -q 20 --phred33 --stringency 3 --length 20 -e 0.1 \
--fastqc --paired --gzip -o \
/data/bioinf/RNA-Bloom_v2.0.1/ \
/data/bioinf/RNA-Bloom_v2.0.1/bulkrnaseq/ERR523093_1.fq.gz /data/bioinf/RNA-Bloom_v2.0.1/bulkrnaseq/ERR523093_2.fq.gz

# Quantify Sample 1
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-quant.jl \
/data/bioinf/RNA-Bloom_v2.0.1/LADC_17_NF_1_val_1.fq.gz /data/bioinf/RNA-Bloom_v2.0.1/LADC_17_NF_2_val_2.fq.gz \
-x /tools/Whippet.jl/bin/index.jls \
-o /tools/Whippet.jl/LADC_17_NF

# Quantify Sample 2
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-quant.jl \
/data/bioinf/RNA-Bloom_v2.0.1/ERR523093_1_val_1.fq.gz /data/bioinf/RNA-Bloom_v2.0.1/ERR523093_2_val_2.fq.gz \
-x /tools/Whippet.jl/bin/index.jls \
-o /tools/Whippet.jl/ERR523093
```

### 4.3 Step 3: Compare Multiple PSI Files

Use `whippet-delta.jl` to compare PSI (Percent Spliced In) files between samples/groups. Pay attention to the comma requirement after sample filenames to avoid errors.

#### 4.3.1 Compare Individual Samples

Critical Note: The comma (,) after each sample filename is **indispensable**; otherwise, an error will occur indicating no files were entered.

```bash
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-delta.jl \
-a ERR523093.psi.gz, \
-b LADC_17_NF.psi.gz, \
-o /tools/Whippet.jl/single
```

#### 4.3.2 Compare Two Groups of Multiple Samples

Separate sample filenames within each group with commas (no trailing comma needed for group-level separation):

```bash
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-delta.jl \
-a ERR523093.psi.gz,SRR1199010.psi.gz \
-b LADC_17_NF.psi.gz,ERR523093.psi.gz \
-o /tools/Whippet.jl/output
```

#### 4.3.3 Compare Groups with Regular Sample Naming

If sample names have a consistent prefix (e.g., sample1-r1.psi.gz, sample1-r2.psi.gz), use the prefix to represent all samples in the group:

```bash
# List all PSI files to confirm naming pattern
ls *.psi.gz
# Example output: sample1-r1.psi.gz sample1-r2.psi.gz sample2-r1.psi.gz sample2-r2.psi.gz

# Compare groups using prefixes
/tools/julia-1.6.7/bin/julia \
/tools/Whippet.jl/bin/whippet-delta.jl \
-a sample1 -b sample2
```

### Key Parameter Explanations

- **whippet-index.jl**:
        

    - --fasta: Path to the reference genome FASTA file.

    - --gtf: Path to the GTF annotation file (must meet GTF2.2 specifications).

    - --bam (optional): Path to the sorted and indexed BAM file for supplementing the index.

    - -x: Path and prefix for the output index file.

- **trim_galore**:
        

    - -q 20: Minimum Phred quality score (20).

    - --phred33: Use Phred33 quality scoring (adjust to --phred64 if needed).

    - --stringency 3: Maximum number of allowed mismatches in the adapter sequence.

    - --length 20: Discard reads shorter than 20 bp after trimming.

    - -e 0.1: Maximum allowed error rate for adapter matching.

    - --fastqc: Run FastQC to assess read quality after trimming.

    - --gzip: Compress the trimmed output fastq files.

    - --paired: Indicate paired-end reads (only for paired-end data).

    - -o: Output directory for trimmed files.

- **whippet-quant.jl**:
        

    - First parameter(s): Path to trimmed fastq file(s) (single file for single-end, two files for paired-end).

    - -x: Path to the pre-built Whippet index file (.jls).

    - -o: Output directory for quantification results (PSI files).

- **whippet-delta.jl**:
        

    - -a: Sample(s) in group A (separated by commas; add trailing comma for single samples).

    - -b: Sample(s) in group B (separated by commas; add trailing comma for single samples).

    - -o: Output directory for differential splicing results.

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths (e.g., reference files, fastq files, output directories, Julia/Whippet paths) when using to avoid running errors.

2. Julia Version: Ensure Julia v1.6.7 is used (Whippet.jl is incompatible with Julia v1.9+). Confirm the path to Julia (e.g., /tools/julia-1.6.7/bin/julia) is correct.

3. GTF File Requirements: Strictly follow the GTF2.2 specification; only exon lines are used, and both `gene_id` and `transcript_id` must be present and distinct. Avoid GTF files from UCSC Table Browser, iGenomes, or Refseq.

4. Read Trimming: Always run `trim_galore` before quantification to remove low-quality reads and adapters, which improves quantification accuracy.

5. PSI Comparison: The trailing comma after single sample filenames in `whippet-delta.jl` is mandatory; omitting it will cause an error.

6. BAM-Supplemented Index: Use this option for non-model organisms with poor annotation to capture more splice sites and exons. Ensure the BAM file is sorted and indexed.

7. Sample Naming (Group Comparison): When using prefixes to represent groups, ensure all samples in the group have the same prefix (e.g., sample1 for sample1-r1.psi.gz, sample1-r2.psi.gz).

8. Output Files: 
        - Quantification results (PSI files) are saved in the directory specified by `-o` in `whippet-quant.jl`.
        - Differential splicing results are saved in the directory specified by `-o` in `whippet-delta.jl` for further downstream analysis.
      

9. Quality Control: Check FastQC reports (generated by `trim_galore`) to ensure read quality meets analysis requirements before proceeding with quantification.