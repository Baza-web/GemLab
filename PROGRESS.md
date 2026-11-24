# GEM LABS — Live Development Progress Report

Platform Owner: Baza Group. First tenant: Gem Research Lab Ceylon (GRL).
This file is updated as work lands — it reflects what is actually built and
tested, not what is planned (planning lives in `docs/roadmap.md`).

## Status: Phase 1 — Foundation (in progress)

### Done
- [x] Planning docs written: architecture, database design, tenant isolation
      strategy, permission matrix, API architecture, certificate engine
      architecture, QR/verification architecture, security plan, module
      plan, roadmap.

### In progress / next up
- [ ] Scaffold independent monorepo (apps/web, apps/api, packages/shared,
      Docker Compose for local Postgres/Redis/MinIO).
- [ ] Phase 1 Prisma schema: Laboratory, Branch, User, Role, Permission,
      RolePermission, UserRole, Plan, PlanFeature, Subscription, AuditLog.
- [ ] Tenant-aware authentication (JWT, Super Admin vs tenant user
      separation, RBAC guards).
- [ ] Super Admin shell: create/activate/suspend laboratory.
- [ ] Seed Gem Research Lab Ceylon as the first tenant with an initial
      Owner user and base roles.

### Not started
Phases 2–10 per `docs/roadmap.md`.

## Environments

| Environment | Status |
|---|---|
| development (local Docker Compose) | being set up |
| staging | not provisioned |
| production | not provisioned — **no production deploy without Owner approval (master spec §51)** |

## Test evidence log

No code shipped yet this phase, so no test evidence yet. Per the ground
rules (master spec §50), nothing in this file will be marked "complete"
without an accompanying test run referenced here.

## Notes

- GEM LABS is an independent product/repo, deliberately not integrated into
  Baza ERP (master spec §3).
- The repository's required development branch is
  `claude/cool-planck-4jaaas`.
