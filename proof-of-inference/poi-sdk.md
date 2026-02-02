---
description: Developer SDK for Proof-of-Inference anchoring and verification
---

# Orynq SDK

The Orynq SDK provides tools for anchoring AI process traces to the Cardano blockchain and verifying those anchors. It supports both direct integration and a managed Anchor-as-a-Service API.

**Orynq** is the umbrella platform for AI verification and attestation services, with **Proof-of-Inference anchoring** as its core capability.

## Features

- **Process Trace Anchoring** - Commit cryptographic proofs of AI execution to Cardano
- **Dual Protocol Support** - x402 (Coinbase standard) for EVM and Flux protocol for Cardano
- **Multi-Chain Payments** - Pay for anchoring in ADA, $AGENT, or EVM stablecoins
- **Auto-Pay Client** - Automatic 402 payment handling with budget controls
- **Verification Tools** - Independently verify any anchor using only the txHash
- **TypeScript & Python** - Full SDK support for both languages

## Installation

### TypeScript/JavaScript

```bash
# Core package
npm install @fluxpointstudios/orynq-sdk-core

# Client with auto-pay
npm install @fluxpointstudios/orynq-sdk-client

# Cardano payer (browser wallets)
npm install @fluxpointstudios/orynq-sdk-payer-cardano-cip30

# Cardano payer (server-side)
npm install @fluxpointstudios/orynq-sdk-payer-cardano-node
```

### Python

```bash
pip install orynq-sdk
```

---

## Quick Start

### Anchoring a Process Trace

```typescript
import { OrynqClient } from '@fluxpointstudios/orynq-sdk-client';
import { createCip30Payer } from '@fluxpointstudios/orynq-sdk-payer-cardano-cip30';

// Connect to a CIP-30 wallet (Nami, Eternl, etc.)
const payer = await createCip30Payer(window.cardano.nami);

// Create client with auto-pay enabled
const client = new OrynqClient({
  payer,
  autoPay: true,
  budget: {
    maxPerRequest: '5000000',  // 5 ADA max per anchor
    maxPerDay: '50000000',     // 50 ADA daily limit
  },
});

// Create a process trace manifest
const manifest = {
  formatVersion: '1.0',
  agentId: 'my-ai-agent-v1',
  rootHash: 'sha256:abc123...',
  manifestHash: 'sha256:def456...',
  merkleRoot: 'sha256:789abc...',
  totalEvents: 47,
  totalSpans: 12,
  createdAt: new Date().toISOString(),
  metadata: {
    model: 'gpt-4',
    sessionId: 'session-001',
  },
};

// Anchor to Cardano (payment handled automatically)
const response = await client.fetch(
  'https://api-v3.fluxpointstudios.com/anchors/process-trace',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ manifest }),
  }
);

const result = await response.json();
console.log('Anchored:', result.txHash);
// View at: https://cardanoscan.io/transaction/{txHash}
```

### Python Example

```python
from orynq_sdk import OrynqClient, BudgetConfig

client = OrynqClient(
    payer=my_payer,
    auto_pay=True,
    budget=BudgetConfig(
        max_per_request="5000000",
        max_per_day="50000000",
    ),
)

manifest = {
    "formatVersion": "1.0",
    "agentId": "my-ai-agent-v1",
    "rootHash": "sha256:abc123...",
    "manifestHash": "sha256:def456...",
    # ... other fields
}

response = await client.fetch(
    "https://api-v3.fluxpointstudios.com/anchors/process-trace",
    method="POST",
    json={"manifest": manifest},
)

result = response.json()
print(f"Anchored: {result['txHash']}")
```

---

## Anchor-as-a-Service API

For applications that don't want to manage wallet infrastructure, use the managed Anchor-as-a-Service API.

### Endpoint

```
POST https://api-v3.fluxpointstudios.com/anchors/process-trace
```

### Request

```json
{
  "manifest": {
    "formatVersion": "1.0",
    "agentId": "your-agent-id",
    "rootHash": "sha256:...",
    "manifestHash": "sha256:...",
    "merkleRoot": "sha256:...",
    "totalEvents": 47,
    "totalSpans": 12,
    "createdAt": "2026-01-30T12:00:00Z",
    "metadata": {
      "model": "gpt-4",
      "sessionId": "..."
    }
  }
}
```

### Response (402 Payment Required)

The first request returns a 402 with payment details:

```json
{
  "x402": {
    "version": "1",
    "paymentRequired": true,
    "accepts": [{
      "scheme": "exact",
      "network": "cardano-mainnet",
      "maxAmountRequired": "2500000",
      "payTo": "addr1..."
    }]
  },
  "invoiceId": "inv_abc123"
}
```

### Response (200 Success)

