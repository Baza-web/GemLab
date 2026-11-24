# Certificate Engine Architecture

Core product feature (§21–22). Goal: enter examination data once, generate a
visually faithful, laboratory-branded certificate automatically, with zero
duplicate data entry.

## Pipeline

```
Examination/Testing records (already in DB)
        |
        v
Select Certificate Template (laboratory-configured, up to plan limit)
        |
        v
Automatic Field Mapping
   - each template version stores a field map: { fieldKey -> source path }
   - source paths read from Gemstone / Examination / Measurement / Customer
     records — never require re-typing a value already captured upstream
        |
        v
Live Preview (render engine, same renderer used for final PDF, so preview
   === final output, not an approximation)
        |
        v
Technical Review -> Approval (ApprovalModule, §27)
        |
        v
Issue
   - transactional: allocate CertificateNumber + create QrIdentifier
     in the same DB transaction (see docs/qr-verification.md)
        |
        v
Render final PDF -> store in object storage (immutable) -> archive
```

## Template model

- `CertificateTemplate` (laboratory-owned) has many `CertificateTemplateVersion`
  rows. A version carries: background/layout asset reference, field
  positions (x/y/page/font/size), photo slots, QR slot, signature slots,
  page count.
- A **visual field-mapping editor** (apps/web) lets an Owner/Admin upload a
  design, place/resize fields on a canvas over the background image, bind
  each field to a data source path, and save a new version. Previous
  versions are kept, never overwritten.
- **Original GRL designs are the reference implementation**: Sapphire
  Report, Advanced Report, Terms & Limitations / Certificate Cover. These are
  recreated pixel-faithful (logo, colours, typography, layout, QR position,
  signature areas, footer) as the first `CertificateTemplateVersion` rows —
  not redesigned.
- A `Certificate` row always stores which exact `CertificateTemplateVersion`
  it was rendered with. If the laboratory edits the template later, every
  previously issued certificate still renders/archives from its original
  version (§22, §28).

## Rendering

- HTML/CSS templates compiled to PDF (headless Chromium rendering, consistent
  with the pre-installed browser tooling in this environment) so the "live
  preview" the operator sees in the browser and the final archived PDF come
  from the same rendering code path — no separate "preview renderer" that can
  drift from the "real" one.
- Rendering runs as a background job (BullMQ) so large batch issuance or slow
  renders don't block the request thread; the API returns once the job is
  enqueued and the UI polls/subscribes for completion.

## Numbering

- See `docs/database-design.md` and `docs/qr-verification.md` for the
  transactional numbering + QR issuance contract. Numbering format is
  configurable per laboratory/branch/year (e.g. `GRL-2026-000001`).

## Amendment / revocation

- Issued certificates are immutable rows. An amendment/supersession/
  revocation creates a new `CertificateVersion` (or a new `Certificate` with
  a `supersedesCertificateId` back-reference, per §28) with its own
  `ApprovalRecord` trail — the original is never edited in place, and its QR
  keeps resolving (now showing the new status) rather than disappearing.
