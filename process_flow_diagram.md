# AI Call Centre Platform — Process Flow Diagrams

## 1. Inbound Call Flow

```mermaid
flowchart TD
    START_IN(["📞 Incoming Call<br/>(PSTN / SIP)"])
    
    START_IN --> TEL_RECV["Telephony Provider<br/>(Exotel/JIO/Ozonetel)<br/>Receives Call"]
    TEL_RECV --> CONSENT{"Recording Consent<br/>Prompt Played?"}
    CONSENT -->|"Accepted"| DID_MAP["DID Number →<br/>Tenant Mapping"]
    CONSENT -->|"Declined"| LOG_DECLINE["Log: Consent Declined"]
    LOG_DECLINE --> DISCONNECT_1(["🔴 Disconnect"])
    
    DID_MAP --> TENANT_LOOKUP["Load Tenant Config<br/>(LLM, STT, TTS,<br/>Prompts, KB)"]
    TENANT_LOOKUP --> IVR_CHECK{"IVR Menu<br/>Configured?"}
    
    IVR_CHECK -->|"Yes"| IVR["Play IVR Menu<br/>Collect DTMF Input"]
    IVR_CHECK -->|"No"| LIVEKIT_ROOM["Create LiveKit Room<br/>+ Join Audio Tracks"]
    IVR --> ROUTE_DECISION{"Route To?"}
    ROUTE_DECISION -->|"AI Agent"| LIVEKIT_ROOM
    ROUTE_DECISION -->|"Human Agent"| HUMAN_QUEUE["Queue for<br/>Human Agent"]
    ROUTE_DECISION -->|"Voicemail"| VOICEMAIL["Record<br/>Voicemail"]
    VOICEMAIL --> SAVE_VM["Save to Storage +<br/>Transcribe"] --> END_VM(["🔴 End Call"])
    
    HUMAN_QUEUE --> HUMAN_AVAIL{"Agent<br/>Available?"}
    HUMAN_AVAIL -->|"Yes"| CONNECT_HUMAN["Connect to<br/>Human Agent"]
    HUMAN_AVAIL -->|"No"| FALLBACK_AI["Route to<br/>AI Agent"]
    FALLBACK_AI --> LIVEKIT_ROOM
    
    LIVEKIT_ROOM --> AUDIO_STREAM["Full-Duplex Audio<br/>Stream Established"]
    AUDIO_STREAM --> NOISE_CANCEL["Noise Cancellation<br/>(RNNoise)"]
    NOISE_CANCEL --> AUDIO_PREPROC["Audio Preprocessing<br/>(AGC + Echo Cancel)"]
    
    AUDIO_PREPROC --> CONVERSATION_LOOP

    subgraph CONVERSATION_LOOP["🔄 Conversation Loop (Real-Time)"]
        direction TB
        VAD_DETECT["Silero VAD<br/>Detects Speech"]
        VAD_DETECT --> STT_PROCESS["STT: Speech → Text<br/>(Google/Sarvam)"]
        STT_PROCESS --> BARGE_CHECK{"Barge-In<br/>Detected?"}
        BARGE_CHECK -->|"Yes"| STOP_TTS["Stop Current TTS<br/>Playback"] --> LANGGRAPH_PROC
        BARGE_CHECK -->|"No"| LANGGRAPH_PROC
        
        LANGGRAPH_PROC["LangGraph<br/>State Machine"]
        LANGGRAPH_PROC --> CONTEXT_LOAD["Load Context<br/>from Redis"]
        CONTEXT_LOAD --> NEED_KB{"Needs Knowledge<br/>Base Lookup?"}
        NEED_KB -->|"Yes"| RAG_SEARCH["RAG: Embed Query →<br/>Vector Search (Qdrant)"]
        RAG_SEARCH --> LLM_PROMPT
        NEED_KB -->|"No"| LLM_PROMPT
        
        LLM_PROMPT["Compose Prompt<br/>(System + Context +<br/>RAG + User Input)"]
        LLM_PROMPT --> LLM_CALL["LLM Generate Response<br/>(Gemini/Sarvam)<br/>Streaming"]
        LLM_CALL --> TOOL_CHECK{"Tool Call<br/>Required?"}
        TOOL_CHECK -->|"CRM Lookup"| CRM_TOOL["Fetch CRM Data"] --> LLM_CALL
        TOOL_CHECK -->|"Transfer"| TRANSFER["Transfer to<br/>Human/Dept"]
        TOOL_CHECK -->|"No"| TTS_PROCESS
        
        TTS_PROCESS["TTS: Text → Speech<br/>(Google/Sarvam)<br/>Streaming"]
        TTS_PROCESS --> PLAY_AUDIO["Stream Audio to<br/>Caller via LiveKit"]
        PLAY_AUDIO --> SAVE_CONTEXT["Save Turn to<br/>Redis Context"]
    end
    
    CONNECT_HUMAN --> LIVE_MONITOR["Admin: Live Transcript<br/>+ Translation View"]

    CONVERSATION_LOOP --> PARALLEL_OPS

    subgraph PARALLEL_OPS["⚡ Parallel Operations (During Call)"]
        direction LR
        TRANSCRIBE_LIVE["Stream Transcript<br/>via WebSocket"]
        TRANSLATE_LIVE["Real-Time Translation<br/>(Bengali↔Hindi↔English)"]
        SENTIMENT_LIVE["Sentiment Analysis<br/>Per Utterance"]
        CRM_UPDATE["Update CRM:<br/>Call Log + Notes"]
    end
    
    PARALLEL_OPS --> CALL_END{"Call Ends?"}
    CALL_END -->|"No"| CONVERSATION_LOOP
    CALL_END -->|"Yes"| POST_CALL

    subgraph POST_CALL["📋 Post-Call Processing"]
        direction TB
        SAVE_RECORDING["Encrypt & Save<br/>Call Recording<br/>(AES-256)"]
        FINAL_TRANSCRIPT["Finalize<br/>Transcript"]
        QUALITY_SCORE["Generate Call<br/>Quality Score"]
        CRM_FINAL["Update CRM:<br/>Outcome, Tags,<br/>Follow-up"]
        ANALYTICS_UPDATE["Update Analytics<br/>Dashboards"]
    end
    
    POST_CALL --> END_IN(["✅ Call Complete"])

    %% Styling
    classDef startEnd fill:#4CAF50,stroke:#2E7D32,color:#FFF,stroke-width:2px
    classDef process fill:#E3F2FD,stroke:#1565C0,color:#000,stroke-width:1px
    classDef decision fill:#FFF3E0,stroke:#E65100,color:#000,stroke-width:2px
    classDef ai fill:#E8F5E9,stroke:#2E7D32,color:#000,stroke-width:1px
    classDef danger fill:#FFEBEE,stroke:#C62828,color:#000,stroke-width:1px
    classDef parallel fill:#F3E5F5,stroke:#7B1FA2,color:#000,stroke-width:1px

    class START_IN,END_IN,END_VM startEnd
    class DISCONNECT_1 danger
    class CONSENT,IVR_CHECK,ROUTE_DECISION,HUMAN_AVAIL,BARGE_CHECK,NEED_KB,TOOL_CHECK,CALL_END decision
    class VAD_DETECT,STT_PROCESS,LANGGRAPH_PROC,LLM_CALL,TTS_PROCESS,RAG_SEARCH,LLM_PROMPT ai
    class TRANSCRIBE_LIVE,TRANSLATE_LIVE,SENTIMENT_LIVE,CRM_UPDATE parallel
```

