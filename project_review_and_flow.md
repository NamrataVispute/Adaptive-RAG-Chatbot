# Adaptive RAG - System Architecture & Technical Flow

This document provides a detailed technical overview of the **Adaptive RAG** system. It details the high-performance local architecture, followed by visual flowcharts and technical breakdowns of the document ingestion and query routing workflows.

---

## 📋 Core Architectural Principles & Component Stack

The system is engineered as a high-speed, cost-effective, and fully local/hybrid Retrieval-Augmented Generation (RAG) pipeline. It operates without heavy external cloud databases or paid API billing dependencies.

| Architectural Component | Technology | Technical Purpose & Value |
| :--- | :--- | :--- |
| **Orchestration Brain** | **LangGraph** | Models dynamic query-routing and self-correcting logic via a stateful, cyclic workflow. |
| **Vector Search Engine** | **FAISS (Local CPU Index)** | Direct in-memory similarity indexing, eliminating the overhead of external database servers. |
| **Text Embeddings** | **HuggingFace (`all-MiniLM-L6-v2`)** | 100% free, offline, local vector calculations with zero API call overhead. |
| **Large Language Model** | **Groq Llama-3** | Delivers ultra-low latency open-source reasoning via Groq's high-speed API engine. |
| **Session Memory** | **Async In-Memory History** | An asynchronous, non-blocking stack that tracks conversation context thread-by-thread. |
| **API Framework** | **FastAPI** | High-throughput asynchronous REST API separating business logic from client interfaces. |
| **User Interface** | **Streamlit** | Responsive, lightweight chat dashboard featuring file uploads and real-time outputs. |

---

## 🔄 System Workflows & Flowcharts

The system operates on two core pipelines: **Document Upload & Ingestion** and **Dynamic Query Routing (Agentic RAG)**.

### 1. Document Upload & Ingestion Flow
This flow is triggered when a document (such as a resume or text file) is uploaded through the Streamlit interface.

```mermaid
graph TD
    A["User Uploads Document (PDF/TXT)"] --> B["Streamlit UI captures file & description"]
    B --> C["Streamlit sends POST to FastAPI /rag/documents/upload"]
    C --> D["FastAPI reads file contents and parses text"]
    D --> E["Text Splitter chunks content into readable segments"]
    E --> F["HuggingFace Local Embeddings calculate vector representations"]
    F --> G["FAISS indexes chunks and stores them in memory"]
    G --> H["Vector database is dynamically bound to the Retriever Tool"]
    H --> I["UI displays success indicator: Document Uploaded Successfully!"]
```

---

### 2. Dynamic Query Routing (LangGraph) Flow
This flowchart represents the stateful query execution graph engineered with **LangGraph**. It intelligently selects the most contextually relevant path to answer any question.

```mermaid
graph TD
    Start["User submits query in Chat Interface"] --> LoadHistory["FastAPI loads session chat history from memory"]
    LoadHistory --> GraphInit["Initialize LangGraph state machine"]
    GraphInit --> Router["Router Node (Classifies Query Type)"]
    
    %% Router Decisions
    Router -- "Option A: Index Query (Requires Document Insight)" --> RetrieveFAISS["Retrieve Node (FAISS similarity search)"]
    Router -- "Option B: Search Query (Out-of-knowledge / Time-sensitive)" --> TavilySearch["Search Node (Tavily Web Search API)"]
    Router -- "Option C: General Query (Logical / General Conversational)" --> DirectLLM["General Node (Direct Groq Llama-3 call)"]
    
    %% FAISS Processing Branch
    RetrieveFAISS --> Grader["Document Grader Node (Relevance check)"]
    Grader -- "Documents are Relevant" --> GenerateAnswer["Generate Node (Answer using Llama-3 with doc context)"]
    Grader -- "Documents are NOT Relevant" --> RewriteQuery["Rewrite Node (Optimizes query for web search)"]
    RewriteQuery --> TavilySearch
    
    %% Web Search Processing Branch
    TavilySearch --> SearchGenerate["Generate Node (Answer using Llama-3 with live web context)"]
    
    %% Final Assembly
    GenerateAnswer --> FinalResponse["Update Chat History & Send JSON response to Streamlit UI"]
    SearchGenerate --> FinalResponse
    DirectLLM --> FinalResponse
    
    FinalResponse --> End["Chat bubble renders response for user"]
```

---

## 🎯 Key Engineering Highlights

1. **Self-Correcting RAG:** The system does not blindly rely on vector store results. If retrieved document segments grade low on relevance, the agent autonomously rewrites the query and executes an external Tavily web search.
2. **Decoupled System Boundaries:** Separating the FastAPI backend from the Streamlit frontend ensures high throughput, cleanly isolating front-end rendering from back-end AI computations.
3. **High-Performance Open-Source Inference:** Utilizing the Groq Llama-3 API allows responses to be generated in milliseconds, proving that open-source RAG architectures can deliver commercial-grade speed and accuracy without high operation costs.
