# Module Implementation Plan

Maps master-spec sections to backend modules (see `docs/architecture.md` §2)
and the roadmap phase each is built in. "Config-gated" means the laboratory
activates only the sub-services it's actually competent/equipped for (§2) —
these are not all-or-nothing modules.

| Module | Spec sections | Phase | Config-gated? |
|---|---|---|---|
| TenancyModule, IdentityModule | 4, 5, 35 | 1 | no (platform core) |
| LaboratoryModule (onboarding) | 6 | 1 | no |
| CustomerModule | 32, 33 | 2 | no |
| SampleIntakeModule | 8, 9 | 2 | no |
| GemstoneModule (incl. digital passport) | 10, 20 | 2–3 | yes, by gemstone category |
| JewelleryModule | 16 | 2–3 | yes |
| TestingModule (methods library + measurements) | 11, 18, 19 | 3 | yes, by method |
| TreatmentModule | 12 | 3 | yes |
| OriginModule | 13 | 3 | yes, origin-opinion capable labs only |
| DiamondModule | 14 | 3 | yes |
| PearlModule | 15 | 3 | yes |
| PreciousMetalModule | 17 | 3 | yes |
| EquipmentModule | 29 | 6 | no, but entries only for owned equipment |
| QualityModule | 30 | 6 | yes, by accreditation scope |
| ValuationModule | 31 | 7 | yes, licensed appraisers only |
| CertificateModule | 21, 22, 23 | 4 | no (core), templates per lab |
| QrVerificationModule | 24, 25, 26 | 5 | no |
| ApprovalModule | 27, 28 | 4–5 | workflow shape configurable |
| FinanceModule | 34 | 7 | yes, currency/tax config |
| DocumentModule | 37 | cross-cutting, introduced phase 2 | no |
| TaskModule (automation/notifications) | 36 | 7 | yes |
| AiAssistantModule | 38 | 8 | yes, opt-in |
| ReportingModule | 39 | 8 | no |
| IntegrationModule | 41 | 8 | yes, per integration |
| SubscriptionModule | 42 | 9 | no (platform core) |
| SuperAdminModule | 4 | 1 | no (platform core) |
| AuditModule | 46 | 1 | no (cross-cutting) |

Internationalization (§40) is not a separate module — it's a cross-cutting
concern (locale, currency, timezone, date/number formatting) baked into
`packages/shared` types and the `LaboratoryModule` config from Phase 1
onward, so later modules don't need retrofitting.
