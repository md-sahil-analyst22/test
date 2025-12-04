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
graph TD

subgraph Client_Side["Client Side"]
  WIDGET["JS Chat Widget<br/>(Embedded on client site)"]
  PY_ADMIN["Python-based Admin UI<br/>(Jinja/HTMX/Streamlit)"]
  REACT_ADMIN["React Admin Dashboard<br/>(Optional)"]
end

subgraph Backend["Backend Layer (80% FastAPI, 20% Django)"]
  APIGW["FastAPI API Layer<br/>/api/v1/*, Webhooks, WS"]
  DJANGO["Django App<br/>Admin, selected views"]
  ORCH["Orchestration Engine<br/>LangChain + LangGraph"]
  SRV_CHAT["Chatbot Service<br/>Session, routing"]
  SRV_CFG["Chatbot Config Service<br/>CRUD Chatbots & Providers"]
  SRV_ANALYTICS["Analytics Service"]
end

subgraph AI_Providers["AI & Voice Providers"]
  GEMINI["Gemini LLM & Embeddings"]
  SARVAM["Sarvam STT/TTS/Translation"]
end

subgraph Data_Layer["Data & Storage Layer"]
  SUPABASE["Supabase (Self-hosted)<br/>Postgres + pgvector"]
  D_APP["App DB Schemas<br/>Users, Chatbots, Conversations"]
  D_VEC["Vector Store Collections<br/>Embeddings per chatbot"]
  D_LOGS["Logs & Analytics Store"]
end

WIDGET -->|"Webhook + WS"| APIGW
PY_ADMIN -->|"REST / Django Views"| APIGW
REACT_ADMIN -->|"REST / GraphQL"| APIGW

APIGW --> DJANGO
APIGW --> SRV_CHAT
APIGW --> SRV_CFG
APIGW --> SRV_ANALYTICS

SRV_CHAT --> ORCH
ORCH --> GEMINI
ORCH --> SARVAM
ORCH --> SUPABASE

SUPABASE --> D_APP
SUPABASE --> D_VEC
SRV_ANALYTICS --> D_LOGS
