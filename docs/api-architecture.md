# API Architecture

## Conventions

- REST, versioned under `/api/v1/...`. Breaking changes ship as `/api/v2`,
  old version kept until all clients (including customer portal integrations)
  migrate.
- Two top-level namespaces:
  - `/api/v1/platform/*` — Super Admin only, never tenant-scoped, never
    touches laboratory business tables.
  - `/api/v1/*` — tenant-scoped; every handler runs behind `TenancyGuard`
    (resolves `laboratoryId` from JWT) and `PermissionGuard`
    (`@RequirePermission(...)`).
  - `/api/v1/public/verify/*` — unauthenticated, read-only, heavily
    rate-limited, returns only the public-safe certificate fields (§25).
- Request/response DTOs are defined once in `packages/shared` and imported by
  both `apps/api` (validation) and `apps/web` (typed client), so the frontend
  and backend cannot silently drift.
- Mutations that create legally/scientifically significant records
  (Certificate issuance, QR creation, Examination submission, Approval
  records) are wrapped in a single DB transaction with the relevant unique
  constraints — the API layer never "issues first, numbers later."
- Idempotency keys are required on certificate issuance and payment
  endpoints to make retried requests safe.
- Standard error shape: `{ code, message, details? }`; codes are stable
  strings (`TENANT_NOT_FOUND`, `CERTIFICATE_ALREADY_ISSUED`, ...), not just
  HTTP status, since the frontend and customer portal need to branch on them.

## Rate limiting & abuse protection

- Public verification endpoints: IP + certificate-number based rate limiting,
  logged as `VerificationEvent` rows for monitoring (not for exposing to the
  public).
- Authenticated endpoints: per-user and per-tenant rate limits, configurable
  by subscription plan (Super Admin controlled, §42).

## Webhooks & integrations

- Outbound webhooks (e.g. certificate issued, payment received) are signed
  (HMAC) and logged with delivery status + retry count (§41).
- Inbound integrations (Gmail, Drive, Sheets, payment gateways) go through
  `IntegrationModule` adapters with per-tenant OAuth credentials stored
  encrypted (§46) — never hard-coded, never in frontend code.
