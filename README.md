<div align="center">
  <h1>IsoformToolsBeachmark</h1>
</div>

## 📌 Overview

In this study, we conducted a comprehensive evaluation of alternative splice analysis tools(ASTs), focusing on their performance in splicing detection, splicing quantification, differential splicing, computational utility, and summarize the results into practical, platform-aware recommendations for AST selection.

<table width="100%" border="0">
  <tr>
    <td colspan="2">
      <img src="https://github.com/user-attachments/assets/9c78a89c-5892-4703-b98d-0f8663632644" width="100%">
    </td>
  </tr>
</table>

---
# ⚙️ Environment Setup
All tools, with the exception of MAJIQ, are provided in Singularity containers to ensure reproducibility and ease of use. Due to specific dependency requirements, BRIE is packaged separately, while all remaining tools are bundled into a single container.

The container files are hosted on Zenodo and can be downloaded using the following links:
- BRIE container: `isoform_brie.sif.zip`  
  https://zenodo.org/records/18885345/files/isoform_brie.sif.zip
- Main tools container: `isoform_tools_v2.sif.zip`  
  https://zenodo.org/records/18885345/files/isoform_tools_v2.sif.zip

Usage
```bash
# Download containers from Zenodo (replace with actual file URLs)
wget https://zenodo.org/records/18885345/files/isoform_brie.sif.zip
wget https://zenodo.org/records/18885345/files/isoform_tools_v2.sif.zip

# Unzip the archives
unzip isoform_brie.sif.zip
unzip isoform_tools_v2.sif.zip

# Run a tool with Singularity
singularity exec isoform_tools_v2.sif python3 -c "import rMATS; print('rMATS environment loaded')"

---
# 📚 Tool Documentation

All 13 tools are documented in the `docs/` directory:

Event-level ASTs
- [BRIE](docs/BRIE%20Pipeline.md)
- [Expedition](docs/Expedition%20Pipeline.md)
- [Leafcutter](docs/Leafcutter%20Pipeline.md)
- [MAJIQ](docs/MAJIQ%20Pipeline.md)
- [MARVEL](docs/MARVEL%20Pipeline.md)
- [Psix](docs/Psix.md)
- [rMATS](docs/rMATS%20Pipeline.md)
- [SUPPA](docs/SUPPA%20Pipeline.md)
- [Whippet](docs/Whippet%20Pipeline.md)

Transcript-level ASTs
- [Kallisto](docs/Kallisto%20Pipeline.md)
- [RSEM](docs/RSEM%20Pipeline.md)
- [XAEM](docs/XAEM%20Pipeline.md)

Gene-level ASTs
- [SpliZ](docs/SpliZ%20Pipeline.md)



