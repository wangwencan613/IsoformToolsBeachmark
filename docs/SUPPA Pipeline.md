# SUPPA2 Alternative Splicing Analysis Pipeline

This document records the complete workflow for alternative splicing analysis using the SUPPA2 tool, including environment preparation, input file specifications, step-by-step running commands (including Salmon-based transcript quantification), and key notes to ensure smooth execution. The workflow covers transcript quantification, alternative splicing event generation, PSI calculation, differential splicing analysis, and cluster analysis.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the SUPPA2 conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate SUPPA2 conda environment
source /tools/miniforge3/bin/activate
conda activate suppa2
```

## 2. Official Documentation Reference

- SUPPA official GitHub repository: https://github.com/comprna/SUPPA

- SUPPA2 detailed tutorial: https://github.com/comprna/SUPPA/wiki/SUPPA2-tutorial

## 3. Input File Preparation

The following files need to be prepared in advance. Note that the transcript expression file (TPM matrix) can be generated via Salmon quantification (Step 4.1-4.3) if not available:

- **GTF file**: Gene and transcript annotation file (e.g., gencode.v44.chr_patch_hapl_scaff.annotation.gtf) for alternative splicing event generation.

- **Reference transcript FASTA file**: Transcript sequence file (e.g., gencode.v44.transcripts.fa) for Salmon index construction.

- **Fastq files**: Paired-end sequencing data (e.g., SRR10777218-221) for transcript quantification using Salmon.

- **Optional: Transcript expression file**: A TPM matrix with multiple samples (generated via Salmon in this workflow).

## 4. Running Commands

The SUPPA2 workflow consists of eight key steps: build Salmon index, Salmon transcript quantification (multiple samples), extract TPM matrix, generate alternative splicing events, calculate PSI per event, compare single event between two groups, differential splicing analysis, and cluster analysis. Run the commands in sequence as follows:

### 4.1 Step 1: Build Salmon Index

Construct a Salmon index using the reference transcript FASTA file, which is required for efficient transcript quantification:

```bash
cd /data/bioinf/suppa
salmon index -t /data/bioinf/ref/gencode/gencode.v44.transcripts.fa -i /data/bioinf/suppa/gencode_44_salmon_index
```

### 4.2 Step 2: Salmon Transcript Quantification (Multiple Samples)

Quantify transcript expression for multiple paired-end samples using the pre-built Salmon index. The example below processes 4 samples (SRR10777218 to SRR10777221):

```bash
# Sample 1: SRR10777218
salmon quant -i /data/bioinf/suppa/gencode_44_salmon_index -l ISF --gcBias \
             -1 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_1.fastq.gz \
             -2 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777218_2.fastq.gz \
             -p 4 -o /data/bioinf/suppa/s218

# Sample 2: SRR10777219
salmon quant -i /data/bioinf/suppa/gencode_44_salmon_index -l ISF --gcBias \
             -1 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777219_1.fastq.gz \
             -2 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777219_2.fastq.gz \
             -p 4 -o /data/bioinf/suppa/s219

# Sample 3: SRR10777220
salmon quant -i /data/bioinf/suppa/gencode_44_salmon_index -l ISF --gcBias \
             -1 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777220_1.fastq.gz \
             -2 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777220_2.fastq.gz \
             -p 4 -o /data/bioinf/suppa/s220

# Sample 4: SRR10777221
salmon quant -i /data/bioinf/suppa/gencode_44_salmon_index -l ISF --gcBias \
             -1 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777221_1.fastq.gz \
             -2 /data/bioinf/scRNaseq_NSCLC/testfastq/SRR10777221_2.fastq.gz \
             -p 4 -o /data/bioinf/suppa/s221
