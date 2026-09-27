# RPI Compliance & Security Standards

> **Part of The Operating System** -- the governance layer of The Machine.
> See [OS.md](OS.md) for the full architecture.
> This document defines the standards -- what the rules ARE.

> **Version**: v2.0
> **Created**: January 25, 2026
> **Updated**: February 19, 2026
> **Policy Number**: SEC-001
> **Scope**: Universal - Applies to ALL RPI Projects, Employees, Contractors, and Agents
> **Status**: Active - Enforced
> **Owner**: CEO (Josh D. Millang)
> **Security Officer**: JDM (formal HIPAA Security Officer, designated 2026-07-06; prior COO John Behn offboarded — Josh sole super-admin, GAM-verified 2026-07-06)

---

## Purpose

This document establishes compliance and security standards for RPI's AI-powered platform. RPI handles:
- **Protected Health Information (PHI)** - Medicare, health conditions, providers, claims
- **Personally Identifiable Information (PII)** - Client demographics, SSNs, DOBs
- **Financial Data** - Account balances, transactions, commissions

Clear standards are required to protect clients, meet regulatory requirements (including HIPAA), and maintain trust. This policy applies to all systems and devices that access, store, or transmit PHI, PII, or financial data.

---

## Part 1: Data Classification

### Classification Levels

| Level | Description | Examples | Handling Requirements |
|-------|-------------|----------|----------------------|
| **PUBLIC** | Non-sensitive, can be shared freely | Company name, public marketing | None |
| **INTERNAL** | Business operations, not client data | Agent performance metrics, internal plans | Internal access only |
| **CONFIDENTIAL** | Client business data | Account balances, policy numbers, premium amounts | Need-to-know basis |
| **RESTRICTED** | PHI/PII requiring regulatory protection | SSN, DOB, health conditions, claims history | HIPAA/regulatory controls |

### Classification by Data Type

| Data Type | Classification | Storage Location | Encryption Required |
|-----------|---------------|------------------|---------------------|
| Client Name | CONFIDENTIAL | MATRIX | At rest |
| SSN | RESTRICTED | MATRIX (masked) | At rest + in transit |
| DOB | RESTRICTED | MATRIX | At rest |
| Health Conditions | RESTRICTED | Health DB (future) | At rest + in transit |
| Medicare Claims | RESTRICTED | Blue Button (future) | At rest + in transit |
| Policy Numbers | CONFIDENTIAL | MATRIX | At rest |
| Account Balances | CONFIDENTIAL | MATRIX | At rest |
| Agent Commissions | INTERNAL | MATRIX | At rest |

---

## Part 2: HIPAA Compliance

> **Status**: HIPAA applies to RPI. BAA signed with Google (February 4, 2026). PHI training deployed and enforced.

RPI is both a **Covered Entity** (handles PHI as part of client service -- Medicare data, health conditions, claims) and a **Business Associate** (processes PHI on behalf of clients). This policy establishes requirements for handling PHI to ensure compliance with HIPAA regulations and protect client privacy.

### Determination

- [x] Is RPI a "Covered Entity" under HIPAA? -- **Yes.**
- [x] Is RPI a "Business Associate" of any Covered Entity? -- **Yes.**
- [x] What Business Associate Agreements (BAAs) are required? -- **One per vendor that touches PHI. The full list, with status, is [BAA-REGISTER.md](BAA-REGISTER.md).** Google Workspace BAA signed Feb 4, 2026.
- [x] Does Google Workspace meet HIPAA requirements? -- **Yes.** HIPAA-compliant with BAA in place.
- [x] What training is required? -- **PHI Training deployed.** See Part 10.

### What PHI Is -- and What It Is Not

> **Added 2026-07-26.** Over-classification was costing more than under-classification: staff and
> warriors were treating any client name as PHI and escalating on *sightings* rather than
> *exposures*. This section is the authoritative test, taken from the regulation rather than
> from custom.

#### The regulatory definition (45 CFR 160.103)

**"Individually identifiable health information"** is health information, including demographic
information, that:

1. Is created or received by a health care provider, health plan, employer, or clearinghouse; **and**
2. **Relates to** one of three things -- (a) the past, present, or future **physical or mental health
   or condition** of an individual; (b) the **provision of health care** to an individual; or (c) the
   past, present, or future **payment for the provision of health care** to an individual; **and**
