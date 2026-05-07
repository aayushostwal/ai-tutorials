.PHONY: install models neo4j notebook stop help

VENV := .venv
PYTHON := $(VENV)/bin/python
PIP := $(VENV)/bin/pip

help:
	@echo "Available targets:"
	@echo "  make install   — create venv and install Python dependencies"
	@echo "  make models    — pull Ollama models (llama3.1:8b, nomic-embed-text)"
	@echo "  make neo4j     — start Neo4j in Docker"
	@echo "  make notebook  — launch JupyterLab"
	@echo "  make stop      — stop the Neo4j Docker container"
	@echo "  make all       — install + models + neo4j (then run 'make notebook' separately)"

all: install models neo4j

install:
	python3 -m venv $(VENV)
	$(PIP) install --upgrade pip
	$(PIP) install -r requirements.txt
	@echo "✓ Dependencies installed. Activate with: source $(VENV)/bin/activate"

models:
	ollama pull llama3.1:8b
	ollama pull nomic-embed-text
	@echo "✓ Ollama models ready"

neo4j:
	docker run -d --rm --name vector-graph-rag-neo4j \
		-p 7474:7474 -p 7687:7687 \
		-e NEO4J_AUTH=neo4j/password \
		neo4j:5
	@echo "✓ Neo4j running at bolt://localhost:7687 (browser: http://localhost:7474)"

notebook:
	$(VENV)/bin/jupyter lab vector_vs_graph_rag_tutorial.ipynb

stop:
	docker stop vector-graph-rag-neo4j
	@echo "✓ Neo4j stopped"
