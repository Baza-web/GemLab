# Security Plan

## Authentication
- Tenant users and Super Admin users are stored and authenticated separately
  (different tables, different login endpoints) so a tenant-side bug can
  never escalate to platform access.
- Passwords hashed with argon2id. JWT access tokens (short-lived) + refresh
  tokens (rotated, revocable). MFA (TOTP) supported, required for Super Admin
  and recommended (plan-configurable to required) for tenant Owner/Admin
  roles.
- Session controls: revoke-all-sessions on password change, device/session
  listing per user.

## Authorization
- `TenancyGuard` + `PermissionGuard` on every tenant route (see
  `docs/api-architecture.md`, `docs/tenant-isolation.md`,
  `docs/permission-matrix.md`).
- Super Admin access to tenant business data is never ad hoc: a support
  session requires an explicit, time-boxed `SupportAccessGrant` record
  (reason, approver if required, expiry) and every read under that grant is
  audit-logged (§4).

## Application security
- Input validation via DTO class-validator decorators on every endpoint
  (reject unknown fields).
- Parameterized queries only (Prisma) — no raw SQL string interpolation;
  the one place raw SQL is used (RLS session variable setting) takes no
  user-controlled string content.
- Output encoding / React's default escaping covers XSS on the frontend;
  rich-text fields (SOPs, comments) are sanitized server-side before storage.
- CSRF protection on cookie-based session endpoints (the customer portal);
  the API-token-based tenant app is not vulnerable to classic CSRF but still
  sets `SameSite=strict` where cookies are used at all.
- Rate limiting: per-IP on public verification and auth endpoints; per-tenant
  and per-user on authenticated API, limits configurable by subscription
  plan.

## Secrets management
- No API keys/credentials in frontend code or in the repository, ever.
  Environment-specific secrets (DB credentials, JWT signing keys, storage
  keys, OAuth client secrets, AI provider keys) live in the deployment
  platform's secret store, injected as environment variables at runtime.
  `.env.example` documents required variable names with placeholder values
  only.

## File & data security
- Uploaded files (photographs, instrument exports, SOPs, signed PDFs) are
  stored in S3-compatible storage with per-laboratory key prefixes and
  signed, short-lived URLs for access — never public-read buckets.
- Encrypted backups (see `docs/backup-dr.md` once written) include database
  and object storage; backup access is restricted and logged.

## Audit logging
- `AuditLog` captures: actor, action, entity type/id, before/after diff
  (for mutable fields), laboratory, timestamp, IP/user-agent. Append-only,
  no update/delete grant for the application DB role.

## AI assistant constraints (§38, enforced in code, not just policy)
- The AI assistant module is read/draft-only: it can search, summarize, and
  draft text, but has no write path to Examination, TestingResult,
  TreatmentFinding, OriginOpinion, Valuation, or Certificate tables. Any
  AI-drafted content inserted into a record is stored with an
  `aiAssisted: true` flag and still requires the same human
  review/approval path as manually entered content.
