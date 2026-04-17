# Leafcutter Single-Cell Alternative Splicing Analysis Pipeline

This document records the complete workflow for alternative splicing analysis using the Leafcutter tool, including environment preparation, step-by-step running commands, and key notes to ensure smooth execution.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the Leafcutter conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate Leafcutter
```

## 2. Official Documentation Reference

Detailed Leafcutter user manual: http://davidaknowles.github.io/leafcutter/

## 3. Running Commands

The Leafcutter analysis workflow consists of three key steps: alignment, BAM to junction conversion, and intron clustering. Run the commands in sequence as follows:

### 3.1 Step 0: Alignment with STAR

Note: Using STAR for alignment requires activating the STAR conda environment first (run `conda activate star` before executing the following command). This step performs read alignment to the reference genome:

```bash
STAR --genomeDir hg19index/ --twopassMode Basic --outSAMstrandField intronMotif --readFilesCommand zcat --outSAMtype BAM Unsorted
```

### 3.2 Step 1: Convert BAM Files to Junction Files (.junc)

Loop through BAM files, index each BAM file, and extract junction information to generate .junc files. Finally, collect all .junc file paths into a text file for subsequent clustering:

```bash
for bamfile in `ls run/geuvadis/*chr1.bam`; do
    echo Converting $bamfile to $bamfile.junc
    samtools index $bamfile
    /tools/regtools/build/regtools junctions extract -a 8 -s RF -m 50 -M 500000 $bamfile -o $bamfile.junc
    echo $bamfile.junc >> test_juncfiles.txt
done
```

### 3.3 Step 2: Intron Clustering

Use Leafcutter’s clustering script to cluster introns based on the junction files collected in Step 1. After clustering, view the generated count file to verify the results:

```bash
python /tools/leafcutter/clustering/leafcutter_cluster_regtools.py -j test_juncfiles.txt \
-m 50 -o testYRIvsEU -l 500000
zcat testYRIvsEU_perind_numers.counts.gz | more
```

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths (e.g., `hg19index/`, `run/geuvadis/*chr1.bam`) when using to avoid running errors.

2. Environment Switch: STAR alignment requires activating the STAR conda environment (`conda activate star`), while other steps use the Leafcutter environment. Ensure the correct environment is activated for each step.

3. Command Sequence: The three steps must be run in sequence (alignment → BAM to junc → intron clustering), and the next step can only be executed after the previous step is completed successfully.

4. Parameter Explanation: 
        - In Step 1: `-a 8` (minimum anchor length), `-s RF` (strand-specific library type), `-m 50` (minimum junction length), `-M 500000` (maximum junction length).
        - In Step 2: `-m 50` (minimum intron length), `-l 500000` (maximum intron length), `-o` (output prefix).
      

5. BAM Index Requirement: Ensure each BAM file is indexed (via `samtools index`) before extracting junctions, otherwise the regtools command will fail.

6. Result Verification: After Step 2, use `zcat testYRIvsEU_perind_numers.counts.gz | more` to view the clustering results and confirm the output is normal.