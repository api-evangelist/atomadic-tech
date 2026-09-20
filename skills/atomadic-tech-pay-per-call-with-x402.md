---
generated: '2026-09-19'
method: generated
name: Pay for a call with x402
description: "Make a metered AAAA-Nexus call as an autonomous agent: read the price manifest, take the 402 challenge, pay\
  \ USDC on Base and retry with the proof \u2014 or use the 3 daily free trial calls."
api: openapi/atomadic-tech-openapi.yml
operations:
- getHealth
- getPricingManifest
- getQuantumRng
- chatInference
source: Grounded in openapi/atomadic-tech-openapi.yml; every operationId verified verbatim. Auth per authentication/atomadic-tech-authentication.yml,
  errors per errors/atomadic-tech-problem-types.yml, replay/reversal semantics per conventions/atomadic-tech-conventions.yml,
  prices per well-known/atomadic-tech-pricing.json.
---

# Pay for a call with x402

Make a metered AAAA-Nexus call as an autonomous agent: read the price manifest, take the 402 challenge, pay USDC on Base and retry with the proof — or use the 3 daily free trial calls.

## Auth
- Free operations (`getHealth`, `getQuantumRng`, `getPricingManifest`) need no credential.
- Metered operations accept EITHER `X-API-Key: <key from https://atomadic.tech/pay>` OR an x402 payment proof. Every metered endpoint also allows 3 credential-free trial calls per day (agent card `trialPolicy`; trial calls add a 2 s delay).
- Base URL: `https://atomadic.tech`.

## Steps
1. **Check the service is up** — `getHealth` (`GET /health`). Free. Returns `status`, `version` and the current entropy `epoch`.
2. **Read the price list** — `getPricingManifest` (`GET /.well-known/pricing.json`). Free. `per_call_micro_usdc` is keyed by path; `tiers.free` lists the no-payment operations; `tiers.credit_packs` prices API keys (500 calls / 8 USDC, 2 500 / 30, 10 000 / 98).
3. **Prove the free path works** — `getQuantumRng` (`GET /v1/rng/quantum`). Free; no header needed.
4. **Fire the metered call without credentials** — `chatInference` (`POST /v1/inference`, body `{"messages":[{"role":"user","content":"..."}],"max_tokens":128}`). If you have trial calls left you get a 200. Otherwise you get **402** with `amount_micro_usdc`, `recipient`, `nonce`, `chain_id` 8453, `token_address` (USDC on Base) and `expires_in_seconds` (300).
5. **Pay and retry** — transfer exactly `amount_micro_usdc` USDC to `recipient` on Base, then repeat the call with `PAYMENT-SIGNATURE: <base64_sig>;<base64_pk>;<txid>;<amount_micro>` (or `X-402-Payment`). The nonce is single-use and dies after 5 minutes; a late retry gets a fresh 402 with a new nonce — pay again, do not reuse the old proof.
6. **Or use a key** — buy a credit pack at `/pay` and send `X-API-Key` instead; keys have no expiry (quickstart).

## Rules
- The spec documents `X-Payment-Proof` and three chains; the live 402 names `PAYMENT-SIGNATURE`/`X-402-Payment` and only Base. Follow the live body (`errors/atomadic-tech-problem-types.yml`).
- Payments are non-refundable once the on-chain transfer confirms (`/legal`). There is no dry-run: a paid call is spent the moment it is accepted. See `conventions/atomadic-tech-conventions.yml` (reversibility).
- Quote `X-Request-Id` from the response when reporting a problem.
