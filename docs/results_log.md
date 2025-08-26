**Date:** 25 August 2025
**Subject:** Completion of Data Collection and Preprocessing Phase

The initial data collection phase is complete. We have successfully gathered a corpus of academic literature from two primary sources: ArXiv and Semantic Scholar.
Initial Collection: The initial queries yielded approximately 130 unique papers from ArXiv and over 2,000 from Semantic Scholar.
Preprocessing: The raw data from both sources was loaded, standardized into a common schema, and combined. A robust, multi-stage de-duplication process was performed, prioritizing records with a DOI and then using cleaned titles as a fallback.
Final Corpus: After cleaning and removing entries with missing abstracts, the final master corpus consists of 1,652 unique papers.
Key Finding: The de-duplication process revealed that Semantic Scholar's index is highly comprehensive, as it included nearly all papers initially sourced from ArXiv. The final dataset retains the richer metadata from the Semantic Scholar records while back-filling ArXiv IDs where applicable. The final, cleaned dataset is saved as data/processed/master_corpus.csv.

---

