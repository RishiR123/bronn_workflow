# 🧭 Bronn Unified Authentication & Workflow Architecture Report

_Last updated: November 2025_  
_Environment: Production_

---

## 🔧 Overview

The Bronn ecosystem uses a **dual-server architecture** for security and modularity:

| Layer | Server | Public IP | Purpose |
|--------|---------|-----------|----------|
| **AuthLayer** | `auth.orionac.in` | `13.203.166.158` | Handles authentication, JWT issuance, orgs, and user management |
| **Workflow Orchestrator** | `bronn.orionac.in` | `13.127.57.20` | Hosts visual workflow editor, proxies integrations, and verifies tokens |

Both share a **common `.env` configuration** stored in `/etc/bronn/.env` which keeps cryptographic parameters synchronized.

---

## 🧩 System Architecture

```text
┌──────────────────────────────────────────────────────────┐
│               🌐 Client (React UI / Browser)              │
│  - Domain: https://bronn.orionac.in                      │
│  - Stores:                                                │
│     • localStorage: token                                 │
│     • cookie: access_token (HttpOnly)                     │
│  - Calls: /auth, /workflow, /integrations                 │
└──────────────────────────────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────────────────┐
│             🧩 AuthLayer (13.203.166.158)                 │
│  - FastAPI (Uvicorn)                                      │
│  - Signs JWTs with shared secret                          │
│  - Routes: /auth/login, /auth/register, /auth/logout      │
│  - Sets HttpOnly cookies + returns JSON tokens            │
│  - Env: /etc/bronn/.env                                   │
└──────────────────────────────────────────────────────────┘
             │
     (JWT Signed Token)
             │
             ▼
┌──────────────────────────────────────────────────────────┐
│     ⚙️ Workflow Orchestrator (13.127.57.20)               │
│  - FastAPI + Flask hybrid                                 │
│  - Routes: /workflow, /run_workflow, /integrations/*      │
│  - Verifies JWT using shared secret                       │
│  - Proxies integrations to AuthLayer                      │
│  - Connects to Temporal workers                           │
│  - Serves frontend React build                            │
└──────────────────────────────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────────────────┐
│     ⏳ Temporal Cluster & Workers                         │
│  - Executes multi-step workflows                          │
│  - Uses envelopes with JWT metadata                       │
└──────────────────────────────────────────────────────────┘
