# 28.08.2025

## Graph-Based Bibliometric Methodology

### Rationale
To systematically map and analyze the literature relevant to *Causal Reinforcement Learning for Smart Charging*, we adopt a **graph-based bibliometric approach**. Instead of pre-clustering papers outside of a graph environment, the literature is represented as a **heterogeneous knowledge graph**, enabling scalable analytics using Neo4j’s Graph Data Science library.

This approach is grounded in best practices from bibliometric research. Empirical comparisons have shown that **direct citation networks** outperform co-citation or bibliographic coupling when the goal is to represent the research front accurately (Boyack & Klavans, 2010; Waltman & van Eck, 2012). Recent surveys further highlight the benefits of graph-based methods for scientific mapping and community detection (Chen et al., 2022).  

### Data Acquisition
- **Primary source**: Semantic Scholar Graph API, providing:
  - Core metadata (paperId, title, abstract, year, venue, fieldsOfStudy, doi, citation/reference counts).
  - References and citations (to build citation edges).
  - Author identifiers and names.
- **Fallback sources**:
  - OpenAlex (for missing DOIs, enriched metadata).
  - CrossRef (for venue normalization).

This ensures a **comprehensive and enriched dataset** suitable for graph construction.

### Graph Schema

**Nodes**
- `Paper {id, title, abstract, year, doi, venue, fieldsOfStudy, citationCount, referenceCount}`
- `Author {id, name}` (optional, for co-authorship networks).
- `Venue {name, type}` (optional, for publication context).
- `Concept {name}` (optional, for extracted keywords or entities).

**Edges**
- `(:Paper)-[:CITES]->(:Paper)`  
  Directed citation relationships between papers (backbone of the graph).
- `(:Paper)-[:SIMILAR_TO {score:float}]->(:Paper)`  
  Weighted semantic similarity edges, based on TF-IDF or embeddings. Useful for **recent papers** that lack citation links.
- `(:Author)-[:COAUTHORED]->(:Paper)`  
- `(:Paper)-[:PUBLISHED_IN]->(:Venue)`  
- `(:Paper)-[:MENTIONS]->(:Concept)`  

In practice, the **minimum viable graph** will consist of **Paper nodes** connected by `CITES` and `SIMILAR_TO` edges, with optional enrichment as needed.

### Analytical Framework (Neo4j GDS)

Once loaded into Neo4j, the graph can be analyzed using the **Graph Data Science (GDS) library**, which supports:

- **Community detection**: Louvain, Leiden, weakly/strongly connected components.
- **Centrality analysis**: PageRank, Eigenvector, Harmonic, Degree centrality to identify influential works.
- **Similarity analysis**: `gds.nodeSimilarity` or embedding-based KNN to cluster related papers.
- **Link prediction**: Adamic-Adar, common neighbors, preferential attachment to infer potential emerging connections.

This hybrid design ensures that both **established literature** (well-cited) and **emerging works** (citation-poor but semantically relevant) are captured.

### References
- Boyack, K. W., & Klavans, R. (2010). Co-citation analysis, bibliographic coupling, and direct citation: Which citation approach represents the research front most accurately? *Journal of the American Society for Information Science and Technology, 61*(12), 2389–2404. https://doi.org/10.1002/asi.21419  

- Waltman, L., & van Eck, N. J. (2012). A new methodology for constructing a publication-level classification system of science. *Journal of the American Society for Information Science and Technology, 63*(12), 2378–2392. https://doi.org/10.1002/asi.22748  

- Chen, C., Song, M., & Heo, G. (2022). Graph-based bibliometric analysis: Methods and applications. *Scientometrics, 127*(6), 3699–3723. https://doi.org/10.1007/s11192-022-04345-6  

Of course. It's an excellent idea to formalize our analytical framework in the log and to plan for the final visuals.

---
**Date:** 03 September 2025
**Subject:** Finalized Analytical Framework and Procedural Plan

The project will now proceed with a three-tiered analytical framework to address the primary research question. The analysis will be conducted using Cypher queries on the populated Neo4j graph. The process will be split into separate, documented Jupyter notebooks for each tier, with all commentary and markdowns written in the third person.

**Tier 1: The Macro View – Quantifying the Landscape (`05_macro_analysis.ipynb`)**
* **Goal:** To establish a high-level, quantitative understanding of the research domain by identifying key entities, their properties, and overall trends.
* **Guiding Questions:**
    * What are the foundational papers (by overall and in-corpus citation count)?
    * Who are the key researchers (by publication count and in-corpus citations)?
    * Where is this research being published (top venues and disciplines)?
    * When did these topics emerge (publication volume over time)?

**Tier 2: The Meso View – Mapping the Structure (`06_meso_analysis.ipynb`)**
* **Goal:** To analyze the network structure of the research field, understanding how entities are connected, how ideas flow, and where structural gaps exist.
* **Guiding Questions:**
    * How are researchers collaborating (co-authorship networks, central hubs, bridge authors)?
    * How do ideas evolve and spread (citation pathways, connections between query topics)?
    * Where are the structural holes and opportunities (weak connections between key concepts)?