---

## 2. Outbound Call Flow

```mermaid
flowchart TD
    START_OUT(["📤 Outbound Call<br/>Trigger"])
    
    START_OUT --> TRIGGER_TYPE{"Trigger<br/>Source?"}
    TRIGGER_TYPE -->|"Manual"| DISPATCH_UI["Operator: Click-to-Call<br/>from Dispatch UI"]
    TRIGGER_TYPE -->|"Campaign"| CAMPAIGN["Campaign Manager:<br/>Upload Contact List"]
    TRIGGER_TYPE -->|"Scheduled"| SCHEDULER["Scheduler Service:<br/>Time-Zone Aware"]
    
    CAMPAIGN --> VALIDATE_LIST["Validate Numbers +<br/>Deduplicate"]
    SCHEDULER --> VALIDATE_LIST
    DISPATCH_UI --> SINGLE_CALL["Single Call<br/>Request"]
    
    VALIDATE_LIST --> DND_CHECK["Check DND<br/>Registry (TRAI)"]
    SINGLE_CALL --> DND_CHECK
    
    DND_CHECK --> DND_RESULT{"On DND<br/>List?"}
    DND_RESULT -->|"Yes"| DND_SKIP["Skip Number +<br/>Log: DND Blocked"]
    DND_RESULT -->|"No"| TIME_CHECK{"Within Allowed<br/>Call Hours?<br/>(9AM-9PM IST)"}
    
    DND_SKIP --> NEXT_NUMBER["Process Next<br/>Number in Queue"]
    
    TIME_CHECK -->|"No"| RESCHEDULE["Reschedule for<br/>Next Window"]
    TIME_CHECK -->|"Yes"| RATE_CHECK{"Within Tenant<br/>Rate Limit?"}
    
    RATE_CHECK -->|"No"| THROTTLE["Queue: Wait for<br/>Rate Limit Window"]
    RATE_CHECK -->|"Yes"| INIT_CALL["Initiate Call via<br/>Telephony Provider"]
    
    INIT_CALL --> CALL_STATUS{"Call<br/>Status?"}
    CALL_STATUS -->|"Answered"| CONSENT_OUT["Play Recording<br/>Consent Prompt"]
    CALL_STATUS -->|"Busy"| RETRY_Q["Add to Retry<br/>Queue"]
    CALL_STATUS -->|"No Answer"| RETRY_Q
    CALL_STATUS -->|"Failed"| LOG_FAIL["Log Failure +<br/>Alert"]
    
    RETRY_Q --> RETRY_CHECK{"Max Retries<br/>Reached?"}
    RETRY_CHECK -->|"No"| BACKOFF["Exponential Backoff<br/>+ Reschedule"] --> INIT_CALL
    RETRY_CHECK -->|"Yes"| LOG_EXHAUST["Log: All Retries<br/>Exhausted"]
    
    CONSENT_OUT --> CONSENT_RESULT{"Consent<br/>Given?"}
    CONSENT_RESULT -->|"Yes"| LOAD_CONFIG["Load Tenant Config +<br/>Campaign Prompt +<br/>Contact Context"]
    CONSENT_RESULT -->|"No"| LOG_NO_CONSENT["Log: No Consent"] --> HANGUP(["🔴 Hangup"])
    
    LOAD_CONFIG --> CREATE_ROOM["Create LiveKit Room +<br/>Establish Full-Duplex"]
    CREATE_ROOM --> AI_GREETING["AI Agent:<br/>Play Greeting<br/>(Campaign-Specific)"]
    AI_GREETING --> CONV_LOOP_OUT["Enter Conversation<br/>Loop<br/>(Same as Inbound)"]
    
    CONV_LOOP_OUT --> OUTCOME{"Call<br/>Outcome?"}
    OUTCOME -->|"Interested"| TAG_HOT["Tag: Hot Lead +<br/>Schedule Follow-Up"]
    OUTCOME -->|"Not Interested"| TAG_COLD["Tag: Cold +<br/>DNC if Requested"]
    OUTCOME -->|"Callback"| TAG_CALLBACK["Schedule Callback<br/>at Preferred Time"]
    OUTCOME -->|"Transfer"| TRANSFER_OUT["Transfer to<br/>Human Agent"]
    
    TAG_HOT --> POST_OUT
    TAG_COLD --> POST_OUT
    TAG_CALLBACK --> POST_OUT
    TRANSFER_OUT --> POST_OUT

    subgraph POST_OUT["📋 Post-Call"]
        direction LR
        UPDATE_CRM_OUT["Update CRM +<br/>Call Outcome"]
        SAVE_REC_OUT["Save Recording +<br/>Transcript"]
        UPDATE_CAMPAIGN["Update Campaign<br/>Progress"]
        NEXT_NUM["Trigger Next<br/>Number"]
    end
    
    POST_OUT --> NEXT_NUMBER
    NEXT_NUMBER --> CAMPAIGN_DONE{"Campaign<br/>Complete?"}
    CAMPAIGN_DONE -->|"No"| DND_CHECK
    CAMPAIGN_DONE -->|"Yes"| REPORT["Generate Campaign<br/>Report"]
    REPORT --> END_OUT(["✅ Campaign<br/>Complete"])

    classDef startEnd fill:#4CAF50,stroke:#2E7D32,color:#FFF,stroke-width:2px
    classDef process fill:#E3F2FD,stroke:#1565C0,color:#000,stroke-width:1px
    classDef decision fill:#FFF3E0,stroke:#E65100,color:#000,stroke-width:2px
    classDef blocked fill:#FFEBEE,stroke:#C62828,color:#000,stroke-width:1px

    class START_OUT,END_OUT startEnd
    class HANGUP blocked
    class TRIGGER_TYPE,DND_RESULT,TIME_CHECK,RATE_CHECK,CALL_STATUS,RETRY_CHECK,CONSENT_RESULT,OUTCOME,CAMPAIGN_DONE decision
```

