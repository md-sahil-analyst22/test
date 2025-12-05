### 3.1 Overall Flow Chart – Text & Voice Request

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
