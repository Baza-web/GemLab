# Development Roadmap

Source of truth for phase sequencing (master spec §48). `PROGRESS.md` at the
repo root tracks live status against this roadmap — update it, don't let it
drift.

| Phase | Scope | Status |
|---|---|---|
| 1 — Foundation | Monorepo, Prisma schema (tenancy/identity), tenant-aware auth, Super Admin shell, GRL tenant seed | **in progress** |
| 2 — Core Laboratory | Sample intake, chain of custody, gemstone/jewellery registration, customer management | not started |
| 3 — Testing Engine | Gemstone examination, advanced gem testing, treatment/origin, diamond/pearl/precious-metal modules, testing methods library, scientific measurement engine | not started |
| 4 — Certificate Engine | Template management, GRL original designs, field mapping editor, live preview, PDF generation, numbering | not started |
| 5 — QR & Security | QR issuance, permanent linkage, public verification site, digital signatures, amendments/revocations | not started |
| 6 — Quality Management | Equipment/calibration, SOPs, audits, CAPA, staff competency | not started |
| 7 — Business Operations | Finance, CRM, customer portal, task automation, reporting | not started |
| 8 — Advanced Features | AI assistant, digital gemstone passport (cross-phase aggregation view), advanced analytics, integrations | not started |
| 9 — Commercial SaaS | Subscription billing, tenant self-onboarding, usage limits, white-label | not started |
| 10 — International Launch | Security audit, performance/load testing, backup/restore testing, penetration testing, UAT, docs, production launch | not started |

## Acceptance scenarios (master spec §49)

Each phase is not "done" until its relevant acceptance scenarios pass with
automated test evidence (not a claim) — see `PROGRESS.md` for current test
coverage per phase. Full scenario list:

1. Create a new laboratory. 2. Register a gemstone. 3. Register jewellery.
4. Assign a sample. 5. Record examination data. 6. Upload photographs.
7. Record instrument readings. 8. Complete technical review.
9. Generate certificate preview. 10. Approve certificate. 11. Issue
certificate. 12. Generate unique QR. 13. Verify certificate publicly.
14. Reprint certificate. 15. Amend certificate. 16. Revoke certificate.
17. Verify historical QR. 18. Cross-tenant isolation. 19. Employee
permissions. 20. Customer portal. 21. Subscription restrictions.
22. Backup restoration.

## Ground rules carried through every phase (master spec §50)

- Never invent scientific findings; never let AI independently certify.
- Never silently modify an issued certificate; never reuse a QR identifier.
- Never claim accreditation without verified accreditation.
- Never deploy an untested migration; never mark a feature complete without
  test evidence.
- Preserve the original GRL certificate designs exactly.
