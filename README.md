# network-rag-api

A minimal retrieval-augmented generation (RAG) API: FastAPI + a persistent Chroma vector store + a local LLM served by Ollama.

## How it works

1. `embed.py` reads `k8s.txt` and stores it in the Chroma collection `docs` at `./db`.
2. `POST /add?text=...` adds a new document (with a UUID id) to the collection.
3. `POST /query?q=...` retrieves the single closest document and sends it with the question to Ollama, then returns `{"answer": ...}`.

## Configuration

- `MODEL_NAME`: the Ollama model to use (default `tinyllama`).
- Requires a running Ollama server reachable by the `ollama` Python client, with the model pulled (`ollama pull tinyllama`).

## Run

Locally:

```bash
pip install fastapi uvicorn chromadb ollama
python embed.py
uvicorn app:app --port 8000
```

With Docker (the `dockerfile` installs the same packages and serves on port 8000):

```bash
docker build -f dockerfile -t network-rag-api .
docker run -p 8000:8000 -e MODEL_NAME=tinyllama network-rag-api
```

Ollama runs outside this container, so the container needs network access to an Ollama host.

## Example (illustrative)

```bash
curl -X POST "http://localhost:8000/query?q=What%20is%20Kubernetes%3F"
# -> {"answer": "..."}
```

## Limitations and eval gaps

- The bundled corpus is one sentence (`k8s.txt`), and retrieval uses only the top-1 document.
- No retrieval or answer-quality evaluation, no tests, and no pinned dependencies (`requirements.txt`) yet.
- The generated `db/` directory is committed to the repo.
