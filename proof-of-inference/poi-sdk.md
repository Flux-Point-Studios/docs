---
description: Developer SDK for Proof-of-Inference anchoring and verification
---

# PoI SDK

The PoI SDK provides tools for anchoring AI process traces to the Cardano blockchain and verifying those anchors. It supports both direct integration and a managed Anchor-as-a-Service API.

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
npm install @fluxpointstudios/poi-sdk-core

# Client with auto-pay
npm install @fluxpointstudios/poi-sdk-client

# Cardano payer (browser wallets)
npm install @fluxpointstudios/poi-sdk-payer-cardano-cip30

# Cardano payer (server-side)
npm install @fluxpointstudios/poi-sdk-payer-cardano-node
```

### Python

```bash
pip install poi-sdk
```

---

## Quick Start

### Anchoring a Process Trace

```typescript
import { PoiClient } from '@fluxpointstudios/poi-sdk-client';
import { createCip30Payer } from '@fluxpointstudios/poi-sdk-payer-cardano-cip30';

// Connect to a CIP-30 wallet (Nami, Eternl, etc.)
const payer = await createCip30Payer(window.cardano.nami);

// Create client with auto-pay enabled
const client = new PoiClient({
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
from poi_sdk import PoiClient, BudgetConfig

client = PoiClient(
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

- **Mainnet**: ~2-3 ADA per anchor (~$1-2 USD)
- **No monthly fees or minimums**
- **Volume discounts available** for enterprise

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

PoI anchors are stored under Cardano metadata **label 2222** with the following schema:

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
| `@fluxpointstudios/poi-sdk-core` | Protocol-neutral types and utilities |
| `@fluxpointstudios/poi-sdk-client` | Auto-pay HTTP client with budget tracking |
| `@fluxpointstudios/poi-sdk-payer-cardano-cip30` | CIP-30 browser wallet payer |
| `@fluxpointstudios/poi-sdk-payer-cardano-node` | Server-side Cardano payer |
| `@fluxpointstudios/poi-sdk-payer-evm-x402` | EIP-3009 gasless EVM payer |
| `@fluxpointstudios/poi-sdk-server-middleware` | Express/Fastify payment middleware |
| `poi-sdk` (Python) | Python SDK with async support |

---

## Try It Now

**Live Demo**: [fluxpointstudios.com/anchor](https://fluxpointstudios.com/anchor)

Try anchoring on testnet for free—no wallet required.

**GitHub**: [github.com/Flux-Point-Studios/poi-sdk](https://github.com/Flux-Point-Studios/poi-sdk)

---

## Support

- **Discord**: [discord.gg/MfYUMnfrJM](https://discord.gg/MfYUMnfrJM)
- **Twitter**: [@fluxpointstudio](https://twitter.com/fluxpointstudio)
- **Email**: support@fluxpointstudios.com
