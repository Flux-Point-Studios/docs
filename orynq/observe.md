---
description: SDK for publishing cryptographically-attested AI model observation receipts to a public registry, anchored to Materios L2 and finalized on Cardano L1.
---

# Orynq Observe

`orynq-observe` is a developer SDK for producing cryptographically-attested AI model observation receipts. Each receipt commits — under a content-addressed hash anchored to Cardano L1 — to a structured claim of the form *"model X exhibited behavior Y at time Z, here is the evidence."*

Receipts are published to a public registry where they are queryable by anyone and cannot be silently withdrawn.

* **Schema:** [`ai_capability_observation_v1`](https://github.com/Flux-Point-Studios/orynq-sdk/blob/main/packages/anchors-materios/src/schemas/ai_capability_observation_v1.ts) — byte-pinned canonical CBOR, Python ↔ TypeScript lockstep.
* **Taxonomy:** [Flux-Point-Studios/ai-capability-taxonomy](https://github.com/Flux-Point-Studios/ai-capability-taxonomy) — public CVE-style stable identifiers for the observed capability classification.
* **Public registry:** browseable at `fluxpointstudios.com/observations` (rolling out).

## Installation

### Python

```bash
pip install orynq-observe
```

### TypeScript / JavaScript

```bash
npm install @orynq/observe
```

Both packages expose the same fluent builder API and the same byte-pinned canonical encoder.

## Quickstart

Time from a fresh `pip install` to a signed canonical receipt is under 10 seconds.

### Python

```python
from orynq_observe import Observation, keypair_from_seed

# Bring your own sr25519 observer key (or generate one with `orynq-observe keygen`)
observer = keypair_from_seed("0x" + "00" * 32)

obs = Observation(
    model_name="claude-opus-4-7",
    model_version="20260201",
    taxonomy_id="AUTO-MONEY-001",
    severity="high",
    observer_context="independent evaluation, agentic-finance harness",
)
obs.add_evidence(prompt="<full prompt>", response="<full response>")

# Optional: full transcript upload (content-addressed; gateway returns artifactRef)
# obs.add_artifact("/path/to/transcript.json")

# Optional: wrap evidence in a TEE attestation
# obs.attest_tee(tier="Acurast", evidence_hex="…")

receipt = obs.submit(signer=observer, network="preprod")
print(receipt.materios_tx, receipt.cardano_anchor_tx)
```

### TypeScript

```typescript
import { Observation, keypairFromSeed } from "@orynq/observe";

const observer = keypairFromSeed("0x" + "00".repeat(32));

const obs = new Observation({
  modelName: "claude-opus-4-7",
  modelVersion: "20260201",
  taxonomyId: "AUTO-MONEY-001",
  severity: "high",
  observerContext: "independent evaluation, agentic-finance harness",
});
obs.addEvidence({ prompt: "<full prompt>", response: "<full response>" });

const receipt = await obs.submit({ signer: observer, network: "preprod" });
console.log(receipt.materiosTx, receipt.cardanoAnchorTx);
```

### CLI

A small CLI ships with both packages for keypair management and one-shot submission:

```bash
orynq-observe keygen --out observer.json
orynq-observe submit \
  --signer observer.json \
  --model claude-opus-4-7 --version 20260201 \
  --taxonomy AUTO-MONEY-001 --severity high \
  --prompt-file prompt.txt --response-file response.txt
```

## Receipt fields

The canonical `ai_capability_observation_v1` wire format:

| Field | Type | Notes |
|---|---|---|
| `model.name` | string | e.g. `"claude-opus-4-7"` |
| `model.version` | string | Model release / build identifier |
| `model.hash` | hex \| `null` | Hash of model weights if known |
| `capability.taxonomyId` | string | Resolves in the taxonomy registry, e.g. `"AUTO-MONEY-001"` |
| `capability.severity` | `low` \| `medium` \| `high` \| `critical` | |
| `observation.promptHash` | hex (sha256) | Hash of the prompt; the prompt itself is **not** put on chain |
| `observation.responseHash` | hex (sha256) | Hash of the response; not put on chain |
| `observation.artifactRef` | content-addressed ref \| `null` | Optional pointer to the full transcript stored off-chain |
| `observation.occurredAt` | ISO 8601 UTC | |
| `observer.ss58` | string | Materios address of the attester |
| `observer.context` | string | Short human-readable context, ≤ 280 chars |
| `observer.teeAttestation.tier` | `ARM-TZ` \| `Acurast` \| `SEV-SNP` \| `build` | Present only if the observer ran inside a hardware-attested environment |
| `observer.teeAttestation.evidence` | hex | TEE evidence blob |

The receipt's `content_hash` is the SHA-256 of the canonical CBOR pre-image. Python and TypeScript produce byte-identical encodings for the same input — verified by cross-language lockstep tests in CI.

## What never goes on chain

* Raw prompt text
* Raw response text
* Any free-form artifact contents (only their content-addressed hash, plus an optional pointer to where the artifact is stored)

Only the structured field set above is committed.

## Attestation flow

1. The SDK signs the canonical CBOR pre-image with the observer's sr25519 key and submits the envelope to the Materios blob gateway.
2. The gateway forwards to the receipt submitter, which calls `submit_receipt_v2(content_hash, schema_hash)` on Materios.
3. The cert-daemon committee independently encodes the receipt, signs over the canonical bytes, and emits an `AvailabilityCertified` event once an M-of-N threshold is reached.
4. The anchor-worker batches certified receipts and writes them to Cardano L1 under metadata label `8746`.

After step 4, anyone with the `content_hash` can verify the full chain of custody from the receipt fields all the way back to a Cardano transaction, without trusting any single party.

## Querying observations

Receipts are queryable through three surfaces:

* **Public registry UI** — `fluxpointstudios.com/observations`, with filters for model, taxonomy ID, severity, observer, TEE tier, and date range. An RSS feed lives at `/observations.rss`.
* **JSON API** — `GET https://materios.fluxpointstudios.com/api/observations` (list) and `GET …/api/observations/:contentHash` (detail). The list endpoint accepts the same filter parameters as the UI.
* **Provenance graph** — every receipt's full lineage is renderable at `fluxpointstudios.com/materios/trace/<contentHash>`.

## Networks

| Network | Materios chain | Cardano L1 | Cost |
|---|---|---|---|
| `preprod` | Materios preprod (v6) | Cardano preprod testnet | Free — sponsored receipt submission, plus a Cardano testnet anchor fee covered by the gateway |
| `mainnet` | Materios mainnet (post-cMATRA launch) | Cardano mainnet | Per-receipt fee in MATRA, paid via the SDK's auto-pay layer |

The preprod path is free for researchers and evaluation orgs. A sponsored Bearer token is required for the gateway-hosted submission path; request one at [Discord](https://discord.gg/MfYUMnfrJM).

## Source

* **SDK packages:** [github.com/Flux-Point-Studios/orynq-sdk](https://github.com/Flux-Point-Studios/orynq-sdk) — under `packages/orynq-observe` (Python) and `packages/orynq-observe-js` (TypeScript).
* **Canonical schema codec:** under `packages/anchors-materios/src/schemas/ai_capability_observation_v1.ts` and `python/orynq_sdk/schemas/ai_capability_observation_v1.py`. The SDK packages consume this canonical codec directly — single source of truth, byte-pinned across languages.
* **Taxonomy:** [github.com/Flux-Point-Studios/ai-capability-taxonomy](https://github.com/Flux-Point-Studios/ai-capability-taxonomy) — MIT, accepts community PRs.

## Related

* [Orynq SDK](../proof-of-inference/orynq-sdk.md) — the broader Orynq SDK family, including process-trace anchoring and the Anchor-as-a-Service API.
* [Materios Partner Chain Overview](../materios/README.md) — the L2 that observation receipts are anchored to.

## Support

* Discord: [discord.gg/MfYUMnfrJM](https://discord.gg/MfYUMnfrJM)
* Email: [support@fluxpointstudios.com](mailto:support@fluxpointstudios.com)