3. Identifies the individual, or there is a reasonable basis to believe it can be used to identify them.

**PHI** is that information in any form or medium. Exclusions: FERPA education records, employment
records held by a covered entity **in its role as employer**, and a person deceased more than 50 years.

**The operative structure: IDENTIFIER + HEALTH INFORMATION, linked.** Both halves. The 18 Safe
Harbor identifiers (45 CFR 164.514(b)(2)(i)) include name, geography below state, dates, phone,
email, SSN, medical record number, **health plan beneficiary number**, account number, and full-face
images -- but an identifier **on its own is not PHI**. A name with no health nexus is CONFIDENTIAL
business data. This is consistent with Part 1, where **Client Name is CONFIDENTIAL, not RESTRICTED**.

#### Tier 1 -- CLINICAL (highest harm; inference counts)

Diagnosis, condition, medication, treatment, provider or specialty, lab result, health intake.

**A clinical detail that lets a reader INFER a condition is PHI even if the condition is never
named.** This is the part most often missed:

| Example | Why it is PHI |
|---|---|
| "Jane Doe, Tamoxifen" | The drug reveals the diagnosis |
| "Jane Doe -- Dr. Smith, Oncologist" | The specialty reveals the diagnosis |
| "Jane Doe, breast cancer" | Condition stated outright |

#### Tier 2 -- COVERAGE + PAYMENT (PHI, but routine)

Plan type, carrier, premium, enrollment status, policy/beneficiary number, claim.

**This is PHI** -- prong 2(c), payment for the provision of health care, squarely covers it, and
"health plan beneficiary number" is an enumerated identifier. **Do not classify it as non-PHI.**

But it is also the substrate of the entire book of business. Handle it need-to-know under the rules
in Part 3 and **keep working**. Encountering it is not an event.

#### Not PHI at all

| Example | Classification | Why |
|---|---|---|
| "Jane Doe" | CONFIDENTIAL (PII) | Identifier with no health nexus |
| "Jane Doe -- invoice #4471" | CONFIDENTIAL | Billing with no health nexus |
| "312 clients enrolled in MAPD" | INTERNAL | Aggregate; not identifiable |
| "Client is on Eliquis" (unnamed, no other identifier) | INTERNAL | Health info; not identifiable |
| Employee's own health record held as employer | Excluded | 160.103 employment-records exclusion |

#### Classification is not escalation

**This is the rule that ends the review marathons.** Determining "this is PHI" means *handle it
correctly* -- storage, masking, logging, minimum necessary. It does **not** mean convene a review.

- **Apply the test yourself and proceed.** It is self-serve. No cross-team adjudication.
- **Escalate actual EXPOSURE** -- PHI that reached a surface or party it should not have
  (see Part 6: Incident Response).
- **A sighting is not an incident. Ambiguity is not an incident.** If you are unsure and nothing
  has been exposed, apply Tier 1 handling and continue working.

### HIPAA Triggers (Active)

| Activity | HIPAA Status | Controls |
|----------|-------------|----------|
| Storing client health conditions | **Active** -- PHI in MATRIX (Google Sheets) | BAA + encryption at rest |
| Blue Button API integration | **Future** -- Requires client consent + security | Design pending |
| AI processing health data | **Active** -- MCP tools access health data | Scoped access, no PHI in logs |
| Sharing health info with carriers | **Active** -- Carrier data exchanges | BAA per carrier (TBD) |

### Controls (Implemented)

1. **Administrative Safeguards**
   - Designated Security Officer: JDM (formal HIPAA designation 2026-07-06; prior COO John Behn offboarded)
   - PHI Training deployed Feb 4, 2026 (see Part 10 for status)
   - Incident response: Report breaches to JDM immediately

2. **Physical Safeguards**
   - Google Workspace handles physical security (SOC 2, ISO 27001)
   - Device policy: No PHI on personal devices outside Google Workspace

3. **Technical Safeguards**
   - Access controls: MATRIX role-based permissions
   - Audit controls: Google Sheets edit history + GAS execution logs
   - Integrity controls: RAPID_API single-source-of-truth for writes
   - Transmission security: Google Workspace TLS encryption in transit
   - PHI masking: SSN shows last 4 only, DOB masked unless task-required
   - No PHI in: logs, error messages, Slack, personal email

---

## Part 3: Data Handling Rules

### Collection

