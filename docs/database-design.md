# Database Design (Phase 1 scope + forward map)

Full normalized schema lives in `apps/api/prisma/schema.prisma` (source of
truth). This document explains the entity groups and the relationships that
matter for correctness, and marks what Phase 1 actually implements vs. what
is modeled here for later phases.

## Entity groups

### Platform / Tenancy (Phase 1 — implemented)
- `Laboratory` — a tenant. Has plan, status (pending/active/suspended), branding, country, timezone, numbering config.
- `Branch` — belongs to a Laboratory. Optional second scoping level.
- `User` — belongs to a Laboratory (tenant user) OR is a Platform user (Super Admin), never both.
- `Role`, `Permission`, `RolePermission`, `UserRole` — RBAC, scoped per laboratory (a Role is laboratory-owned, except built-in platform roles).
- `Subscription`, `Plan`, `PlanFeature` — plan/feature/limit configuration, editable by Super Admin, never hard-coded in app logic.
- `AuditLog` — append-only, cross-cutting. Every mutation of a tenant-sensitive record writes one.

### Customer & Sample (Phase 2)
- `Customer` — belongs to a Laboratory.
- `Sample` — intake record; unique `sampleId` per laboratory. Not the same identifier space as certificate QR (§8 rule).
- `ChainOfCustodyEvent` — append-only log of every physical movement of a Sample (§9). Never updated, only inserted.

### Gemstone / Jewellery / Testing (Phase 3)
- `Gemstone` — the digital passport root (§20). One row per physical gemstone, persists across many examinations and certificates.
- `JewelleryItem`
- `Examination` — one row per examination event; references a `TestingMethodVersion`, never a mutable "current method."
- `TestingMethod`, `TestingMethodVersion` — versioned method library (§18). An Examination stores a FK to the exact version used, immutable thereafter.
- `Measurement` — raw reading, FK to Examination. `rawValue` is stored separately from any `derivedValue` + `calculationMethod` (§19). Raw values are never edited after insert; corrections create a new Measurement row with a `supersedesId` back-reference.
- `TreatmentFinding`, `OriginOpinion`, `DiamondGrading`, `PearlAssessment`, `PreciousMetalTest` — specialized result tables, each FK'd to an Examination.

### Equipment & Quality (Phase 6)
- `Equipment`, `CalibrationRecord`
- `Sop`, `Audit`, `Nonconformity`, `CorrectiveAction`, `CompetencyRecord`, `ProficiencyTest`

### Certificates & QR (Phase 4–5 — Phase 1 lays the FK groundwork only)
- `CertificateTemplate`, `CertificateTemplateVersion` — versioned, per laboratory (max 10 active templates enforced by subscription limit, not schema).
- `Certificate` — immutable once issued. Status enum: DRAFT, PENDING_APPROVAL, ISSUED, AMENDED, SUPERSEDED, REVOKED, CANCELLED.
- `CertificateVersion` — every amendment/supersession creates a new version row; the original is never edited in place (§28).
- `QrIdentifier` — **one-to-one, immutable, with `Certificate`**, created only at issuance (not at preview). Unique constraint on `certificateId` and on the public `qrCode` value. See `docs/qr-verification.md` for the full rule set.
- `VerificationEvent` — append-only log of every public lookup (for analytics + abuse detection), never exposes customer/financial data.
- `ApprovalRecord` — one row per approval/rejection step in the workflow, FK to Certificate and to the approving User, never deleted.

### Finance (Phase 7)
- `ServiceCatalogItem`, `Quotation`, `Invoice`, `Payment`, `CreditNote`, `Expense`

### Documents, Tasks, Integrations, AI (Phase 7–8)
- `Document` (storage pointer + version chain), `Task`, `Notification`, `IntegrationConnection`, `AiAssistantQuery` (logged, never authoritative)

## Key integrity rules enforced at the DB layer, not just application code

1. `(laboratoryId, certificateNumber)` unique — no duplicate certificate numbers within a tenant (§23).
2. `QrIdentifier.certificateId` unique, `QrIdentifier.qrCode` globally unique — one certificate, one QR, forever (§24).
3. `Measurement`, `ChainOfCustodyEvent`, `ApprovalRecord`, `AuditLog`, `VerificationEvent` are insert-only from the application's perspective — no `UPDATE`/`DELETE` grants on these tables for the application's runtime DB role; corrections are new rows with a back-reference.
4. Every tenant-owned table has `laboratoryId NOT NULL` with an FK to `Laboratory`, indexed, and (where Phase 1+ turns on RLS) a row-security policy.
5. `Examination.testingMethodVersionId` is NOT NULL and immutable after insert — the exact method version used is permanent history (§18).

## Phase 1 implementation scope

`apps/api/prisma/schema.prisma` Phase 1 migration includes only: Laboratory,
Branch, User, Role, Permission, RolePermission, UserRole, Plan,
PlanFeature, Subscription, AuditLog. Later groups are added module-by-module
in the phases listed in `docs/roadmap.md`, each phase arriving with its own
migration — never one giant upfront migration for tables with no code using
them yet.
