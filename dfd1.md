### 4.2 DFD Level-1

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
