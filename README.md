# Source Mesh

A hybrid document intelligence system for uploading PDFs, building searchable knowledge layers, and asking grounded questions across your source material.

Source Mesh combines semantic retrieval, keyword matching, entity graphs, and cross-encoder reranking to help users explore complex documents with more context than a simple vector search pipeline can provide. It is built as a full-stack research assistant with a FastAPI backend, a Next.js chat interface, PDF ingestion, conversation memory, summarization, and citation-aware answers.

## What Source Mesh Does

Source Mesh turns uploaded PDFs into a connected retrieval mesh.

Instead of treating every chunk as an isolated text block, the system extracts document text, cleans it, chunks it, embeds it, indexes it, identifies entities, builds keyword search structures, and expands retrieval through graph relationships. When a user asks a question, Source Mesh searches through multiple paths, reranks the best candidates, and sends a focused context window to the language model.

The result is a practical RAG application for research papers, reports, manuals, policy documents, and other dense PDF collections.

## Core Capabilities

### Multi-PDF Ingestion

Upload one or more PDF files through the API or web interface. Each document receives a unique document ID, is stored locally at runtime, parsed into page-level text, cleaned, chunked, indexed, and registered for retrieval.

### Hybrid Retrieval

Source Mesh uses a layered retrieval pipeline:

1. Dense vector search for semantic similarity
2. BM25 keyword search for exact-term anchoring
3. Entity graph expansion for concept-level coverage
4. Cross-encoder reranking for final relevance ordering

This gives the system both fuzzy understanding and precise matching.

### Graph-Aware Context Expansion

The project extracts entities from document chunks and links related concepts through a graph structure. For broader or comparison-style questions, graph expansion helps pull in adjacent chunks that may not appear in the first vector-only result set.

### Cross-Encoder Reranking

After Source Mesh gathers candidates from vector, lexical, and graph retrieval, a cross-encoder reranker scores query and chunk pairs directly. This creates a precision layer before the final context is passed to the LLM.

### Question Answering

Users can ask natural-language questions against the uploaded documents. Source Mesh retrieves relevant chunks, builds a grounded prompt, generates an answer, and returns supporting citations.

### Document Summarization

The chat endpoint supports a summarization mode for all uploaded documents or selected document IDs. A token guard prevents oversized summarization requests from being sent to the model.

### Conversation Memory

Each session keeps short-term chat history. Follow-up questions can be rewritten using recent conversation context, making the experience feel more natural without forcing users to repeat every detail.

### Document Management

The backend includes endpoints for uploading PDFs, listing loaded documents, removing documents, checking token counts, asking questions, summarizing content, and clearing conversation history.

## Architecture

```text
PDF Upload
   ↓
PDF Text Extraction
   ↓
Text Cleaning
   ↓
Chunking
   ↓
Entity Extraction
   ↓
Vector Index + BM25 Index + Entity Graph
   ↓
Hybrid Retrieval
   ↓
Graph Expansion
   ↓
Cross-Encoder Reranking
   ↓
Prompt Construction
   ↓
LLM Response
   ↓
Answer + Citations
```

## Tech Stack

### Backend

* Python 3.10+
* FastAPI
* Uvicorn
* Pydantic
* Groq API client
* OpenAI-compatible model support
* Qdrant
* ChromaDB
* Sentence Transformers
* Cross-Encoder reranking
* spaCy
* PyMuPDF
* BM25
* NetworkX
* LangChain packages
* Ruff, Black, isort, and pre-commit

### Frontend

* Next.js 14
* React 18
* TypeScript
* Tailwind CSS
* Lucide React icons
* Browser localStorage for session and uploaded-document state

### Deployment

* Docker backend support
* Hugging Face Spaces compatible backend container
* Vercel-ready frontend

## Repository Structure

```text
.
├── backend/
│   └── app/
│       ├── api/
│       │   ├── routes_chat.py
│       │   ├── routes_chat_langchain.py
│       │   └── routes_docs.py
│       ├── core/
│       │   ├── embeddings.py
│       │   ├── llm.py
│       │   └── prompts.py
│       ├── ingestion/
│       │   ├── chunking.py
│       │   ├── cleaning.py
│       │   ├── entities.py
│       │   ├── indexing.py
│       │   ├── pdf_loader.py
│       │   └── pipeline.py
│       ├── memory/
│       │   ├── conversation.py
│       │   └── query_rewriter.py
│       ├── models/
│       └── retrieval/
│           ├── citation_filter.py
│           ├── graph_utils.py
│           ├── keyword_index.py
│           ├── reranker.py
│           ├── retrieve.py
│           └── vector_store.py
├── frontend/
│   └── app/
│       └── page.tsx
├── Dockerfile
├── pyproject.toml
├── requirements.txt
└── README.md
```

## Getting Started

### Prerequisites

Install the following before running Source Mesh locally:

* Python 3.10 or newer
* Node.js 18 or newer
* Git
* A Groq API key

## Backend Setup

Clone the repository:

```bash
git clone https://github.com/sanskarmodi8/Atlas-RAG.git
cd Atlas-RAG
```

Create and activate a Python environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

Install backend dependencies:

