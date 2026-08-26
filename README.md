# Self-RAG with LangGraph

This project demonstrates how to build a Self-RAG system in four phases using LangChain, LangGraph, Mistral AI, FAISS, and a company policy PDF.

The system retrieves information only when required, checks document relevance, verifies answer support, evaluates usefulness, and rewrites failed queries.

## Project Structure

```text
Self RAG/
├── Company_Policies.pdf
├── self_rag_step1.ipynb
├── self_rag_step2.ipynb
├── self_rag_step3.ipynb
├── self_rag_step4.ipynb
└── README.md
```

## Technologies

- Python
- LangChain
- LangGraph
- Mistral AI
- FAISS
- PyPDF
- Pydantic
- Jupyter Notebook

## Setup

Create a virtual environment in Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
pip install langchain langchain-community langchain-mistralai langchain-text-splitters langgraph faiss-cpu pypdf pydantic python-dotenv jupyter
```

Add your Mistral API key to `.env`:

```env
MISTRAL_API_KEY=your_mistral_api_key
```

Do not commit `.env` to Git.

## How to Run

Run the notebooks in order:

```text
1. self_rag_step1.ipynb
2. self_rag_step2.ipynb
3. self_rag_step3.ipynb
4. self_rag_step4.ipynb
```

Open the project in VS Code:

```powershell
code .
```

Make sure `Company_Policies.pdf` is in the project root before running the notebooks.

---

## Phase 1: Basic RAG

File: `self_rag_step1.ipynb`

The first phase builds the basic retrieval pipeline:

1. Load `Company_Policies.pdf`.
2. Split the document into smaller chunks.
3. Create embeddings using Mistral AI.
4. Store the embeddings in FAISS.
5. Retrieve relevant document chunks.
6. Generate an answer using the retrieved context.

This phase establishes the foundation for the complete Self-RAG system.

---

## Phase 2: Retrieval Decision

File: `self_rag_step2.ipynb`

The second phase adds a retrieval decision step.

The LLM decides whether the question requires information from company documents.

```text
Question
   ├── General question → Direct answer
   └── Company-specific question → Retrieve documents
```

General questions can be answered directly, while company-specific questions use the vector store.

---

## Phase 3: Document and Answer Evaluation

File: `self_rag_step3.ipynb`

The third phase improves answer quality by evaluating the retrieval and generation process.

The system:

- Checks whether retrieved documents are relevant.
- Generates an answer from relevant documents.
- Verifies whether answer claims are supported by the context.
- Revises answers that contain unsupported information.

The support evaluation returns one of three results:

- `fully_supported`
- `partially_supported`
- `no_support`

---

## Phase 4: Complete Self-RAG with LangGraph

File: `self_rag_step4.ipynb`

The fourth phase combines all components into a LangGraph workflow.

### Main Graph Nodes

- `decide_retrieval` — decides whether retrieval is required.
- `generate_direct` — answers general questions.
- `retrieve` — searches the FAISS vector store.
- `is_relevant` — filters documents by relevance.
- `generate_from_context` — generates an answer using retrieved context.
- `is_sup` — checks answer support.
- `revise_answer` — removes unsupported claims.
- `is_use` — checks whether the answer addresses the question.
- `rewrite_question` — creates a better retrieval query.
- `no_answer_found` — handles failed retrieval attempts.

### Complete Workflow

```text
START
  ↓
Decide whether retrieval is needed
  ├── No → Generate direct answer → END
  └── Yes
        ↓
      Retrieve documents
        ↓
      Check document relevance
        ├── No relevant documents → No answer found → END
        └── Relevant documents
                ↓
          Generate answer from context
                ↓
          Verify answer support
                ├── Unsupported → Revise answer → Verify again
                └── Supported
                        ↓
                  Check answer usefulness
                    ├── Useful → END
                    ├── Not useful → Rewrite query → Retrieve again
                    └── Retry limit reached → END
```

### Retry Controls

The notebook limits repeated attempts using:

```python
MAX_RETRIES = 10
MAX_REWRITE_TRIES = 3
```

These values can be adjusted depending on cost, latency, and answer quality requirements.

## Example Question

```python
"Describe NexaAI's company culture."
```

The final output displays:

- Whether retrieval was needed
- Number of retrieved documents
- Number of relevant documents
- Support verification result
- Evidence quotes
- Number of retries
- Final answer

## Environment Variables

The `.env` file should contain:

```env
MISTRAL_API_KEY=your_mistral_api_key
```

Keep API keys private and ensure `.env` is included in `.gitignore`.

## Summary

This project evolves from a basic RAG pipeline into a complete Self-RAG workflow:

```text
Basic RAG
   ↓
Retrieval Decision
   ↓
Relevance and Support Evaluation
   ↓
Self-RAG Workflow with LangGraph
```

The final system can decide, retrieve, evaluate, revise, and retry automatically.
