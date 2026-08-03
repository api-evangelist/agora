---
name: Read AUSD supply metrics
description: Fetch real-time AUSD total and circulating supply, aggregate and per-chain, with no credentials.
api: openapi/agora-openapi-original.yml
operations:
  - GET /v0/metrics            # operationId: get (Metrics)
  - GET /v0/metrics/total-supply        # operationId: totalsupply
  - GET /v0/metrics/circulating-supply  # operationId: circulatingsupply
---

# Read AUSD supply metrics

The Metrics endpoints are **public** — no API key or session JWT required.

## Steps

1. Call `GET https://api.agora.finance/v0/metrics` for the aggregate + per-chain snapshot of AUSD supply across every supported network.
2. For a single figure, call `GET /v0/metrics/total-supply` or `GET /v0/metrics/circulating-supply`.
3. Parse supply amounts as **decimal strings** (six decimals of precision) — never as JSON numbers, or you will truncate. Use `decimal.Decimal` / `BigNumber.js` / `rust_decimal`.
4. Log the `Request-Id` response header for support tracing.

## Rules
- No auth header needed; do not attach a bearer token.
- Metrics may report partial per-chain state — check each chain entry rather than assuming completeness.
- On `429 rate_limit_exceeded`, honor the `Retry-After` header with exponential backoff.
