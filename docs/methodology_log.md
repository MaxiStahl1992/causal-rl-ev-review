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