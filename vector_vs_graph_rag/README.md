# Vector vs Graph RAG with Local Ollama

This tutorial compares two retrieval patterns over the same small corpus of noisy enterprise documents:

1. `Vector RAG`: embed each long document, run similarity search, and pass the top match to an LLM.
2. `Graph RAG`: extract entities and relationships from prose, traverse connected nodes, and pass the structured context to an LLM.

The notebook is designed to run locally first and then show what you would change for production. The comparison is intentionally set up to stress a **naive vector baseline** over messy company docs, so the limitations are visible instead of hidden.

## Files

| Path | Purpose |
| --- | --- |
| `vector_vs_graph_rag_tutorial.ipynb` | End-to-end tutorial notebook with setup, retrieval logic, benchmarks, and production notes |
| `data/*.txt` | Realistic company-style onboarding, policy, incident, and launch documents used by both retrieval methods |
| `requirements.txt` | Lightweight Python dependencies for the notebook |

## Local setup

1. Create and activate a Python environment.
2. Install the dependencies:

```bash
pip install -r requirements.txt
```

3. Install Ollama and pull the models used in the notebook:

```bash
ollama pull llama3.1:8b
ollama pull nomic-embed-text
ollama serve
```

4. Open the notebook:

```bash
jupyter lab
```

## What the notebook covers

- Why long enterprise documents are a bad fit for a naive document-level vector baseline
- How to build both retrieval approaches from the same noisy corpus
- How to extract a graph from prose instead of pre-labeled edges
- How to measure retrieval latency and answer support on a benchmark designed around multi-hop operational questions
- What needs to change when you move from local development to production

## Production direction

The notebook keeps everything local and transparent on purpose. For production, the main changes are:

- Replace the notebook's naive document-level vector search with chunked retrieval in a vector database such as pgvector, Qdrant, Weaviate, or Milvus
- Replace the in-memory `networkx` graph with a graph database such as Neo4j or Amazon Neptune
- Add offline evaluation, request tracing, caching, retries, and authorization boundaries
- Treat the Ollama-based flow as a local development baseline, then swap the generation and embedding backends behind stable interfaces

## Suggested flow

Start with the notebook, then use the tutorial README as the checklist for hardening the design into a service.
