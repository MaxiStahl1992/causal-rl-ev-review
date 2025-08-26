**Date:** 22 August 2025
**Subject:** Foundational Methodology for AI-Driven Literature Review

Traditional Systematic Literature Reviews (SLRs) are often labor-intensive, time-consuming, and prone to error, particularly in rapidly evolving research fields[cite: 968, 2733]. Recent studies highlight significant gaps in the automation of critical phases such as data extraction, quality assessment, and data synthesis[cite: 2709, 2944, 3001, 3006]. To address these challenges, this study adopts a semi-automated pipeline that leverages Large Language Models (LLMs) and advanced Natural Language Processing (NLP) to enhance the efficiency and rigor of the review process[cite: 182, 969].
Our approach is directly informed by recently proposed frameworks like PROMPTHEUS, which demonstrate the feasibility of an end-to-end automated SLR pipeline[cite: 994, 995, 1131]. The methodology consists of three main stages. First, a **systematic search and screening** phase will use an LLM to generate a robust search query and Sentence-BERT embeddings to filter for the most relevant papers based on semantic similarity[cite: 1131, 1142, 1181]. Second, a **data extraction and topic modeling** phase will employ the BERTopic algorithm to cluster the selected literature into coherent themes[cite: 214, 1001, 1202]. This technique leverages UMAP for dimensionality reduction and HDBSCAN for clustering, providing a robust method for thematic analysis[cite: 344, 345, 386]. Finally, we will construct and analyze a **knowledge graph** to map the relationships between concepts, papers, and authors, providing a structural overview of the research landscape[cite: 1717, 2277]. This comprehensive, AI-assisted methodology aims to produce a systematic and data-driven analysis while significantly reducing the manual workload associated with traditional review methods[cite: 970].

---

**Date:** 25 August 2025
**Subject:** Refined Plan for Analysis Phase: Topic Modeling and Knowledge Graph

With a clean corpus of 1,652 unique papers established, the project now moves into the analysis phase. This phase will be executed in two main stages: (1) Topic Modeling to identify thematic clusters, and (2) Knowledge Graph construction to map the relationships within the research landscape.

### Stage 1: Topic Modeling (03_topic_modeling.ipynb)
The primary objective of this stage is to automatically discover and label the core research themes present in the master_corpus.csv.

Algorithm: We will use the BERTopic algorithm, which leverages transformer embeddings for semantic understanding and HDBSCAN for clustering.

Number of Topics: A key advantage of this approach is that we do not need to pre-define the number of topics. BERTopic's use of HDBSCAN will automatically determine the optimal number of clusters based on the data's density, ensuring a data-driven result.

Output: The process will yield a new DataFrame where each paper from our corpus is assigned a topic_id and a descriptive topic_name. This topic-enriched dataset will be the primary input for the knowledge graph.

### Stage 2: Knowledge Graph Construction and Analysis (04_knowledge_graph_analysis.ipynb)
The goal of this stage is to model our dataset as a rich network to uncover deeper structural insights.

Technology: We will use a Neo4j graph database.

Schema: The graph will be structured with three types of nodes (Paper, Author, Topic) and two primary relationships (AUTHORED, BELONGS_TO). This model will allow for complex, multi-dimensional queries.

Analysis: We will query the graph to answer an expanded set of analytical questions, including identifying influential authors and papers within specific topics, mapping collaboration networks, and discovering authors who act as bridges between different research themes. This will provide the data-driven narrative for the final research essay.

---
**Date** 26 August 2025
**Subject** Methodology Update: From Embedding-Only to Citation-Anchored Clustering

**Summary**
We’re shifting from pure embedding-based topic modeling toward a citation-anchored clustering approach. This method anchors clusters in the scholarly network—more stable, interpretable, and grounded in how researchers actually structure knowledge.
	1.	Construct a Citation Graph
Each research paper becomes a node; directed or undirected edges represent citation links, capturing the academic field’s intellectual structure (Šubelj, van Eck, & Waltman, 2015; Klavans & Boyack, 2015).
	2.	Apply the Leiden Algorithm for Community Detection
We use the Leiden algorithm (Traag, Waltman, & van Eck, 2019), which outperforms Louvain by guaranteeing that each community is internally connected, converging to locally optimal partitions and doing so faster (Traag et al., 2019).
	3.	Why This Beats Topic-Only Clustering
Head-to-head evaluations show that citation-based clustering (CC) more faithfully captures scientific micro-communities, whereas topic modeling (TM) may obscure them due to thematic overlap (Xie & Waltman, 2023).
	4.	Labeling Communities with Text + Entities
After detecting communities, we generate human-readable labels using class-based TF-IDF on titles and abstracts (as in BERTopic methodology), enhanced with domain-specific entities (e.g., causal inference terms, grid vocabulary) via scientific NER.

**References (APA Style)**
	•	Klavans, R., & Boyack, K. W. (2015). Which type of citation analysis generates the most accurate taxonomy of scientific and technical knowledge? arXiv. https://doi.org/10.48550/arXiv.1511.05078
	•	Šubelj, L., van Eck, N. J., & Waltman, L. (2015). Clustering scientific publications based on citation relations: A systematic comparison of different methods. arXiv. https://doi.org/10.48550/arXiv.1512.09023
	•	Traag, V. A., Waltman, L., & van Eck, N. J. (2019). From Louvain to Leiden: Guaranteeing well-connected communities. Scientific Reports. https://doi.org/10.1038/s41598-019-41695-z
	•	Xie, Q., & Waltman, L. (2023). A comparison of citation-based clustering and topic modeling for science mapping. ArXiv. https://doi.org/10.48550/arXiv.2309.06160

⸻