# Chatbot Builder Platform – Architecture Blueprint & Data Flow (Draft v1)

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