| Rule | Implementation |
|------|----------------|
| **Minimum necessary** | Only collect/access data required for the task. Do not access client records without a business need. |
| **Consent** | Document client consent for data usage |
| **Source verification** | Validate data comes from authorized sources |
| **Minimum disclosure** | Share only the PHI necessary for the recipient's legitimate purpose. Do not disclose to coworkers who do not require it. |

### Storage

| Rule | Implementation |
|------|----------------|
| **Encryption at rest** | Google Workspace provides encryption |
| **Access controls** | MATRIX permissions by role |
| **Retention limits** | Define how long data is kept (TBD) |
| **Backup** | Google Workspace automatic backup |

**Approved PHI Storage Locations:**

| System | Purpose | Security |
|--------|---------|----------|
| Google Workspace (Sheets/Drive/Docs) | Client records, MATRIX | Google BAA, encryption, access controls |
| Gmail (@retireprotected.com) | Business communication | Google BAA, TLS encryption |
| Google Apps Script applications | Internal tools (PRODASHX, etc.) | Organization-only access, Google infrastructure |

**PHI is PROHIBITED in:**
- Personal email accounts
- Personal cloud storage (Dropbox, iCloud, OneDrive, personal Google accounts)
- Messaging platforms (Slack, text messages, WhatsApp)
- Unapproved third-party applications
- Personal devices not managed by RPI
- Physical documents removed from RPI premises

### Processing

| Rule | Implementation |
|------|----------------|
| **Purpose limitation** | Only use data for stated purpose |
| **Audit logging** | Log who accessed what, when |
| **AI processing** | Document what AI sees and does |

### Sharing & Transmission

| Rule | Implementation |
|------|----------------|
| **Need-to-know** | Only share with those who require access |
| **Third-party agreements** | Contracts with vendors handling data |
| **Client authorization** | Get consent before sharing externally |

**Internal transmission:**
- PHI transmitted via Gmail between @retireprotected.com addresses is encrypted in transit (TLS)
- Use Google Confidential Mode for sensitive PHI when additional protection is needed

**External transmission:**
- PHI may only be transmitted to external parties with proper authorization
- Use encrypted methods (secure portal, encrypted email) for external PHI transmission
- Never send PHI to personal email addresses of external parties

**Prohibited transmission methods:**
- Unencrypted email to non-Google recipients
- Text/SMS messages
- Fax (unless encrypted/secure fax service)
- Social media or messaging apps

### Disposal

| Rule | Implementation |
|------|----------------|
| **Secure deletion** | Purge data that's no longer needed |
| **Retention schedule** | Define retention periods by data type |

---

## Part 4: Access Control

### Role-Based Access (MATRIX)

| Role | Access Level | Data Visible |
|------|--------------|--------------|
| **Executive** | Full | All data |
| **Service Team** | Client-focused | Assigned clients, accounts, health |
| **Sales Team** | Pipeline-focused | Prospects, assigned clients |
| **BD Team** | Agent-focused | Agent data, production |
| **AI Agents** | Task-scoped | Only data needed for task |

**Access Control Rules:**
- Access to PHI/PII is granted on a need-to-know basis aligned with job responsibilities
- All users must authenticate via Google Workspace with mandatory 2FA
- Access permissions are reviewed quarterly
- Terminated employees have access revoked same-day

> For current role tables, MDJ instance access assignments, and detailed access matrices, see [POSTURE.md](POSTURE.md).

### Google Workspace OU Semantics

> **Rehomed here 2026-07-26** from `toMachina/docs/warriors/shared/phi-governance.md`, which is
> retired. This is now the canonical home. Compliance-critical for any user-admin, offboarding,
> or email-routing action.

| OU | Meaning | Who goes here |
|----|---------|---------------|
| `/RPI- Archived Users` | FINRA email archiving via Global Relay | **Active LICENSED users only.** Do NOT move non-licensed users here. |
| `/RPI- Offboarded` | Suspended/departed employees | Departed team members. NOT archived to Global Relay. |
| `/RPI- Non-Archived Users` | Active employees NOT under securities email archiving | Non-licensed active staff. |

**A departing employee goes to `/RPI- Offboarded`** -- never `/RPI- Archived Users`, which
requires FINRA licensed-user status.

**Super Admins:** read the live Workspace admin audit (`audit_admin_roles` / Admin SDK), never a
hardcoded list. Super Admin access is not self-grantable. **JDM is sole Super Admin --
GAM-verified 2026-07-06** (prior COO offboarded; see the Security Officer designation at the top
of this document).

