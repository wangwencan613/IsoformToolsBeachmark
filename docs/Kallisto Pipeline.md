# Kallisto & kb-python Single-Cell Transcriptome Quantification Pipeline

This document records the complete workflow for single-cell transcriptome quantification using Kallisto and kb-python tools, including environment preparation, index construction, input file requirements, and running commands for different platforms.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the Kallisto conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate Kallisto
```

## 2. Official Documentation Reference

- Kallisto official repository: https://github.com/pachterlab/kallisto

- Kallisto getting started guide: https://pachterlab.github.io/kallisto/starting

- Kallisto bus tools: https://www.kallistobus.tools

## 3. Kallisto Analysis

### 3.1 Index Construction

First, download the transcript FASTA file (if not available) and construct the Kallisto index. The number of threads (-t) can be adjusted according to server resources:

```bash
cd /data/bioinf/ref/
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_44/gencode.v44.transcripts.fa.gz
cd /data/bioinf/kallisto/

kallisto index -t 50 -i /data/bioinf/ref/gencode/GRCh38.p14.transcripts.idx \ 
/data/bioinf/ref/gencode/gencode.v44.transcripts.fa
```

### 3.2 For Well-based Platforms (e.g., Smart-seq2)

#### 3.2.1 Input File Preparation

The following files need to be prepared in advance for quantification:

- Fastq files (paired-end)

- Kallisto index file (constructed in Section 3.1)

#### 3.2.2 Quantification Commands

Run the following commands for transcriptome quantification (adjust threads (-t) and bootstrap number (-b) as needed):

```bash
# Quantification (Sample 1: SRR10777218)
kallisto quant -t 30 -i /data/bioinf/ref/gencode/GRCh38.p14.transcripts.idx \
-o /data/bioinf/kallisto/218/ -b 100 \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_1.fastq.gz \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_2.fastq.gz

# Quantification (Sample 2: SRR10777219)
kallisto quant -t 30 -i /data/bioinf/ref/gencode/GRCh38.p14.transcripts.idx \
-o /data/bioinf/kallisto/219/ -b 100 \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777219_1.fastq.gz \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777219_2.fastq.gz

# Quantification with genome BAM generation
kallisto quant -t 30 -i /data/bioinf/ref/gencode/GRCh38.p14.transcripts.idx \
-b 30 -o /data/bioinf/kallisto/ --genomebam \
--gtf /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf \
--chromosomes /data/bioinf/ref/gencode/hg38.chrom.sizes \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_1.fastq.gz \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_2.fastq.gz

# Quantification with pseudo BAM generation
kallisto quant -t 30 -i /data/bioinf/ref/gencode/GRCh38.p14.transcripts.idx \
-b 30 -o /data/bioinf/kallisto/ --pseudobam \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_1.fastq.gz \
/data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_2.fastq.gz
```

## 4. kb-python Analysis

### 4.1 Index Construction

Construct the index for kb-python, which includes the kallisto index, transcript-to-gene mapping, and cDNA FASTA file. The parameters are explained as follows:

- -i INDEX: Path to the kallisto index to be constructed (prefix if multiple indices are generated).

- -g T2G: Path to the transcript-to-gene mapping file to be generated.

- -f1 FASTA: Path to the cDNA FASTA file to be generated.

```bash
kb ref -i /data/bioinf/ref/gencode/kb_GRCh38.p14.genome.idx \
       -g /data/bioinf/ref/gencode/transcripts_to_genes \
       -f1 /data/bioinf/ref/gencode/gencode.v44.transcripts.fa \
          /data/bioinf/ref/gencode/GRCh38.p14.genome.fa \
                   /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf
```

### 4.2 For Droplet-based Platforms (e.g., 10X Genomics)

#### 4.2.1 Input File Preparation

The following files need to be prepared in advance for quantification:

- Fastq files (paired-end, multiple lanes if applicable)

- kb-python index file (constructed in Section 4.1)

- Transcript-to-gene mapping file (output during index construction in Section 4.1)

#### 4.2.2 Quantification Commands

Generate count matrices from single-cell FASTQ files. Run `kb --list` to view supported single-cell technologies. Note to confirm whether the platform is 10XV3 or 10XV2 (check via `zcat sample_S1_L001_R1_001.fastq.gz | head -n 8`):

```bash
cd /data/bioinf/kallisto/kb

kb count -t 30 -m 70G -i /data/bioinf/ref/gencode/kb_GRCh38.p14.genome.idx \
         -g /data/bioinf/ref/gencode/transcripts_to_genes \
         -x 10XV3 -o /data/bioinf/kallisto/kb/ \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L001_R1_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L001_R2_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L002_R1_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L002_R2_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L003_R1_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L003_R2_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L004_R1_001.fastq.gz \
            /data/bioinf/KD6_mRNA/KD6_mRNA_S25_L004_R2_001.fastq.gz
```

#### 4.2.3 Optional Arguments (kb count)

- -w WHITELIST: Path to the whitelisted barcodes (pre-packaged if not provided and supported by bustools).

- --workflow: Workflow type (standard/lamanno/nucleus/kite/kite:10xFB); use `lamanno` for RNA velocity, `kite` for feature barcoding.

- --mm: Include reads that pseudoalign to multiple genes.

- --tcc: Generate a TCC matrix instead of a gene count matrix.

- --filter: Produce a filtered gene count matrix (default: bustools).

- --overwrite: Overwrite the existing output.bus file.

- --dry-run: Perform a dry run without actual execution.

- --loom/--h5ad: Generate loom/h5ad file from the count matrix.

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths when using to avoid running errors.

2. Environment Confirmation: Ensure the "Kallisto" conda environment is successfully activated before running the commands, otherwise the tools may not be found.

3. Index Dependence: Both Kallisto and kb-python require their respective index files; ensure the index is constructed correctly before quantification.

4. Platform Distinction: Clearly distinguish between well-based (Smart-seq2) and droplet-based (10X) platforms when selecting commands and input files.

5. 10X Version Note: Confirm whether the droplet-based platform uses 10XV3 or 10XV2 to set the correct `-x` parameter in `kb count`.

6. Resource Adjustment: Adjust the number of threads (-t) and memory (-m for kb count) according to server resources to avoid excessive resource occupation.

7. Command Sequence: For each tool, complete index construction first, then proceed with quantification commands.