---

## 3. Admin & Tenant Onboarding Flow

```mermaid
flowchart TD
    ADMIN_START(["🔧 Admin / Tenant<br/>Management"])

    ADMIN_START --> ADMIN_TYPE{"Action<br/>Type?"}
    
    ADMIN_TYPE -->|"New Tenant"| SIGNUP["Tenant Signup<br/>Registration Form"]
    ADMIN_TYPE -->|"Configure"| CONFIG_MENU["Configuration<br/>Dashboard"]
    ADMIN_TYPE -->|"Monitor"| MONITOR_MENU["Monitoring<br/>Dashboard"]
    
    %% Onboarding Flow
    SIGNUP --> EMAIL_VERIFY["Email Verification"]
    EMAIL_VERIFY --> PLAN_SELECT["Select Plan<br/>(Starter/Pro/Enterprise)"]
    PLAN_SELECT --> PAYMENT["Setup Payment<br/>(Razorpay)"]
    PAYMENT --> WIZARD["Onboarding Wizard"]
    
    WIZARD --> STEP1["Step 1: Connect<br/>Telephony Provider"]
    STEP1 --> STEP2["Step 2: Select<br/>AI Config<br/>(LLM/STT/TTS)"]
    STEP2 --> STEP3["Step 3: Upload<br/>Knowledge Base"]
    STEP3 --> STEP4["Step 4: Configure<br/>Prompts & Persona"]
    STEP4 --> STEP5["Step 5: Test Call<br/>(Sandbox)"]
    STEP5 --> GO_LIVE["Go Live!"]
    
    %% Configuration
    CONFIG_MENU --> CFG_TEL["Telephony:<br/>Provider, SIP, DID"]
    CONFIG_MENU --> CFG_AI["AI: LLM, STT,<br/>TTS, VAD"]
    CONFIG_MENU --> CFG_DB["Database:<br/>PG/Supabase/Qdrant"]
    CONFIG_MENU --> CFG_PROMPT["Prompts:<br/>Templates, Versions"]
    CONFIG_MENU --> CFG_KB["Knowledge Base:<br/>Docs, FAQs, URLs"]
    CONFIG_MENU --> CFG_BRAND["White-Label:<br/>Logo, Colors, Domain"]
    
    %% Monitoring
    MONITOR_MENU --> MON_LIVE["Live Calls<br/>Dashboard"]
    MONITOR_MENU --> MON_TRANS["Real-Time<br/>Transcripts"]
    MONITOR_MENU --> MON_ANALYTICS["Analytics:<br/>Volume, Quality,<br/>Sentiment"]
    MONITOR_MENU --> MON_BILLING["Billing:<br/>Usage, Invoices"]
    MONITOR_MENU --> MON_AUDIT["Audit Logs"]
    
    MON_TRANS --> TRANSLATION_VIEW["Side-by-Side<br/>Translation View"]

    classDef startEnd fill:#4CAF50,stroke:#2E7D32,color:#FFF,stroke-width:2px
    classDef decision fill:#FFF3E0,stroke:#E65100,color:#000,stroke-width:2px
    classDef process fill:#E3F2FD,stroke:#1565C0,color:#000,stroke-width:1px
    classDef wizard fill:#E8F5E9,stroke:#2E7D32,color:#000,stroke-width:1px

    class ADMIN_START startEnd
    class ADMIN_TYPE decision
    class STEP1,STEP2,STEP3,STEP4,STEP5 wizard
```

