# 🧾️ Bronn Orchestrator — OAuth + Workflow Integration System (Developer Handover Report)

### 🕝 Updated: November 11, 2025

### 👤 Author: Rishi

### 🤠 Reviewer: ChatGPT (GPT-5)

### 🔧 Environment: Ubuntu (FastAPI + Flask hybrid + Temporal)

---

## 1. Overview

The **Bronn Orchestrator** is a workflow automation backend similar to Zapier or Make.com, designed to integrate 3rd-party apps (like Gmail, Notion, Slack, etc.) through a unified visual builder (`/workflow` React UI).

The system has two major parts:

| Component                  | Role                                                                                                             |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Orchestrator (main.py)** | Exposes `/workflow` React app + API proxy for integrations, and connects to Temporal for workflow orchestration. |
| **Auth API (FastAPI)**     | Manages authentication, JWTs, OAuth integrations, and stores user tokens securely in PostgreSQL.                 |

---

## 2. Current System Status

✅ **OAuth Flow (Google Gmail)** — Fully functional
✅ **JWT Authentication** — Stable (no guest fallbacks for logged users)
✅ **PostgreSQL Schema** — Healthy with correct foreign keys
✅ **Integration Refresher Thread** — Running (no due refreshes yet)
✅ **Frontend** — `/workflow` React app builds & authenticates correctly
⚠️ **Next Phase Needed:** Implement automatic token refresh and Gmail API workers.

---

## 3. Database Structure (AuthDB)

| Table                 | Purpose                                                                            |
| --------------------- | ---------------------------------------------------------------------------------- |
| **users**             | Stores registered user accounts (UUID primary key).                                |
| **oauth_state**       | Temporary table for pending OAuth handshakes (`state_key`, `user_id`, `provider`). |
| **user_integrations** | Persistent store for OAuth credentials and metadata.                               |

### Verified User IDs

| user_id                                | email                                                         |
| -------------------------------------- | ------------------------------------------------------------- |
| `f5c8a8d9-7ea3-4d68-80c3-9245b47922d1` | [rishiininternet@gmail.com](mailto:rishiininternet@gmail.com) |
| `cb90e0b6-e572-4b5a-9366-23298d22de10` | [rishi@orionac.in](mailto:rishi@orionac.in)                   |

---

## 4. OAuth Implementation (Gmail)

### Endpoints

| Endpoint                               | Description                                                                     |
| -------------------------------------- | ------------------------------------------------------------------------------- |
| `GET /integrations/connect/{provider}` | Generates Google OAuth URL, stores state, and redirects to Google.              |
| `GET /integrations/callback/google`    | Handles Google’s redirect, exchanges code → token, stores credentials.          |
| `GET /integrations/status/{provider}`  | Returns integration status: `active`, `expired`, `revoked`, or `not_connected`. |

### Flow Summary

1. React UI triggers `/integrations/connect/gmail`
2. API validates JWT → gets current user
3. API inserts `state_key` into `oauth_state`
4. Redirects to Google consent screen
5. User approves
6. Google redirects → `/integrations/callback/google`
7. Server exchanges `code` for tokens
8. Saves credentials → `user_integrations`
9. Logs event → `log_integration_event`
10. Returns success HTML page

---

## 5. Fixes & Improvements Done

### Fix 1: Foreign Key Violation

**Problem:** `insert or update on table "oauth_state" violates foreign key constraint`
**Fix:** Updated `serve_react()` in `main.py` to issue guest tokens only for local dev; real JWT is used for logged users.

### Fix 2: Redirect URI Mismatch

**Problem:** Google error `redirect_uri_mismatch`.
**Fix:** Added `https://api.bronn.orionac.in/integrations/callback/google` to Google Cloud OAuth Client settings.

### Fix 3: JSON Encode Error

**Problem:** `Json.dumps() missing 1 required positional argument: 'obj'`.
**Fix:** Replaced `Json(token_data).dumps()` with `json.dumps(token_data)` in `app/routes/integration_routes.py`.

