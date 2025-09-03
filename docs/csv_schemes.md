# Papers.csv (nodes)

One row per paper. paperId from Semantic Scholar is the primary key. Keep a DOI if present for cross-refs.

column	type	required	description
paper_id	STRING	✅	Semantic Scholar paperId (e.g., ARXIV:2101.00001 or S2:XXXXXXXX)
doi	STRING	optional	DOI (normalized lowercase; with or without https://doi.org/)
title	STRING	✅	Paper title
abstract	STRING	optional	Abstract text (can be empty)
year	INTEGER	optional	Publication year
venue	STRING	optional	Journal/conference name
fields_of_study	STRING	optional	Pipe-separated list (e.g., `Computer Science
citation_count	INTEGER	optional	Total citations (from S2)
reference_count	INTEGER	optional	Total references (from S2)
s2_url	STRING	optional	S2 URL (handy for UI)
openalex_id	STRING	optional	OpenAlex Work ID (e.g., W123456789) if resolved
added_at	DATETIME	optional	ETL timestamp (ISO 8601)

Header example

paper_id,doi,title,abstract,year,venue,fields_of_study,citation_count,reference_count,s2_url,openalex_id,arxiv_id,added_at,source

Row example

S2:1a2b3c,,Causal RL for EV Smart Charging,"We propose...",2024,IEEE TPS,Computer Science|Energy,12,35,https://www.semanticscholar.org/paper/1a2b3c,W4210706243,2401.12345,2025-08-28T10:00:00Z,SemanticScholar


⸻

# Cites.csv (edges: (:Paper)-[:CITES]->(:Paper))

Directed edges from the citing paper to the referenced paper.

column	type	required	description
src_paper_id	STRING	✅	Citing paper’s paper_id
dst_paper_id	STRING	✅	Cited paper’s paper_id
src_year	INT	optional	Year of the citing paper (for temporal filtering)
dst_year	INT	optional	Year of the cited paper
source	STRING	optional	SemanticScholar
added_at	DATETIME	optional	ETL timestamp

Header

src_paper_id,dst_paper_id,src_year,dst_year,source,added_at

Row example

S2:1a2b3c,S2:9f8e7d,2024,2020,SemanticScholar,2025-08-28T10:00:00Z


⸻

# Similarity.csv (edges: (:Paper)-[:SIMILAR_TO {score}]→(:Paper))

Undirected or symmetric-directed edges capturing semantic proximity. Build via TF-IDF or sentence embeddings; store top-k per node (e.g., k=10). If you add both directions, set undirected=true in your GDS projections.

column	type	required	description
src_paper_id	STRING	✅	Paper A
dst_paper_id	STRING	✅	Paper B
score	FLOAT	✅	Similarity in [0,1] (cosine)
method	STRING	optional	tfidf, MiniLM, SciBERT, etc.
k	INT	optional	k used in KNN
added_at	DATETIME	optional	ETL timestamp

Header

src_paper_id,dst_paper_id,score,method,k,added_at

Row

S2:1a2b3c,S2:7a6b5c,0.73,MiniLM,10,2025-08-28T10:00:00Z


⸻

# Authors.csv (optional node file: (:Author))

column	type	required	description
author_id	STRING	✅	S2 authorId
name	STRING	✅	Author name
orcid	STRING	optional	ORCID if available

Header

author_id,name,orcid

Row

A:998877,Jane Doe,0000-0002-1825-0097


⸻

# Authorship.csv (edges: (:Author)-[:COAUTHORED]->(:Paper))

column	type	required	description
author_id	STRING	✅	From Authors.csv
paper_id	STRING	✅	From Papers.csv
position	INT	optional	Author order
added_at	DATETIME	optional	ETL timestamp

Header

author_id,paper_id,position,added_at

Row

A:998877,S2:1a2b3c,1,2025-08-28T10:00:00Z


⸻

# Venues.csv (optional node file: (:Venue))

column	type	required	description
venue_id	STRING	✅	Deterministic hash of name/type or OpenAlex venue id
name	STRING	✅	Venue name
type	STRING	optional	journal, conference, etc.

Header

venue_id,name,type

Row

V:ieee_tps,IEEE Transactions on Power Systems,journal


⸻

# PublishedIn.csv (edges: (:Paper)-[:PUBLISHED_IN]->(:Venue))

column	type	required	description
paper_id	STRING	✅	Paper id
venue_id	STRING	✅	Venue id
year	INT	optional	Year

Header

paper_id,venue_id,year

Row

S2:1a2b3c,V:ieee_tps,2024


⸻

# Concepts.csv (optional node file: (:Concept))

column	type	required	description
concept_id	STRING	✅	Slug (e.g., causal-reinforcement-learning)
name	STRING	✅	Human-readable name

Header

concept_id,name

Row

causal-rl,Causal Reinforcement Learning


⸻

# Mentions.csv (edges: (:Paper)-[:MENTIONS]->(:Concept))

column	type	required	description
paper_id	STRING	✅	Paper id
concept_id	STRING	✅	Concept id
weight	FLOAT	optional	TF-IDF or keyphrase weight

Header

paper_id,concept_id,weight

Row

S2:1a2b3c,causal-rl,0.42


⸻

# Neo4j setup (constraints + quick import)

Constraints

CREATE CONSTRAINT paper_id_unique IF NOT EXISTS
FOR (p:Paper) REQUIRE p.paper_id IS UNIQUE;

CREATE CONSTRAINT author_id_unique IF NOT EXISTS
FOR (a:Author) REQUIRE a.author_id IS UNIQUE;

CREATE CONSTRAINT venue_id_unique IF NOT EXISTS
FOR (v:Venue) REQUIRE v.venue_id IS UNIQUE;

CREATE CONSTRAINT concept_id_unique IF NOT EXISTS
FOR (c:Concept) REQUIRE c.concept_id IS UNIQUE;

Bulk import (CSV → nodes/edges)
Use neo4j-admin database import for a fresh DB (fastest) or use LOAD CSV/apoc.periodic.iterate for incremental loads.

Example (incremental, safe)

// Papers
USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM 'file:///Papers.csv' AS row
MERGE (p:Paper {paper_id: row.paper_id})
SET p.title = row.title,
    p.abstract = row.abstract,
    p.year = toInteger(row.year),
    p.doi = row.doi,
    p.venue = row.venue,
    p.fields_of_study = row.fields_of_study,
    p.citation_count = toInteger(row.citation_count),
    p.reference_count = toInteger(row.reference_count),
    p.s2_url = row.s2_url,
    p.openalex_id = row.openalex_id,
    p.arxiv_id = row.arxiv_id,
    p.source = row.source,
    p.added_at = row.added_at;

// CITES
USING PERIODIC COMMIT 10000
LOAD CSV WITH HEADERS FROM 'file:///Cites.csv' AS row
MATCH (src:Paper {paper_id: row.src_paper_id})
MATCH (dst:Paper {paper_id: row.dst_paper_id})
MERGE (src)-[e:CITES]->(dst)
SET e.src_year = toInteger(row.src_year),
    e.dst_year = toInteger(row.dst_year),
    e.source = row.source,
    e.added_at = row.added_at;

// SIMILAR_TO (store as undirected by mirroring edges or set a property)
USING PERIODIC COMMIT 10000
LOAD CSV WITH HEADERS FROM 'file:///Similarity.csv' AS row
MATCH (a:Paper {paper_id: row.src_paper_id})
MATCH (b:Paper {paper_id: row.dst_paper_id})
MERGE (a)-[s:SIMILAR_TO]->(b)
SET s.score = toFloat(row.score),
    s.method = row.method,
    s.k = toInteger(row.k),
    s.added_at = row.added_at;

GDS projection (example)

// Citation-only directed projection
CALL gds.graph.project(
  'papers_cites',
  'Paper',
  { CITES: { orientation: 'NATURAL' } }
);

// Citation + Similarity undirected weighted projection
CALL gds.graph.project(
  'papers_cites_sim',
  'Paper',
  {
    CITES: { orientation: 'UNDIRECTED' },
    SIMILAR_TO: { orientation: 'UNDIRECTED', properties: 'score' }
  }
);


⸻

Mapping from Semantic Scholar fields
	•	paperId → paper_id
	•	externalIds.DOI → doi
	•	title, abstract → title, abstract
	•	year → year
	•	venue → venue
	•	fieldsOfStudy[] → fields_of_study (join with |)
	•	citationCount, referenceCount → counts
	•	url → s2_url
	•	authors[].authorId/name → Authors.csv + Authorship.csv
	•	references[].paperId → Cites.csv (as dst_paper_id)
	•	citations[].paperId → (optional; usually you only need references to build edges)

⸻

Minimal ETL checklist
	1.	Pull batch metadata for your 1650 seed paperIds (and recursively their references if you want more coverage).
	2.	Normalize IDs, dedupe by paper_id.
	3.	Build Papers.csv and Cites.csv (SIMILAR_TO optional initially).
	4.	Import into Neo4j, create constraints, then run GDS algorithms:
	•	CC/WCC, Louvain/Leiden,
	•	PageRank/Eigenvector/Harmonic,
	•	nodeSimilarity / KNN on embeddings,
	•	Adamic-Adar link prediction.

