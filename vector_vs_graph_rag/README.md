# Vector RAG vs Graph RAG — Tutorial

A hands-on Jupyter notebook that compares naive vector retrieval against graph-oriented retrieval on a small corpus of interconnected company documents. The tutorial covers the basics and then goes deeper into **production failure modes** with working code fixes.

---

## What this tutorial covers

### Part 1 — The core comparison
- Why **naive document-level vector RAG** breaks down on multi-hop enterprise questions
- Building a vector baseline (one embedding per full document, cosine similarity retrieval)
- Extracting entity relationships from prose using deterministic rules
- Loading a knowledge graph into **Neo4j** and retrieving via Cypher traversal
- A 6-question benchmark designed to stress the vector baseline
- Side-by-side answer generation using both methods with a local **Ollama** LLM

### Part 2 — Production failure modes (with code)
Five failure modes that teams commonly encounter after their initial graph RAG implementation, each demonstrated and then fixed:

| # | Failure | What goes wrong | Fix shown |
|---|---|---|---|
| 1 | **Entity Disambiguation** | "Delta Battery" doesn't match "Delta Batteries" → zero seed nodes, empty context | Fuzzy matching with `SequenceMatcher` |
| 2 | **Brittle Extraction** | Paraphrased relationships are invisible to regex patterns | LLM-based extraction prompt |
| 3 | **Shallow Graph Depth** | 4-hop causal chain only partially covered at `max_hops=2` | Per-query-type adaptive hop depth |
| 4 | **Hallucination** | LLM fabricates answers when retrieved context doesn't support the query | Safe refusal prompt + post-hoc grounding check |
| 5 | **Latency Bottlenecks** | Repeated embedding calls dominate P95 latency | In-process embedding cache |

---

## The corpus

Five short (~130 word) documents about a fictional company, **Lumina Manufacturing**. All five share the same entities and are interconnected so that most interesting questions require facts from multiple documents:

| File | Content |
|---|---|
| `01_employee_onboarding_handbook.txt` | Product stack (Orion Sensor → Atlas Gateway → Nimbus Analytics), team ownership, regional SLAs |
| `02_procurement_and_vendor_policy.txt` | Supplier links (BlueRiver Circuits → Orion Sensor, Delta Batteries → Atlas Gateway), fallback routing |
| `03_monsoon_incident_retro.txt` | Cascading failure: Monsoon → Chennai Port → Harbor Logistics → Delta Batteries → Atlas Gateway → Pune at risk |
| `04_launch_readiness_brief.txt` | Launch dependencies, regional priority (Pune before Singapore), dashboard requirements |
| `05_support_operating_playbook.txt` | SLAs, two-path escalation (Platform Reliability vs Field Engineering), multi-document reasoning |

---

## Local setup

```bash
make install   # create venv + install dependencies (includes JupyterLab)
make models    # pull llama3.1:8b and nomic-embed-text via Ollama
make neo4j     # start Neo4j 5 in Docker
make notebook  # launch JupyterLab
```

To stop Neo4j when you're done:

```bash
make stop
```

Override Neo4j defaults if needed:

```bash
export NEO4J_URI=bolt://localhost:7687
export NEO4J_USERNAME=neo4j
export NEO4J_PASSWORD=password
export NEO4J_DATABASE=neo4j
```

Open the notebook from the `vector_vs_graph_rag/` directory so relative paths resolve correctly.

---

## Key takeaway

Naive vector retrieval is the right starting point — and often the wrong stopping point. When questions require connecting supplier dependencies, escalation ownership, routing fallbacks, or cascading failure chains across multiple documents, graph RAG becomes the correct next layer.

The production failure modes section shows that deploying graph RAG reliably also requires fuzzy entity matching, robust extraction, adaptive traversal depth, grounding verification, and latency management. Each failure mode is silent (no exception — just a wrong or empty answer), which is exactly why they matter in real deployments.
