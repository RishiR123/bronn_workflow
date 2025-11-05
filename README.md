# 🚀 API Endpoints Reference — Auth + Org + Subscription + Integrations

Comprehensive endpoint list with I/O schemas and example usage.

---

## 🔐 Authentication

### **POST /auth/register**

**Request:**

```json
{
  "email": "rishi@example.com",
  "password": "test123",
  "full_name": "Rishi Dev"
}
```

**Response:**

```json
{
  "status": "ok",
  "user_id": "uuid"
}
```

---

### **POST /auth/login**

**Request:**

```json
{
  "email": "rishi@example.com",
  "password": "test123"
}
```

**Response:**

```json
{
  "access_token": "<JWT>",
  "token_type": "bearer"
}
```

---

## 🏢 Organizations

### **POST /org/create**

**Request:**

```json
{
  "name": "Quantium",
  "slug": "quantium"
}
```

**Response:**

```json
{
  "status": "ok",
  "organization_id": "uuid",
  "slug": "quantium"
}
```

---

### **GET /org/list**

**Response:**

```json
{
  "organizations": [
    {
      "id": "uuid",
      "name": "Quantium",
      "slug": "quantium",
      "role": "admin",
      "created_at": "2025-11-05T15:54:29.666989"
    }
  ]
}
```

---

### **POST /org/invite**

**Request:**

```json
{
  "org_id": "uuid",
  "email": "teammate@example.com",
  "role": "member"
}
```

**Response:**

```json
{
  "status": "ok",
  "message": "User teammate@example.com added as member"
}
```

---

### **POST /org/invite/generate**

**Request:**

```json
{
  "org_id": "uuid",
  "email": "newjoin@example.com",
  "role": "member"
}
```

**Response:**

```json
{
  "status": "ok",
  "invite_id": "uuid",
  "code": "ABC123XYZ",
  "invite_url": "https://your-auth-domain.com/join/ABC123XYZ",
  "expires_at": "2025-11-12T16:03:37.604614"
}
```

---

## 💳 Subscriptions

### **POST /subscriptions/activate**

**Request:**

```json
{
  "organization_id": "uuid",
  "plan_id": "uuid",
  "provider": "manual"
}
```

**Response:**

```json
{
  "status": "ok",
  "subscription_id": "uuid",
  "active_until": "2025-12-05T16:13:44.700216",
  "provider": "manual"
}
```

---

## 🔗 Integrations

### **POST /integrations/connect**

**Request:**

```json
{
  "organization_id": "uuid",
  "integration_key": "gmail",
  "type": "oauth",
  "credentials": "eyJ0b2tlbiI6ICJ4eXoifQ=="
}
```

**Response:**

```json
{
  "status": "ok",
  "message": "gmail connected successfully"
}
```

---

### **GET /integrations/list**

**Response:**

```json
{
  "integrations": [
    {
      "key": "gmail",
      "type": "oauth",
      "revoked": false,
      "expires_at": "2025-11-05T17:43:51.010926",
      "updated_at": "2025-11-05T16:43:51.013949"
    }
  ]
}
```

---

### **GET /integrations/status/{integration_key}**

**Example:** `/integrations/status/gmail`

**Response:**

```json
{
  "integration": "gmail",
  "revoked": false,
  "expired": false,
  "status": "active"
}
```

---

### **POST /integrations/refresh/{integration_key}**

**Example:** `/integrations/refresh/gmail`

**Response:**

```json
{
  "status": "ok",
  "integration": "gmail",
  "expires_at": "2025-11-05T17:43:51.010926"
}
```

---

### **POST /integrations/revoke/{integration_key}**

**Example:** `/integrations/revoke/gmail`

**Response:**

```json
{
  "status": "ok",
  "message": "gmail revoked"
}
```

---

### **GET /integrations/logs**

**Response:**

```json
{
  "logs": [
    {
      "integration": "gmail",
      "action": "REFRESH_SUCCESS",
      "message": "Token refreshed until 2025-11-05T17:43:51.010926",
      "timestamp": "2025-11-05T16:43:51.013949"
    }
  ]
}
```

---

## ⚙️ Notes

* All endpoints (except `/auth/*`) require a valid JWT Bearer token.
* `credentials` for integrations are base64-encoded JSON.
* The background worker automatically refreshes expiring integrations every 30 minutes.
* Integration logs capture all refresh/revoke actions with timestamps.

---

**Author:** Quantium Engineering Team
**Version:** Beta 1.0 — November 2025
