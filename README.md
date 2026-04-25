# Master-thesis-project

Designed a question answering system that retrieves answers from three independent sources in parallel: direct LLM generation, structured knowledge graph querying, and domain-specific document retrieval.
For knowledge graph access, a dedicated LLM translates the user’s natural language into SPARQL and queries the Wikidata database. Simultaneously, a RAG pipeline retrieves relevant context from a user-uploaded document. The three candidate answers are then evaluated by a separate LLM-as-a-Judge, which either selects the most accurate response or synthesizes a combined final answer.

<img width="1414" height="1840" alt="image" src="https://github.com/user-attachments/assets/8a3ac2d0-9c34-4665-979f-8848a6b28764" />

## How it works

1. **Question improvement** — The user's raw input is passed to GPT-4o, which corrects spelling, resolves ambiguities, and rephrases the question for better retrieval.
2. **Wikidata pipeline** — GPT-4o extracts named entities and relations, Falcon 2.0 links them to Wikidata QIDs/PIDs, GPT-4o generates a SPARQL query, and the query is executed against the Wikidata endpoint.
3. **Direct LLM answer** — The improved question is sent directly to GPT-3.5-turbo (or GPT-4o) for a parametric answer with no retrieval.
4. **RAG pipeline** — If a document is uploaded, it is chunked (500 chars, 50 overlap), embedded with OpenAI embeddings, stored in a FAISS index, and a RetrievalQA chain retrieves the top-4 chunks to answer the question.
5. **LLM-as-a-Judge** — GPT-4o receives all three answers and scores them for correctness, relevance, and completeness, returning a structured JSON evaluation.

All five steps are orchestrated through a Gradio interface.

---

## Setup

### Requirements

```bash
pip install langchain openai faiss-cpu pypdf docx2txt langchain-openai langchain-community gradio
```

### API key

This notebook is designed for Google Colab. Add your OpenAI key to Colab secrets:

1. Open the 🔑 **Secrets** panel in Colab (left sidebar)
2. Add a secret named `OPENAI_API_KEY` with your key as the value
3. The notebook reads it automatically via `userdata.get('OPENAI_API_KEY')`

---

## Configuration

| Parameter | Default | Description |
|---|---|---|
| `CHUNK_SIZE` | 500 | Characters per document chunk |
| `CHUNK_OVERLAP` | 50 | Overlap between consecutive chunks |
| `TOP_K` | 4 | Number of chunks retrieved per query |
| `LLM_MODEL` | `gpt-3.5-turbo` | Model used for direct LLM answers |

---

## Supported document formats

The RAG pipeline accepts the following file types:

- `.pdf` — loaded via PyPDFLoader
- `.txt` — loaded via TextLoader
- `.docx` — loaded via Docx2txtLoader

---

## Example questions

| Question | Best source |
|---|---|
| Who is the president of France? | Wikidata |
| What is MiCA? | Direct LLM |
| What does this contract say about termination? | RAG (upload contract) |
| How many ingredients are in creatine? | Wikidata + RAG |
| What is the capital of Albania? | Wikidata |

---

## Limitations

- Wikidata SPARQL can time out for complex or multi-hop questions. The system avoids property path wildcards (`*`, `+`) to reduce this risk and limits queries to 3–4 triple patterns.
- RAG answers are bounded by the content of the uploaded document — the system is instructed not to hallucinate outside it.
- The direct LLM answer has no grounding and may hallucinate for niche or recent topics.
