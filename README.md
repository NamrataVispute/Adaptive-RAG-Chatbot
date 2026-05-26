# Adaptive RAG - Intelligent Agentic AI Chatbot

Adaptive RAG is an intelligent, high-performance Retrieval-Augmented Generation (RAG) system built on an agentic AI architecture. By combining dynamic query routing, local semantic document search, and real-time internet lookups, the system intelligently selects the most accurate path to answer any user query.

Rather than running expensive external cloud databases, the system is fully optimized for local execution using offline sentence embeddings, local vector indexing, and ultra-fast, open-source LLM inference.

---

## 🏗️ Architecture & Workflow

The system uses **LangGraph** to construct a stateful, cyclic workflow (Agentic loop) that processes query requests dynamically:

```
                      ┌───────────────────────────┐
                      │    Streamlit Web UI       │ <─── User uploads files & queries
                      └───────────────────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │    FastAPI REST Backend   │
                      └───────────────────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │  LangGraph Router Node    │ <─── Analyzes and classifies query
                      └───────────────────────────┘
                                    │
           ┌────────────────────────┼────────────────────────┐
           ▼                        ▼                        ▼
┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────┐
│  Vector Store Path  │  │   Web Search Path   │  │  Direct LLM Path    │
│  (Local FAISS Index)│  │     (Tavily API)    │  │  (General Knowledge)│
└─────────────────────┘  └─────────────────────┘  └─────────────────────┘
           │                        │                        │
           └────────────────────────┼────────────────────────┘
                                    ▼
                      ┌───────────────────────────┐
                      │    Llama 3 Response Gen   │
                      │  (Groq API - Low Latency) │
                      └───────────────────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │   Response back to User   │
                      └───────────────────────────┘
```

### Key Workflow Steps:
1. **Intelligent Query Classification**: Analyzes the query and routes it to the most relevant pipeline:
   - **Index**: Specific queries about uploaded documents.
   - **Search**: Time-sensitive or out-of-knowledge queries requiring live web search.
   - **General**: Queries answerable with general logic.
2. **Local Semantic Search**: Generates vector search queries against uploaded files using high-fidelity offline embeddings.
3. **Active Web Integration**: Leverages web-search APIs to pull real-time internet facts.
4. **Self-Correction & Refinement**: Rewrites ambiguous search queries and grades search relevance to prevent hallucinations.

---

## ⚡ Tech Stack

* **Core Language**: Python 3.9+
* **AI Orchestration & Agents**: LangGraph (cyclic state machines), LangChain
* **LLM Engine**: Groq (Llama-3-8b-Instant / Llama-3-70b-Versatile)
* **Local Embeddings**: HuggingFace (`sentence-transformers/all-MiniLM-L6-v2`)
* **Vector Index**: FAISS (Facebook AI Similarity Search - offline, fast CPU index)
* **Web Search Engine**: Tavily Search API
* **Backend API Framework**: FastAPI (Asynchronous REST API)
* **Frontend Web App**: Streamlit (Interactive, session-based UI)

---

## 📂 Project Structure

```text
Adaptive-Rag-main/
│
├── src/
│   ├── api/
│   │   └── routes.py             # FastAPI REST endpoints (Upload / Query)
│   │
│   ├── config/
│   │   ├── settings.py           # YAML prompts configurations
│   │   └── prompts.yaml          # System prompts for agents & routers
│   │
│   ├── core/
│   │   ├── config.py             # Environment configurations
│   │   └── logger.py             # Event logger
│   │
│   ├── llms/
│   │   └── openai.py             # LLM initialization with Groq/Llama-3 fallback
│   │
│   ├── memory/
│   │   └── chathistory_in_memory.py # High-speed local session chat log manager
│   │
│   ├── models/
│   │   └── query_request.py      # Pydantic data schemas
│   │
│   ├── rag/
│   │   ├── document_upload.py    # Document chunking and parsing (PDF/TXT)
│   │   ├── graph_builder.py      # LangGraph state machine configuration
│   │   ├── reAct_agent.py        # Reasoning + Acting loops
│   │   └── retriever_setup.py    # Offline HuggingFace embeddings & FAISS Setup
│   │
│   └── main.py                   # FastAPI server entry point
│
├── streamlit_app/
│   ├── pages/
│   │   └── chat.py               # Main UI Chat interface with resume upload
│   └── home.py                   # Application dashboard & entry point
│
├── .env                          # Configuration keys and secrets
├── requirements.txt              # Production-ready dependencies list
└── README.md                     # This file
```

---

## 🚀 Setup & Execution

### 1. Install Dependencies
Ensure you have Python 3.9+ installed. Run:
```bash
pip install -r requirements.txt
```

### 2. Configure Environment (`.env`)
Create a `.env` file in the root folder with the following keys:
```env
# Groq API Key for ultra-fast Llama-3 reasoning
GROQ_API_KEY=your_groq_api_key_here

# HuggingFace Token for offline vector embeddings
HF_TOKEN=your_huggingface_token_here

# Tavily API Key for real-time web search Fallback
TAVILY_API_KEY=your_tavily_api_key_here
```

### 3. Run the Backend API Server
Start the high-performance **FastAPI** server:
```bash
uvicorn src.main:app --reload --port 8000
```
*The API documentation will be available at `http://localhost:8000/docs`.*

### 4. Run the Streamlit Frontend Web App
In a new terminal window, start the **Streamlit** user interface:
```bash
streamlit run streamlit_app/home.py --server.port 8501
```
*The interactive web interface will launch automatically at `http://localhost:8501`.*

---

## 🌟 Key Highlights for AIML Roles
* **Agentic Workflows**: Demonstrates mastery of cyclic AI design patterns and workflow state machines using **LangGraph** instead of rigid, single-run prompt chains.
* **Cost & Performance Optimization**: Leverages local **HuggingFace** sentence models for zero-cost embedding and the blazing-fast **Groq** hardware API for Llama-3 inference.
* **Hybrid Data Routing**: Implements dynamic classification algorithms to choose the best knowledge source, balancing local document vector storage (FAISS) with external live searching (Tavily).
