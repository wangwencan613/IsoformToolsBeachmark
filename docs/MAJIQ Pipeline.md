# MAJIQ Single-Cell Alternative Splicing Analysis Pipeline
This document describes the complete workflow for single-cell alternative splicing analysis using the **MAJIQ** framework, including environment setup, input requirements, and step-by-step commands for well‑based sequencing platforms (e.g., Smart‑seq2).

## 1. Environment Preparation
MAJIQ requires user registration prior to download and installation.
Users must install the software and configure the runtime environment by following official instructions.

## 2. Official Documentation & Access
- MAJIQ quick start guide: https://biociphers.bitbucket.io/majiq/quick.html
- Registration & download page: https://majiq.biociphers.org/app_download/

## 3. Analysis for Well‑based Platforms (e.g., Smart‑seq2)

### 3.1 Input File Preparation
The following files are required to run the pipeline:
- Sorted BAM files (`.bam`) and corresponding index files (`.bai`)
- Reference genome sequence (`.fa`), available from GENCODE, UCSC, NCBI, etc.
- Gene annotation file (`.gtf` or `.gff3`), matching the reference genome version
- Configuration (`.ini`) file defining paths to BAM files, reference genome, and sample prefixes

### 3.2 Running Commands

#### 3.2.1 MAJIQ Builder
Build splice graph indices from the annotation and BAM files specified in the configuration:
```bash
majiq build \
/data/bioinf/ref/gencode/gencode.v44.annotation.gff3 \
-c /data/bioinf/MAJIQ/workshop_example/settings.ini \
-o /data/bioinf/MAJIQ/workshop_example/output1 \
-j 8
```

#### 3.2.2 MAJIQ Quantifier – PSI Quantification
Loop through MAJIQ output files to calculate PSI (Percent Spliced In) values per sample:
```bash
for file in /data/bioinf/MAJIQ/workshop_example/output1/*_Aligned.sortedByCoord.out.majiq; do
    sample=$(basename "${file%_Aligned.sortedByCoord.out.majiq}")
    input="/data/bioinf/MAJIQ/workshop_example/output1/${sample}_Aligned.sortedByCoord.out.majiq"
    majiq psi -o /data/bioinf/MAJIQ/workshop_example/TKIpsi -n $sample $input
done
```

#### 3.2.3 MAJIQ Quantifier – DeltaPSI Quantification
Compute differential splicing (DeltaPSI) between two groups of samples:
```bash
majiq deltapsi \
-o /data/bioinf/MAJIQ/workshop_example/deltapsi \
-n A1 C1 \
-grp1 /data/bioinf/MAJIQ/workshop_example/output1/SRR10777215_Aligned.sortedByCoord.out.majiq \
      /data/bioinf/MAJIQ/workshop_example/output1/SRR10777216_Aligned.sortedByCoord.out.majiq \
-grp2 /data/bioinf/MAJIQ/workshop_example/output1/SRR10777217_Aligned.sortedByCoord.out.majiq \
      /data/bioinf/MAJIQ/workshop_example/output1/SRR10777218_Aligned.sortedByCoord.out.majiq
```

### Key Parameter Explanations
- `majiq build`: Constructs splice graphs for downstream quantification
  - `-c`: Path to the MAJIQ configuration file
  - `-o`: Output directory for built MAJIQ files
  - `-j`: Number of parallel threads
- `majiq psi`: Quantifies PSI values for individual samples
  - `-n`: Sample name prefix for output files
- `majiq deltapsi`: Detects differential splicing between two groups
  - `-n`: Names for the two comparison groups
  - `-grp1` / `-grp2`: MAJIQ files for each group of samples

## Notes
- All paths in this document are examples; replace them with your actual file paths.
- BAM files **must be sorted and indexed** before running MAJIQ.
- The genome FASTA and GTF/GFF3 annotations must use the same genome build.
- Ensure the configuration file correctly points to all input BAMs and references.
- Group names and sample lists in `deltapsi` can be adjusted according to experimental design.