---
description: Developer SDK for Proof-of-Inference anchoring and verification
---

# Orynq SDK

The Orynq SDK provides tools for anchoring AI process traces to the Cardano blockchain and verifying those anchors. It supports both direct integration and a managed Anchor-as-a-Service API.

**Orynq** is the umbrella platform for AI verification and attestation services, with **Proof-of-Inference anchoring** as its core capability.

## Features

- **Process Trace Anchoring** - Commit cryptographic proofs of AI execution to Cardano
- **Self-Hosted Anchoring** - Use your own wallet to anchor directly—no API fees
- **OpenClaw/Claude Code Integration** - Zero-config anchoring for AI coding sessions
- **Dual Protocol Support** - x402 (Coinbase standard) for EVM and Flux protocol for Cardano
- **Multi-Chain Payments** - Pay for anchoring in ADA, $AGENT, or EVM stablecoins
- **Auto-Pay Client** - Automatic 402 payment handling with budget controls
- **Verification Tools** - Independently verify any anchor using only the txHash
- **TypeScript & Python** - Full SDK support for both languages

## Installation

### TypeScript/JavaScript

```bash
# Process tracing (instrument your AI agent)
npm install @fluxpointstudios/poi-sdk-process-trace

# Self-hosted anchoring (use your own wallet)
npm install @fluxpointstudios/poi-sdk-anchors-cardano lucid-cardano

# OR use the managed API with auto-pay client
npm install @fluxpointstudios/poi-sdk-client
npm install @fluxpointstudios/poi-sdk-payer-cardano-cip30  # Browser wallets
npm install @fluxpointstudios/poi-sdk-payer-cardano-node   # Server-side
```

### Python

```bash
pip install poi-sdk
```

---

## OpenClaw Integration

