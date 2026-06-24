# T Backend Developer Guide

<img width="400" height="400" alt="agentT_realistic" src="https://github.com/user-attachments/assets/cef3c0e4-c20d-4ae5-a6e1-7c00ab4fb0a1" />

**Welcome to T Backend!** This guide helps you integrate with the **T API** and understand the core request flows.

### Quick Links

- **Base URL**: `https://api-v3.fluxpointstudios.com`
- **Swagger API Docs**: [API Docs](https://api-v3.fluxpointstudios.com/docs)
- **Support**: `contact@fluxpointstudios.com` · [Discord](https://discord.gg/MfYUMnfrJM)

---

## Authentication

### Service API key (required)

Every request to `POST /chat` and `POST /token-analysis` **requires** a service `api-key` header. The key authenticates you as a trusted, registered caller:

```bash
curl -s https://api-v3.fluxpointstudios.com/chat \
  -H "api-key: YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{"message":"hello"}'
```

A missing, invalid, inactive, or expired key returns **`401 Unauthorized`**.

If you need an API key, contact `contact@fluxpointstudios.com` or reach out on [Discord](https://discord.gg/MfYUMnfrJM).

### Optional wallet session (metered agent mode)

`POST /chat` accepts an **optional** `Authorization: Bearer <wallet-session>` header on top of the api-key:

- **No bearer** → **free conversational mode** (the default; the SaturnSwap web-app path). No ledger debit/refund.
- **Valid bearer** → **metered agent mode** (the autonomous-agent runtime). The reply includes `balance_after` from the server ledger.
- A **malformed / expired / revoked** bearer fails closed with **`401`** — only the *absence* of a bearer downgrades to free mode.

> The api-key is always required in both modes. The bearer only adds *spending identity*; it never replaces the api-key.

### Browser / extension callers

CORS is `allow_origins: *`, which works for **header-based** auth (it does not cover cookies). From a browser or extension:

- Send `credentials: "omit"` (do not rely on cookies).
- Send a **normal User-Agent** — the Cloudflare WAF may return `403` for unusual or empty UAs.

---

## Public API Surface

The customer-facing Swagger docs are scoped to:

- `GET /health`, `GET /api`
- `POST /chat`
- `POST /token-analysis`
- `POST /images/*`

If you have additional capabilities enabled for your deployment, you'll receive separate documentation for those routes.

---

## 1) Chat (`POST /chat`)

Send a message and receive a reply. **`/chat` is authenticated with the `api-key` header only — it is not a payment endpoint** (no x402 invoices on this route).

### Request schema

| Field | Type | Required | Notes |
|---|---|---|---|
| `message` | string | **yes** | 1–32000 chars. |
| `session_id` | string | no | **Logging / correlation only.** It does **not** carry conversation memory — the server keeps no per-session history. (See "Conversation context" below.) |
| `image_data` | string | no | Base64 data URI for multimodal input (e.g. `data:image/png;base64,...`). |
| `stream` | bool | no | Default `false`. `true` → Server-Sent Events (see Streaming). |
| `system` | string | no | System prompt override. |
| `persona` | string | no | Persona selector. |
| `profile` | string | no | Profile selector. |
| `temperature` | float | no | `0`–`2`. |
| `top_p` | float | no | `0`–`1`. |
| `max_tokens` | int | no | `1`–`262144`. Default `2048`. |
| `meta` | object | no | Free-form metadata. |
| `context` | object | no | Public wallet snapshot, treated as untrusted LLM context. The server never reads client-sent balances for entitlement. |
| `request_id` | string | no | Per-turn idempotency key (UUID v4), stable across retries. Keys debit/refund in metered mode. |

> There is **no `model` field** (the server selects the model — Kimi by default) and **no `history` / `messages` array**. Each call is a single turn.

### Response schema

```json
{
  "reply": "…",            // always set (never empty, even on provider failure)
  "used_tools": { },        // tool-call summary, or null
  "reply_json": null,       // always null — see note below
  "provider": "kimi",
  "model": "…",
  "reasoning": "…",        // Kimi thinking content, or null
  "balance_after": 0        // metered (bearer) mode only; null in free mode
}
```

> **Note:** `reply_json` is always `null`. There is no structured-output / JSON-mode feature — parse `reply` (a string) yourself if you need structured data.

### Basic example

```python
import requests

r = requests.post(
    "https://api-v3.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={"message": "What is Cardano?"},
)
print(r.json()["reply"])
```

### Conversation context

`session_id` is used only to correlate logs/telemetry — it does **not** make the server remember previous turns. If you need multi-turn context, include the relevant prior content in `message` (or `system`) yourself.

### Streaming (Server-Sent Events)

Set `"stream": true` to receive `text/event-stream` output. Each frame is `data: {json}\n\n` where the JSON has a `type`:

- `content`  — `{"type":"content","text":"…"}` (answer tokens)
- `reasoning` — `{"type":"reasoning","text":"…"}` (Kimi thinking tokens)
- `tool_call` — `{"type":"tool_call","name":"…","args":{…}}`
- `tool_result` — `{"type":"tool_result","name":"…","ok":true|false}`

The stream terminates with `data: [DONE]\n\n`.

```python
import requests

with requests.post(
    "https://api-v3.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={"message": "Summarize Cardano in 5 bullets.", "stream": True},
    stream=True,
) as resp:
    resp.raise_for_status()
    for line in resp.iter_lines():
        if line:
            print(line.decode("utf-8"))  # e.g. b'data: {"type":"content","text":"…"}'
```

### Image input (optional)

Provide `image_data` as a base64 data URI:

```python
import requests

r = requests.post(
    "https://api-v3.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={
        "message": "What's in this image?",
        "image_data": "data:image/png;base64,AAAA...",  # your base64
    },
)
print(r.json()["reply"])
```

### Errors

- **`401`** — missing/invalid `api-key` (or a malformed `Authorization: Bearer`).
- **`422`** — request body fails validation (e.g. `message` too long, `temperature` out of range).
- Otherwise the endpoint degrades to `200` with a descriptive `reply` rather than returning a 5xx for chat-logic failures.

---

## 2) Token Analysis (`POST /token-analysis`)

Analyze a token. Requires the `api-key` header (same as `/chat`). The request accepts **both snake_case and camelCase** field names.

### Request schema

| Field (camel / snake) | Type | Notes |
|---|---|---|
| `token` / `tokenName` | string | Token symbol or name (e.g. `SNEK`). Required (unless it is a known internal ticker). |
| `policyId` / `policy_id` | string | The Cardano policy id (56-hex). Required for non-internal tokens. |
| `assetNameHex` / `asset_name_hex` | string | Cardano asset name in hex. **Required for V2 Cardano** (omitting it returns `400`; pass an empty string `""` if the asset name is empty). |
| `network` | string | Default `cardano`. Allowed: `cardano`, `ethereum`, `base`, `solana`, `polygon`, `avalanche`. |
| `api_version` / `apiVersion` (body) **or** `X-API-Version` (header) | string | `"1"` = LLM analysis (default). `"2"` = deterministic risk engine. Header wins over body. |

> `network` defaults to `cardano`. The version can be set in the body (`api_version`/`apiVersion`) or via the `X-API-Version` header; the header takes precedence, and the default is `"1"`.

### Example (V2, Cardano)

```python
import requests

r = requests.post(
    "https://api-v3.fluxpointstudios.com/token-analysis",
    headers={
        "api-key": "YOUR_API_KEY",
        "Content-Type": "application/json",
        "X-API-Version": "2",
    },
    json={
        "token": "SNEK",
        "policyId": "279c909f348e533da5808898f87f9a14bb2c3dfbbacccd631d927a3f",
        "assetNameHex": "534e454b",   # REQUIRED for V2 Cardano ("" if the asset name is empty)
        "network": "cardano",
    },
)
print(r.json())
```

### Response shapes

The response is a JSON object whose shape depends on `api_version`.

**V2 (`"2"`)** — a deterministic `AnalysisReport`:

```json
{
  "token_name": "SNEK",
  "network": "cardano",
  "api_version": "2",
  "risk_engine": { "overall_score": 0.42, "risk_level": "…", "subscores": { }, "confidence": 0.8, "…": "…" },
  "market":      { "price_usd": 0.0, "market_cap_usd": 0.0, "volume_24h_usd": 0.0, "…": "…" },
  "liquidity":   { "pool_depth_usd": 0.0, "pool_count": 0, "spread_bps": 0 },
  "holders":     { "holder_count": 0, "top_10_holder_pct": 0.0 },
  "security":    { "contract_verified": false, "registry_present": false, "…": "…" },
  "data_quality":{ "coverage": 0.0, "sources_used": [], "sources_failed": [], "…": "…" },
  "_meta":       { "request_id": "…", "from_cache": false, "…": "…" }
}
```

(Null fields are omitted. The response is also augmented with a `_meta` block — request id, cache flag, scoring details.)

**V1 (`"1"`, default)** — LLM analysis:

```json
{
  "token": "SNEK",
  "network": "cardano",
  "analysis": "…",                  // LLM-generated text
  "riskLevel": "…",
  "marketData": { "price": 0.0, "marketCap": 0.0, "volume24h": 0.0, "priceChange7d": 0.0, "priceChange30d": 0.0 },
  "dataQuality": { "coverage": 0.0, "sourcesUsed": [] }
}
```

### Notes & errors

- Results may be cached for performance. Bypass with `x-bypass-cache: 1`.
- **`400`** — missing `token`/`network`, a non-alphanumeric `policyId`/`token`, an unknown `network`, or (V2 Cardano) a missing `assetNameHex`.
- **`401`** — missing/invalid `api-key`.
- **`422`** — body fails type parsing.

---

## 3) Images (`/images/*`)

### Generate (`POST /images/generate`)

```python
import requests

r = requests.post(
    "https://api-v3.fluxpointstudios.com/images/generate",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={"prompt": "A futuristic Cardano validator node in space", "size": "1024x1024", "n": 1},
)
data = r.json()
if data.get("success"):
    print("images:", len(data.get("images") or []))
else:
    print("error:", data)
```

### Edit (`POST /images/edit`)

- `input_images` is a list of base64 images
- Each image is validated to **<= 25MB**

```python
import requests

r = requests.post(
    "https://api-v3.fluxpointstudios.com/images/edit",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={
        "prompt": "Add plants and warm natural lighting",
        "input_images": ["data:image/png;base64,AAAA..."],
        "n": 1,
        "size": "1024x1024",
    },
)
print(r.json())
```

### Upload + edit (`POST /images/upload-and-edit`)

Use this if you want to upload an image file directly instead of base64. Refer to the Swagger docs for the exact multipart form fields.

### Variations (`POST /images/variations`)

API-key only. Refer to the Swagger docs for the request format.

---

## Operational Notes

### Request IDs

You may send `X-Request-Id` and the server will echo `X-Request-ID` back on responses. This helps correlate logs and support requests.

### Common status codes

- **401**: missing/invalid `api-key` (or a malformed `Authorization: Bearer`).
- **400**: invalid request parameters (e.g. unknown network, missing `assetNameHex` for V2 Cardano).
- **413**: payload too large (commonly for image base64 uploads).
- **422**: request body fails validation.
- **5xx**: transient server or upstream failures; retry with backoff. (Note: `/chat` degrades to a `200` with a descriptive `reply` instead of returning 5xx for chat-logic failures.)

---

## Troubleshooting

### 1) "401 Unauthorized"
- Ensure the `api-key` header is present and correct.
- Confirm the key is active and not expired (contact support if unsure).
- If you send `Authorization: Bearer …`, make sure the wallet session is valid — a malformed/expired bearer also 401s.

### 2) "403 Forbidden" (browser / extension)
- The Cloudflare WAF can block unusual or empty User-Agent strings. Send a normal UA.
- Use `credentials: "omit"` — CORS is `allow_origins: *` for header auth, not cookies.

### 3) "400 Bad Request" on `/token-analysis`
- For V2 Cardano you must include `assetNameHex` (use `""` if the asset name is empty).
- Check `policyId` and `token` are present and alphanumeric, and `network` is one of the supported values.

### Debug tip

```python
import requests

resp = requests.post(url, headers=headers, json=payload)
print(resp.status_code, resp.text)
```

---

## Additional Resources

- **Swagger docs**: [API Docs](https://api-v3.fluxpointstudios.com/docs)

---

## You're Ready

Quick checklist:
- [ ] Get an API key (`contact@fluxpointstudios.com` or Discord)
- [ ] `GET /health`
- [ ] `POST /chat` (with `api-key`)
- [ ] `POST /token-analysis` (include `assetNameHex` for V2 Cardano)
- [ ] `POST /images/generate`
