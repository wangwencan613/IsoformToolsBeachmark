# MARVEL Single-Cell Alternative Splicing Analysis Pipeline

This document records the complete workflow for single-cell alternative splicing analysis using the MARVEL tool, including environment preparation, input file requirements, and step-by-step running commands for both droplet-based and well-based platforms.

## 1. Environment Preparation

First, load the runtime environment via Singularity to ensure a clean environment with complete dependencies, then activate the MARVEL conda environment:

```bash
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate MARVEL
```

## 2. Official Documentation Reference

- MARVEL official GitHub repository: https://github.com/wenweixiong/MARVEL

- Single-cell plate-based alternative splicing analysis guide: https://wenweixiong.github.io/MARVEL_Plate.html

- Single-cell droplet-based alternative splicing analysis guide: https://wenweixiong.github.io/MARVEL_Droplet.html

## 3. For Droplet-based Platforms (e.g., 10X Genomics)

### 3.1 Input File Preparation

The following files need to be prepared in advance to ensure the analysis runs smoothly:

- Genome sequence file (genome.fasta)

- Gene annotation file (gene.gtf)

- Cell Ranger reference file

- Sample metadata

- Cell barcode whitelist

- Raw gene count matrix

- Raw splice junction count matrix

- Normalised gene expression matrix

- Cell embeddings (e.g., UMAP coordinates)

### 3.2 Running Commands

#### 3.2.1 Create a MARVEL Object

First, read input files (gene expression, splice junction counts, cell coordinates, GTF), then create and annotate the MARVEL object, followed by data filtering and validation to prepare for downstream analysis:

```r
df.gene.norm <- readMM("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/norm/matrix_normalised.mtx")
df.gene.norm.pheno <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/norm/phenoData.txt", sep="\t", header=TRUE)
df.gene.norm.feature <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/norm/featureData.txt", sep="\t", header=TRUE)

df.gene.count <- readMM("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/matrix.mtx")
df.gene.count.pheno <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/phenoData.txt", sep="\t", header=TRUE)
df.gene.count.feature <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/Gene/filtered/filter-raw/featureData.txt", sep="\t", header=TRUE)

df.sj.count <- readMM("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/SJ/raw/sjcount/matrix.mtx")
df.sj.count.pheno <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/SJ/raw/sjcount/phenoData.txt", sep="\t", header=TRUE)
df.sj.count.feature  <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/SJ/raw/sjcount/featureData.txt", sep="\t", header=TRUE)
rownames(df.sj.count) = df.sj.count.feature[,1]
colnames(df.sj.count) = df.sj.count.pheno[,1]

df.coord <- read.table("/data/bioinf/10xtest/Droplet_data/SRR9008752/SRR9008752/Solo.out/umap/UMAP_coordinate.txt", sep="\t", header=TRUE)
# Process GTF file
gtf <- data.table::fread("/data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf", sep="\t", header=FALSE) %>% as.data.frame() 

gtf$V9 = gsub("gene_type","gene_biotype",gtf$V9)
gtf$V1 = gsub("chr","",gtf$V1)
library(MARVEL)
# Create MARVEL object
marvel <- CreateMarvelObject.10x(gene.norm.matrix=df.gene.norm,
                                 gene.norm.pheno=df.gene.norm.pheno,
                                 gene.norm.feature=df.gene.norm.feature,
                                 gene.count.matrix=df.gene.count,
                                 gene.count.pheno=df.gene.count.pheno,
                                 gene.count.feature=df.gene.count.feature,
                                 sj.count.matrix=df.sj.count,
                                 sj.count.pheno=df.sj.count.pheno,
                                 sj.count.feature=df.sj.count.feature,
                                 pca=df.coord,
                                 gtf=gtf
                                 )

# Annotate genes
marvel <- AnnotateGenes.10x(MarvelObject=marvel)
# Annotate junctions
marvel <- AnnotateSJ.10x(MarvelObject=marvel)  
# Validate junctions
marvel <- ValidateSJ.10x(MarvelObject=marvel)  
# Subset protein-coding genes
marvel <- FilterGenes.10x(MarvelObject=marvel,
                          gene.type="protein_coding"  
                          )
# Check alignment quality (prepares data for downstream analysis)
marvel <- CheckAlignment.10x(MarvelObject=marvel)

# Save MARVEL object
save(marvel, file="/data/bioinf/10xtest/Droplet_data/SRR9008752/MARVEL.RData")
```

