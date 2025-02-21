## Indexing Phase:

1.  **Document:**
    *   **Input:** The raw text:
        ```
        Tesla reported total revenue of $23.35 billion in Q3 2023, marking a 9% increase year-over-year (YoY) from $21.45 billion in Q3 2022. The automotive segment remained the primary revenue driver, contributing $19.63 billion, up 5% YoY. Energy generation and storage revenue saw significant growth, reaching $1.56 billion, a 40% increase YoY.
        ```
    *   **Action:** This is the starting point - your piece of information.

2.  **Document Parsing:**
    *   **Input:** The raw text (already quite clean in this case).
    *   **Action:** In this simple example, parsing is minimal. If it were a PDF or webpage, this step would involve extracting the text content and cleaning it (removing HTML tags, etc.). Here, we essentially just pass the text through.
    *   **Output:** Clean text:
        ```
        Tesla reported total revenue of $23.35 billion in Q3 2023, marking a 9% increase year-over-year (YoY) from $21.45 billion in Q3 2022. The automotive segment remained the primary revenue driver, contributing $19.63 billion, up 5% YoY. Energy generation and storage revenue saw significant growth, reaching $1.56 billion, a 40% increase YoY.
        ```

3.  **Document Chunking:**
    *   **Input:** Clean text.
    *   **Action:** Let's say we use sentence-based chunking for simplicity. We'll split the text into sentences:
        *   **Chunk 1:** `"Tesla reported total revenue of $23.35 billion in Q3 2023, marking a 9% increase year-over-year (YoY) from $21.45 billion in Q3 2022."`
        *   **Chunk 2:** `"The automotive segment remained the primary revenue driver, contributing $19.63 billion, up 5% YoY."`
        *   **Chunk 3:** `"Energy generation and storage revenue saw significant growth, reaching $1.56 billion, a 40% increase YoY."`
    *   **Output:** List of document chunks.

4.  **Contextualization (Contextual Retrieval):**
    *   **Input:** Original Document Text (all sentences together) and each Chunk (Chunk 1, Chunk 2, Chunk 3).
    *   **Action:** For each chunk, we use an LLM (like GPT-4) with a prompt to generate context based on the entire document. Let's imagine the LLM generates these contexts:
        *   **Context for Chunk 1:**  "This document provides a financial summary of Tesla's performance in Q3 2023. This section introduces the overall revenue figures and year-over-year growth."
        *   **Context for Chunk 2:** "Continuing the Tesla Q3 2023 revenue report, this section focuses on the automotive division, highlighting its role as the main revenue source."
        *   **Context for Chunk 3:** "Completing the Q3 2023 revenue breakdown, this section details the energy generation and storage sector's revenue and its significant growth rate."
    *   **Output: Contextualized Chunks:**
        *   **Contextualized Chunk 1:**
            ```
            Context: This document provides a financial summary of Tesla's performance in Q3 2023. This section introduces the overall revenue figures and year-over-year growth.

            Chunk Content: Tesla reported total revenue of $23.35 billion in Q3 2023, marking a 9% increase year-over-year (YoY) from $21.45 billion in Q3 2022.
            ```
        *   **Contextualized Chunk 2:**
            ```
            Context: Continuing the Tesla Q3 2023 revenue report, this section focuses on the automotive division, highlighting its role as the main revenue source.

            Chunk Content: The automotive segment remained the primary revenue driver, contributing $19.63 billion, up 5% YoY.
            ```
        *   **Contextualized Chunk 3:**
            ```
            Context: Completing the Q3 2023 revenue breakdown, this section details the energy generation and storage sector's revenue and its significant growth rate.

            Chunk Content: Energy generation and storage revenue saw significant growth, reaching $1.56 billion, a 40% increase YoY.
            ```

