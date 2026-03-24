# Bootcamp Assignment -- Diagnostic Multimodal RAG

## Problem Statement

Modern diesel engine ECUs generate fault codes when anomalies are detected.
Service engineers must locate the correct diagnostic procedure in a dense
technical manual containing text, tables, and wiring schematic images.
This system solves that problem with a Multimodal RAG pipeline.

## Architecture

```
INGESTION
PDF --> PyMuPDF (text) + pdfplumber (tables) + PyMuPDF (images)
                                                     |
                                         Claude Vision VLM
                                         (image -> text summary)
                                                     |
                                  sentence-transformers/all-MiniLM-L6-v2
                                                     |
                                           ChromaDB (persistent)

QUERY
Question --> embed --> ChromaDB search --> context --> Claude Sonnet --> Answer + Sources
```

## Technology Choices

| Component | Choice | Reason |
|---|---|---|
| PDF Parser | PyMuPDF + pdfplumber | Best for text/images and tables |
| VLM | Claude Opus 4.5 | Superior wiring diagram comprehension |
| Embedding | all-MiniLM-L6-v2 | Local, free, 384-dim, strong semantic search |
| Vector Store | ChromaDB | Native metadata filtering by chunk type and page |
| LLM | Claude Sonnet 4.6 | Fast, cost-effective, strong technical reasoning |
| Framework | FastAPI (custom) | Full transparency, minimal dependencies |

## Setup Instructions

```bash
cp .env.example .env
pip install -r requirements.txt
python3 main.py
# In a second terminal:
curl -X POST http://localhost:8000/ingest \
  -F "file=@sample_documents/Diagnostic_Document.pdf"
curl -X POST http://localhost:8000/query \
  -H "Content-Type: application/json" \
  -d '{"question": "What does fault code P0087 mean?"}'
```

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | /health | System status, model readiness, uptime |
| POST | /ingest | Upload PDF, parse, embed, index |
| POST | /query | Natural language question, returns answer + sources |
| GET | /documents | List all indexed documents |
| DELETE | /delete | Remove a document from the index |
| GET | /docs | Swagger UI (auto-generated) |

## Limitations

- VLM image processing adds ~90 seconds ingestion time
- No conversation memory between queries
- Character-based chunking (semantic chunking would be better)