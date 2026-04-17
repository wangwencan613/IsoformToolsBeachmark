<div align="center">
  <h1>IsoformToolsBeachmark</h1>
</div>

## 📌 Overview

In this study, we conducted a comprehensive evaluation of alternative splice analysis tools, focusing on their performance in splicing detection, splicing quantification, differential splicing, computational utility, and summarize the results into practical, platform-aware recommendations for AST selection.

<table width="100%" border="0">
  <tr>
    <td colspan="2">
      <img src="https://github.com/user-attachments/assets/9c78a89c-5892-4703-b98d-0f8663632644" width="100%">
    </td>
  </tr>
</table>

---
## ⚙️ Environment Setup
All tools are integrated into a single Conda environment.
```bash
# Create and activate the environment
conda env create -f environment.yaml
conda activate isoform_tools

##📚 Tool Documentation

We provide detailed documentation for each evaluated tool, including installation notes, usage examples, and common pitfalls. All Markdown files are stored in the docs/ folder:
Event-level AST tools

BRIE2
Expedition
LeafCutter
MAJIQ2
MARVEL
Psix
rMATS
SUPPA2
Whippet
Transcript-level quantification tools

kallisto
RSEM
XAEM
Gene-level quantification tool

SpliZ