```

### 4.3 Step 3: Extract TPM Matrix from Salmon Output

Use `multipleFieldSelection.py` to extract the TPM values (field 4) and transcript IDs (field 1) from all Salmon quantification outputs, generating a combined TPM matrix for downstream SUPPA2 analysis:

```bash
multipleFieldSelection.py -i /data/bioinf/suppa/*/quant.sf -k 1 -f 4 -o /data/bioinf/suppa/iso_tpm.txt
```

### 4.4 Step 4: Generate Alternative Splicing Events

Use SUPPA2 to generate alternative splicing events from the GTF file, then merge all event files into a single file for subsequent analysis:

```bash
# Generate alternative splicing events
suppa.py generateEvents -i /data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf \
         -o /data/bioinf/suppa/event -f ioe --pool-genes -e SE SS MX RI FL -b S -t 20

# Merge all .ioe event files into a single file (skip duplicate headers)
cd /data/bioinf/suppa/event
awk 'FNR==1 && NR!=1 { while (/^<header>/) getline; }
    1 {print}' *.ioe > all_events.ioe
```

Note: The `-e` parameter specifies the types of alternative splicing events to generate: SE (Skipped Exon), SS (Alternative Splice Site), MX (Mutually Exclusive Exons), RI (Retained Intron), FL (Alternative First/Last Exon).

### 4.5 Step 5: Calculate PSI per Alternative Splicing Event

Compute PSI (Percent Spliced In) values for each alternative splicing event using the TPM matrix generated in Step 4.3:

```bash
suppa.py psiPerEvent -i /data/bioinf/suppa/event/all_events.ioe \
                     -e /data/bioinf/suppa/iso_tpm.txt -o TRA2_events
```

### 4.6 Step 6: Compare Single Event Between Two Groups

Generate a boxplot to compare the PSI values of a specific alternative splicing event between two groups. Ensure each group has at least 2 samples (critical for statistical reliability):

```bash
python /data/bioinf/suppa/SUPPA/scripts/generate_boxplot_event.py \
       -i /data/bioinf/suppa/TRA2_events.psi \
       -e "ENSG00000198677.13;RI:chr5:95516525:95516586-95516672:95517060:-" \
       -g 1-2,3-4 -c NC,KD -o /data/bioinf/suppa/
```

Critical Note: The `-g` parameter specifies sample groups (column ranges in the PSI file), and each group must contain at least 2 samples.

### 4.7 Step 7: Differential Splicing Analysis with Local Events

First, split the TPM matrix and PSI file into two groups (NC and KD) using `split_file.R`, then run SUPPA2 differential splicing analysis with empirical method:

```bash
# Split TPM matrix into two groups (NC: s218,s219; KD: s220,s221)
/data/bioinf/suppa/SUPPA/scripts/split_file.R \
      iso_tpm.txt s218,s219 s220,s221 TRA2_NC_iso.tpm TRA2_KD_iso.tpm -i

# Split PSI file into two groups (matching TPM groups)
/data/bioinf/suppa/SUPPA/scripts/split_file.R \
      TRA2_events.psi s218,s219 s220,s221 TRA2_NC_events.psi TRA2_KD_events.psi -e

# Run differential splicing analysis
suppa.py diffSplice -m empirical -gc -i /data/bioinf/suppa/event/all_events.ioe \
         -p TRA2_KD_events.psi TRA2_NC_events.psi \
         -e TRA2_KD_iso.tpm TRA2_NC_iso.tpm -o TRA2_diffSplice --save_tpm_events
```

### 4.8 Step 8: Cluster Analysis of Differential Splicing Events

Perform cluster analysis on significant differential splicing events using the OPTICS method, with parameters adjusted for event clustering:

```bash
suppa.py clusterEvents \
         --dpsi /data/bioinf/suppa/TRA2_diffSplice.dpsi \
         --psivec /data/bioinf/suppa/TRA2_diffSplice.psivec \
         -st 0.05 --eps 0.2 --separation 0.11 -dt 0.2 \
         --min-pts 10 --groups 1-2,3-4 -c OPTICS \
         -o /data/bioinf/suppa/cluster
```

### Key Parameter Explanations

- **Salmon**:
        

    - index -t: Path to the reference transcript FASTA file.

    - index -i: Path and prefix for the output Salmon index.

    - quant -i: Path to the pre-built Salmon index.

    - quant -l ISF: Library type (Illumina Single-End Forward, adjusted for paired-end with -1/-2).

    - quant --gcBias: Enable GC bias correction (improves quantification accuracy).

    - quant -1/-2: Paths to paired-end fastq files.

    - quant -p: Number of threads used for parallel computing (set to 4 in this example).

    - quant -o: Output directory for Salmon quantification results.

- **multipleFieldSelection.py**:
        

    - -i: Input Salmon quantification files (using wildcard * to include all samples).

    - -k 1: Extract field 1 (transcript ID) as the key.

    - -f 4: Extract field 4 (TPM value) as the data.

    - -o: Path to the output TPM matrix file.

- **suppa.py generateEvents**:
        

    - -i: Path to the GTF annotation file.

    - -o: Output directory for alternative splicing events.

    - -f ioe: Output format (ioe = SUPPA2 event format).

    - --pool-genes: Pool events from the same gene.

    - -e: Types of alternative splicing events to generate (SE, SS, MX, RI, FL).

    - -b S: Strand-specific mode (S = stranded).

    - -t 20: Number of threads used for parallel computing.

- **suppa.py psiPerEvent**:
        

    - -i: Path to the merged alternative splicing events file (all_events.ioe).

    - -e: Path to the TPM matrix file (iso_tpm.txt).

    - -o: Prefix for the output PSI file.

- **generate_boxplot_event.py**:
        

    - -i: Path to the PSI file (TRA2_events.psi).

    - -e: Specific alternative splicing event to compare (format: gene_id;event_type:chromosome:coordinates:strand).

    - -g: Sample groups (column ranges in the PSI file, e.g., 1-2 = columns 1-2 as group 1).

    - -c: Group labels (corresponding to -g, e.g., NC,KD).

    - -o: Output directory for the boxplot.

- **split_file.R**:
        

    - First parameter: Input file (TPM matrix or PSI file).

    - Second parameter: Sample names for group 1 (separated by commas).

    - Third parameter: Sample names for group 2 (separated by commas).

    - Fourth parameter: Output file for group 1.

    - Fifth parameter: Output file for group 2.

    - -i: Indicate input is a TPM matrix; -e: Indicate input is a PSI file.

- **suppa.py diffSplice**:
        

    - -m empirical: Statistical method for differential splicing (empirical distribution).

    - -gc: Enable GC bias correction.

    - -i: Path to the merged alternative splicing events file.

    - -p: PSI files for the two groups (group 1 first, group 2 second).

    - -e: TPM files for the two groups (matching the order of -p).

    - -o: Prefix for the output differential splicing results.

    - --save_tpm_events: Save TPM values for events in the output.

- **suppa.py clusterEvents**:
        

    - --dpsi: Path to the differential splicing dpsi file (TRA2_diffSplice.dpsi).

    - --psivec: Path to the PSI vector file (TRA2_diffSplice.psivec).

    - -st/--sig-threshold: p-value threshold to consider an event significant (set to 0.05).

    - -dt/--dpsi-threshold: Lower bound for absolute delta PSI to include in clustering (set to 0.2).

    - -e/--eps: Maximum distance (0-1) to consider two events in the same cluster (set to 0.2).

    - -s/--separation: Maximum distance of an event to a cluster (required for OPTICS method, set to 0.11).

    - --min-pts: Minimum number of events required per cluster (set to 10).

    - -g/--groups: Sample groups (column ranges, matching the PSI file).

    - -c OPTICS: Clustering method (OPTICS).

    - -o: Output directory for cluster results.

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths (e.g., reference files, fastq files, output directories, SUPPA2/Salmon paths) when using to avoid running errors.

2. Environment Confirmation: Ensure the "suppa2" conda environment is successfully activated before running the commands. Also, confirm Salmon and SUPPA2 tools are installed in the environment.

3. Sample Group Requirement: For group comparison and differential analysis, each group must contain at least 2 samples to ensure statistical reliability. The `-g` parameter in boxplot and cluster analysis must specify valid column ranges with no overlaps.

4. Alternative Splicing Events: The `-e` parameter in `suppa.py generateEvents` specifies event types; adjust based on your research focus (SE, SS, MX, RI, FL are the most common types).

5. Salmon Quantification: The `--gcBias` parameter is recommended to correct for GC bias in sequencing data, improving transcript quantification accuracy. Adjust the `-p` (threads) parameter based on server resources.

6. Event Merging: The `awk` command skips duplicate headers when merging .ioe files, ensuring the combined file has a single header and valid event entries.

7. Clustering Parameters: Adjust parameters like `--eps`, `--separation`, `--min-pts`, and`-dt` based on your data characteristics to obtain meaningful clusters.

8. Output Files: 
        - Salmon index and quantification results are saved in the specified directories.
        - TPM matrix (iso_tpm.txt) and PSI file (TRA2_events.psi) are generated for downstream analysis.
        - Differential splicing results (e.g., TRA2_diffSplice.dpsi, TRA2_diffSplice.psivec) and cluster results are saved in the specified output directories for further functional analysis.
      

9. Reference Consistency: Ensure the GTF file, reference transcript FASTA file, and fastq files are all based on the same reference genome (e.g., Gencode v44) to avoid alignment and event generation errors.

10. Script Execution: Ensure the paths to SUPPA2 scripts (generate_boxplot_event.py, split_file.R) are correct; adjust the paths if the scripts are located in a different directory.