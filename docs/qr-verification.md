# Permanent QR & Public Verification Architecture

Mandatory core requirement (§24–26). This document is the authoritative rule
set; the Prisma schema and `CertificateModule`/`QrVerificationModule` code
must match it exactly.

## Identity model

```
Certificate (1) ──── (1) QrIdentifier
     |                      |
     | certificateNumber    | qrCode (public, opaque, unguessable)
     | e.g. GRL-2026-000001 | e.g. QR-GRL-000001
                            |
                            v
          https://gemlab.baza-boutique.com/verify/{certificateNumber}
          (or /verify/qr/{qrCode} for scanned lookups)
```

## Hard rules (enforced in schema + service layer, not just UI)

1. A `QrIdentifier` is created **only** at the moment a `Certificate`
   transitions to `ISSUED` — never at preview/draft time. Preview renders use
   a visibly watermarked placeholder QR image that does not resolve on the
   public verification endpoint (checked by a `isPreview` flag the
   verification endpoint rejects).
2. `QrIdentifier.certificateId` has a `UNIQUE` DB constraint — one QR per
   certificate, enforced by the database, not just application logic.
3. `QrIdentifier.qrCode` has a `UNIQUE` DB constraint globally (across all
   tenants) — QR codes are never reused or reassigned, ever, even after
   revocation.
4. Reprinting a certificate re-serves the **same** `QrIdentifier`/PDF; it
   never allocates a new one.
5. A new Certificate (new examination, new report) always gets a brand-new
   `CertificateNumber` and a brand-new `QrIdentifier`, even for the same
   physical Gemstone.
6. Amending a certificate creates a new `CertificateVersion`/successor
   `Certificate`; the **original** QR keeps resolving — it now reports the
   current status (AMENDED/SUPERSEDED/REVOKED) and, where applicable, links
   forward to the superseding certificate. The QR is never deleted or
   redirected to a different gemstone.
7. Revoked and superseded certificates remain publicly verifiable — the
   verification page must clearly show REVOKED/SUPERSEDED, not a 404.
8. The verification server always re-reads the authoritative `Certificate`
   + `QrIdentifier` rows from the database on every lookup. The QR image
   itself (just an encoded URL) proves nothing by itself — it is a pointer,
   not a credential.

## Verification statuses

`VALID | AMENDED | SUPERSEDED | REVOKED | CANCELLED | NOT_FOUND`

## Public endpoint contract (`/api/v1/public/verify/:certificateNumber`)

Returns only:
- certificate number, status, issuing laboratory (public identity/branding)
- gemstone/jewellery public identification summary (variety, weight,
  dimensions, treatment disclosure, as approved for public display)
- approved photograph(s)
- issue date, approval/authorization statement (not personal employee data)

Never returns: customer identity/contact info, pricing/financial data,
internal notes, raw instrument data, employee personal details (§25).

Every lookup (success or not-found) writes a `VerificationEvent` (IP hash,
timestamp, certificate number queried, result) for abuse monitoring —
analytics only, never exposed publicly.

## Digital signature (independent of QR)

Each issued PDF is cryptographically signed at render time (detached
signature + stored hash of the final PDF bytes). The QR answers "does a
certificate with this number exist and what's its status"; the signature
answers "has this exact PDF file been altered since the laboratory issued
it." Both are implemented — neither substitutes for the other (§26).

## White-labeling

`QrIdentifier`/verification URLs are laboratory-brandable (custom
sub-path or domain) while still resolving through the same authoritative
backend record — branding is presentation only, never a second source of
truth.
