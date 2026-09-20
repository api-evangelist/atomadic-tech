---
generated: '2026-09-19'
method: generated
name: Check EU AI Act compliance and certify drift
description: Run a compliance check and EU AI Act certificate for a described system, monitor model drift, and write the evidence
  to the audit vault.
api: openapi/atomadic-tech-openapi.yml
operations:
- checkCompliance
- euAiActCertificate
- driftCheck
- getDriftCertificate
- logAuditEvent
- getAuditTrail
source: Grounded in openapi/atomadic-tech-openapi.yml; every operationId verified verbatim. Auth per authentication/atomadic-tech-authentication.yml,
  errors per errors/atomadic-tech-problem-types.yml, replay/reversal semantics per conventions/atomadic-tech-conventions.yml,
  prices per well-known/atomadic-tech-pricing.json.
---

# Check EU AI Act compliance and certify drift

Run a compliance check and EU AI Act certificate for a described system, monitor model drift, and write the evidence to the audit vault.

## Auth
- Metered (`X-API-Key` or x402): compliance check and EU AI Act certificate 60 000 micro-USDC each, audit log 40 000. Three free trial calls per endpoint per day apply.

## Steps
1. **General check** — `checkCompliance` (`POST /v1/compliance/check`) with the system description; returns the framework verdicts the provider computes (EU AI Act, NIST AI RMF, ISO 42001 per `/compliance`).
2. **EU AI Act certificate** — `euAiActCertificate` (`POST /v1/compliance/eu-ai-act`, quickstart body `{"system_description":"...","risk_category":"high"}`). The provider describes the output as a signed, machine-checkable certificate; store the response verbatim — it is the artifact a regulator would be shown.
3. **Drift** — `driftCheck` (`POST /v1/drift/check`) on the model, then `getDriftCertificate` (`GET /v1/drift/certificate?model_id=...`) for the 47-epoch certificate.
4. **Write the audit record** — `logAuditEvent` (`POST /v1/audit/log`) with the decision and the certificate ids; read back with `getAuditTrail` (`GET /v1/audit/trail?agent_id=...`). The vault is described as tamper-proof hash-chained.

## Rules
- These calls score what YOU describe; they are not an audit of your system. The provider's own trust page says its SOC 2 / ISO 27001 third-party audit has not happened yet (`conformance/atomadic-tech-conformance.yml`).
- Audit log writes have no undo and no idempotency key — deduplicate on your side before logging (`conventions/`).