Automatically anchor your [OpenClaw](https://openclaw.ai) AI coding sessions to the blockchain with zero configuration. Every coding session becomes a verifiable, tamper-proof record.

### One-Line Install

```bash
# Linux/macOS
curl -fsSL https://raw.githubusercontent.com/Flux-Point-Studios/orynq-sdk/main/scripts/install-openclaw.sh | bash

# Windows (PowerShell)
irm https://raw.githubusercontent.com/Flux-Point-Studios/orynq-sdk/main/scripts/install-openclaw.ps1 | iex
```

This installs:
1. **OpenClaw** (official installer)
2. **Orynq OpenClaw recorder** as a background daemon

### Manual Install

```bash
npx @fluxpointstudios/orynq-openclaw install --service
```

### Configuration

After installation, add your Orynq partner key to enable on-chain anchoring:

```bash
# Linux/macOS
echo "ORYNQ_PARTNER_KEY=your_key_here" >> ~/.config/orynq-openclaw/service.env

# Windows
echo ORYNQ_PARTNER_KEY=your_key_here >> %APPDATA%\orynq-openclaw\service.env
```

Without a partner key, the recorder still creates local process traces—they just won't be anchored to the blockchain.

### Commands

| Command | Description |
|---------|-------------|
| `orynq-openclaw status` | Check daemon status |
| `orynq-openclaw logs -f` | View logs (follow mode) |
| `orynq-openclaw start` | Run in foreground |
| `orynq-openclaw restart-service` | Restart the daemon |
| `orynq-openclaw doctor` | Check configuration |
| `orynq-openclaw uninstall --service --purge` | Remove completely |

### How It Works

```
OpenClaw Session → JSONL Logs → Orynq Recorder → Process Trace → Cardano Anchor
```

1. **Tails** OpenClaw JSONL session logs in real-time
2. **Builds** cryptographic process traces with:
   - Rolling hash chains for event ordering
   - Merkle trees for selective disclosure
   - SHA-256 content hashes for integrity
3. **Anchors** manifests to Cardano via Orynq API
4. **Stores** local bundles, manifests, and receipts

**Privacy**: Only cryptographic hashes are sent to the blockchain—never raw prompts, code, or responses.

### Daemon Support

| Platform | Daemon Type | Location |
|----------|-------------|----------|
| Linux | systemd user service | `~/.config/systemd/user/` |
| macOS | launchd LaunchAgent | `~/Library/LaunchAgents/` |
| Windows | Task Scheduler | User tasks |

### Output Structure

```
~/.openclaw/orynq/recorder/
├── spool/           # Pending events by bundle
├── bundles/         # Complete process trace bundles
├── manifests/       # Anchor manifests (hashes only)
├── receipts/        # Anchor confirmations with txHash
├── chunks/          # Large bundle chunks
└── state/           # Tail position & anchor state
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
- [Starter Plan ($49/mo)](https://buy.stripe.com/7sY7sL6wXgNe9cl9sJfrW09)
- [Growth Plan ($199/mo)](https://buy.stripe.com/fZu14n7B168A88h0WdfrW0a)

**All plans include:**
- No wallet management required
- API key authentication (X-Partner header)
- Usage dashboard at [fluxpointstudios.com/orynq/enterprise-dashboard](https://fluxpointstudios.com/orynq/enterprise-dashboard)
- Pay via invoice (USD)
- Dedicated support
- SLA available

Contact [sales@fluxpointstudios.com](mailto:sales@fluxpointstudios.com) for custom Scale pricing.

---

## Self-Hosted Anchoring

For maximum control and privacy, you can anchor directly to Cardano using your own wallet—no API or service fees required. This is ideal for:

- **Privacy-conscious users** who don't want to route through a third-party API
- **High-volume anchoring** where per-anchor fees add up
- **Air-gapped environments** with no external API access
- **Custom integrations** with existing Cardano infrastructure

### Installation

```bash
npm install @fluxpointstudios/poi-sdk-process-trace \
            @fluxpointstudios/poi-sdk-anchors-cardano \
            lucid-cardano
```

### Complete Example

```typescript
import {
  createTrace,
  addSpan,
  addEvent,
  closeSpan,
  finalizeTrace,
} from "@fluxpointstudios/poi-sdk-process-trace";

import {
  createAnchorEntryFromBundle,
  buildAnchorMetadata,
  serializeForCbor,
  POI_METADATA_LABEL,
} from "@fluxpointstudios/poi-sdk-anchors-cardano";

import { Lucid, Blockfrost } from "lucid-cardano";

// 1. Instrument your AI agent
const run = await createTrace({ agentId: "my-agent" });
const span = addSpan(run, { name: "task" });

await addEvent(run, span.id, {
  kind: "observation",
  content: "User requested code review",
  visibility: "public",
});

await addEvent(run, span.id, {
  kind: "decision",
  content: "Will check for security issues first",
  visibility: "public",
});

await closeSpan(run, span.id);
const bundle = await finalizeTrace(run);

// 2. Build anchor metadata
const entry = createAnchorEntryFromBundle(bundle);
const metadata = serializeForCbor(buildAnchorMetadata(entry));

// 3. Submit with your own wallet
const lucid = await Lucid.new(
  new Blockfrost("https://cardano-mainnet.blockfrost.io/api/v0", "your-project-id"),
  "Mainnet"
);
lucid.selectWalletFromSeed("your seed phrase here");

const tx = await lucid
  .newTx()
  .attachMetadata(POI_METADATA_LABEL, metadata[POI_METADATA_LABEL])
  .complete();

const txHash = await tx.sign().complete().then(t => t.submit());
console.log("Anchored:", txHash);
```

### Cost

- **Mainnet**: ~0.2-0.3 ADA per anchor (~$0.10-0.20 USD) — just the Cardano tx fee
- **Preprod testnet**: Free (get test ADA from the [faucet](https://docs.cardano.org/cardano-testnets/tools/faucet/))

No service fees, no subscriptions—you pay only the blockchain transaction fee.

### cardano-cli Alternative

If you prefer cardano-cli over Lucid:

```typescript
import { serializeForCardanoCli } from "@fluxpointstudios/poi-sdk-anchors-cardano";

const cliJson = serializeForCardanoCli(buildAnchorMetadata(entry));
fs.writeFileSync("metadata.json", cliJson);
```

```bash
cardano-cli transaction build \
  --tx-in <UTXO> \
  --change-address <YOUR_ADDRESS> \
  --metadata-json-file metadata.json \
  --out-file tx.raw

cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --out-file tx.signed

cardano-cli transaction submit --tx-file tx.signed
```

### Full Example

See the complete runnable example: [github.com/Flux-Point-Studios/orynq-sdk/tree/main/examples/self-anchor](https://github.com/Flux-Point-Studios/orynq-sdk/tree/main/examples/self-anchor)

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
| `@fluxpointstudios/poi-sdk-process-trace` | Cryptographic process trace builder |
| `@fluxpointstudios/poi-sdk-anchors-cardano` | Cardano anchor builder & verifier (for self-hosted anchoring) |
| `@fluxpointstudios/poi-sdk-core` | Protocol-neutral types and utilities |
| `@fluxpointstudios/poi-sdk-client` | Auto-pay HTTP client with budget tracking |
| `@fluxpointstudios/poi-sdk-payer-cardano-cip30` | CIP-30 browser wallet payer |
| `@fluxpointstudios/poi-sdk-payer-cardano-node` | Server-side Cardano payer |
| `@fluxpointstudios/poi-sdk-payer-evm-x402` | EIP-3009 gasless EVM payer |
| `@fluxpointstudios/poi-sdk-server-middleware` | Express/Fastify payment middleware |
| `poi-sdk` (Python) | Python SDK with async support |

---

## Try It Now

**Live Demo**: [fluxpointstudios.com/orynq](https://fluxpointstudios.com/orynq)

Try anchoring on testnet for free—no wallet required.

**GitHub**: [github.com/Flux-Point-Studios/orynq-sdk](https://github.com/Flux-Point-Studios/orynq-sdk)

---

## Support

- **Discord**: [discord.gg/MfYUMnfrJM](https://discord.gg/MfYUMnfrJM)
- **Twitter**: [@fluxpointstudio](https://twitter.com/fluxpointstudio)
- **Email**: support@fluxpointstudios.com
