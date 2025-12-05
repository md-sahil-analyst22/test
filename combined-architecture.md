# Chatbot Builder Platform 

## 1. System Overview

This document describes the **current demo / PoC architecture** of the Chatbot Builder Platform with the following technical baseline:

- **Backend**
  - **80% FastAPI** – primary API layer, orchestration endpoints, webhooks, chat APIs.
  - **20% Django** – selected use-cases: admin/config UI, some server-rendered pages, optional built-in auth/admin.
- **Frontend**
  - Python-based frontends (e.g. Jinja2/HTMX, Streamlit, etc.) for internal tools.
  - Optional **React-based admin dashboard** and **JS chat widget** embedded into client sites.
- **AI & Orchestration**
  - **Gemini** – main LLM + embedding generator.
  - **Sarvam** – STT, TTS, translation (demo/testing).
  - **LangChain + LangGraph** – orchestration of RAG, tools, and conversation flows.
  - **Self-hosted Supabase (Postgres + pgvector)** – vector database for knowledge base.
- **Integration**
  - A **webhook endpoint** (FastAPI) is generated per chatbot and used by the **chat widget** to send/receive messages.

---

## 2. High-Level Architecture Blueprint

### 2.1 Component View

```mermaid
graph TB
    %% Styles
    classDef comp fill:#eef7ff,stroke:#5b8ff6,stroke-width:1px,color:#000
    classDef group fill:#fff7e6,stroke:#ffa940,stroke-width:1px,color:#000

    %% =============================
    %% CLIENT SIDE (COMPONENTS)
    %% =============================
    subgraph CLIENT["Client Side"]
        WIDGET["JS Chat Widget\n(Embedded on client site)"]:::comp
        PY_UI["Python-based Admin UI\n(Jinja/HTMX/Streamlit)"]:::comp
        REACT_UI["React Admin Dashboard\n(Optional)"]:::comp
    end

    %% =============================
    %% BACKEND LAYER (COMPONENTS)
    %% =============================
    subgraph BACKEND["Backend Layer (80% FastAPI / 20% Django)"]
        FASTAPI["FastAPI API Layer\n(/api/v1/*, Webhooks, WS)"]:::comp
        DJANGO["Django App\n(Admin, selected views)"]:::comp
        CHAT_SVC["Chatbot Service\n(Session, routing)"]:::comp
        CFG_SVC["Chatbot Config Service\n(CRUD Chatbots & Providers)"]:::comp
        ANALYTICS_SVC["Analytics Service"]:::comp
        ORCH["Orchestration Engine\n(LangChain + LangGraph)"]:::comp
    end

    %% =============================
    %% AI & VOICE PROVIDERS
    %% =============================
    subgraph AI["AI & Voice Providers"]
        GEMINI["Gemini\nLLM & Embeddings"]:::comp
        SARVAM["Sarvam\nSTT / TTS / Translation"]:::comp
    end

    %% =============================
    %% DATA & STORAGE LAYER
    %% =============================
    subgraph DATA["Data & Storage Layer"]
        SUPABASE["Supabase (Self-hosted)\nPostgres + pgvector"]:::comp
        LOGS["Logs & Analytics Store"]:::comp
    end

    %% =============================
    %% RELATIONSHIPS (STRUCTURAL)
    %% =============================

    %% Client → Backend contracts
    WIDGET ---|"Webhook + WebSocket"| FASTAPI
    PY_UI ---|"REST / Django Views"| FASTAPI
    REACT_UI ---|"REST / API / GraphQL"| FASTAPI

    %% Backend internal components
    FASTAPI --- DJANGO
    FASTAPI --- CHAT_SVC
    FASTAPI --- CFG_SVC
    FASTAPI --- ANALYTICS_SVC

    CHAT_SVC --- ORCH
    CFG_SVC --- SUPABASE
    ANALYTICS_SVC --- LOGS

    %% Orchestration dependencies
    ORCH --- GEMINI
    ORCH --- SARVAM
    ORCH --- SUPABASE
```

### 2.2 Overall Flow Chart – Text & Voice Request

