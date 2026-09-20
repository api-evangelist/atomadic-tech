---
generated: '2026-09-19'
method: generated
name: Secure a session with RatchetGate
description: "Register a 47-epoch ratcheting session, advance and verify it, inspect its status and revoke it \u2014 the provider's\
  \ mitigation for MCP session-fixation (CVE-2025-6514)."
api: openapi/atomadic-tech-openapi.yml
operations:
- ratchetRegisterSession
- ratchetAdvanceSession
- ratchetProbe
- ratchetStatus
- ratchetVerify
- ratchetList
- ratchetRevoke
source: Grounded in openapi/atomadic-tech-openapi.yml; every operationId verified verbatim. Auth per authentication/atomadic-tech-authentication.yml,
  errors per errors/atomadic-tech-problem-types.yml, replay/reversal semantics per conventions/atomadic-tech-conventions.yml,
  prices per well-known/atomadic-tech-pricing.json.
---

# Secure a session with RatchetGate

Register a 47-epoch ratcheting session, advance and verify it, inspect its status and revoke it — the provider's mitigation for MCP session-fixation (CVE-2025-6514).

## Auth
- All seven operations are metered (`X-API-Key` or x402); register is 20 000 micro-USDC per the pricing manifest (the spec's prose says $0.002 — the manifest is the machine-readable source, trust it).

## Steps
1. **Register** — `ratchetRegisterSession` (`POST /v1/ratchet/register`, required `session_id`, e.g. `sess-001`; the quickstart also passes `agent_id`). Response: `session_id`, `registered_at`, `status: registered`.
2. **Advance each epoch** — `ratchetAdvanceSession` (`POST /v1/ratchet/advance`) on every sensitive step; the ratchet is 47 epochs deep with a formal re-key schedule (agent card skill `ratchetgate`).
3. **Probe / inspect** — `ratchetProbe` (`GET /v1/ratchet/probe`) and `ratchetStatus` (`GET /v1/ratchet/status?session_id=...`) to read the current epoch without advancing it.
4. **Verify a presented session** — `ratchetVerify` (`POST /v1/ratchet/verify`) before trusting a counterparty's claimed session state.
5. **Enumerate** — `ratchetList` (`GET /v1/ratchet/list`) for the sessions your key owns.
6. **Revoke** — `ratchetRevoke` (`POST /v1/ratchet/revoke`) when the session ends or looks compromised. Revocation is the reversal path; no reinstatement window is documented, so treat it as final.

## Rules
- Advance is not idempotent by design (each call moves the epoch). Never retry an advance blindly after a timeout — call `ratchetStatus` first and compare epochs.
- `session_id` is client-chosen; use an unguessable value, the whole point of the mechanism is fixation resistance.
