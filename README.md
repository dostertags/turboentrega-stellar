# turboentrega-stellar

Proof-of-delivery attestation and delivery-versus-payment escrow on Stellar (Soroban), built for **TurboEntrega** — a delivery-operations platform whose offline-first driver app captures geofenced, photo- and signature-backed proof of delivery.

> **Status:** proposed Stellar Instawards sprint (Stellar Ambassador Chapter Chile). Work begins within 5 business days of award notice.
> **Testnet only** — no mainnet deployment, no real funds, no token issuance.

## The idea

Soroban escrows that release payment on delivery already exist; they depend on a "trusted attestor" to say the goods arrived. TurboEntrega already produces the evidence that attestation should rest on. This repository turns that evidence into the on-chain trigger:

1. When a driver completes a delivery, the evidence (GPS fix, server-recomputed geofence result, photo, signature, receiver name) is salted, canonicalised and hashed inside the private application.
2. The hash is attested on-chain. The registry is append-only: an attestation can never be overwritten.
3. An independent confirmer recomputes the hash themselves and confirms it; the contract rejects any confirmation that does not match.
4. Escrowed funds release only when a matching attestation *and* confirmation exist, and refund to the payer if the deadline passes first. Settlement happens once.

## What this repository will contain

| Path | Contents |
|---|---|
| `contracts/pod-escrow/` | Soroban contract (Rust): `__constructor`, `attest`, `confirm`, `lock`, `release`, `refund`, `get_attestation`, `get_escrow`. Non-upgradeable. |
| `packages/evidence/` | Canonical, salted evidence hashing (TypeScript) used by the delivery pipeline. |
| `apps/verify/` | Public verifier that reads contract state directly from Soroban RPC. |
| `apps/confirm/` | Static confirmation page: recomputes the evidence hash in the confirmer's own browser and asks them to sign `confirm` in their Freighter wallet. |
| `.github/workflows/` | CI for all of the above. |

## Sprint plan (30 days, 120 engineering hours)

| # | Deliverable | Days |
|---|---|---|
| 1 | Evidence layer, tests & CI | 1–5 |
| 2 | Soroban attestation & escrow contract | 6–16 |
| 3 | Pipeline integration, independent confirmation & pilot setup | 17–23 |
| 4 | Public verifier, seven-day Testnet pilot & technical report | 24–30 |

## Verify a delivery without us

Once the contract is deployed, anyone can check a delivery with the Stellar CLI alone, without using any TurboEntrega infrastructure:

```sh
stellar contract invoke --network testnet --id <CONTRACT_ID> -- get_attestation --delivery_id <DELIVERY_ID>
```

## Privacy

Only salted hashes ever reach the contract, the verifier, or this repository. Names, addresses, photos and signatures stay inside the private TurboEntrega application.

## Toolchain

Rust 1.84+ with the `wasm32v1-none` target (the only Wasm target the Soroban runtime supports) and `stellar-cli`. Exact versions are pinned in this file on day 1 of the sprint.

## License

MIT. See [LICENSE](LICENSE).