**Tier 3: The Micro View – Content Synthesis (`07_content_analysis.ipynb`)**
* **Goal:** To synthesize the content of a focused subset of approximately 50 papers (selected based on insights from Tiers 1 & 2) to directly answer the research question and identify a research gap.
* **Initial Guiding Questions:**
    * What specific causal models and CRL algorithms are proposed for smart charging?
    * How do these methods claim to provide explainability and trust?
    * How is robustness defined and tested in the context of grid stability?
    * What are the common limitations and stated future work in the key literature?

---

Of course. That's an excellent and critical step. A strong methodology must be grounded in established academic principles.

You are right to want to formalize this before proceeding. The methods we've discussed are indeed based on standard, citable concepts from the field of **bibliometrics** and **scientometrics**. The paper you found by Ge et al. (2025) on Knowledge Graph Analysis is a perfect source for us to cite as a precedent for these techniques.

Here is the updated methodology log entry that formalizes our plan with the supporting concepts and citations.

---
### ## Entry for `docs/methodology_log.md`

**Date:** 04 September 2025

**Subject:** Grounding the Phase 1 Selection Methodology in Bibliometric Principles (Ge et al. (2025) )

To ensure a rigorous and defensible paper selection process, the Phase 1 methodology is explicitly grounded in established principles from bibliometrics and scientometrics. The "Two-Bucket" strategy is designed to mitigate the common bias towards older publications by actively seeking both foundational and emerging works.

#### ### Bucket A: Identifying "Foundational Pillars"

The selection of foundational papers is based on a **composite importance score**. This approach, a form of multi-criteria analysis, provides a more robust measure of influence than any single metric alone. The score combines several standard bibliometric indicators:

* [cite_start]**Overall and In-Corpus Citation Counts:** These are standard measures of a paper's impact and influence[cite: 158].
* **Network Influence (PageRank):** This metric assesses a paper's importance within the citation network, where a paper is considered influential if it is cited by other influential papers.
* **Co-citation Analysis:** This technique identifies papers that are frequently cited together, suggesting they are intellectually linked and foundational to a specific school of thought. [cite_start]This use of co-citation networks to understand a field's structure is a standard bibliometric method[cite: 347, 348].

#### ### Bucket B: Identifying "Rising Stars"

To identify important recent papers that have not yet accumulated high citation counts, this methodology employs a **citation velocity** metric.

* [cite_start]This approach is a practical implementation of **citation burst analysis**, a well-established technique for discovering research topics and papers that are gaining significant attention over a short period[cite: 494, 495]. [cite_start]Identifying such "bursts" is critical for mapping emerging trends and cutting-edge directions in a research field[cite: 442, 501].

This refined and academically-grounded methodology provides a transparent and defensible process for selecting a balanced corpus of the most relevant foundational and emerging papers for our deep analysis in Phase 2.

---

Of course. This is the perfect time to consolidate our entire strategy into a single, comprehensive plan. This document will serve as our final roadmap, summarizing what has been achieved and detailing the precise, academically-grounded steps we will take to complete the project.

---
## ## A Methodological Framework for the Research Essay

### **1. Summary of Progress (Phase 1 Completion)**

The initial phase of the project, **Corpus Characterization & Candidate Selection**, has been successfully executed. The goal of this phase was to move from a broad topic to a defensible and balanced list of key academic papers. This was achieved through a systematic, multi-stage process:

1.  **Data Collection:** A corpus of over 2,000 papers relevant to Reinforcement Learning, Causality, and EV Charging was collected from academic APIs (arXiv, Semantic Scholar).
2.  **Knowledge Graph Modeling:** The collected metadata was structured into a rich knowledge graph in Neo4j, capturing entities (Papers, Authors, Venues, etc.) and their relationships (e.g., `CITES`, `AUTHORED`).
3.  **Bibliometric Analysis:** A series of "Macro" and "Meso" analyses were performed on the graph to map the research landscape. This included identifying influential papers, prolific authors, key collaboration networks, and disciplinary trends.
4.  **Candidate Selection:** A rigorous **"Three-Bucket" methodology** was employed to select a final candidate list of 225 papers, ensuring a balance between established and emerging research. This included identifying:
    * **Foundational Pillars:** Highly influential papers based on a composite score of citation counts and network centrality.
    * **Rising Stars:** Recent papers with high "citation velocity," indicating growing impact.
    * **The Pre-publication Frontier:** The newest, most relevant preprints identified via semantic similarity.

The outcome of Phase 1 is a comprehensive, data-backed understanding of the research landscape and a final candidate list of 225 papers ready for deep analysis.

---
### ### 2. Detailed Plan for Phase 2: Targeted Analysis & Final Selection

**Goal:** The primary goal of Phase 2 is to conduct a **Targeted Synthesis & Argument Construction**. This involves using a hybrid computational and qualitative approach to analyze the 225 candidate papers, extract their core arguments, and produce a final, defensible shortlist of ~50-75 papers for in-depth manual reading.