```mermaid
flowchart TD
    classDef step fill:#eef7ff,stroke:#5b8ff6,stroke-width:1px,color:#000
    classDef decision fill:#fff7e6,stroke:#ffa940,stroke-width:1px,color:#000
    classDef ext fill:#ffe7ba,stroke:#d46b08,stroke-width:1px,color:#000
    classDef data fill:#f6ffed,stroke:#73d13d,stroke-width:1px,color:#000

    U[User]:::ext --> W["Client Chat Widget (Web / App)"]:::step

    W --> T1{"Input Type?"}:::decision

    %% TEXT PATH
    T1 -->|Text| T_API["FastAPI Chat/Webhook Endpoint (/api/v1/chat or /webhook)"]:::step
    T_API --> T_VAL["Validate chatbot_id, auth, load chatbot config"]:::step

    T_VAL --> T_OK{"Valid & Active Chatbot?"}:::decision
    T_OK -->|No| T_ERR["Return error (invalid / disabled chatbot)"]:::step
    T_ERR --> T_END_TEXT["Render error in widget"]:::step

    T_OK -->|Yes| T_STATE["Build conversation state (session, history, config)"]:::step
    T_STATE --> T_GRAPH["Invoke LangGraph flow (standard_graph)"]:::step

    T_GRAPH --> T_RAG{"Need KB / RAG?"}:::decision
    T_RAG -->|Yes| T_VEC_REQ["Embed / Query via LangChain (Gemini embeddings → Supabase)"]:::step
    T_VEC_REQ --> T_VEC_DB["Supabase pgvector (retrieve chunks)"]:::data
    T_VEC_DB --> T_CTX["Return context to graph"]:::step

    T_RAG -->|No| T_DIRECT["Skip RAG – direct LLM call"]:::step

    T_CTX --> T_LLM
    T_DIRECT --> T_LLM

    T_LLM["Call Gemini LLM (prompt + context)"]:::step
    T_LLM --> T_POST["Post-process response (formatting, safety, truncation)"]:::step
    T_POST --> T_LOG["Log conversation & metrics (Analytics Service / DB)"]:::step
    T_LOG --> T_RESP["Return JSON response (text reply)"]:::step
    T_RESP --> T_WIDGET["Widget renders reply (chat bubble)"]:::step

    %% VOICE PATH
    T1 -->|Voice| V_WS["WebSocket connection (audio stream)"]:::step
    V_WS --> V_STT["Stream audio to Sarvam STT (speech → text)"]:::step
    V_STT --> V_TXT["Transcribed text (normalized message)"]:::step

    %% Voice uses same FastAPI text pipeline
    V_TXT --> T_API

    %% OPTIONAL TTS RETURN
    T_POST --> V_TTS_DEC{"Voice reply needed?"}:::decision
    V_TTS_DEC -->|No| T_LOG
    V_TTS_DEC -->|Yes| V_TTS["Call Sarvam TTS (text → audio)"]:::step
    V_TTS --> V_STREAM["Stream audio back via WebSocket"]:::step
    V_STREAM --> V_PLAY["Widget plays audio reply"]:::step
```

### 2.3 DFD 

#### 2.3.1 DFD Level-0

```mermaid
flowchart TD

subgraph EXT_Users["External Entities"]
  EU[End User]
  ADM[Admin User]
end

subgraph EXT_Providers["External AI Providers"]
  GEM[GEMINI API]
  SAR[SARVAM API]
end

subgraph SYS["Chatbot Builder Platform"]
  P0[0.0 Chatbot Platform]
end

DB_APP[(App DB<br/>Supabase/Postgres)]
DB_VEC[(Vector Store<br/>pgvector in Supabase)]
DB_LOG[(Logs & Analytics)]

EU -->|"User Message / Voice"| P0
P0 -->|"Chatbot Reply / Voice"| EU

ADM -->|"Config, CRUD Chatbots, Providers"| P0
P0 -->|"Dashboards, Analytics, Status"| ADM

P0 --> GEM
GEM --> P0

P0 --> SAR
SAR --> P0

P0 --> DB_APP
DB_APP --> P0

P0 --> DB_VEC
DB_VEC --> P0

P0 --> DB_LOG
DB_LOG --> P0
```

#### 2.3.2 DFD Level-1