---

## Part 5: Audit & Logging

### What Must Be Logged

| Event | Log Contains | Retention |
|-------|--------------|-----------|
| **Data access** | Who, what record, when | 90 days (TBD) |
| **Data modification** | Who, what changed, old/new values | 1 year (TBD) |
| **AI queries** | What was asked, what data returned | 90 days (TBD) |
| **Export/download** | Who, what data, when | 1 year (TBD) |
| **Access failures** | Who, what denied, when | 90 days (TBD) |

**Logging Rules:**
- All access to PHI systems is logged via Google Workspace audit logs
- Logs include: who accessed what, when, and what actions were taken
- Logs are retained per Google Workspace retention settings
- Logs are reviewed as part of quarterly security reviews

> For current logging implementation status by system, see [MONITORING.md](MONITORING.md).

---

## Part 6: Incident Response

### Incident Categories

| Category | Definition | Response Time |
|----------|------------|---------------|
| **CRITICAL** | Data breach, unauthorized access to PHI | Immediate |
| **HIGH** | System outage affecting client service | 4 hours |
| **MEDIUM** | Security vulnerability discovered | 24 hours |
| **LOW** | Policy violation, minor issue | 1 week |

### Notification Timeframes

| Scenario | Who to Notify | Timeline |
|----------|---------------|----------|
| PHI breach | HHS + affected individuals | 60 days |
| PII breach (state laws apply) | State AG + affected individuals | Varies by state (Iowa: 60 days) |
| Security incident (internal) | Leadership | Immediate |

> For the incident response procedure (7-step process), notification details, and reporting instructions, see [OPERATIONS.md](OPERATIONS.md).

---

## Part 7: AI-Specific Considerations

### AI Data Processing Rules

| Rule | Rationale |
|------|-----------|
| **No PHI in prompts by default** | Minimize exposure |
| **Scoped MCP access** | Each AI instance only sees what it needs |
| **No training on client data** | RPI data doesn't train external models |
| **Audit trail for AI decisions** | Know what AI recommended and why |

### MDJ Guardrails (Future)

| Guardrail | Implementation |
|-----------|----------------|
| **Can't share data cross-instance** | MDJ-Sales can't access MDJ-Service data |
| **Can't take actions without approval** | AI recommends, humans decide |
| **Can't access raw PHI** | Only aggregated/anonymized health data |
| **Can't export bulk data** | Rate limits on data access |

### API Contract Standards (Layer 2 — Compile-Time DTOs)

All API routes in `services/api/src/routes/` must use typed DTOs from `@tomachina/core/api-types/` on every `successResponse<T>()` call. This is the compile-time layer of the 3-layer API safety model.

| Rule | Rationale |
|------|-----------|
| **Every `successResponse()` must have a `<DtoType>` generic** | Typed generics document the contract and enable grep-based auditing |
| **DTOs live in `packages/core/src/api-types/`** | Single source of truth — both API and frontend reference the same types |
| **Use `as unknown as DtoType` cast on Firestore data** | Firestore returns `Record<string, unknown>` which requires explicit narrowing |
| **New routes must define DTOs before shipping** | Hookify rule `warn-untyped-api-response` enforces this at code-write time |

**DTO file structure:** 10 files (1 barrel + 1 common + 8 route group files) covering all 54 API routes with 300+ named types.

**3-layer enforcement model:**
- **Layer 1 (code-write):** Hookify `warn-untyped-api-response` warns on bare `successResponse()` calls
- **Layer 2 (compile-time):** DTOs catch shape mismatches at `npm run type-check` (this section)
- **Layer 3 (runtime):** Valibot schemas in `fetchValidated` catch mismatches at runtime (see below)

### API Response Validation (Layer 3)

Critical API consumers in `packages/ui/src/modules/` must use `fetchValidated` (not raw `fetchWithAuth`) for all JSON API calls. `fetchValidated` validates response shapes at runtime using Valibot schemas, catching type mismatches that TypeScript cannot detect across HTTP boundaries.

| Rule | Rationale |
|------|-----------|
| **Use `fetchValidated` for all JSON API calls** | Runtime shape validation prevents render crashes from unexpected API response shapes |
| **Add type parameter on typed state assignments** | `fetchValidated<MyType[]>(url)` ensures `result.data` is correctly typed |
| **Schemas validate critical render fields only** | Shape schemas in `packages/core/src/schemas/` check 5-15 fields per entity with passthrough for the rest |
| **Non-blocking in production** | Validation mismatches warn in dev (`console.warn`), never crash in prod |