5.  **BM25 Indexing (on Contextualized Chunks):**
    *   **Input:** Contextualized Chunk 1, 2, and 3.
    *   **Action:** BM25 Indexing processes each Contextualized Chunk:
        *   **Tokenization:**  Breaks each chunk (including context and content) into tokens. For example, for Contextualized Chunk 3, tokens would include: "Context", "Completing", "Tesla", "Q3", "2023", "revenue", "breakdown", "section", "details", "energy", "generation", "storage", "sector", "revenue", "significant", "growth", "rate", "Chunk", "Content", "Energy", "generation", "storage", "revenue", "saw", "significant", "growth", "reaching", "$1.56", "billion", "40", "increase", "YoY".
        *   **TF Calculation:** Counts term frequencies within each Contextualized Chunk.  "revenue" appears multiple times in Chunk 3, so its TF will be higher there.
        *   **IDF Calculation:** Calculates Inverse Document Frequency for each term across all Contextualized Chunks. "Tesla" or "revenue" might have moderate IDF, while very common words like "the", "of", "a" will have low IDF.  More specific terms might have higher IDF if they are rarer across all indexed content (though in this tiny example, it's less relevant).
        *   **Inverted Index Creation:** Builds an inverted index mapping tokens to Contextualized Chunks and their associated weights.
    *   **Output:** **BM25 Index**.

6.  **Embedding Generation (for Semantic Search):**
    *   **Input:** Contextualized Chunk 1, 2, and 3 (same as for BM25).
    *   **Action:** For each Contextualized Chunk, send it to an embedding model (e.g., Azure OpenAI Embeddings). The model generates a vector embedding that represents the semantic meaning of the *entire Contextualized Chunk* (context + content).
    *   **Output:** **Vector Embeddings** for each Contextualized Chunk. Let's say we get vectors V1, V2, V3 for Chunk 1, 2, 3 respectively.

7.  **Vector Database Indexing:**
    *   **Input:** Vector Embeddings V1, V2, V3.
    *   **Action:** Store these vectors in a Vector Database (like FAISS). The database indexes these vectors for efficient similarity search.
    *   **Output:** **Vector Database** with indexed embeddings.

## Retrieval Phase: (Example Query: "What was Tesla's energy revenue in Q3 2023?")

1.  **User Query:**
    *   **Input:** `"What was Tesla's energy revenue in Q3 2023?"`

2.  **Query Processing:**
    *   **Action:** Tokenize the query: `["What", "was", "Tesla's", "energy", "revenue", "in", "Q3", "2023", "?"]`

3.  **BM25 Retrieval:**
    *   **Input:** Processed query tokens and **BM25 Index**.
    *   **Action:** Search the BM25 Index using the query tokens. BM25 algorithm calculates scores for each Contextualized Chunk based on term matching, TF, IDF, etc.
    *   **BM25 Results:** Ranks Contextualized Chunks based on BM25 scores.  Likely, **Contextualized Chunk 3** would get a high score.

4.  **Semantic Retrieval:**
    *   **Input:** User Query and **Vector Database**.
    *   **Action:**
        *   Generate an embedding for the user query using the *same* embedding model used for chunk embeddings. Let's say the query embedding is V_query.
        *   Perform a similarity search in the Vector Database to find the Contextualized Chunk embeddings (V1, V2, V3) that are most similar to V_query (e.g., using cosine similarity).
    *   **Semantic Results:** Ranks Contextualized Chunks based on semantic similarity. Again, **Contextualized Chunk 3** is very likely to be ranked highly.

5.  **Rank Fusion (Optional):**
    *   **Input:** Ranked lists from **BM25 Retrieval** and **Semantic Retrieval**.
    *   **Action:** Combine the rankings (e.g., using Reciprocal Rank Fusion) to produce a single, unified ranked list of Contextualized Chunks.

6.  **Reranking (Optional):**
    *   **Input:** Ranked list of Contextualized Chunks (from Rank Fusion or just from the best individual retrieval method).
    *   **Action:**  Use a reranking model (like Cohere Rerank) to further refine the ranking.

7.  **Relevant Chunks:**
    *   **Output:** Top-ranked **Contextualized Chunk(s)**.  In this scenario, **Contextualized Chunk 3** is highly likely to be selected.

8.  **Answer Generation (LLM):**
    *   **Input:**  User Query: "What was Tesla's energy revenue in Q3 2023?" and **Relevant Chunk** (Contextualized Chunk 3):
        ```
        Context: Completing the Q3 2023 revenue breakdown, this section details the energy generation and storage sector's revenue and its significant growth rate.

        Chunk Content: Energy generation and storage revenue saw significant growth, reaching $1.56 billion, a 40% increase YoY.
        ```
    *   **Action:** Send a prompt to an LLM (like Azure OpenAI's GPT-4) that includes the user query and the relevant chunk. The prompt instructs the LLM to answer the question based on the provided information.

9.  **Answer to User:**
    *   **Output:**  A concise and accurate answer generated by the LLM, such as:  `"Tesla's energy generation and storage revenue in Q3 2023 was $1.56 billion."`
