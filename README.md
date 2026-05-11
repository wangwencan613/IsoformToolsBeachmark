<div align="center">
  <h1>IsoformToolsBeachmark</h1>
</div>

# 📌 Overview

In this study, we conducted a comprehensive evaluation of alternative splice analysis tools(ASTs), focusing on their performance in splicing detection, splicing quantification, differential splicing, computational utility, and summarize the results into practical, platform-aware recommendations for AST selection.

<table width="100%" border="0">
  <tr>
    <td colspan="2">
      <img src="https://raw.githubusercontent.com/JYuan-Lab/IsoformToolsBeachmark/main/assets/image/figure1.tif" width="100%">
    </td>
  </tr>
</table>

---

# ⚙️ Environment Setup
All tools, with the exception of MAJIQ2 (requires permission), are provided in Singularity containers to ensure reproducibility and ease of use. Due to specific dependency requirements, BRIE2 is packaged separately, while all remaining tools are bundled into a single container.

The container files are hosted on Zenodo and can be downloaded using the following links:
- BRIE container: 
  https://zenodo.org/records/18885345
- Main tools container and the readme documentation of all methods:  
  https://zenodo.org/records/20067728

Usage
```bash
# Download containers from Zenodo (replace with actual file URLs)
wget https://zenodo.org/records/18885345/files/isoform_brie.sif.zip
wget https://zenodo.org/records/20067728/files/singularity_and_readme.zip?download=1

# Unzip the archives
unzip isoform_brie.sif.zip
unzip singularity_and_readme.zip
unzip isoform_tools_v2.sif.zip

# Run a tool with Singularity (e.g., rMATS)
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash
# --cleanenv: Clean the host environment variables to ensure a pure container environment
# --no-home: Do not mount the host's home directory to avoid configuration interference
# -B /data/bioinf:/data/bioinf: Mount the host data directory to the container
# isoform_tools_v2.sif: Packaged transcriptome analysis tool container
# bash: Start an interactive terminal inside the container

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate rmats

# Run a tool with Singularity (e.g., rMATS)
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash
# --cleanenv: Clean the host environment variables to ensure a pure container environment
# --no-home: Do not mount the host's home directory to avoid configuration interference
# -B /data/bioinf:/data/bioinf: Mount the host data directory to the container
# isoform_tools_v2.sif: Packaged transcriptome analysis tool container
# bash: Start an interactive terminal inside the container

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate rmats

```
---
# 📚 Tool Documentation

Event-level ASTs
- BRIE2<br>https://brie.readthedocs.io/en/latest/<br>https://github.com/huangyh09/briekit/wiki
- Expedition<br>http://yeolab.github.io/outrigger/
- Leafcutter<br>http://davidaknowles.github.io/leafcutter/
- MAJIQ2<br>https://biociphers.bitbucket.io/majiq/
- MARVEL<br>https://github.com/wenweixiong/MARVEL
- Psix<br>https://github.com/lareaulab/psix
- rMATS<br>https://github.com/Xinglab/rmats-turbo/tree/v4.1.2
- SUPPA2<br>https://github.com/comprna/SUPPA
- Whippet<br>https://github.com/timbitz/Whippet.jl

Transcript-level ASTs
- Kallisto<br>https://github.com/pachterlab/kallisto
- RSEM<br>https://github.com/deweylab/RSEM
- XAEM<br>https://github.com/WenjiangDeng/XAEM

Gene-level ASTs
- SpliZ<br>https://github.com/salzman-lab/SpliZ/

---
# 📊 Evaluations
We benchmarked the performance of all tools across four core dimensions: Detection, Quantification, Differential Splicing Analysis (DSA), and Utility. Below are the key metrics and their definitions.

## 1. Detection Metrics
To evaluate the ability of tools to correctly identify alternative splicing events.

| Metric | Formula | Description |
|--------|---------|-------------|
| **Number** | - | Total count of splicing events/genes detected by the tool. |
| **Similarity (Dice Similarity Coefficient, DSC)** |$$\text{Dice} = \frac{2 * TP}{(TP + FN) + (TP + FP)} $$ | Measures the overlap between detected events and ground truth events. |
| **Recall (Sensitivity)** | $$\text{Recall} = \frac{TP}{TP + FN}$$ | Proportion of true positive events (TP) among all known true events (TP + FN). Measures the tool's ability to capture all real events. |

## 2. Quantification
To evaluate the accuracy of splicing event quantification.

| Metric                | Formula | Description |
|-----------------------|---------|-------------|
| **Correlation (Pearson)** | $$r = \frac{\sum(x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum(x_i - \bar{x})^2} \cdot \sqrt{\sum(y_i - \bar{y})^2}}$$ | Measures the strength of association between predicted and true values. |
| **Root Mean Squared Error** |  $$\text{RMSE} =\sqrt{\frac{1}{N}\sum_{i=1}^{n}(Y_i - f(x_i))^2}$$ | Measures the average magnitude of the error between predicted values and ground truth. Lower = better.|
| **Error Rate** |  $ER = &#124;\bar{PSI_p} - \bar{PSI_g} &#124; \times 100$  | Measures the relative deviation of predictions from ground truth. |
| **IQR (Interquartile Range)** | $$\text{IQR} = Q_3 - Q_1$$ | Measures the spread of quantification results across replicates or conditions. Lower = more consistent. |

## 3. Differential Splicing Analysis (DSA)
To evaluate the ability to identify statistically significant splicing differences between conditions.

| Metric                | Formula                                                                 | Description                                                                                                 |
|-----------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| **Recall (Sensitivity)** | $`\text{Recall} = \text{Sensitivity} = \text{TPR} = \dfrac{TP}{TP + FN}`$  | Proportion of true positive events (TP) among all known true events (TP + FN). Measures the tool's ability to capture all real events. |
| **Precision**           | $`\text{Precision} = \dfrac{TP}{TP + FP}`$   | Proportion of true positive events (TP) among all events predicted by the tool (TP + FP). Measures the tool's ability to avoid false positives. |
| **Specificity**|$`\text{Specificity} = \text{TNR} = \dfrac{TN}{TN + FP}`$ |Proportion of true negative events (TN) among all true negative events (TN + FP). Measures the tool's ability to correctly identify non-events.|
| **Accuracy**|$`\text{Accuracy} = \dfrac{TP + TN}{TP + FP + TN + FN}`$ |Overall proportion of correct predictions (TP + TN) among all predictions.|
| **F1 Score**            | $`F_1\text{ Score} = 2 \times \dfrac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}`$ | Harmonic mean of precision and recall, providing a balanced measure of detection performance.               |
| **Matthews Correlation Coefficient** |$`\text{MCC} = \dfrac{TP \cdot TN - FP \cdot FN}{\sqrt{(TP + FP) \cdot (TP + FN) \cdot (TN + FP) \cdot (TN + FN)}}`$ |Balanced correlation coefficient, robust to class imbalance, ranges from -1 to 1.|
| **Odds Ratio** |$`\text{OR} = \dfrac{TP \times TN}{FN \times FP}`$|Measures the strength of association between predicted differential events and true differential events.|

## 4. Utility
Practical metrics for usability in real-world applications.

|Metric	                |Description                                                              |
|-----------------------|-------------------------------------------------------------------------|
| **Runtime**  |	Wall-clock time required to process a dataset.|
| **Dependency** |	Complexity of software installation and required dependencies (e.g., Conda/Singularity compatibility, external tools).|
| **User-friendliness** |	Subjective assessment of documentation quality, command-line interface, error handling, and ease of use.|
