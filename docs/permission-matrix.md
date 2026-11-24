# Permission Matrix (Phase 1 model)

Permissions are `module:action` strings (e.g. `certificate:issue`,
`gemstone:examine`, `finance:invoice.create`), grouped into Roles. Roles are
laboratory-owned (a laboratory can customize/add roles) except the platform
`SUPER_ADMIN` role, which is global and cannot be assigned to a tenant user.

A Permission can be additionally scoped by:
- **Module** (e.g. only Gemstone module)
- **Branch** (user restricted to one or more branches)
- **Action** (read / create / update / approve / issue / revoke)
- **Record ownership** (e.g. "own examinations only" for junior roles)

## Built-in roles (seed data, customizable per tenant)

| Role | Typical permissions |
|---|---|
| Laboratory Owner | all modules, all actions, billing |
| Laboratory Admin | all modules except billing/subscription |
| Laboratory Manager | operational modules, approvals, reports |
| Senior Gemologist | gemstone/jewellery examine + review, cannot issue alone |
| Gemologist | gemstone/jewellery examine (own records) |
| Diamond Specialist | diamond module examine + review |
| Jewellery Testing Specialist | jewellery + precious metal examine |
| Precious Metal Analyst | precious metal module examine |
| Laboratory Assistant | sample intake, chain of custody, read-only elsewhere |
| Reception Officer | customer + intake create, no testing access |
| Quality Manager | quality module full, read-only elsewhere |
| Technical Reviewer | review step only, cannot self-approve own examinations |
| Certificate Approval Officer | approval step only |
| Certification Officer | certificate issuance, QR generation |
| Finance Officer | finance module full |
| Customer Service | customer + portal support, no testing/finance |
| Auditor | read-only, cross-module, including audit log |
| Read-Only User | read-only, configurable scope |

## Separation-of-duties rule (hard rule, not configurable away)

The same user cannot be both the **examiner** and the sole **approver** of
the same Certificate when the laboratory's workflow requires independent
review (§27). The `ApprovalModule` enforces this at the service layer
regardless of role permission flags — a permission can grant the *ability*
to approve, but the workflow engine still blocks self-approval on records
the same user authored, unless the laboratory explicitly configures
one-person approval for that certificate type.

## Phase 1 implementation

Phase 1 ships the RBAC schema (`Role`, `Permission`, `RolePermission`,
`UserRole`), the guard/decorator mechanism (`@RequirePermission('module:action')`),
and the seed roles above for the GRL tenant. Per-record/branch-level
restriction and the separation-of-duties enforcement are wired in as the
owning modules (Testing, Certificate, Approval) are built in later phases —
the matrix above is the target shape they are built against.