**Academic Grounding:** This hybrid methodology is a form of **Computationally-Assisted Systematic Review**. Our approach is directly supported by recent academic literature. The use of LLMs for classification and extraction is a primary application of AI in modern systematic reviews (Sundaram & Berleant, 2023). Specifically, our plan to use a large language model to "extract and categorize key themes and concepts" is a practical application of the methods demonstrated by Gana et al. (2024) and is a core component of end-to-end AI-driven review frameworks like PROMPTHEUS (Torres et al., 2025).

#### **Step 2.1: Top-Down Thematic Classification**

* **Methodology:** Instead of unsupervised clustering, a top-down classification approach will be used, guided by the primary research question. The 225 candidate papers will be programmatically scored against three pre-defined **Thematic Pillars**:
    1.  **Pillar 1: Causal Reinforcement Learning (CRL) Methods**
    2.  **Pillar 2: EV Smart Charging Applications**
    3.  **Pillar 3: Explainability & Trust**
* To ensure reproducibility, this scoring will be performed using the **Gemini API** with the `temperature` set to `0` and a fixed model version. All prompts and raw API responses will be logged.
* **Outcome:** A structured dataset where each of the 225 papers has a numerical relevance score for each of the three pillars.

#### **Step 2.2: LLM-Powered Structured Information Extraction**

* **Methodology:** The Gemini API will be used to extract key structured data from the abstract of each of the 225 papers. Targeted prompts will be designed to identify and extract:
    * **Stated Methodology** (e.g., "Algorithm Development," "Case Study," "Survey")
    * **Core Contribution** (a one-sentence summary of the main finding)
* **Outcome:** The dataset will be further enriched with structured tags, enabling more granular sorting and filtering.

#### **Step 2.3: Final Selection of the Reading List (50-75 Papers)**

* **Methodology:** The final reading list will be curated from the "smart" dataset created in the previous steps, using a balanced portfolio strategy to ensure comprehensive coverage. The selection will include:
    * **"Triple Threats":** The top 15-20 papers scoring highly across all three pillars.
    * **"Pillar Champions":** The top 10-15 papers from each individual pillar.
    * **"Bridge Papers":** Important interdisciplinary works with high scores in two pillars.
* The **"Foundational Score"** and **"Rising Star"** status calculated in Phase 1 will be used as a tie-breaker.
* **Outcome:** A final, justified list of 50-75 papers for deep manual reading.

---
### ### 3. Detailed Plan for Phase 3: Qualitative Synthesis & Essay Composition

**Goal:** To perform a deep reading of the final selection of papers, synthesize the findings to construct the central argument that answers the research question, and compose the final essay.

#### **Step 3.1: Guided Manual Reading**

* **Methodology:** The final 50-75 papers will be read in-depth, guided by our Tier 3 content-based questions (e.g., "How is 'robustness' defined and tested?," "What are the common 'limitations' and 'future work' sections?").

#### **Step 3.2: Argument Construction & Gap Identification**

* **Methodology:** The findings from the manual reading will be synthesized with the structured data extracted by the LLM in Phase 2. The primary goal is to formulate the "red line"—the central thesis that answers the main research question. A key part of this is to clearly articulate the **research gap** by synthesizing the limitations and open questions identified across the most important papers.

#### **Step 3.3: Essay Writing**

* **Methodology:** The final essay will be structured according to the university's guidelines. The data and insights from our three-phase process will be mapped directly to the essay's structure:
    * **Introduction:** The rationale will be supported by our temporal analysis from Phase 1, showing the topic's growing importance. The research question will be followed by a clear statement of the identified research gap.
    * **Main Body:** The argument will be structured around the Thematic Pillars. Each section will present the synthesized findings from the deep reading (Phase 3), supported by quantitative evidence from our landscape analysis (Phase 1).
    * **Conclusion:** The essay will conclude by summarizing the argument, directly answering the research question, and proposing the identified research gap as a compelling avenue for future work (forming the basis for a Master's thesis).

### References

* Gana, B., Leiva-Araos, A., Allende-Cid, H., & García, J. (2024). Leveraging LLMs for Efficient Topic Reviews. *Applied Sciences, 14*(17), 7675. [https://doi.org/10.3390/app14177675](https://doi.org/10.3390/app14177675)
* Sundaram, G., & Berleant, D. (2023). Automating Systematic Literature Reviews with Natural Language Processing and Text Mining: a Systematic Literature Review. In *Proceedings of the Eighth International Congress on Information and Communication Technology (ICICT 2023)* (Vol. 1, pp. 73-92). Springer.
* Torres, J., Mulligan, C., Jorge, J., & Moreira, C. (2025). PROMPTHEUS: A Human-Centered Pipeline to Streamline Systematic Literature Reviews with Large Language Models. *Information, 16*(5), 420. [https://doi.org/10.3390/info16050420](https://doi.org/10.3390/info16050420)

---