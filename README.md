# Multi-Modal-Research-Assistant

### Enterprise-Grade AI Knowledge System · n8n · GPT-4o Vision · Pinecone · PostgreSQL · Tavily

---

## 📌 Project Overview

This is an **advanced, production-ready AI Research Assistant** that processes **any file type** — PDFs, images, spreadsheets, Word docs — extracts knowledge using GPT-4o Vision, stores it in a high-dimensional vector database, and powers a multi-tool AI agent with real-time web search, chart generation, and full session memory.

**What makes this different from a basic RAG chatbot:**

| Feature | Basic RAG | This Project |
|---|---|---|
| File types | PDF only | PDF, DOCX, XLSX, PNG, JPEG, MD |
| Embedding dims | 1536 | **3072** (text-embedding-3-large) |
| Image handling | ❌ | ✅ GPT-4o Vision |
| Web search | ❌ | ✅ Tavily API (real-time) |
| Chart generation | ❌ | ✅ Dynamic chart tool |
| Session persistence | In-memory | ✅ PostgreSQL |
| Analytics | None | ✅ Hourly monitoring + Telegram |
| Ingestion | Manual | ✅ Auto every 30 min |
| Namespaces | Single | ✅ documents + images |

---

## 🚀 Core Capabilities

- 🖼️ **Multi-modal ingestion** — GPT-4o Vision reads charts, figures, diagrams inside images
- 🔢 **3072-dimension embeddings** — 2× the precision of standard 1536-dim models
- 🔍 **Semantic search** — top-8 retrieval with metadata filtering
- 🌐 **Real-time web search** — Tavily integration for knowledge beyond uploaded docs
- 📊 **Dynamic chart generation** — agent can create charts from data on demand
- 🧮 **Built-in calculator** — performs statistical computations inline
- 🧠 **Windowed conversation memory** — 20-message context window per session
- 📱 **Telegram + Gmail alerts** — processing complete, hourly health reports
- 🗄️ **Full PostgreSQL backend** — document registry, chat sessions, analytics logs
- 📈 **Live analytics dashboard** — beautiful dark-mode UI with real-time metrics

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    WORKFLOW 1: INGESTION PIPELINE                │
│                                                                  │
│  Schedule (30min) ──► Google Drive ──► Filter File Types        │
│                           │                                      │
│                    ┌──────┴──────┐                               │
│                    ▼            ▼                                │
│              Text Extract   GPT-4o Vision                        │
│                    └──────┬──────┘                               │
│                           ▼                                      │
│              Semantic Chunker (1200 chars, 150 overlap)          │
│                           │                                      │
│              OpenAI Embed (text-embedding-3-large, 3072-dim)     │
│                           │                                      │
│                ┌──────────┴──────────┐                           │
│                ▼                     ▼                           │
│          Pinecone Upsert       PostgreSQL Registry               │
│          (namespaced)          (document metadata)               │
│                │                                                 │
│         Telegram + Gmail notifications                           │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                   WORKFLOW 2: AI AGENT CHAT ENGINE               │
│                                                                  │
│  POST /research-chat ──► Parse Request ──► Load PG Context       │
│                                │                                 │
│                         GPT-4o Agent                             │
│                        (Tools available)                         │
│               ┌──────────┬──────────┬──────────┐                │
│               ▼          ▼          ▼          ▼                │
│        Pinecone     Tavily Web    Chart     Calculator           │
│        Semantic     Search       Generator                       │
│        (top-8)      (real-time)                                  │
│               └──────────┴──────────┴──────────┘                │
│                                │                                 │
│                  Format Response + Citations                      │
│                          │                                       │
│              PostgreSQL: Save session + messages                  │
│                          │                                       │
│                    Return JSON Response                           │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│              WORKFLOW 3: ANALYTICS & HEALTH MONITOR              │
│                                                                  │
│  Hourly Trigger ──► Query PostgreSQL Stats                       │
│                          │                                       │
│              Pinecone Index Health Check                         │
│                          │                                       │
│               Compile Report + Log Analytics                     │
│                          │                                       │
│              Telegram Notification                               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| **n8n** | 1.70+ | Workflow automation engine |
| **OpenAI GPT-4o** | Latest | Chat + Vision processing |
| **OpenAI Embeddings** | text-embedding-3-large | 3072-dim vector generation |
| **Pinecone** | Serverless | Vector database (namespaced) |
| **PostgreSQL** | 15+ | Sessions, registry, analytics |
| **Google Drive** | API v3 | Document source storage |
| **Tavily** | API v1 | Real-time web search |
| **Telegram Bot** | API | Notifications & alerts |
| **Gmail** | OAuth2 | Email reports |

---

## 📂 Project Structure