---

## 4. Real-Time Voice AI Pipeline (Detail)

```mermaid
flowchart LR
    subgraph INPUT["🎤 Caller Audio Input"]
        RAW["Raw Audio<br/>(16kHz PCM)"]
    end

    subgraph PREPROCESS["🔧 Preprocessing"]
        AGC["Automatic<br/>Gain Control"]
        ECHO["Echo<br/>Cancellation"]
        DENOISE["Noise Cancel<br/>(RNNoise)"]
    end

    subgraph DETECTION["🔍 Detection"]
        SILERO["Silero VAD"]
        BARGE["Barge-In<br/>Detector"]
    end

    subgraph TRANSCRIBE["📝 Speech-to-Text"]
        STT_STREAM["Streaming STT<br/>(Google/Sarvam)"]
        PARTIAL["Partial<br/>Results"]
        FINAL["Final<br/>Transcript"]
    end

    subgraph UNDERSTAND["🧠 Understanding & Generation"]
        LANG_GRAPH["LangGraph<br/>Orchestrator"]
        REDIS_CTX["Redis:<br/>Load Context"]
        RAG_Q["RAG: Vector<br/>Search"]
        PROMPT_BUILD["Build<br/>Prompt"]
        LLM_STREAM["LLM Streaming<br/>(Gemini/Sarvam)"]
    end

    subgraph SPEAK["🔊 Text-to-Speech"]
        TTS_STREAM["Streaming TTS<br/>(Google/Sarvam)"]
        AUDIO_OUT["Audio<br/>Output"]
    end

    subgraph PARALLEL_LIVE["📊 Parallel Streams"]
        WS_TRANSCRIPT["WebSocket →<br/>Transcript UI"]
        WS_TRANSLATE["Translation<br/>Engine"]
        WS_SENTIMENT["Sentiment<br/>Scorer"]
    end

    RAW --> AGC --> ECHO --> DENOISE
    DENOISE --> SILERO
    DENOISE --> BARGE
    SILERO -->|"Speech Detected"| STT_STREAM
    BARGE -->|"Interrupt"| TTS_STREAM
    STT_STREAM --> PARTIAL
    STT_STREAM --> FINAL
    FINAL --> LANG_GRAPH
    LANG_GRAPH --> REDIS_CTX
    LANG_GRAPH --> RAG_Q
    REDIS_CTX --> PROMPT_BUILD
    RAG_Q --> PROMPT_BUILD
    PROMPT_BUILD --> LLM_STREAM
    LLM_STREAM --> TTS_STREAM --> AUDIO_OUT

    FINAL --> WS_TRANSCRIPT
    FINAL --> WS_TRANSLATE
    FINAL --> WS_SENTIMENT

    classDef input fill:#FFCDD2,stroke:#C62828,color:#000
    classDef preprocess fill:#E1F5FE,stroke:#0277BD,color:#000
    classDef detect fill:#FFF9C4,stroke:#F9A825,color:#000
    classDef stt fill:#C8E6C9,stroke:#2E7D32,color:#000
    classDef brain fill:#D1C4E9,stroke:#512DA8,color:#000
    classDef tts fill:#FFE0B2,stroke:#E65100,color:#000
    classDef parallel fill:#F3E5F5,stroke:#7B1FA2,color:#000

    class RAW input
    class AGC,ECHO,DENOISE preprocess
    class SILERO,BARGE detect
    class STT_STREAM,PARTIAL,FINAL stt
    class LANG_GRAPH,REDIS_CTX,RAG_Q,PROMPT_BUILD,LLM_STREAM brain
    class TTS_STREAM,AUDIO_OUT tts
    class WS_TRANSCRIPT,WS_TRANSLATE,WS_SENTIMENT parallel
```

---

## Latency Budget

| Stage | Target Latency | Notes |
|-------|---------------|-------|
| Audio Preprocessing + VAD | <50ms | Local processing |
| STT (Streaming) | <150ms | First partial result |
| LangGraph + RAG | <100ms | Redis cache + Qdrant |
| LLM (First Token) | <200ms | Streaming mode |
| TTS (First Chunk) | <100ms | Streaming synthesis |
| **Total First-Byte** | **<500ms** | **End-to-end target** |
