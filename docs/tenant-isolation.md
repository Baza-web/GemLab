# Tenant Isolation Strategy

## Model

A **tenant** = a Laboratory organisation (e.g. "Gem Research Lab Ceylon"). A
Laboratory may have multiple Branches. All tenant-scoped data carries a
`laboratoryId`. Branch-level scoping (`branchId`) is a second, optional layer
under the same laboratory.

The **Super Admin** (Baza Group / Platform Owner) is not a tenant — it is a
separate, platform-level principal that can see cross-tenant metadata
(subscription status, usage, support tickets) but not laboratory business
data, except through an explicit, audited, time-limited support-access grant
(§4 of the master spec — "never unrestricted access by default").

## Phase 1–8 approach: shared database, enforced row isolation

Start with a **single PostgreSQL database, shared schema, `laboratoryId` on
every tenant-owned table**, enforced at three layers so a bug in any one layer
is not enough to leak data:

1. **Application layer (primary defense).** Every tenant-scoped Prisma query
   goes through a `TenantScopedRepository` base that injects
   `where: { laboratoryId: ctx.laboratoryId }` automatically. Direct use of
   the raw Prisma client for tenant tables outside this base is disallowed by
   lint rule / code review checklist.
2. **Request layer.** A `TenancyGuard` resolves `laboratoryId` from the
   authenticated JWT (never from a client-supplied body/query param) and
   attaches it to request context before any handler runs. Super Admin
   endpoints live under a separate `/api/v1/platform/*` namespace that never
   resolves a `laboratoryId` and never touches tenant business tables.
3. **Database layer (defense in depth).** PostgreSQL Row-Level Security (RLS)
   policies on tenant tables, keyed off a `current_setting('app.laboratory_id')`
   set at the start of each transaction. Even a raw/forgotten-filter query
   cannot cross tenants if RLS is on. RLS is introduced as soon as the schema
   stabilizes in Phase 1, not deferred to "later hardening."

Certificate numbering sequences are **transactional and unique per
laboratory (and per branch where configured)** — enforced with a DB unique
constraint on `(laboratoryId, branchId, certificateNumber)`, not just
application logic (see `docs/qr-verification.md`).

## Enterprise option: database-per-tenant

For laboratories requiring stronger isolation (Enterprise plan, regulatory
requirement, or sheer scale), the architecture supports migrating a single
tenant to its own database: the `TenantScopedRepository` + connection
resolution layer already requires knowing "which DB does this tenant live
in," so adding a per-tenant connection pool is additive, not a rewrite.
This is evaluated per-tenant, not adopted platform-wide in phase 1.

## What isolation must never depend on

- The QR code or certificate PDF itself. Verification always re-checks the
  authoritative certificate record server-side (§24, §26).
- Client-supplied tenant/laboratory identifiers in request bodies or query
  strings.
- Soft conventions ("just remember to filter by laboratoryId") without a
  structural enforcement (guard + base repository + RLS).

## Testing requirement

Cross-tenant isolation is a first-class automated test suite: for every
tenant-scoped endpoint, a test asserts that User A (Lab 1) cannot read,
list, or mutate a record belonging to Lab 2, including by guessing/
enumerating IDs. This is acceptance scenario #18 in `docs/roadmap.md`.