**Exception:** Non-JSON endpoints (e.g., HTML roadmap) may use `fetchWithAuth` directly with a comment explaining why.

### Config Registry Pattern

All platform configuration that changes at a business cadence (carrier maps, thresholds, tax brackets, product catalogs) must be stored in the `config_registry` Firestore collection and editable from the Admin Module's Config Registry tab — never hardcoded as the primary source.

| Rule | Rationale |
|------|-----------|
| **Configs live in `config_registry` collection** | Single source of truth, no code deploy to change a carrier name |
| **Hardcoded values are FALLBACK ONLY** | If Firestore is unavailable, code constants provide defaults |
| **All reads via `getConfig(key, fallback)`** | Shared helper with 60s TTL cache, consistent access pattern |
| **PUT validation per config type** | Sliders have min/max, tables require non-empty keys, checklists reject dupes |
| **Seed script for initial population** | `seed-config-registry.ts` — idempotent, manual, one-time after deploy |

**12 configs currently managed:** dedup thresholds, carrier/charter map, status map, carrier aliases, product type map, tax brackets, IRMAA brackets, carrier products, ATLAS stages, content block types, excluded statuses, rate limits.

### AI Disclosure

When AI interacts with clients (if ever):
- [ ] Disclose that AI is being used
- [ ] Provide human escalation path
- [ ] Don't make AI pretend to be human

---

## Part 8: Vendor & Third-Party Standards

### Current Platforms

| Platform | Data Stored | Security Status |
|----------|-------------|-----------------|
| **Google Workspace** | MATRIX, Drive, Gmail | SOC 2, ISO 27001, HIPAA BAA available |
| **GitHub** | Code only (no client data) | SOC 2 |
| **Slack** | Internal comms (no PHI) | SOC 2, HIPAA (Enterprise) |
| **GAS Web Apps** | Processed data | Google security |

### Third-Party Requirements

Before using new vendors that handle client data:
- [ ] Security assessment completed
- [ ] Data handling agreement signed
- [ ] HIPAA BAA (if applicable)
- [ ] Encryption verified
- [ ] Access controls confirmed

---

## Part 9: Disaster Recovery

### Backup Strategy

| Data | Backup Method | Frequency | Retention |
|------|---------------|-----------|-----------|
| **MATRIX** | Google Sheets version history | Automatic | 30 days |
| **Code** | GitHub | Every commit | Unlimited |
| **Documents** | Google Drive | Automatic | Unlimited |
| **GAS Apps** | `clasp push` to GAS | Manual | Versions |

### Recovery Procedures

| Scenario | Recovery Method | RTO Target |
|----------|-----------------|------------|
| Accidental data deletion | Sheets version history | 1 hour |
| Code rollback | Git revert | 15 minutes |
| GAS app rollback | `clasp deploy` to previous version | 30 minutes |
| Account compromise | Google account recovery | 4 hours |
| Full system outage | Contact Google support | 24 hours |

### What's NOT Covered (Gaps)

- [ ] Point-in-time recovery for MATRIX beyond 30 days
- [ ] Cross-region redundancy
- [ ] Offline backups
- [ ] Formal DR testing schedule

---

## Part 10: Training & Awareness

### Required Training

| Role | Training Required | Frequency |
|------|-------------------|-----------|
| All team members | PHI handling | Annual |
| Service/Sales | PHI/PII handling | Annual |
| Technical | Security best practices | Annual |
| Leadership | Incident response | Annual |

**Training Rules:**
- All workforce members must complete PHI handling training before accessing PHI
- Annual refresher training is required
- Training completion is documented via signed acknowledgment

> For training schedule, completion status, and roster tracking, see [OPERATIONS.md](OPERATIONS.md).

---

## Part 11: Compliance Checklists

### Before Launching New Features

- [ ] Data classification reviewed
- [ ] Access controls defined
- [ ] Logging implemented
- [ ] PHI/PII handling documented
- [ ] No hardcoded credentials
- [ ] Error messages don't expose sensitive data

### Before Using New Vendor

- [ ] Security assessment
- [ ] Data handling agreement
- [ ] BAA (if handling PHI)
- [ ] Documented in vendor inventory

