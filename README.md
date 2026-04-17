<div align="center">
  <h1>IsoformToolsBeachmark</h1>
</div>

# 📌 Overview

In this study, we conducted a comprehensive evaluation of alternative splice analysis tools(ASTs), focusing on their performance in splicing detection, splicing quantification, differential splicing, computational utility, and summarize the results into practical, platform-aware recommendations for AST selection.

<table width="100%" border="0">
  <tr>
    <td colspan="2">
      <img src="https://github.com/user-attachments/assets/9c78a89c-5892-4703-b98d-0f8663632644" width="100%">
    </td>
  </tr>
</table>

---
# 📚 Tool Documentation

All 13 tools are documented in the `docs/` directory:

Event-level ASTs
- [BRIE2](docs/BRIE%20Pipeline.md)
- [Expedition](docs/Expedition%20Pipeline.md)
- [Leafcutter](docs/Leafcutter%20Pipeline.md)
- [MAJIQ2](docs/MAJIQ%20Pipeline.md)
- [MARVEL](docs/MARVEL%20Pipeline.md)
- [Psix](docs/Psix.md)
- [rMATS](docs/rMATS%20Pipeline.md)
- [SUPPA2](docs/SUPPA%20Pipeline.md)
- [Whippet](docs/Whippet%20Pipeline.md)

Transcript-level ASTs
- [Kallisto](docs/Kallisto%20Pipeline.md)
- [RSEM](docs/RSEM%20Pipeline.md)
- [XAEM](docs/XAEM%20Pipeline.md)

Gene-level ASTs
- [SpliZ](docs/SpliZ%20Pipeline.md)


# ⚙️ Environment Setup
All tools, with the exception of MAJIQ2 (requires permission), are provided in Singularity containers to ensure reproducibility and ease of use. Due to specific dependency requirements, BRIE2 is packaged separately, while all remaining tools are bundled into a single container.

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

# Run a tool with Singularity (eg. rMATS)
singularity exec --cleanenv --no-home -B /data/bioinf:/data/bioinf isoform_tools_v2.sif bash
# --cleanenv：清空宿主环境变量，保证容器环境纯净
# --no-home：不挂载宿主home目录，避免配置干扰
# -B /data/bioinf:/data/bioinf：将宿主机数据目录挂载到容器内
- isoform_tools_v2.sif：打包好的转录本分析工具容器
- bash：在容器内启动交互式终端

# Activate conda environment
source /tools/miniforge3/bin/activate
conda activate rmats