### Fix 4: React Hook Errors

**Problem:** Conditional hook calls blocked production build.
**Fix:** Refactored `WorkflowBuilder.js` to ensure hooks are called unconditionally.

---

## 6. Token Storage Structure

Stored in `user_integrations.credentials` (base64-encoded JSON):

```json
{
  "access_token": "ya29.a0AfH6SMA...",
  "refresh_token": "1//04WZx...",
  "scope": "https://mail.google.com/",
  "token_type": "Bearer",
  "expires_in": 3599
}
```

**Decode in PostgreSQL:**

```sql
SELECT convert_from(decode(credentials, 'base64'), 'UTF8')
FROM user_integrations
WHERE integration_key = 'gmail';
```

---

## 7. Background Refresher Thread

Logs:

```
🚀 Integration auto-refresher started...
🕒 [Refresher] No integrations due for refresh.
⏳ Sleeping for 30.0 minutes...
```

✅ Thread runs correctly.
⚙️ Next: Add refresh logic using stored `refresh_token` to renew Gmail access tokens.

---

## 8. Code Path Summary

| File                               | Description                                                                          |
| ---------------------------------- | ------------------------------------------------------------------------------------ |
| `main.py`                          | Hybrid FastAPI + Flask app, hosts React, proxies integrations, connects to Temporal. |
| `app/routes/integration_routes.py` | Handles OAuth connect, callback, token save, and integration status.                 |
| `app/utils/deps.py`                | Validates JWTs from headers or query params.                                         |
| `app/utils/oauth_clients.py`       | Provides OAuth client factory for scalable multi-provider use.                       |

---

## 9. Next Developer Actions

| Priority | Task                   | Description                                                            |
| -------- | ---------------------- | ---------------------------------------------------------------------- |
| 🟩       | Add Token Auto-Refresh | Use `refresh_token` to renew Gmail access tokens periodically.         |
| 🟩       | Add Gmail Worker       | Create Temporal worker to send/read emails using Gmail API.            |
| 🟨       | UI Integration Status  | Display `Connected`/`Expired` status via `/integrations/status/gmail`. |
| 🟨       | Add More Providers     | Extend OAuth client factory for Notion, Slack, Drive, etc.             |
| 🟨       | Security Hardening     | Enforce HTTPS + stricter cookie policies.                              |
| 🟥       | Production Sync        | Align `.env` across orchestrator & auth servers.                       |

---

## 10. Final Status Snapshot

| Component             | Status                                               |
| --------------------- | ---------------------------------------------------- |
| JWT Auth Flow         | ✅ Stable                                             |
| OAuth2 Gmail          | ✅ Working end-to-end                                 |
| Token Persistence     | ✅ Verified                                           |
| Refresher Thread      | ✅ Running                                            |
| Frontend UI           | ✅ Functional                                         |
| Temporal Orchestrator | ✅ Connected                                          |
| Next Phase            | 🔁 Auto-refresh Gmail tokens + add workflow triggers |

---

## 11. Quick Verification Checklist

| Step                 | Command                                          | Expected                                      |
| -------------------- | ------------------------------------------------ | --------------------------------------------- |
| Check integrations   | `SELECT integration_key FROM user_integrations;` | gmail                                         |
| Check token expiry   | `SELECT expires_at FROM user_integrations;`      | Future timestamp                              |
| Check state table    | `SELECT COUNT(*) FROM oauth_state;`              | 0                                             |
| Check refresher logs | Console                                          | `[Refresher] No integrations due for refresh` |

---

## 12. Summary

The Bronn Orchestrator is now at a **production-ready OAuth integration baseline**:

* Clean JWT propagation from orchestrator → auth service.
* Fully functional Google OAuth2 flow.
* Secure token persistence with PostgreSQL.
* Verified background refresher operation.
* Ready for next-phase Gmail and Notion integrations.

---
