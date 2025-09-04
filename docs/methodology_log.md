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