#### 3.2.2 Generate Equivalence Class Table & Update Expression Matrices

Use XAEM tool to generate equivalence class table, create Y count matrix, and update X matrix and transcript expression using AEM algorithm:

```bash
# Generating the equivalence class table
/tools/XAEM-binary-0.1.2/bin/XAEM \
-i /data/bioinf/XAEM/data/TxIndexer_idx \
-l IU \
-1 <(gunzip -c /data/bioinf/scRNaseq_NSCLC/fastq/SRR10777215_1.fastq.gz) \
-2 <(gunzip -c /data/bioinf/scRNaseq_NSCLC/fastq/SRR10777215_2.fastq.gz) \
-p 4 -o ./SRR10777215

# Creating Y count matrix
Rscript /tools/XAEM-binary-0.1.2/R/Create_count_matrix.R workdir=$PWD  core=8  design.matrix=/data/bioinf/XAEM/data/X_matrix.RData

# Updating the X matrix and transcript expression using AEM algorithm
Rscript /tools/XAEM-binary-0.1.2/R/AEM_update_X_beta.R workdir=$PWD  core=8  design.matrix=/data/bioinf/XAEM/data/X_matrix.RData isoform.out=XAEM_isoform_expression.RData paralog.out=XAEM_paralog_expression.RData
```

#### 3.2.3 Explore Expression Patterns

Define cell groups based on sample metadata, then explore the percentage of cells expressing genes and splice junctions in different cell groups:

```r
# Retrieve sample metadata and define cell groups
sample.metadata <- marvel$sample.metadata
sample.metadata$cell.type<-rep(c("iPSC","Cardio day 2","Cardio day 4","Cardio day 10"),times = 2164)   

# Group 1 (reference group: iPSC)
index <- which(sample.metadata$cell.type=="iPSC")
cell.ids.1 <- sample.metadata[index, "cell.id"]
length(cell.ids.1)

# Group 2 (comparison group: Cardio day 10)
index <- which(sample.metadata$cell.type=="Cardio day 10")
cell.ids.2 <- sample.metadata[index, "cell.id"]
length(cell.ids.2)
 
# Explore percentage of cells expressing genes
marvel <- PlotPctExprCells.Genes.10x(MarvelObject=marvel,
                                     cell.group.g1=cell.ids.1,
                                     cell.group.g2=cell.ids.2,
                                     min.pct.cells=5
                                     )
head(marvel$pct.cells.expr$Gene$Data)

# Explore percentage of cells expressing splice junctions (SJ)
marvel <- PlotPctExprCells.SJ.10x(MarvelObject=marvel,
                                  cell.group.g1=cell.ids.1,
                                  cell.group.g2=cell.ids.2,
                                  min.pct.cells.genes=5,
                                  min.pct.cells.sj=5,
                                  downsample=TRUE,
                                  downsample.pct.sj=10
                                  )
head(marvel$pct.cells.expr$SJ$Data)
```

#### 3.2.4 Differential Analysis

Perform differential splicing analysis and differential gene expression analysis, then export the results:

```r
# Differential splicing analysis
marvel <- CompareValues.SJ.10x(MarvelObject=marvel,
                               cell.group.g1=cell.ids.1,
                               cell.group.g2=cell.ids.2,
                               min.pct.cells.genes=10,
                               min.pct.cells.sj=10,
                               min.gene.norm=1.0,
                               seed=1,
                               n.iterations=100,
                               downsample=TRUE,
                               show.progress=FALSE
                               )

# Differential gene expression analysis
marvel <- CompareValues.Genes.10x(MarvelObject=marvel,
                                  show.progress=FALSE
                                  )
 
# View and export differential splicing results
head(marvel$DE$SJ$Table)
write.table(marvel$DE$SJ$Table,"/data/bioinf/10xtest/Droplet_data/SRR9008752/MARVEL_diff.txt",sep="\t",col.names=TRUE,row.names=FALSE,quote=F)
```

#### 3.2.5 Gene-Splicing Dynamics Analysis

Analyze the relationship between gene expression and splicing events, which can be classified into four categories: coordinated, opposing, isoform-switching, and complex. Export the results for further analysis:

```r
marvel <- IsoSwitch.10x(MarvelObject=marvel,
                        pval.sj=0.05,
                        delta.sj=2,
                        min.gene.norm=1.0,
                        pval.adj.gene=0.05,
                        log2fc.gene=0.03
                        )

# View gene-splicing relationship results
head(marvel$SJ.Gene.Cor$Data[,c("coord.intron", "gene_short_name", "cor.complete")])

# Export results and save updated MARVEL object
write.table(marvel$SJ.Gene.Cor$Data,"/data/bioinf/10xtest/Droplet_data/SRR9008752/SJ.Gene.Cor.txt",sep="\t",col.names=TRUE,row.names=FALSE,quote=F)
save(marvel, file="/data/bioinf/10xtest/Droplet_data/SRR9008752/MARVEL_diff_cor.RData")
```

## 4. For Well-based Platforms (e.g., Smart-seq2)

### 4.1 Input File Preparation

The following files need to be prepared in advance for analysis:

- Sample metadata

- Genome sequence file (genome.fasta)

- Gene annotation file (gene.gtf)

- Sample fastq files

- Splice junction counts matrix

- Splicing event metadata

- Intron count matrix

- Gene expression matrix (e.g., TPM matrix)

- Gene metadata

### 4.2 Running Commands

#### 4.2.1 Create a MARVEL Object

Read input files (splice junctions, intron counts, gene expression, GTF), then create the MARVEL object for well-based platform data:

```r
library(data.table)
library(MARVEL)

# Read sample metadata
df.pheno <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/SJ_phenoData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")

# Read splice junction data
sj <- as.data.frame(fread("/data/bioinf/scRNaseq_NSCLC/starsjnext/SJ.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA"))

# Read splicing event feature data (SE, MXE, RI, A5SS, A3SS)
df.feature.se <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/SE_featureData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")
df.feature.mxe <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/MXE_featureData.txt",sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")
df.feature.ri <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/RI_featureData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")
df.feature.a5ss <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/A5SS_featureData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")
df.feature.a3ss <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/A3SS_featureData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA")
df.feature.list <- list(df.feature.se, df.feature.mxe, df.feature.ri, df.feature.a5ss, df.feature.a3ss)
names(df.feature.list) <- c("SE", "MXE", "RI", "A5SS", "A3SS")

# Read intron count matrix
df.intron.counts <- as.data.frame(fread("/data/bioinf/scRNaseq_NSCLC/starsjnext/rMATS/RI_Counts_Total/Counts_by_Region.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE, na.strings="NA"))

# Read gene expression (TPM) matrix and process column names
df.tpm <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/rsem/TPM.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE)
colnames(df.tpm)<-gsub(".genes.results","",colnames(df.tpm))
colnames(df.tpm)[1]<-"gene_id"

# Read and process GTF file
gtf <- as.data.frame(data.table::fread("/data/bioinf/ref/gencode/gencode.v44.chr_patch_hapl_scaff.annotation.gtf", sep="\t", header=FALSE, stringsAsFactors=FALSE, quote=""))

# Read gene feature data
df.tpm.feature <- read.table("/data/bioinf/scRNaseq_NSCLC/starsjnext/TPM_featureData.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE)

# Create MARVEL object for well-based platform
marvel <- CreateMarvelObject(SpliceJunction=sj,
                             SplicePheno=df.pheno,
                             SpliceFeature=df.feature.list,
                             IntronCounts=df.intron.counts,
                             GeneFeature=df.tpm.feature,
                             Exp=df.tpm,
                             GTF=gtf
                             )
```

