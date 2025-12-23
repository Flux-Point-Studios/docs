# T Backend Developer Guide

<img width="400" height="400" alt="agentT_realistic" src="https://github.com/user-attachments/assets/cef3c0e4-c20d-4ae5-a6e1-7c00ab4fb0a1" />

**Welcome to T Backend!** This guide helps you integrate with the **T API** and understand the core request flows.

### Quick Links

- **Base URL**: `https://api-v2.fluxpointstudios.com`
- **Swagger API Docs**: [API Docs](https://api-v2.fluxpointstudios.com/docs)
- **Support**: `contact@fluxpointstudios.com` · [Discord](https://discord.gg/MfYUMnfrJM)

---

## Authentication

### API Key

All authenticated requests use an `api-key` header:

```bash
curl -s https://api-v2.fluxpointstudios.com/health \
  -H "api-key: YOUR_API_KEY_HERE"
```

If you need an API key, contact `contact@fluxpointstudios.com` or reach out on [Discord](https://discord.gg/MfYUMnfrJM).

### Pay‑Per‑Call (x402)

If you omit `api-key` on protected endpoints, the server may return:

- `402 Payment Required` with an **invoice payload**
- You pay the invoice on-chain
- You retry the same request with:
  - `X-Invoice-Id: <invoiceId>`
  - `X-Payment: <tx_hash_or_signed_cbor>`

**Partner billing (recommended)**: include `X-Partner: <partner_name>` and `X-Wallet-Address: <addr...>` so your pricing/tier rules apply.

---

## Public API Surface

The customer-facing Swagger docs are intentionally scoped to:

- `GET /health`, `GET /api`
- `POST /chat`
- `POST /token-analysis`
- `POST /images/*`
- `GET/POST /payments/*` (when payments are enabled)

If you have additional capabilities enabled for your deployment, you’ll receive separate documentation for those routes.

---

## 1) Chat (`POST /chat`)

Send a message and receive a reply. Use `session_id` to preserve context across turns.

### Basic example

```python
import requests

r = requests.post(
    "https://api-v2.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={"message": "What is Cardano?", "session_id": "my-session-1"},
)
print(r.json()["reply"])
```

### Structured output (optional)

If you want a JSON object back (when the assistant returns JSON), set `output=structured` (query param) or `output_mode="structured"` (body):

```python
import requests

r = requests.post(
    "https://api-v2.fluxpointstudios.com/chat?output=structured",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={
        "message": "Return JSON with keys: summary, risks (array), next_steps (array).",
        "session_id": "my-session-2",
        "output_mode": "structured",
    },
)
print(r.json().get("reply_json"))
```

### Streaming (Server‑Sent Events)

Set `"stream": true` to stream tokens as SSE (client must support SSE):

```python
import requests

with requests.post(
    "https://api-v2.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={"message": "Summarize Cardano in 5 bullets.", "session_id": "sse-1", "stream": True},
    stream=True,
) as resp:
    resp.raise_for_status()
    for line in resp.iter_lines():
        if line:
            print(line.decode("utf-8"))
```

### Image input (optional)

Provide `image_data` as a base64 data URI:

```python
import requests

r = requests.post(
    "https://api-v2.fluxpointstudios.com/chat",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={
        "message": "What’s in this image?",
        "session_id": "vision-1",
        "image_data": "data:image/png;base64,AAAA...",  # your base64
    },
)
print(r.json()["reply"])
```

---

## 2) Token Analysis (`POST /token-analysis`)

Analyze a token by ticker and policy id (Cardano).

```python
import requests

r = requests.post(
    "https://api-v2.fluxpointstudios.com/token-analysis",
    headers={"api-key": "YOUR_API_KEY", "Content-Type": "application/json"},
    json={
        "token": "SNEK",
        "policyId": "279c909f348e533da5808898f87f9a14bb2c3dfbbacccd631d927a3f",
        "network": "cardano",
    },
)
print(r.json())
```

Notes:
- The endpoint may cache results for performance.
- You can bypass cache using `x-bypass-cache: 1`.

---

## 3) Images (`/images/*`)

### Generate (`POST /images/generate`)

```python
import requests

r = requests.post(
    "https://api-v2.fluxpointstudios.com/images/generate",
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
    "https://api-v2.fluxpointstudios.com/images/edit",
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

Use this if you want to upload an image file directly instead of base64. Refer to Swagger docs for the exact multipart form fields.

### Variations (`POST /images/variations`)

This endpoint is **API-key only** (not pay-per-call). Refer to Swagger docs for the request format.

---

## Payments & Access Control (x402)

### What a 402 invoice looks like

When payment is required, you’ll get HTTP 402 with a body like:

```json
{
  "detail": {
    "version": "0.1",
    "invoiceId": "…",
    "chain": "cardano-mainnet",
    "asset": "ADA",
    "amountUnits": "10000000",
    "decimals": 6,
    "payTo": "addr1…",
    "timeout": 300,
    "extra": {
      "lovelace": true,
      "babel": false,
      "partner": "your_partner_name",
      "batchTarget": 100
    }
  }
}
```

Key fields:
- **`payTo`**: destination address
- **`amountUnits` + `decimals`**: what to pay
- **`asset` + `chain`**: network/asset label

### Revenue share / split outputs (optional)

If your partner tier has revenue share enabled, the invoice may include `detail.extra.outputs` / `detail.extra.split`.

In that case, your payment transaction must include those outputs; the verifier enforces them.

### Paying + retrying

After you pay, retry the original request with:

- `X-Invoice-Id: <invoiceId>`
- `X-Payment: <tx_hash_or_signed_cbor>`

Example (retrying `/chat`):

```bash
curl -s https://api-v2.fluxpointstudios.com/chat \
  -H "X-Partner: your_partner_name" \
  -H "X-Wallet-Address: addr1..." \
  -H "X-Invoice-Id: YOUR_INVOICE_ID" \
  -H "X-Payment: YOUR_TX_HASH_OR_SIGNED_CBOR" \
  -H "Content-Type: application/json" \
  -d '{"message":"hello","session_id":"pay-1"}'
```

### Payment helpers

- **Poll invoice status**:

```bash
curl -s https://api-v2.fluxpointstudios.com/payments/status/YOUR_INVOICE_ID
```

- **Check prepaid balance (ADA partners)**:

```bash
curl -s "https://api-v2.fluxpointstudios.com/payments/balance?partner=your_partner_name&wallet=addr1..."
```

---

## Operational Notes

### Request IDs

You may send `X-Request-Id` and the server will echo `X-Request-ID` back on responses. This helps correlate logs and support requests.

### Common status codes

- **401**: missing/invalid `api-key`
- **402**: payment required (x402 invoice returned)
- **403**: forbidden (insufficient privileges)
- **413**: payload too large (commonly for image base64 uploads)
- **422**: invalid request body
- **5xx**: transient server or upstream failures; retry with backoff

---

## Troubleshooting

### 1) “401 Unauthorized”
- Ensure `api-key` header is present and correct.
- Confirm the key is active (contact support if unsure).

### 2) “402 Payment Required”
- Read the `invoiceId`, `asset`, `amountUnits`, `decimals`, and `payTo`.
- Pay on-chain.
- Retry with `X-Invoice-Id` + `X-Payment`.

### 3) “wallet_address_required_for_prepaid_batches”
- Some partner billing modes require `X-Wallet-Address` so prepaid balance can be tracked.

### Debug tip

```python
import requests

resp = requests.post(url, headers=headers, json=payload)
print(resp.status_code, resp.text)
```

---

## Additional Resources

- **Swagger docs**: [API Docs](https://api-v2.fluxpointstudios.com/docs)

---

## You’re Ready

Quick checklist:
- [ ] Get an API key (or partner config for x402)
- [ ] `GET /health`
- [ ] `POST /chat`
- [ ] `POST /token-analysis`
- [ ] `POST /images/generate`
- [ ] If using x402: run a full 402 → pay → retry loop
* [ ] Generate an image
* [ ] Explore the knowledge graphs
