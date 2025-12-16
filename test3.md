``` mermaid
flowchart TD
  A[User Message Arrives] --> B{Signed in?}

  %% Anonymous flow
  B -- No --> C1[Set owner = session_id]
  C1 --> D1[Load last N messages for thread]
  D1 --> E1[Build prompt: system + window + current]
  E1 --> F1[LLM generate response]
  F1 --> G1[Store chat_messages for session]
  G1 --> Z[End]

  %% Signed-in flow
  B -- Yes --> C2[Set owner = user_id]
  C2 --> D2[Load last N messages for thread]
  D2 --> R[Embed current message with Gemini]
  R --> S[Vector search memory_chunks top-K filtered by user_id and not deleted]
  S --> E2[Build prompt: system + retrieved memory + window + current]
  E2 --> F2[LLM generate response]
  F2 --> G2[Store chat_messages for user]
  G2 --> I[Chunk new content and embed]
  I --> J[Upsert chat_memory_chunks in pgvector]
  J --> Z
```