```bash
pip install -r requirements.txt
pip install -e .
```

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key_here
DEFAULT_MODEL=openai/gpt-oss-120b
QDRANT_PATH=/tmp/qdrant
DOCS_PATH=/tmp/docs
MAX_SUMMARY_TOKENS=6000
```

Run the backend:

```bash
uvicorn backend.app.main:app --reload
```

For the deployed container-style path, run:

```bash
uvicorn backend.app.main:app --host 0.0.0.0 --port 7860
```

The local API will be available at:

```text
http://127.0.0.1:8000
```

FastAPI docs are available at:

```text
http://127.0.0.1:8000/docs
```

## Frontend Setup

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend will run at:

```text
http://localhost:3000
```

By default, the frontend points to the deployed backend. For local development, update the `API_BASE` value in the frontend app to your local FastAPI server:

```ts
const API_BASE = 'http://127.0.0.1:8000';
```

## API Overview

### Upload PDFs

```http
POST /docs/upload
```

Accepts one or more PDF files and returns extracted chunks grouped by document ID.

### Remove a Document

```http
DELETE /docs/remove/{doc_id}
```

Removes document chunks from memory, deletes indexed vector points, and removes the stored PDF file.

### List Documents

```http
GET /docs/list
```

Returns document count, chunk count, and loaded document IDs.

### Check Token Counts

```http
GET /docs/token-counts
```

Returns approximate token counts for loaded documents and the configured summarization limit.

### Ask a Question

```http
POST /chat/ask
```

Example payload:

```json
{
  "query": "How does attention replace recurrence?",
  "mode": "qa",
  "top_k": 5,
  "session_id": "default",
  "doc_ids": null
}
```

### Summarize Documents

```http
POST /chat/ask
```

Example payload:

```json
{
  "query": "Summarize the selected documents",
  "mode": "summarize",
  "top_k": 5,
  "session_id": "default",
  "doc_ids": ["document-id-here"]
}
```

### Clear Conversation Memory

```http
POST /chat/clear?session_id=default
```

Clears the stored conversation history for a session.

## Retrieval Pipeline

Source Mesh retrieves context in several stages.

### 1. Broad Seed Retrieval

The backend performs vector search and BM25 search with a larger candidate pool than the final requested `top_k`. This gives the system more raw material before reranking.

### 2. Entity Extraction

The query is processed for entity-like terms. If named entity recognition does not find useful terms, the system falls back to longer query tokens.

### 3. Graph Expansion

Source Mesh builds a graph from extracted document entities. Related concepts are expanded through adaptive hops, allowing the retriever to include connected chunks beyond pure vector similarity.

### 4. Candidate Merge

Vector hits, BM25 hits, and graph-expanded chunks are merged into a shared candidate pool.

### 5. Cross-Encoder Reranking

The reranker scores each query and chunk pair, then sorts candidates by relevance.

### 6. Final Context Selection

The best candidates are passed to the prompt builder. Comparison-style queries preserve multiple viewpoints before final truncation.

## Configuration

Source Mesh uses environment variables through Pydantic settings.

| Variable             |               Default | Purpose                                            |
| -------------------- | --------------------: | -------------------------------------------------- |
| `GROQ_API_KEY`       |                 empty | API key used for Groq chat completions             |
| `DEFAULT_MODEL`      | `openai/gpt-oss-120b` | Default model name                                 |
| `QDRANT_PATH`        |         `/tmp/qdrant` | Local Qdrant storage path                          |
| `DOCS_PATH`          |           `/tmp/docs` | Runtime PDF storage path                           |
| `MAX_SUMMARY_TOKENS` |                `6000` | Maximum estimated tokens allowed for summarization |

## Docker

Build the backend image:

```bash
docker build -t source-mesh-backend .
```

Run the backend container:

```bash
docker run \
  -p 7860:7860 \
  -e GROQ_API_KEY=your_groq_api_key_here \
  source-mesh-backend
```

The container exposes the backend on:

```text
http://localhost:7860
```

## Code Quality

Install pre-commit hooks:

```bash
pre-commit install
```

Run linting:

```bash
ruff check .
```

Format code:

```bash
ruff format .
```

Optional formatting tools are included in the dependency set for stricter local development workflows.

## Example Workflow

1. Start the backend.
2. Start the frontend.
3. Upload one or more PDFs.
4. Ask a focused question.
5. Review the generated answer and citations.
6. Ask follow-up questions using the same session.
7. Switch to summarization mode when you want a broader overview.
8. Remove documents when they are no longer needed.

## Use Cases

Source Mesh is useful for:

* Academic paper exploration
* Technical report analysis
* Policy and compliance review
* Product documentation Q&A
* Internal knowledge-base experiments
* RAG architecture demos
* Retrieval evaluation experiments
* Multi-document summarization prototypes

## Notes and Limitations

* The current document pipeline is designed around PDF files.
* Runtime document and vector storage use local paths by default.
* The summarization flow uses an approximate token check based on character count.
* The frontend stores uploaded-document metadata and session IDs in browser localStorage.
* Production deployments should tighten CORS, persistence, authentication, file limits, and tenant isolation before handling private documents.

## Roadmap Ideas

* Persistent document metadata storage
* User authentication
* Multi-tenant workspaces
* Streaming chat responses
* Configurable model provider selection
* Source-level document titles in citations
* Async ingestion for large PDFs
* Background index rebuilds
* More formal retrieval evaluation dashboards
* Support for DOCX, HTML, Markdown, and text files

## License

This project is released under the MIT License.

## Author

Built by Sanskar Modi.

## Project Name

This README presents the project as **Source Mesh**, a clearer name for a system that connects document sources through semantic, lexical, and graph-based retrieval.
