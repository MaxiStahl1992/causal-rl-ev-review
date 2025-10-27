# Causal Reinforcement Learning in Electric Vehicle Applications: A Systematic Review

This repository contains the data, analysis notebooks, and visualizations for a systematic literature review on causal reinforcement learning methods applied to electric vehicle domains.

## Project Structure

```
├── data/
│   ├── raw/                    # Original data sources
│   ├── processed/              # Cleaned and processed datasets
│   └── artifacts/              # Generated embeddings and intermediate files
├── notebooks/                  # Jupyter notebooks for analysis
├── figs/                      # Generated visualizations
└── requirements.txt           # Python dependencies
```

## Key Notebooks

1. **01_data_collection.ipynb** - Data gathering from academic sources
2. **05_candidate_selection.ipynb** - Paper selection methodology
3. **06_llm_analysis_and_selection.ipynb** - LLM-powered thematic analysis
4. **07_qualitative_synthesis.ipynb** - Qualitative analysis and synthesis
5. **08_thematic_synthesis.ipynb** - Thematic classification results
6. **09_cluster_map.ipynb** - Cluster visualization and analysis

## Main Findings

The review identified 70 papers across four main thematic clusters:

- **Causal Methods** (19 papers) - Causal discovery and modeling approaches
- **Economic Optimisation** (18 papers) - Cost optimization and economic modeling
- **Grid Stability & Services** (26 papers) - Grid integration and stability services
- **Explainability & Interpretability** (7 papers) - Trust and transparency methods

## Getting Started

1. Install dependencies:

```bash
pip install -r requirements.txt
```

2. Run the notebooks in order, starting with data collection

## Data Sources

- Semantic Scholar API
- OpenAlex database
- Neo4j knowledge graph (requires manual setup)
- Manual curation and expert review

## Citation

If you use this work, please cite [paper details to be added].
