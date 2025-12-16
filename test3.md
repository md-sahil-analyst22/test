``` mermaid
flowchart TB
    A["User Message Arrives"] --> B{"Signed in?"}
    B -- No --> C1["Owner=session_id"]
    C1 --> D1["Load last N msgs for thread"]
    D1 --> E1["Build prompt: system + window + current msg"]
    E1 --> F["LLM Generate Response"]
    F --> G["Store chat_messages (session)"] & G2["Store chat_messages (user)"]
    G --> H1["END"]
    B -- Yes --> C2["Owner=user_id"]
    C2 --> D2["Load last N msgs for thread"]
    D2 --> R["Embed current msg Gemini embeddings"]
    R --> S["Vector search memory_chunks top-K filter user_id + deleted_at null"]
    S --> E2["Build prompt: system + memories + window + current msg"]
    E2 --> F
    G2 --> I["Chunk+Embed new content"]
    I --> J["Upsert chat_memory_chunks pgvector"]
    J --> H2["END"]
```