```mermaid
flowchart TD

%% External Entities
EU[End User]
ADM[Admin User]
GEM[GEMINI API]
SAR[SARVAM API]

%% Data Stores
DB_APP[(D1 App DB<br/>Users, Chatbots, Conversations)]
DB_VEC[(D2 Vector Store<br/>Embeddings / KB)]
DB_LOG[(D3 Logs & Analytics)]

%% Processes
P1[1.0 API Gateway & Auth<br/>FastAPI + Django]
P2[2.0 Orchestration Engine<br/>LangChain + LangGraph]
P3[3.0 Vector & KB Manager<br/>Supabase + Embeddings]
P4[4.0 Voice & Translation Integrator<br/>Sarvam]
P5[5.0 Analytics & Logging]
P6[6.0 Admin & Config Management<br/>Django/FastAPI]

%% Data Flows – User
EU -->|"1.1 Chat Messages / Voice Frames"| P1
P1 -->|"1.2 Text Reply / Voice Stream"| EU

%% P1 -> P2
P1 -->|"2.1 Normalized Request<br/>(user, session, chatbot config)"| P2
P2 -->|"2.2 Response Text / Structured Output"| P1

%% P2 <-> P3
P2 -->|"3.1 Embed & Query Request"| P3
P3 -->|"3.2 Retrieved Context / Docs"| P2

%% P3 <-> DBs
P3 -->|"Write / Read Metadata"| DB_APP
DB_APP -->|"Chatbot Config, Docs, etc."| P3

P3 -->|"Store / Fetch Embeddings"| DB_VEC
DB_VEC -->|"Chunks, Vectors"| P3

%% P2 <-> GEMINI
P2 -->|"LLM & Embedding Calls"| GEM
GEM -->|"Model Outputs / Embeds"| P2

%% P2 <-> P4 (Voice)
P2 -->|"4.1 Text Response for TTS"| P4
P4 -->|"4.2 Audio Response"| P2

P1 -->|"4.3 Audio Frames for STT"| P4
P4 -->|"4.4 Transcribed Text"| P2

P4 -->|"Translation Requests"| SAR
SAR -->|"Translated Text / Audio"| P4

%% P2 -> P5
P2 -->|"5.1 Conversation Events / Metrics"| P5
P5 -->|"5.2 Aggregated Metrics"| DB_LOG

%% Admin Flows
ADM -->|"6.1 Admin Actions<br/>Create/Update Chatbots, Providers"| P6
P6 -->|"6.2 Persist Config"| DB_APP
DB_APP -->|"6.3 Config Data, Chatbot List"| P6
P6 -->|"6.4 Config Data / Dashboard to Admin"| ADM
```

# Chatbot Platform – Project Structure