> For periodic review schedule (quarterly access audits, annual vendor reviews, etc.), see [MONITORING.md](MONITORING.md).

---

## Part 12: Enforcement

Violations of these standards may result in:
- Additional training requirements
- Restricted system access
- Disciplinary action up to and including termination
- Potential legal liability for willful violations

**There is no penalty for good-faith reporting of incidents.** Report to CEO (JDM).

---

## Appendix A: Definitions

| Term | Definition |
|------|------------|
| **PHI** | Protected Health Information - individually identifiable health information transmitted or maintained in any form |
| **ePHI** | Electronic PHI - PHI in electronic form |
| **PII** | Personally Identifiable Information - data that can identify a person |
| **BAA** | Business Associate Agreement - HIPAA contract with vendors |
| **HIPAA** | Health Insurance Portability and Accountability Act |
| **SOC 2** | Service Organization Control 2 - security audit standard |
| **Encryption at rest** | Data encrypted when stored |
| **Encryption in transit** | Data encrypted when transmitted |
| **Minimum Necessary** | Limiting PHI access and disclosure to the minimum needed for the intended purpose |
| **Covered Entity** | Healthcare providers, health plans, and healthcare clearinghouses subject to HIPAA |
| **Business Associate** | Entity that handles PHI on behalf of a Covered Entity |

---

## Appendix B: Open Questions

1. ~~**HIPAA Status**: Is RPI a Covered Entity or Business Associate?~~ -- **RESOLVED.** Yes to both. BAA signed with Google Feb 4, 2026.
2. **State Laws**: What state privacy laws apply (Iowa, others)? -- Iowa Consumer Data Protection Act (effective Jan 1, 2025)
3. **Retention**: What are legal retention requirements for different data types?
4. **Consent**: What consent language is needed for AI processing of client data?
5. **Breach Notification**: What are our exact notification obligations? -- Iowa: 60 days, HHS: 60 days for PHI
6. ~~**Training**: What formal training is legally required?~~ -- **RESOLVED.** PHI Training deployed, acknowledgment form live.

---

## Appendix C: Related Documents

| Document | Purpose |
|----------|---------|
| [OS.md](OS.md) | The Operating System architecture |
| [POSTURE.md](POSTURE.md) | Current security posture, role tables, access assignments |
| [MONITORING.md](MONITORING.md) | Logging status, periodic review schedules |
| [OPERATIONS.md](OPERATIONS.md) | Incident response procedures, training tracking, policy acknowledgment |
| `reference/compliance/SECURITY_COMPLIANCE.md` | Security framework and audit trail |
| `reference/maintenance/WEEKLY_HEALTH_CHECK.md` | Operational verification |
| `reference/maintenance/PROJECT_AUDIT.md` | Full compliance audit |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v0.1 | Jan 25, 2026 | Initial skeleton |
| v1.0 | Feb 13, 2026 | HIPAA status resolved (BAA signed), PHI training status updated (10/13 complete), version upgraded from draft to active |
| v2.0 | Feb 19, 2026 | Merged from COMPLIANCE_STANDARDS.md + PHI_POLICY.md into OS kernel. Operational content moved to OPERATIONS.md, monitoring content to MONITORING.md, posture details to POSTURE.md. Added enforcement section, transmission security rules, approved storage locations, expanded definitions. |
| v2.1 | Mar 22, 2026 | Added API Contract Standards (Layer 2) section — compile-time DTO enforcement via @tomachina/core/api-types/. 3-layer API safety model documented. |


---

<!-- Landed 2026-06-13 (MEGAZORD, OS governance pass). Cites COMPLIANCE.md spine; does not re-derive the §164.312 matrix or the 2-gate model. -->

## Standards — Multi-Tenant Custody + InfoSec/HIPAA

> **Regulatory weight class:** RPI is a multi-tenant custodian of partner firms' credentials AND their clients' PHI. Every subsection below carries an InfoSec/HIPAA obligation. Laxity in any layer is a reportable incident.

---

### 1. Tenancy Model

RPI operates a three-tier credential topology. Each tier has a distinct Firestore path, a distinct access-control predicate, and a distinct data-sensitivity classification.

| Tier | Firestore Path | Sensitivity |
|---|---|---|
| **RPI Team** | `users/{email}/access_items` | Internal credentials |
| **Partner** | `partner_vault/{tenant}/carriers/{key}` | Partner PHI + carrier credentials |
| **Client** | `clients/{clientId}/access_items` | Client PHI |

