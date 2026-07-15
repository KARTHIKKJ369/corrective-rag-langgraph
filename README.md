# Recall

> A local-first implementation of **Corrective Retrieval-Augmented Generation (CRAG)** powered by **LangGraph, ChromaDB, and Ollama**.

Recall is an experimental AI retrieval system that improves traditional Retrieval-Augmented Generation by **evaluating retrieved context before generation**. Instead of blindly passing retrieved documents to the language model, Recall introduces a corrective feedback loop that grades document relevance, rewrites weak queries, and retries retrieval until sufficient evidence is collected.

Everything runs locally using Ollama and ChromaDB, making Recall suitable for privacy-focused AI applications, research, and experimentation.

---

## Features

- Local-first architecture
- Corrective RAG (CRAG) workflow
- LangGraph state machine orchestration
- Maximum Marginal Relevance (MMR) retrieval
- Automatic relevance grading
- Query rewriting when retrieval quality is poor
- Web, PDF, Markdown and Text ingestion
- Persistent Chroma vector database
- Modular ingestion pipeline
- Configurable local LLMs through Ollama
- Environment-driven configuration

---

# Architecture

```text
                User Question
                      │
                      ▼
              Vector Retrieval
                      │
                      ▼
          Relevance Grading (LLM)
              │             │
      Relevant          Not Relevant
          │                  │
          ▼                  ▼
    Answer Generation   Query Rewrite
          ▲                  │
          └──────────────────┘
                 Retry
```

The system continuously improves retrieval quality before allowing answer generation.

---

# Technology Stack

| Component | Technology |
|------------|------------|
| Orchestration | LangGraph |
| LLM | Ollama |
| Embeddings | Sentence Transformers |
| Vector Database | ChromaDB |
| Parsing | LangChain Community |
| Language | Python 3.13 |

---

# Project Structure

```
.
├── main.py                 # Main CRAG workflow
├── retriver.py             # Build vector database
├── rag_ingest.py           # Source ingestion
├── ingest.py               # Chunk inspection
├── debug_openrouter.py     # OpenRouter testing
├── agent.py                # Legacy prototype
├── chroma_db/              # Local vector store
├── requirements.txt
└── .env.example
```

---

# CRAG Pipeline

## 1. Document Ingestion

Supported sources include:

- Web pages
- PDF documents
- Markdown
- Plain text

Documents are parsed while preserving structural headers whenever possible before semantic chunking.

---

## 2. Embedding

Chunks are embedded using

```
sentence-transformers/all-MiniLM-L6-v2
```

and stored inside ChromaDB.

---

## 3. Retrieval

The retriever performs

- similarity search
- Maximum Marginal Relevance (MMR)

to retrieve diverse and relevant context.

---

## 4. Relevance Grading

Unlike conventional RAG systems, retrieved documents are evaluated by a lightweight LLM.

If retrieval quality is sufficient:

```
Retrieve
      ↓
Generate
```

Otherwise:

```
Retrieve
      ↓
Grade
      ↓
Rewrite Query
      ↓
Retrieve Again
```

This corrective loop reduces hallucinations caused by poor retrieval.

---

## 5. Generation

Once relevant evidence has been collected, the final answer is generated using a local Ollama model.

The model is instructed to answer **only from retrieved context**, reducing unsupported responses.

---

# Installation

Clone the repository

```bash
git clone https://github.com/<username>/recall.git
cd recall
```

Create a virtual environment

```bash
python -m venv recall-env
source recall-env/bin/activate
```

Install dependencies

```bash
pip install -r requirements.txt
```

Copy environment variables

```bash
cp .env.example .env
```

Start Ollama

```bash
ollama serve
```

Pull your preferred model

```bash
ollama pull gemma4:12b
```

---

# Configuration

| Variable | Description |
|-----------|-------------|
| ARTICLE_URL | Default ingestion source |
| CHROMA_DIR | Vector database location |
| EMBEDDING_MODEL | Embedding model |
| OLLAMA_MODEL | Generation model |
| RETRIEVER_K | Retrieved documents |
| FETCH_K | Initial retrieval candidates |
| MAX_REWRITE_LOOPS | Maximum retry attempts |

---

# Usage

## Build Vector Database

```bash
python retriver.py
```

Use custom sources

```bash
python retriver.py notes.md paper.pdf https://example.com
```

Run Recall

```bash
python main.py
```

Ask a question

```bash
python main.py "Explain task decomposition."
```

Inspect chunking

```bash
python ingest.py
```

---

# Why Corrective RAG?

Traditional Retrieval-Augmented Generation follows a simple pipeline:

```
Question
     ↓
Retrieve
     ↓
Generate
```

Poor retrieval often leads directly to hallucinations.

Recall introduces a corrective stage:

```
Question
     ↓
Retrieve
     ↓
Grade
 ┌────┴────┐
 │         │
Good     Poor
 │         │
 ▼         ▼
Generate Rewrite
           │
           ▼
      Retrieve Again
```

This iterative process significantly improves answer reliability by ensuring generation is based on stronger evidence.

---

# Current Limitations

- Single-user local deployment
- No web interface
- No hybrid search
- No reranking model
- No streaming responses

---

# Roadmap

- [ ] Hybrid BM25 + Dense Retrieval
- [ ] Cross-encoder reranking
- [ ] Streaming generation
- [ ] Web UI
- [ ] Multi-document collections
- [ ] Citation support
- [ ] Agentic retrieval
- [ ] Knowledge graph integration

---

# License

This project is released under the MIT License.
