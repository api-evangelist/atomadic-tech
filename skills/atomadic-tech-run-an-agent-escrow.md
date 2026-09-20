---
generated: '2026-09-19'
method: generated
name: Run an agent-to-agent escrow
description: Lock funds between two agents, watch the escrow state, release on completion or dispute it, and record the outcome
  to the reputation ledger.
api: openapi/atomadic-tech-openapi.yml
operations:
- registerAgent
- createEscrow
- getEscrowStatus
- releaseEscrow
- disputeEscrow
- recordReputation
- getReputationScore
source: Grounded in openapi/atomadic-tech-openapi.yml; every operationId verified verbatim. Auth per authentication/atomadic-tech-authentication.yml,
  errors per errors/atomadic-tech-problem-types.yml, replay/reversal semantics per conventions/atomadic-tech-conventions.yml,
  prices per well-known/atomadic-tech-pricing.json.
---

# Run an agent-to-agent escrow

Lock funds between two agents, watch the escrow state, release on completion or dispute it, and record the outcome to the reputation ledger.

## Auth
- `registerAgent` is in the free tier; everything else here is metered (`X-API-Key` or x402 — see the *Pay for a call with x402* skill). Prices from `/.well-known/pricing.json`: create 40 000 micro-USDC, release 20 000, dispute 60 000.

## Steps
1. **Both parties exist** — `registerAgent` (`POST /v1/agents/register`) for payer and payee if they are not already registered; the `agent_id` you choose is the id every later call references.
2. **Create the escrow** — `createEscrow` (`POST /v1/escrow/create`, required `payer_id`, `payee_id`, `amount`, `currency`; optional `conditions` object). Response carries `escrow_id`, `status`, `created_at`. Keep `escrow_id`.
3. **Poll state** — `getEscrowStatus` (`GET /v1/escrow/status?escrow_id=...`). States named on the status page: funded, released, disputed, resolved.
4. **Happy path: release** — `releaseEscrow` (`POST /v1/escrow/release`, body `{"escrow_id"}`) once the task is done; response `status: released`, `released_at`.
5. **Unhappy path: dispute** — `disputeEscrow` (`POST /v1/escrow/dispute`) with evidence. The status page says a dispute "triggers arbitration window" and arbiters auto-resolve after 3 votes (simple majority); the window length is NOT published — do not assume one.
6. **Record the outcome** — `recordReputation` (`POST /v1/reputation/record`: success, quality score, latency, context) and read it back with `getReputationScore` (`GET /v1/reputation/score?agent_id=...`) — tiers run platinum to untrusted and drive a fee multiplier.

## Rules
- `createEscrow` has no idempotency key. A retried create after a timeout can lock funds twice; check `getEscrowStatus`/your own ledger before retrying (`conventions/`, idempotency: none).
- Release is the reversal of create, dispute is the reversal of release-by-default; neither has a documented time window, so the reversibility grade is *documented*, not *verified*.
- Escrow state is opaque beyond the four names above; there is no webhook — poll.