#### 4.2.2 Detect Additional Splicing Events

Detect additional alternative splicing events (AFE: Alternative First Exon, ALE: Alternative Last Exon) using the MARVEL object:

```r
# Detect Alternative First Exon (AFE) events
marvel <- DetectEvents(MarvelObject=marvel,
                       min.cells=0,
                       min.expr=1,
                       track.progress=FALSE,
                       EventType="AFE"
                       )

# Detect Alternative Last Exon (ALE) events
marvel <- DetectEvents(MarvelObject=marvel,
                       min.cells=50,
                       min.expr=1,
                       track.progress=FALSE,
                       EventType="ALE"
                       )
```

#### 4.2.3 Compute PSI Values

Check alignment quality, then validate, filter, and compute PSI (Percent Spliced In) values for all types of splicing events:

```r
# Check splicing junction data quality
marvel <- CheckAlignment(MarvelObject=marvel, level="SJ")

# Compute PSI for SE (Skipped Exon) events
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     UnevenCoverageMultiplier=10,
                     EventType="SE"
                     )

# Compute PSI for MXE (Mutually Exclusive Exons) events    
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     UnevenCoverageMultiplier=10,
                     EventType="MXE"
                     )

# Compute PSI for RI (Retained Intron) events      
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     EventType="RI",
                     thread=4
                     )

# Compute PSI for A5SS (Alternative 5' Splice Site) events  
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     EventType="A5SS"
                     )

# Compute PSI for A3SS (Alternative 3' Splice Site) events  
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     EventType="A3SS"
                     )

# Compute PSI for AFE (Alternative First Exon) events     
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     EventType="AFE"
                     )
    
# Compute PSI for ALE (Alternative Last Exon) events      
marvel <- ComputePSI(MarvelObject=marvel,
                     CoverageThreshold=10,
                     EventType="ALE"
                     )

# Save MARVEL object and export SE PSI results
save(marvel, file="/data/bioinf/scRNaseq_NSCLC/starsjnext/MARVEL.RData")
marvel$PSI$SE
write.table(marvel$PSI$SE, "/data/bioinf/scRNaseq_NSCLC/starsjnext/marvel-PSI-SE.txt", sep="\t", col.names=TRUE, row.names=FALSE, quote=FALSE)
```

## Notes

1. Path Description: All paths in this document are example paths; replace them with actual file paths when using to avoid running errors.

2. Environment Confirmation: Ensure the "MARVEL" conda environment is successfully activated before running the commands. For XAEM-related commands, ensure the XAEM tool path is correct.

3. Input File Requirements: 
        - For droplet-based platforms: Ensure all count matrices (gene, splice junction) and cell embeddings are correctly formatted and matched.
        - For well-based platforms: Prepare all required splicing event feature files and ensure the TPM matrix column names are consistent with sample metadata.
     

4. Command Sequence: Each step must be run in sequence (e.g., create MARVEL object → annotate → filter → downstream analysis); the next step can only be executed after the previous step is completed successfully.

5. R Package Dependencies: Ensure required R packages (MARVEL, data.table, etc.) are installed in the MARVEL conda environment to avoid package loading errors.

6. Parameter Adjustment: Adjust parameters (e.g., `min.cells`, `CoverageThreshold`, `thread`) according to data characteristics and server resources.

7. Result Saving: Regularly save the MARVEL object during analysis to avoid data loss due to unexpected errors.

8. Platform Distinction: Use the corresponding MARVEL object creation function for different platforms (`CreateMarvelObject.10x` for droplet-based, `CreateMarvelObject` for well-based).