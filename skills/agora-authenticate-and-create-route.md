---
name: Authenticate, register an account, and create a mint/redeem route
description: Exchange an API key for a session JWT, register an account, create a reusable AUSD mint/redeem route, and track its transactions.
api: openapi/agora-openapi-original.yml
operations:
  - POST /v0/auth/token        # operationId: token (Auth)
  - POST /v0/accounts          # operationId: create (Accounts)
  - GET /v0/accounts           # operationId: get (Accounts)
  - POST /v0/routes            # operationId: create (Routes)
  - GET /v0/routes/{routeId}   # operationId: getbyid (Routes)
  - GET /v0/transactions       # operationId: list (Transactions)
---

# Authenticate, register an account, and create a route

Accounts, Routes, and Transactions are **authenticated** (Beta, access gated). Obtain an API key from Agora first.

## Steps

1. **Exchange the API key.** `POST /v0/auth/token` with `Authorization: Bearer <API_KEY>` and no body → `{ sessionJwt }`. The JWT is valid **15 minutes**.
2. **Register an account.** `POST /v0/accounts` (bearer = session JWT) to register a bank account or blockchain wallet. On `409 account_already_exists`, fetch it via `GET /v0/accounts` instead of re-creating.
3. **Create a route.** `POST /v0/routes` to define a reusable mint/redeem path (fiat ↔ AUSD or stablecoin ↔ AUSD). On `409 route_already_exists`, read `context.routeId` and fetch the existing route via `GET /v0/routes/{routeId}` — route ids are deterministic.
4. **Track transactions.** `GET /v0/transactions` to list settled transactions and per-leg detail settling along your routes.

## Rules
- Attach `Authorization: Bearer <sessionJwt>` on every call except Metrics.
- On `401` with `context.reason: token_expired`, re-exchange the API key (step 1); there is no refresh-token endpoint.
- The API is **not idempotent** — there is no idempotency-key header. Rely on `409` conflict codes and deterministic route ids rather than retrying blind writes.
- Amount fields are decimal strings (6 dp); parse with a decimal library.
- Match on the stable error `code`, never on `message`. Log the `Request-Id` header.
- Validation failures return `400 parameter_invalid` with a `context.issues[]` array — inspect it for the offending field.
