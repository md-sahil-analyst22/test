### 4.1 DFD Level-0

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
