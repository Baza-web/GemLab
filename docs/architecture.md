# GEM LABS — System Architecture

Platform Owner: Baza Group
Product: GEM LABS (independent product — not integrated into Baza ERP)
First tenant: Gem Research Lab Ceylon (GRL)
Primary domain: gemlab.baza-boutique.com

## 1. Shape of the system

GEM LABS is a **modular monolith**, not a microservices system, for phase 1–8. A single
backend process is organized into clearly bounded modules (see §3) with explicit
interfaces between them. Microservices are deferred until there is a measured,
specific reason (e.g. a module needs independent scaling or a different runtime).

Two deployable applications, one shared package, one database at first:

```
apps/
  web/        Next.js (App Router) + TypeScript — tenant web app, customer portal,
              public verification site, Super Admin console
  api/        NestJS + TypeScript — the modular monolith backend
packages/
  shared/     Shared TypeScript types/DTOs/enums used by both web and api
              (no business logic — just contracts)
```

Independent from Baza ERP:
- own GitHub repository (this one)
- own database (PostgreSQL, own instance/schema)
- own backend, own frontend, own auth, own API
- own file storage bucket(s)
- own deployment pipeline and own backups

Initial deployment may reuse Baza Group's existing VPS, but nothing in the
application may assume co-location with Baza ERP. No shared database, no shared
auth, no direct in-process calls into Baza ERP code.

## 2. Request flow (high level)

```
Browser / Mobile
      |
      v
Next.js (apps/web)  --- public verification pages (SSR, no auth)
      |                 --- tenant app (SPA-like, authenticated)
      |                 --- super admin console (authenticated, platform role)
      v
NestJS API (apps/api) --- REST, versioned (/api/v1/...)
      |
      +-- AuthModule          (tenant-aware JWT auth, Super Admin auth)
      +-- TenancyModule        (laboratory/branch context, isolation guard)
      +-- LaboratoryModule     (labs, branches, onboarding)
      +-- IdentityModule       (users, roles, permissions)
      +-- CustomerModule       (CRM, customer portal data)
      +-- SampleIntakeModule   (sample registration, chain of custody)
      +-- GemstoneModule       (gemstone records, digital passport)
      +-- JewelleryModule
      +-- TestingModule        (methods library, examinations, measurements)
      +-- TreatmentModule
      +-- OriginModule
      +-- DiamondModule
      +-- PearlModule
      +-- PreciousMetalModule
      +-- EquipmentModule      (instruments, calibration)
      +-- QualityModule        (ISO 17025 support: SOPs, audits, NC/CAPA)
      +-- CertificateModule    (templates, field mapping, rendering, numbering)
      +-- QrVerificationModule (QR issuance, public verification)
      +-- ApprovalModule       (workflow: review -> approval -> issuance)
      +-- FinanceModule        (quotations, invoices, payments, expenses)
      +-- DocumentModule       (file storage abstraction, versioning)
      +-- TaskModule           (queues, automation, notifications)
      +-- AiAssistantModule    (read-only assistant, never authoritative)
      +-- ReportingModule      (BI/analytics, read models)
      +-- SubscriptionModule   (plans, limits, billing)
      +-- SuperAdminModule     (platform owner operations)
      +-- AuditModule          (cross-cutting audit log sink)
      |
      v
PostgreSQL (primary data)   Redis + BullMQ (jobs, queues)   S3-compatible storage (files)
```

Modules communicate through injected services and domain events inside the
process (Nest's event emitter) rather than HTTP, until/unless a module is
split out.

## 3. Module boundaries and ownership

Each module in §2 owns its own tables and exposes a narrow service interface
to other modules. No module reaches into another module's Prisma models
directly — cross-module reads go through the owning module's service. This is
what keeps the monolith splittable later without a rewrite.

## 4. Environments

Three environments, each with its own database, storage bucket, and secrets:

- **development** — local Docker Compose (Postgres, Redis, MinIO for S3-compatible storage)
- **staging** — mirrors production, used for UAT before any tenant-facing release
- **production** — never touched by an untested migration; deploys require Owner approval (§51)

CI runs lint, typecheck, unit tests, and migration dry-runs on every change;
staging deploys are automatic on merge to the default branch, production
deploys are manual/gated.

## 5. Why this stack

See `docs/roadmap.md` for phased delivery and `docs/tenant-isolation.md`,
`docs/security-plan.md`, `docs/certificate-engine.md`, `docs/qr-verification.md`
for the architecture of the platform's hardest requirements.

- **Next.js/TypeScript** — one codebase for tenant app, customer portal, Super
  Admin console, and the public verification site, with SSR for fast, SEO-safe
  public verification pages.
- **NestJS/TypeScript** — structured modules, DI, guards, and interceptors map
  directly onto "configurable modules per tenant" and "enforce isolation on
  every endpoint" requirements.
- **PostgreSQL** — strong relational integrity, transactions, and unique
  constraints are required for certificate numbering and QR permanence; these
  are correctness requirements, not just performance preferences.
- **Prisma** — typed schema/migrations shared conceptually across modules,
  with a single source of truth for the ER model in `docs/database-design.md`.
- **Redis + BullMQ** — background jobs (PDF rendering, notifications,
  calibration alerts, webhook retries).
- **S3-compatible storage** — photographs, instrument readings, signed
  certificate PDFs; MinIO locally, real object storage in staging/production.
