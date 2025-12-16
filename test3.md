``` mermaid
flowchart TD
  A[User Message Arrives] --> B[Resolve owner key<br/>owner_type: user or session<br/>owner_id: user_id or session_id<br/>thread_id]
  B --> C[Fetch last N messages for owner+thread<br/>(ordered newest->oldest)]
  C --> D[Build prompt: system + window + current]
  D --> E[LLM generate response]
  E --> F[Persist user msg + assistant msg]
  F --> G[Enforce window limit N]
  G --> H{Count > N?}
  H -- No --> Z[Return response]
  H -- Yes --> I[Delete oldest messages beyond N<br/>for owner+thread]
  I --> Z

```


``` mermaid
flowchart TB
  %% =========================
  %% CONTEXT MEMORY ARCHITECTURE (Hard bounded N for all users)
  %% =========================

  subgraph CLIENT["Client"]
    UI["Web / App UI"]
    SID["Session ID store<br/>(cookie/localStorage)"]
    LOGIN["Optional Login<br/>(yields user_id)"]
    UI --> SID
    UI --> LOGIN
  end

  subgraph API["Backend API (FastAPI)"]
    CHAT["POST /chat/send<br/>message + thread_id + (session_id|user_id)"]
  end

  UI --> CHAT

  subgraph ORCH["Memory Policy Orchestrator (LangGraph)"]
    ID["Identity Resolver<br/>owner_type + owner_id + thread_id"]
    LOAD["Load Window Memory<br/>last N turns/messages"]
    PROMPT["Prompt Builder<br/>system + window + current"]
    GEN["LLM Call (via LangChain wrapper)"]
    SAVE["Persist Messages"]
    ENFORCE["Enforce hard limit N<br/>count -> delete oldest"]
  end

  CHAT --> ID --> LOAD --> PROMPT --> GEN --> SAVE --> ENFORCE --> OUT["Return response"]

  subgraph DATA["Supabase Postgres (source of truth)"]
    MSG["Table: chat_messages<br/>owner_type, owner_id, thread_id<br/>role, content, created_at"]
    IDX["Indexes<br/>owner_type+owner_id+thread_id+created_at"]
  end

  LOAD --> MSG
  SAVE --> MSG
  ENFORCE --> MSG
  IDX --- MSG

  subgraph CACHE["Optional Cache (Redis)"]
    RWIN["Hot window cache<br/>owner+thread -> last N"]
  end

  LOAD -. optional .-> RWIN
  SAVE -. optional .-> RWIN
  ENFORCE -. optional .-> RWIN


```

``` mermaid
flowchart TD
  A[After saving new user+assistant] --> B[Compute total pairs for owner+thread]
  B --> C{Pairs > N?}
  C -- No --> Z[Done]
  C -- Yes --> D[Find oldest pairs beyond N]
  D --> E[Delete both messages of each pair]
  E --> Z


```