**Tenant isolation is a HIPAA technical access-control safeguard under §164.312(a)(1).** Isolation is enforced at the Firestore Rules layer, not the application layer, so it cannot be bypassed by a compromised API route.

#### Rule Helpers (canonical — do not inline these predicates)

```
// Per-partner tenancy check
function isPartnerOfTenant(t) {
  return request.auth.token.tenant == t;
}

// Owner-gate for hub and vault collections
function isOwner() {
  return resource.data.owner_email == request.auth.token.email || isHubAdmin();
}
```

- `isPartnerOfTenant(t)` — gates all `partner_vault/{tenant}/**` reads and writes. A partner session must carry the `tenant` custom claim (set server-side at login via Firebase Auth Admin SDK) and that claim must match the path segment.
- `isOwner()` — gates hub/vault collection documents where a single human owns the record (e.g. `dojo_messages`, approval cards, per-user OAuth tokens). A document is readable/writable only by the email that owns it, or by a verified hub admin.

**Auth domain restriction:** Firebase Auth is locked to `@retireprotected.com`. No account outside that domain can obtain an ID token that satisfies any `isRPIUser()` predicate in `firestore.rules`.

---

### 2. PHI Handling — Permitted Surfaces

PHI is ONLY permitted on BAA-covered surfaces. **What is covered, and by which signed agreement, lives in one place: [BAA-REGISTER.md](BAA-REGISTER.md).** Google Workspace's BAA was signed 2026-02-04. Google Cloud (Firestore, BigQuery, Cloud Run …) is a **separate** Google agreement, signed 2026-09-27 (not before). (Corrected 2026-09-27: this line used to say the Workspace BAA "covers Google Workspace and GCP", which no signed record supported.) **Slack is NOT covered by the BAA.**

| Surface | PHI Permitted | Notes |
|---|---|---|
| Firestore (Native mode) | **Yes** | Primary PHI store. Covered by the Google Cloud BAA (see register). |
| Google Workspace (Drive, Sheets, Docs) | **Yes** | BAA-covered. |
| Cloud Run logs | **No** | Never log PHI — hook-enforced (`block-phi-in-logs`). |
| Slack (any channel or DM) | **No** | Not in BAA. Routing PHI to Slack is a reportable breach. |
| `dojo_messages` / Approval Hub cards | **Treat as PHI surface** | Hub messages may carry client context. Owner-gate applies. Never forward to Slack. |
| Local disk / tmux panes | **No** | Transient processing only; never persist. |

PHI extends to: client names when paired with health information, Medicare IDs, DOB paired with coverage data, and any carrier-account credential that could be used to access a client's health record.

---

### 3. Encryption at Rest

**Standard:** Partner and carrier credentials are NEVER stored in plaintext. PHI fields in Firestore that would be sensitive if the Firestore project were compromised MUST be encrypted before write.

**Cipher module:** `@tomachina/core/crypto/vault-cipher`

```typescript
import { encrypt } from '@tomachina/core/crypto/vault-cipher';

const field: EncryptedField = encrypt(plaintext);
// Returns: { ciphertext, iv, authTag, encrypted_at, key_version }
```

- Algorithm: AES-256-GCM (authenticated encryption — ciphertext integrity is verified on decrypt).
- Key material: `VAULT_ENCRYPTION_KEY` — a base64-encoded 32-byte key stored in **Google Secret Manager only**. Never in env files, never in source, never in Slack.
- Key retrieval (authorized services only):
  ```bash
  gcloud secrets versions access latest --secret=VAULT_ENCRYPTION_KEY
  ```
- The `key_version` field in `EncryptedField` enables future key rotation without re-encrypting in-place — increment the version, re-encrypt on next write.

**HIPAA cite:** AES-256-GCM at rest satisfies §164.312(a)(2)(iv) (Encryption and Decryption) and §164.312(e)(2)(ii) (Encryption of data at rest).

**Enforcement:** Any Write or Edit that stores a carrier credential, OAuth token, or PHI field without routing through `vault-cipher` is a Tier 1 violation. Hook `block-hardcoded-secrets` enforces the secret-in-source dimension; the vault-cipher usage standard covers the plaintext-in-Firestore dimension.

---

### 4. Firestore Rules — Deploy Standard