```text
chatbot-platform/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI application entry
│   │   ├── config.py                  # Settings & environment config
│   │   │
│   │   ├── api/                       # API endpoints
│   │   │   ├── __init__.py
│   │   │   ├── deps.py                # Dependency injection
│   │   │   └── v1/
│   │   │       ├── auth.py            # Authentication endpoints
│   │   │       ├── chatbots.py        # Chatbot CRUD
│   │   │       ├── chat.py            # Chat endpoints
│   │   │       ├── webhooks.py        # Webhook management
│   │   │       ├── analytics.py       # Analytics & reporting
│   │   │       └── providers.py       # Provider configuration
│   │   │
│   │   ├── models/                    # SQLAlchemy ORM models
│   │   │   ├── user.py
│   │   │   ├── chatbot.py
│   │   │   ├── conversation.py
│   │   │   ├── webhook.py
│   │   │   └── analytics.py
│   │   │
│   │   ├── schemas/                   # Pydantic schemas
│   │   │   ├── user.py
│   │   │   ├── chatbot.py
│   │   │   ├── chat.py
│   │   │   └── provider.py
│   │   │
│   │   ├── services/                  # Business logic layer
│   │   │   ├── chatbot_service.py     # Core chatbot logic
│   │   │   ├── webhook_service.py     # Webhook generation
│   │   │   ├── session_service.py     # Session management
│   │   │   ├── analytics_service.py   # Analytics processing
│   │   │   └── billing_service.py     # Usage & cost tracking
│   │   │
│   │   ├── providers/                 # 🔌 Provider Abstraction Layer
│   │   │   ├── __init__.py
│   │   │   ├── base.py                # Abstract base classes
│   │   │   ├── factory.py             # Factory & Registry pattern
│   │   │   │
│   │   │   ├── llm/                   # LLM providers
│   │   │   │   ├── openai_provider.py
│   │   │   │   ├── anthropic_provider.py
│   │   │   │   ├── cohere_provider.py
│   │   │   │   └── ollama_provider.py
│   │   │   │
│   │   │   ├── vector_db/             # Vector database providers
│   │   │   │   ├── qdrant_provider.py
│   │   │   │   ├── pinecone_provider.py
│   │   │   │   └── weaviate_provider.py
│   │   │   │
│   │   │   ├── stt/                   # Speech-to-Text providers
│   │   │   │   ├── sarvam_provider.py
│   │   │   │   ├── whisper_provider.py
│   │   │   │   └── deepgram_provider.py
│   │   │   │
│   │   │   ├── tts/                   # Text-to-Speech providers
│   │   │   │   ├── sarvam_provider.py
│   │   │   │   ├── elevenlabs_provider.py
│   │   │   │   └── openai_tts_provider.py
│   │   │   │
│   │   │   ├── translation/           # Translation providers
│   │   │   │   ├── sarvam_provider.py
│   │   │   │   ├── google_provider.py
│   │   │   │   └── deepl_provider.py
│   │   │   │
│   │   │   ├── ocr/                   # OCR providers
│   │   │   │   ├── tesseract_provider.py
│   │   │   │   └── google_vision_provider.py
│   │   │   │
│   │   │   └── image/                 # Image processing
│   │   │       ├── opencv_provider.py
│   │   │       └── vision_llm_provider.py
│   │   │
│   │   ├── orchestration/             # LangChain / LangGraph orchestration
│   │   │   ├── __init__.py
│   │   │   ├── base_state.py          # Shared conversation state
│   │   │   ├── common_tools.py        # Tools exposed to graphs
│   │   │   ├── standard_graph.py      # Default text-chat graph
│   │   │   ├── voice_graph.py         # Voice pipeline graph (STT → LLM → TTS)
│   │   │   └── supervisor_graph.py    # Routing / multi-agent supervisor
│   │   │
│   │   ├── websocket/                 # WebSocket handlers
│   │   │   ├── connection_manager.py  # Connection pool management
│   │   │   ├── voice_chat_handler.py  # Real-time voice streaming
│   │   │   └── chat_handler.py        # Text chat streaming
│   │   │
│   │   ├── core/                      # Core utilities
│   │   │   ├── security.py            # JWT, encryption
│   │   │   ├── cache.py               # Redis caching utilities
│   │   │   ├── exceptions.py          # Custom exceptions
│   │   │   └── logging.py             # Logging configuration
│   │   │
│   │   ├── db/                        # Database configuration
│   │   │   ├── session.py             # Database session
│   │   │   └── base.py                # Base model
│   │   │
│   │   └── tasks/                     # Celery background tasks
│   │       ├── celery_app.py          # Celery configuration
│   │       ├── document_processing.py # Document embedding (LangChain)
│   │       ├── analytics.py           # Analytics aggregation
│   │       └── cleanup.py             # Maintenance tasks
│   │
│   ├── alembic/                       # Database migrations
│   │   ├── versions/
│   │   └── env.py
│   │
│   ├── tests/                         # Test suite
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   │
│   ├── requirements.txt               # Python deps (FastAPI, Django, LangChain, LangGraph, etc.)
│   ├── Dockerfile                     # Backend container
│   └── docker-compose.yml             # Local backend-only dev setup
│
├── frontend/
│   ├── admin-dashboard/               # React admin application
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   └── main.tsx
│   │   ├── package.json
│   │   └── Dockerfile
│   │
│   └── chat-widget/                   # Embeddable widget
│       ├── src/
│       │   ├── widget.js              # Main widget code
│       │   └── styles.css
│       ├── package.json
│       └── dist/                      # Built widget files
│
├── k8s/                               # Kubernetes manifests
│   ├── namespace.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   ├── redis-deployment.yaml
│   ├── postgres-deployment.yaml
│   ├── ingress.yaml
│   └── configmap.yaml
│
├── scripts/                           # Utility scripts
│   ├── init_db.py                     # Database initialization
│   ├── seed_data.py                   # Sample data
│   └── backup.sh                      # Backup script
│
├── docs/                              # Documentation
│   ├── api.md                         # API documentation
│   ├── providers.md                   # Provider integration guide
│   ├── deployment.md                  # Deployment guide
│   └── architecture.md                # Architecture details
│
├── .env.example                       # Environment variables template
├── .gitignore
├── docker-compose.yml                 # Full stack composition (backend + frontend + DB + Redis)
├── docker-compose.prod.yml            # Production composition
├── Makefile                           # Common commands
└── README.md                          # Project overview