```
project/
│
├── workflows/
│   ├── 01_document_ingestion_pipeline.json     ← Import to n8n first
│   ├── 02_ai_agent_chat_engine.json            ← Import to n8n second
│   └── 03_analytics_monitor.json              ← Import to n8n third
│
├── dashboard/
│   └── index.html                              ← Live monitoring dashboard
│
├── data/
│   ├── database_schema.sql                     ← Run this in PostgreSQL first
│   └── sample_docs/                            ← Drop your research files here
│
└── README.md
```

---

## ⚙️ Setup Instructions

### Step 1: PostgreSQL Database

```sql
-- Create database
CREATE DATABASE research_assistant;

-- Connect and run schema
\c research_assistant
\i data/database_schema.sql
```

### Step 2: Pinecone Index

1. Go to [pinecone.io](https://pinecone.io) → Create Index
2. Index name: `research-assistant-v2`
3. Dimensions: **3072**
4. Metric: **cosine**
5. Create namespaces: `documents`, `images`

### Step 3: API Keys Required

```env
OPENAI_API_KEY=sk-...              # OpenAI (GPT-4o + Embeddings)
PINECONE_API_KEY=...               # Pinecone
TAVILY_API_KEY=tvly-...            # Tavily Web Search
TELEGRAM_BOT_TOKEN=...             # Telegram Bot (via @BotFather)
TELEGRAM_CHAT_ID=...               # Your chat ID
GOOGLE_DRIVE_FOLDER_ID=...         # Target Google Drive folder
```

### Step 4: n8n Import

1. Open n8n → Workflows → Import from File
2. Import in order:
   - `01_document_ingestion_pipeline.json`
   - `02_ai_agent_chat_engine.json`
   - `03_analytics_monitor.json`
3. Configure credentials for each workflow node
4. Activate all three workflows

### Step 5: Upload Documents

Drop any of these file types into your Google Drive folder:
- `.pdf` — Research papers, reports
- `.docx` — Word documents, notes
- `.xlsx` — Spreadsheets with data
- `.md` — Markdown notes
- `.png` / `.jpeg` — Diagrams, charts, figures

The system auto-processes new files every 30 minutes.

---

## 💬 Chat API

### Request

```bash
curl -X POST https://your-n8n.com/webhook/research-chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Compare sparse and dense retrieval in RAG systems",
    "sessionId": "user123_session_1",
    "userId": "user123",
    "mode": "research"
  }'
```

### Response

```json
{
  "requestId": "req_abc123xyz",
  "sessionId": "user123_session_1",
  "query": "Compare sparse and dense retrieval in RAG systems",
  "response": "Based on your research documents... [full answer with citations]",
  "sources": [
    {
      "fileName": "rag_survey_paper.pdf",
      "chunkIndex": 12,
      "webViewLink": "https://drive.google.com/...",
      "relevanceScore": 0.94
    }
  ],
  "mode": "research",
  "timestamp": "2025-05-10T12:34:56.789Z",
  "processingTimeMs": 1420,
  "metadata": {
    "model": "gpt-4o",
    "embeddingModel": "text-embedding-3-large",
    "vectorDimensions": 3072,
    "chunksRetrieved": 8
  }
}
```

### Query Modes

| Mode | Description | Best For |
|---|---|---|
| `research` | Deep Q&A with citations | Technical questions |
| `summarize` | Structured summary | Papers, reports |
| `compare` | Side-by-side analysis | Multiple documents |
| `extract` | Pull specific data points | Stats, quotes, figures |

---

## 📊 Dashboard

Open `dashboard/index.html` in any browser for a live view of:

- Real-time metrics: documents, vectors, sessions, response time
- Query volume & user analytics charts
- Mode distribution (research/summarize/compare/extract)
- Recent processing activity table
- Interactive AI chat interface (demo mode)
- Knowledge base document browser
- Full system architecture view
- Live system status indicators

---

## 🔒 Production Recommendations

1. **Rate limiting** — Add n8n rate limit node before Pinecone queries
2. **Auth** — Add API key validation in the webhook node
3. **Error workflow** — Create dedicated error handling workflow
4. **Backup** — Schedule daily PostgreSQL dump to Google Drive
5. **Cost control** — Set OpenAI usage limits in your dashboard
6. **Monitoring** — Connect to Grafana via PostgreSQL data source

---

## 📈 Key Differences vs Basic RAG

This system was inspired by a basic Python notes RAG chatbot and evolved into a production-grade research platform:

**From** a single file type, fixed knowledge base, in-memory chat, manual uploads, no analytics

**To** multi-modal ingestion, real-time web augmentation, persistent sessions, auto-scheduled processing, full analytics & monitoring, dynamic tool use, 2× embedding precision

---

*Built with n8n · OpenAI · Pinecone · PostgreSQL · Tavily*