The Firestore Rules file is a **whole-file last-writer-wins deploy**. Deploying from a stale local copy silently clobbers every match block added by other warriors since the last pull. This is a security incident, not just a merge conflict.

#### Required Procedure (no exceptions)

1. **Fetch the live ruleset** before writing a single line.
   - REST: `GET https://firebaserules.googleapis.com/v1/projects/PROJECT/releases/cloud.firestore` → extract `rulesetName`.
   - `GET https://firebaserules.googleapis.com/v1/{rulesetName}` → `source.files[0].content` is the authoritative current ruleset.
2. **Insert your match block** into the fetched content (before the documents-block closing brace).
3. **POST a new ruleset.** The file name in the payload body MUST be `'firestore.rules'` — not a full path, not a directory-prefixed string.
4. **PATCH the release** to point at the new ruleset:
   ```
   PATCH https://firebaserules.googleapis.com/v1/projects/PROJECT/releases/cloud.firestore
   Body: {
     "release": { "name": "projects/PROJECT/releases/cloud.firestore", "rulesetName": "projects/PROJECT/rulesets/<id>" },
     "updateMask": "rulesetName"
   }
   ```
   The `rulesetName` value is the **bare resource name** `projects/PROJECT/rulesets/<id>` — not a URL, not a full `https://` path.
5. **Auth:** `GOOGLE_APPLICATION_CREDENTIALS=<sa-key> gcloud auth application-default print-access-token`
6. **Land the identical content to `main` in the same session.** `repo == live` is the invariant. A live-only hand-patch that does not land to `main` creates drift and will be silently overwritten on the next deploy.
7. **Verify the rule body**, not just that the match block is present. Confirm the `allow` condition evaluates correctly against a test token before closing the session.

**HIPAA cite:** Correctly scoped Firestore Rules are the technical implementation of §164.312(a)(1) access controls. A clobber event that removes a match block is an unintended PHI access-control regression and must be treated as a potential incident.

---

### 5. In-App OAuth — Per-User Consent Pattern

When a feature requires access to a user's external account (e.g. a carrier portal, a Google service beyond base SSO), use the in-app OAuth consent pattern. Do not pre-provision shared service-account tokens for per-human resources.

**Flow:**

1. A **'Connect'** button initiates an OAuth consent popup scoped to exactly the permissions needed.
2. On successful consent, store the resulting token in an owner-gated Firestore document where the document ID equals `request.auth.token.email`. The Firestore rule enforces that only that email (or a hub admin) can read or write the document.
3. The token never transits Slack, is never written to Cloud Run logs, and is never stored outside Firestore.

**Firebase Compat SDK gotcha (do not repeat this error):**

```typescript
// WRONG — returns null in compat SDK:
const credential = GoogleAuthProvider.credentialFromResult(result);
const token = credential?.accessToken; // null

// CORRECT — read directly from result in compat:
const token = result.credential.accessToken;

// To add scopes to an already-signed-in user:
const result = await reauthenticateWithPopup(currentUser, provider);
```

**HIPAA relevance:** Per-user OAuth tokens that grant access to a system containing PHI (e.g. a carrier portal) are themselves PHI-adjacent credentials. Owner-gating in Firestore is the access-control safeguard; vault-cipher encryption is the encryption-at-rest safeguard. Both apply.

---

### 6. §164.312 Coverage — see the spine matrix (no double-maintenance)

The canonical §164.312 technical-safeguards matrix lives in the **compliance spine** (`COMPLIANCE.md` §3) — it is the SSOT and indexes every control documented across Standards/Posture/Operations. This Standards doc is the *implementation layer*; it does not re-maintain the matrix.

**Where each Standards section maps in:**
- §1 Tenancy model → spine: Access control §164.312(a)(1) + Tenant isolation + Person/entity auth §164.312(d)
- §3 Encryption at rest → spine: Encryption §164.312(a)(2)(iv) + §164.312(e)(2)(ii)
- §2 PHI permitted-surfaces + §4 deploy-from-main → spine: PHI-on-new-surfaces (§4) + Access control
- **Audit controls §164.312(b): VERIFIED 2026-06-13** — `vault_audit` per cred op + immutable `dojo_messages` + `bigquery-stream` write-stream; two caveats (best-effort; per-partner activation via `BQ_STREAM_PARTNERS`) carried in spine §3 + open item §7. Not re-asserted here.