After payment verification:

```json
{
  "success": true,
  "txHash": "abc123def456...",
  "requestId": "req_xyz789",
  "network": "cardano-mainnet",
  "explorerUrl": "https://cardanoscan.io/transaction/abc123def456..."
}
```

### Pricing

#### Pay-Per-Use (Self-Service)

Use your own Cardano wallet with the x402 payment protocol:

- **~2-3 ADA per anchor** (at current network fees)
- No monthly fees or minimums
- Pay only when you anchor

#### Managed Anchoring

No crypto wallet needed • API key authentication • Pay by card

| Tier | Monthly | Included Anchors | Overage |
|------|---------|------------------|---------|
| **Starter** | $49/mo | 500 anchors | $0.15/anchor |
| **Growth** | $199/mo | 2,500 anchors | $0.10/anchor |
| **Scale** | Custom | Unlimited | Volume pricing |

**Subscribe now:**
- [Starter Plan ($49/mo)](https://buy.stripe.com/test_6oU9AT1cD7cE74d6gxfrW01)
- [Growth Plan ($199/mo)](https://buy.stripe.com/test_4gMeVd8F57cE3S134lfrW02)

**All plans include:**
- No wallet management required
- API key authentication (X-Partner header)
- Usage dashboard at [fluxpointstudios.com/orynq/enterprise-dashboard](https://fluxpointstudios.com/orynq/enterprise-dashboard)
- Pay via invoice (USD)
- Dedicated support
- SLA available

Contact [sales@fluxpointstudios.com](mailto:sales@fluxpointstudios.com) for custom Scale pricing.

---

## Verification

### Verify via API

```bash
curl "https://api-v3.fluxpointstudios.com/anchors/verify?txHash=abc123..."
```

Response:

```json
{
  "verified": true,
  "network": "cardano-mainnet",
  "txHash": "abc123...",
  "onChainData": {
    "schema": "poi-anchor-v1",
    "type": "process-trace",
    "manifestHash": "sha256:def456...",
    "rootHash": "sha256:abc123...",
    "timestamp": "2026-01-30T12:00:00Z"
  }
}
```

### Verify Independently

Anyone can verify an anchor using Blockfrost or Cardanoscan:

```bash
# Fetch transaction metadata (label 2222)
curl -H "project_id: $BLOCKFROST_KEY" \
  "https://cardano-mainnet.blockfrost.io/api/v0/txs/{txHash}/metadata"
```

The metadata under label `2222` contains:

```json
{
  "schema": "poi-anchor-v1",
  "type": "process-trace",
  "manifestHash": "sha256:...",
  "rootHash": "sha256:...",
  "timestamp": "..."
}
```

---

## On-Chain Data Format

Orynq anchors are stored under Cardano metadata **label 2222** with the following schema:

| Field | Type | Description |
|-------|------|-------------|
| `schema` | string | Always `"poi-anchor-v1"` |
| `type` | string | Anchor type (e.g., `"process-trace"`) |
| `manifestHash` | string | SHA-256 hash of the full manifest |
| `rootHash` | string | Root hash of the process trace |
| `merkleRoot` | string | Merkle root of events (optional) |
| `timestamp` | string | ISO 8601 timestamp |

**Important**: Only cryptographic hashes are stored on-chain. Raw prompts, responses, and sensitive data are **never** written to the blockchain.

---

## SDK Packages

| Package | Description |
|---------|-------------|
| `@fluxpointstudios/orynq-sdk-core` | Protocol-neutral types and utilities |
| `@fluxpointstudios/orynq-sdk-client` | Auto-pay HTTP client with budget tracking |
| `@fluxpointstudios/orynq-sdk-payer-cardano-cip30` | CIP-30 browser wallet payer |
| `@fluxpointstudios/orynq-sdk-payer-cardano-node` | Server-side Cardano payer |
| `@fluxpointstudios/orynq-sdk-payer-evm-x402` | EIP-3009 gasless EVM payer |
| `@fluxpointstudios/orynq-sdk-server-middleware` | Express/Fastify payment middleware |
| `orynq-sdk` (Python) | Python SDK with async support |

---

## Try It Now

**Live Demo**: [fluxpointstudios.com/anchor](https://fluxpointstudios.com/anchor)

Try anchoring on testnet for free—no wallet required.

**GitHub**: [github.com/Flux-Point-Studios/orynq-sdk](https://github.com/Flux-Point-Studios/orynq-sdk)

---

## Support

- **Discord**: [discord.gg/MfYUMnfrJM](https://discord.gg/MfYUMnfrJM)
- **Twitter**: [@fluxpointstudio](https://twitter.com/fluxpointstudio)
- **Email**: support@fluxpointstudios.com
