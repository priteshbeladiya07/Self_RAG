# Self-RAG with LangGraph

A four-phase Self-RAG pipeline that decides when to retrieve documents, evaluates retrieved content, verifies generated answers, and retries when necessary.

## Project Structure

```text
Self RAG/
├── Company_Policies.pdf
├── self_rag_step4.ipynb
└── README.md
```

## Requirements

- Python 3.10+
- Mistral API key
- Jupyter or Visual Studio Code

## Installation

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install langchain langchain-community langchain-mistralai langchain-text-splitters langgraph faiss-cpu pypdf pydantic python-dotenv jupyter
```

Create a `.env` file:

```env
MISTRAL_API_KEY=your_mistral_api_key
```

Place `Company_Policies.pdf` in the project directory.

## Four Phases

### Phase 1: Retrieval Decision

The `decide_retrieval` node uses an LLM to determine whether the question requires company-specific information.

- General questions are answered directly.
- Company-specific questions continue to retrieval.

### Phase 2: Document Retrieval and Relevance

The PDF is loaded, split into chunks, embedded with Mistral embeddings, and indexed using FAISS.

The graph then:

1. Retrieves relevant chunks.
2. Uses an LLM to filter documents by topic relevance.
3. Generates an answer from the relevant context.

### Phase 3: Answer Support Verification

The `is_sup` node checks whether the generated answer is supported by the retrieved context.

Possible results:

- `fully_supported`
- `partially_supported`
- `no_support`

Unsupported answers are revised using context-only quotes and verified again.

### Phase 4: Answer Usefulness and Query Rewriting

The `is_use` node checks whether the answer actually addresses the user’s question.

If the answer is not useful:

1. The retrieval query is rewritten.
2. Documents are retrieved again.
3. The answer-generation and verification process repeats.

The graph stops after a configurable number of retries.

## Graph Flow

```text
START
  ↓
Decide Retrieval
  ├── Generate Direct Answer → END
  └── Retrieve Documents
          ↓
      Check Relevance
       ├── No Answer Found → END
       └── Generate from Context
                    ↓
                Verify Support
                 ├── Revise Answer → Verify Support
                 └── Check Usefulness
                              ├── Useful → END
                              ├── Rewrite Query → Retrieve
                              └── Retry Limit → END
```

## Running the Notebook

Open the notebook:

```powershell
code .\self_rag_step4.ipynb
```

Run the cells from top to bottom. Update the question in the `initial_state` cell:

```python
initial_state = {
    "question": "Describe NexaAI's company culture.",
    "retrieval_query": "",
    "rewrite_tries": 0,
    "docs": [],
    "relevant_docs": [],
    "context": "",
    "answer": "",
    "issup": "",
    "evidence": [],
    "retries": 0,
    "isuse": "not_useful",
    "use_reason": "",
}
```

The final cell displays:

- Whether retrieval was needed
- Retrieved and relevant documents
- Verification status
- Evidence quotes
- Retry counts
- Final answer

## Configuration

Adjust retry limits in the notebook:

```python
MAX_RETRIES = 10
MAX_REWRITE_TRIES = 3
```

The retriever currently returns four documents:

```python
retriever = vector_store.as_retriever(
    search_kwargs={"k": 4}
)
```

## Technologies

- LangGraph — workflow orchestration
- LangChain — LLM and retrieval components
- Mistral AI — chat model and embeddings
- FAISS — vector similarity search
- PyPDF — PDF loading
- Pydantic — structured LLM outputs
- Python TypedDict — graph state definition
