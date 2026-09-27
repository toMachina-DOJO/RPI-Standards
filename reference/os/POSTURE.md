# RPI Security Posture

> **Part of The Operating System** — the governance layer of The Machine.
> See [OS.md](OS.md) for the full architecture.
> This document defines the posture — who has access and what's verified.

> **Owner:** CEO (JDM) with operational support from Fractional CTO
> **Version:** 1.0
> **Last Updated:** 2026-03-19 (OS Audit refresh)
> **Review Frequency:** Quarterly

---

## Part 1: Security Statement (For Clients & Partners)

### Standard Security Statement

> "RPI's applications are built on Google Cloud infrastructure, which maintains SOC 2 Type II, ISO 27001, and HIPAA certifications. Access to our systems is restricted to authenticated personnel within our organization. We follow security best practices including mandatory two-factor authentication, credential management protocols, and regular access reviews."

### If Asked About Specific Certifications

| Question | Response |
|----------|----------|
| "Are you SOC 2 certified?" | "Our infrastructure runs on Google Cloud, which is SOC 2 Type II certified. RPI has not pursued independent SOC 2 certification as our architecture delegates infrastructure security to Google." |
| "Are you HIPAA compliant?" | "Yes, we maintain a Business Associate Agreement with Google and operate on HIPAA-eligible infrastructure." |
| "What security measures do you have?" | See Security Controls section below |
| "Can you sign a BAA with us?" | "We can execute a Business Associate Agreement. Our infrastructure is HIPAA-eligible through Google Cloud." |
| "Do you have encryption?" | "Yes - all data is encrypted both in transit (TLS 1.3) and at rest (AES-256). Gmail uses TLS for all transmissions, and Google encrypts all stored data automatically." |
| "Is email encrypted?" | "Yes. Gmail-to-Gmail is always encrypted via TLS. External email uses opportunistic TLS (encrypted when recipient supports it, which most servers do). For sensitive content, we can use Google Confidential Mode for additional restrictions." |

### Security Controls Summary (For Client Conversations)

1. **Authentication:** Google Workspace SSO with mandatory 2FA
2. **Authorization:** Application access restricted to RPI organization members
3. **Encryption:** TLS 1.3 in transit, AES-256 at rest (Google-managed)
4. **Infrastructure:** Google Cloud Platform (SOC 2, ISO 27001, HIPAA certified)
5. **Access Reviews:** Quarterly review of user access
6. **Credential Management:** API keys and secrets stored in secure Script Properties, not in code
7. **Audit Logging:** Google Workspace audit logs retained per Google policy

---

## Part 2: Compliance Checklist

### Google Workspace Security Settings

| Item | Required State | Status | Verified Date |
|------|---------------|--------|---------------|
| 2FA enforced for all users | ON | [x] | 2026-02-13 |
| Less secure app access | OFF | [ ] | |
| Third-party app access | Restricted to approved apps | [x] | 2026-02-13 |
| External sharing (Drive) | Restricted or warn | [ ] | |
| Email authentication (SPF/DKIM/DMARC) | Configured | [x] | 2026-03-19 |
| Mobile device management | Basic or higher | [x] | 2026-02-13 |
| Admin roles | Principle of least privilege | [x] | 2026-02-13 |
| BAA with Google | Signed (if handling PHI) | [x] | Workspace 2026-02-04 · Cloud 2026-09-27 (BAA-REGISTER.md) |

### Application Security

| Item | Required State | Status | Verified Date |
|------|---------------|--------|---------------|
| No hardcoded credentials in code | All secrets in Script Properties | [~] | 2026-02-14 (known gap: CORE_Config.gs syncAllProperties pattern stores plaintext secrets in source — remediation planned) |
| Apps deployed as "Organization only" | All internal apps restricted | [x] | 2026-02-15 (13 GAS web apps: 12 DOMAIN verified, 1 approved exception RAPID_API; see Part 5) |
| No alert()/confirm()/prompt() | Using custom modals | [x] | 2026-02-04 |
| API endpoints authenticated | API key validation where needed | [x] | 2026-02-04 |
| Git repos private | All RPI repos private on GitHub | [x] | 2026-02-04 |

### Access Management

| Item | Required State | Status | Verified Date |
|------|---------------|--------|---------------|
| User access list current | No former employees have access | [x] | 2026-02-13 |
| Admin access limited | Only necessary personnel | [x] | 2026-02-13 |
| Offboarding checklist exists | Documented process | [x] | 2026-02-13 |
| Contractor access reviewed | Time-limited, minimal access | [x] | 2026-02-13 |

### Source Code vs. Deployment Access (CRITICAL)

> **Discovery Date:** 2026-02-14
> **Impact:** Silent security regression on `clasp push`

The GAS editor UI and `appsscript.json` are **two different settings** that can diverge:

| Setting Location | What It Controls | How It's Set |
|-----------------|-----------------|--------------|
| GAS Editor UI (Deploy -> Manage) | The LIVE deployment's access | Manual click in browser |
| `appsscript.json` `"webapp.access"` | What `clasp push` + `clasp deploy` will SET | Source code file |

**The danger:** If someone fixes access in the GAS editor UI but the source file still says `ANYONE_ANONYMOUS`, the next `clasp push --force` + `clasp deploy` silently reverts to public access.

**Rule:** Always fix `appsscript.json` first. Always verify after deploy. Audits must check the source file, not just the GAS editor.

**Projects with known disconnect (as of 2026-02-14):** ~~RIIMO, RPI-Command-Center, DAVID-HUB~~ -- All 3 remediated via "Phase 0: Security hardening" commits. Verified 2026-02-15.

---

## Part 3: Access Control

> Imported from COMPLIANCE_STANDARDS.md. For the underlying data classification standards that govern these access levels, see [STANDARDS.md](STANDARDS.md).

### Role-Based Access (MATRIX)

| Role | Access Level | Data Visible |
|------|--------------|--------------|
| **Executive** | Full | All data |
| **Service Team** | Client-focused | Assigned clients, accounts, health |
| **Sales Team** | Pipeline-focused | Prospects, assigned clients |
| **BD Team** | Agent-focused | Agent data, production |
| **AI Agents** | Task-scoped | Only data needed for task |

### MDJ Instance Access (Future)

| MDJ Instance | Data Scope |
|--------------|------------|
| MDJ-Service-Medicare | Service clients + health data |
| MDJ-Service-Retirement | Service clients + account data |
| MDJ-Sales-Medicare | Prospects + plan data (no health) |
| MDJ-Sales-Retirement | Prospects + account data |
| MDJ-BD | Agents + production (no client PII) |
| MDJ-Executive | All (aggregated, not individual PHI) |

### Super Admin Access

Super Admin locked to **Josh only** — verified live 2026-07-06 (`gam print users query isAdmin=True` → 1 result: Josh@retireprotected.com). Q1 2026 audit reduced 5→2 (Josh + John Behn); John Behn was subsequently offboarded (account `johnbehn.archived@`, suspended, `/RPI- Offboarded`), leaving Josh as sole super-admin.

### Organizational Unit (OU) Structure

| OU | Purpose | Who Belongs Here |
|----|---------|-----------------|
| `/RPI- Archived Users` | FINRA email archiving via Global Relay | Active securities-licensed users (Josh, Nikki, Angelique) |
| `/RPI- Non-Archived Users` | Active employees NOT under securities email archiving | All other active employees |
| `/RPI- Offboarded` | Suspended/departed employees | Former employees, suspended accounts (NOT archived to Global Relay) |

**Critical distinction:** `/RPI- Archived Users` is for FINRA compliance — Global Relay archives all email in this OU. Offboarded users go to `/RPI- Offboarded` so they are NOT included in FINRA archiving. The auto-offboard trigger (`API_Compliance.gs`) moves suspended users to `/RPI- Offboarded` automatically.

### Google Groups (Organizational Email Addresses)

These are the email addresses referenced on the retireprotected.com legal pages (Privacy Policy, Terms of Service, SMS Terms).

| Group Email | Purpose | Owner | External Posting | Join Policy | Created |
|-------------|---------|-------|-----------------|-------------|---------|
| contact@retireprotected.com | General website inquiries and client communications | Josh | Enabled | Public join | 2026-03-08 |
| compliance@retireprotected.com | Privacy rights requests, HIPAA compliance, and security concerns | Josh | Enabled | Invite-only | 2026-03-08 |

**Monitoring:** Periodically verify these groups are receiving mail correctly (see MONITORING.md Section 2.4).

---

## Part 4: HIPAA Status

### BAA Status

**Every vendor, with its status, is in [BAA-REGISTER.md](BAA-REGISTER.md).** That's the only place a BAA counts as on record.

- [x] BAA signed with Google Workspace
- [x] Date signed: February 4, 2026
- [x] Location of signed BAA: Google Admin Console -> Account -> Legal and Compliance
- [x] **Google Cloud BAA** (Firestore, BigQuery, Cloud Run …) signed **2026-09-27**: "Reviewed and accepted on Sep 27, 2026 by Josh@retireprotected.com", Cloud console -> IAM & Admin -> Privacy & Security (project claude-mcp-484718). A separate agreement from the Workspace BAA, and **not accepted before this date**. **BBP2-015** (Blue Button Phase II, toMachina#5265): this line is the durable record that Gate **G0a** (Google Cloud BAA accepted) checks against, and G0a is **met** on it. Source: SHINOB1's report and the Cloud console readback quoted above, both 2026-09-27; the full row, with the before and after screenshots, is in [BAA-REGISTER.md](BAA-REGISTER.md). G0a covers the BAA only. Gate G0c (CMS clearing the "Google Cloud" wording in the Blue Button policy) is separate and **still open**.
- [ ] **Non-Google vendors carrying PHI with no BAA** (2026-09-27 sweep): Anthropic, Twilio, PostGrid, DocuSign, and GoHighLevel (records conflict). SendGrid refuses to sign, so its PHI flow must stop. Actions and owners are in the register.

For PHI handling policies, see [STANDARDS.md](STANDARDS.md).

---

## Part 5: Completed Security Actions (Audit Trail)

### Initial Security Audit (2026-02-04)

#### Infrastructure Security
- [x] Enabled 2FA enforcement for Google Workspace (1-week grace period for existing users)
- [x] Accepted HIPAA Business Associate Amendment with Google
- [x] Verified GitHub repos are private

#### Application Security
- [x] Removed hardcoded credentials from code (CEO-Dashboard, RAPID_API)
- [x] Rotated exposed Slack tokens
- [x] Updated all MCP config files with new tokens
- [x] Added API key authentication to RAPID_API
- [x] Replaced forbidden UI patterns (alert/confirm/prompt)
- [x] Verified PRODASHX has organization-only access

#### Documentation
- [x] Created security compliance documentation
- [x] Added org-only access enforcement to CLAUDE.md
- [x] Added deploy-time security hook for access verification

#### Organization-Only Access Verification (Complete)

| App | Access | Verified Date | Notes |
|-----|--------|---------------|-------|
| PRODASHX | DOMAIN | 2026-02-04 | Already compliant |
| SENTINEL | DOMAIN | 2026-02-13 | All 21 deploys DOMAIN (19 stale ANYONE deploys updated to v381) |
| SENTINEL v2 | DOMAIN | 2026-02-13 | All 2 deploys already DOMAIN -- clean |
| DEX | DOMAIN | 2026-02-13 | All 20 deploys DOMAIN (19 stale ANYONE deploys updated to v64) |
| RIIMO | DOMAIN | 2026-02-15 | Fixed -- "Phase 0: Security hardening" commit `85854e0`. Source file + deployment verified. |
| CAM | DOMAIN | 2026-02-13 | Fixed from ANYONE -> DOMAIN (v51) |
| CEO-Dashboard | DOMAIN | 2026-02-13 | Fixed from ANYONE_ANONYMOUS -> DOMAIN (v32). Was CRITICAL -- no auth required. |
| C3 | DOMAIN | 2026-02-13 | Already compliant (v127) |
| RAPID_API | ANYONE_ANONYMOUS | 2026-02-14 | Intentional for SPARK webhook reception -- approved exception. Document rationale. |
| RAPID_COMMS | N/A (library) | 2026-02-19 | Standalone GAS library -- no web app deployment. executionApi DOMAIN only. Twilio SHAKEN/STIR approved. A2P 10DLC campaign pending. Toll-free SMS verification pending. |
| RAPID_IMPORT | DOMAIN | 2026-02-14 | Verified -- appsscript.json has `"access": "DOMAIN"` in both webapp and executionApi |
| RPI-Command-Center | DOMAIN | 2026-02-15 | Fixed -- "Phase 0: Security hardening" commit `d265ac5`. Source file + deployment verified. |
| QUE-Medicare | DOMAIN | 2026-02-15 | Verified -- appsscript.json `"access": "DOMAIN"` confirmed. |
| DAVID-HUB | DOMAIN | 2026-02-15 | Fixed -- "Phase 0: Security hardening" commit `fcc5011`. Source file + deployment verified. |

### Supply Chain + Code Security (2026-03-22)

#### Automated CI Security
- [x] *(REVISED 2026-05-12 per ZRD-UNINVITE-DESTRUCTABOT-001)* CVE scanning via own `cve-scan.yml` workflow — `npm audit --production --audit-level=high` weekly Monday 13:00 UTC + `workflow_dispatch` for ad-hoc; opens GitHub Issue + fails workflow on high/critical findings, silent on clean. *Replaces Dependabot, which was uninvited after surfacing doctrine drift: prior POSTURE claim of "Dependabot CVE scanning" was fictional — `dependabot_security_updates` was disabled at GitHub Settings level; only the weekly minor-and-patch update PRs ran, closing 9 of 10 unmerged. Own scanner = real coverage we control.*
- [x] Enabled CodeQL static analysis for JavaScript/TypeScript (weekly Sunday — GitHub-hosted, off PR gate per JDM-CI-CODEQL-OFFPR-001 2026-06-25)
- [x] Removed Cursor Bugbot GitHub App (unauthorized third-party code reviewer)
- [x] Verified only authorized GitHub Apps remain: Claude, Firebase App Hosting

#### E2E Test Coverage (2026-03-22)
- [x] Vitest backend pipeline tests: 4 intake wire tests (ACF_UPLOAD, MAIL, SPC_INTAKE, ACF_SCAN)
- [x] Playwright UI visual tests: 10 module tests (Contacts, Accounts, Comms, Connect, Admin, MyRPI, FORGE, Pipeline Studio, Quick Intake, Sidebar+Header)
- [x] CI jobs: e2e-ui (pre-deploy) + e2e-intake (post-deploy)
- [ ] CI auth tuning: Firebase REST + Workload Identity — pending (jobs run but fail on auth, non-blocking)
- Coverage: 4 pipeline paths + 10 UI modules verified automatically. ~50 items remain manual-only (subjective UX, new features not yet covered).

### Public Website Pages (retireprotected.com)

Published legal/compliance pages — public-facing, no authentication required. Built from existing RPI compliance docs (Client Guide, PII/PHI Data Protection Policy, Security Compliance Framework).

| Page | URL | WP Page ID | Published |
|------|-----|------------|-----------|
| Privacy Policy | https://retireprotected.com/privacy-policy/ | 639 | 2026-03-08 |
| Terms of Service | https://retireprotected.com/terms-of-service/ | 640 | 2026-03-08 |
| SMS Terms & Conditions | https://retireprotected.com/sms-terms/ | 641 | 2026-03-08 |

**Purpose:** Required for A2P 10DLC campaign registration (CTA verification), toll-free SMS compliance, and general regulatory compliance. These URLs are referenced in the Twilio A2P campaign submission.

**Monitoring:** Periodic availability checks (see MONITORING.md).

### Twilio Compliance Posture (2026-02-19)

| Registration | ID | Status | Date |
|-------------|-----|--------|------|
| SHAKEN/STIR (Voice) | BU7eec3e064ae2c8c5df6e62adca27610e | **Approved** | 2026-02-16 |
| A2P Brand | BN1dab2dca942aaf25f01a72acf2457d11 | **Approved** | 2026-02-19 |
| A2P Campaign (10DLC) | CM62449e61cacd4e8dadfdd0472705b101 | **REJECTED** — CTA verification failed, resubmission pending with new URLs | 2026-03-08 |
| Toll-Free SMS Verification | HH4ac6f9e2b149ebf1ece6bd250f1ad957 | **Pending review** | 2026-02-19 |
| Messaging Service | MGf7c81d2233fdca246b45bc63079a6e55 | Active | 2026-02-19 |
| Customer Profile | BU3d7b6cc9f05d41fee354c6af90a003b8 | Twilio-approved | 2026-02-16 |

**Phone Numbers:**
| Number | Type | Voice | SMS |
|--------|------|-------|-----|
| +18886208587 | Toll-free | LIVE (SHAKEN/STIR) | Pending (toll-free verification) |
| +15155002308 | Local (515) | Available | Pending (A2P 10DLC) |

**Check status:** Tell Claude "check A2P status" or "check SMS status" in any session.

### Q1 2026 Compliance Audit (2026-02-13)

#### Token Revocations (26 total)

| User | Tokens Revoked | Details |
|------|---------------|---------|
| Josh | 3 | cloudHQ (full Gmail+Drive), Adobe Acrobat (admin scope), Zoominfo (gmail.readonly) |
| Christa | 17 | Full deprovision -- all tokens revoked |
| Allison | 2 | Microsoft + Chrome -- already suspended, tokens cleaned |
| Nikki | 1 | Adobe Acrobat (admin.directory.user.readonly) |

#### User Suspensions (2 new)

| User | Action |
|------|--------|
| rpifax@ | Suspended -- 382+ days since last login |
| christa@ | Suspended + fully deprovisioned (17 tokens revoked) |

#### OU Corrections (2)

| User | From -> To | Reason |
|------|-----------|--------|
| nikki@ | Non-Archived -> Archived | Securities/FINRA email archiving compliance |
| talan@ | / (root) -> Non-Archived | Proper OU placement |

### OU Structure Fix (2026-03-02)

**Issue:** `runAutoOffboard_()` was moving suspended users to `/RPI- Archived Users` (FINRA compliance OU), mixing departed employees with active securities-licensed users whose email must be archived by Global Relay.

**Fix:** Created `/RPI- Offboarded` OU. Updated auto-offboard code to use new OU. Moved 5 suspended users:

| User | From | To | Reason |
|------|------|----|--------|
| alex@ | /RPI- Archived Users | /RPI- Offboarded | Departed — should not be in FINRA archiving OU |
| jmdconsulting@ | /RPI- Archived Users | /RPI- Offboarded | Departed — should not be in FINRA archiving OU |
| allison@ | /RPI- Non-Archived Users | /RPI- Offboarded | Suspended — proper offboarded placement |
| rpifax@ | /RPI- Non-Archived Users | /RPI- Offboarded | Suspended — proper offboarded placement |
| christa@ | / (root) | /RPI- Offboarded | Suspended — proper offboarded placement |

#### Super Admin Downgrades (3)

| User | Before -> After |
|------|----------------|
| shane@ | Super Admin -> Delegated Admin (Billing, Groups, Storage, Help Desk) |
| matt@ | Super Admin -> No admin roles |
| jmdconsulting@ | Super Admin -> No admin roles |

**Result: Super Admins reduced 5 -> 2 (Josh + John Behn only)**

#### 2FA Status
- Enforced org-wide: 19/19 users
- Enrolled: 5/19 (Josh, Angelique, Christa, Robert, Vince)
- Grace period set in Admin Console for remaining users

#### Automation Built
- [x] Created `API_Compliance.gs` in RAPID_API -- automated quarterly audit with Slack posting
- [x] Added `delete_mobile_device` tool to rpi-workspace-mcp (10 -> 11 admin tools)
- [x] Quarterly/weekly/monthly triggers for continuous monitoring

---

## Part 6: Remaining Action Items

### Immediate (Post Q1 Audit)
- [ ] Delete 13 stale mobile devices (requires MCP restart for `delete_mobile_device` tool)
- [ ] JDM: Run `SETUP_ComplianceTrigger` in RAPID_API -> API_Compliance.gs (authorizes Admin SDK + creates trigger)
- [ ] JDM: Set SLACK_BOT_TOKEN and SLACK_CHANNEL_ADMIN in RAPID_API Script Properties (for audit Slack notifications)

### This Month (Priority 3)
- [ ] Brief team on 2FA enrollment requirement (grace period active)
- [ ] Review Josh's ~45 SSO-only app tokens for cleanup (low risk, clutter reduction)
- [ ] Optionally enable weekly + monthly monitors: `SETUP_WeeklyStaleUserMonitor`, `SETUP_MonthlyTokenHygiene`

---

## Cross-References (Content Moved to Other OS Documents)

- **Scheduled security tasks and cadences** (automated triggers, manual quarterly/annual tasks): See [MONITORING.md](MONITORING.md)
- **Offboarding checklist** (employee/contractor departure procedure): See [OPERATIONS.md](OPERATIONS.md)
- **Incident response procedure** (contain, assess, notify, remediate, document): See [OPERATIONS.md](OPERATIONS.md)
- **PHI handling policies** (storage, logging, display, breach notification rules): See [STANDARDS.md](STANDARDS.md)

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-02-04 | Initial security compliance document created | Claude Code |
| 2026-02-04 | 2FA enforcement enabled, HIPAA BAA accepted | JDM + Claude Code |
| 2026-02-04 | Merged IMMEDIATE_ACTIONS.md into compliance doc | Claude Code |
| 2026-02-13 | 8 of 13 GAS apps verified org-only. Fixed DEX, CAM, CEO-Dashboard. | Claude Code |
| 2026-02-13 | Stale deployment cleanup: 38 old ANYONE deploys updated to DOMAIN. | Claude Code |
| 2026-02-13 | Q1 2026 audit: 26 tokens revoked, 2 users suspended, 3 Super Admins downgraded, 2 OU moves | Claude Code |
| 2026-02-13 | Part 2 checklist updated. Automated triggers added. Completed/remaining actions expanded. | Claude Code |
| 2026-02-13 | API_Compliance.gs added to RAPID_API -- automated quarterly/weekly/monthly compliance | Claude Code |
| 2026-02-14 | Corrected app count (13 identified, 8 verified). Fixed RIIMO false-positive. Added source-code-vs-deployment section. | Claude Code |
| 2026-02-15 | All 13 apps verified DOMAIN (12 compliant + 1 approved exception). RIIMO, RPI-Command-Center, DAVID-HUB remediated. QUE-Medicare verified. | Claude Code |
| 2026-02-19 | Evolved from SECURITY_COMPLIANCE.md + imported access control from COMPLIANCE_STANDARDS.md | Claude Code |
| 2026-03-02 | Created /RPI- Offboarded OU. Fixed auto-offboard to use correct OU. Moved 5 suspended users. Documented OU structure. | Claude Code |
| 2026-03-08 | Added Public Website Pages section (Privacy Policy, ToS, SMS Terms). Updated A2P campaign status to REJECTED — CTA verification failed. | Claude Code |
| 2026-03-08 | Added Google Groups section (contact@ and compliance@) — referenced on retireprotected.com legal pages. | Claude Code |

---

*This document should be reviewed quarterly and updated as RPI's security posture evolves.*


---

<!-- Landed 2026-06-13 (MEGAZORD, OS governance pass). Cites COMPLIANCE.md spine; does not re-derive the §164.312 matrix or the 2-gate model. -->

## 1. New Firestore collections + access posture (verification table additions)

| Collection | Rule posture | Notes |
|---|---|---|
| `dojo_threads` | owner-gated: read `owner_email==token.email \|\| isHubAdmin()`; create owner-only; update/delete `false` | Hub comms backbone (thread list + previews). Doc-id `{sanitized_owner_email}__{warrior_id}`. |
| `dojo_messages` | owner-gated: read owner-or-hubadmin; create owner-only + `from=='me'`; update/delete `false` | The conversation log. Ringer reads via Admin SDK (bypass). `delivered:true` = ✓✓ receipt. |
| `approval_requests` | read owner-or-hubadmin; create `isRPIUser()` + status `pending` + `owner_email` is string + `!decided_by`; update owner-decides + `final_text` | Multi-tenant Approval Hub cards. |
| `approval_secrets` | create owner-gated; **read `false`** | Server-only (Admin SDK). Secret values never client-readable — held through both rule clobbers this week. |
| `partner_vault/{tenant}/**` | **no explicit match → deny-by-default via `{document=**} → if false` catch-all** | Encrypted partner carrier creds. Server-only (Admin SDK). Verified secure (the "partner_vault" string in the ruleset is only a comment; the catch-all is the real guard). |
| `chat_oauth` | **RETIRED 2026-06-13** | gchat tossed by JDM. Token revoked + doc deleted; owner-only rule remains DORMANT/harmless in live ruleset — candidate for removal on the next rules pass. |

**3-tenancy vault map (encode as the canonical credential topology):**
- RPI team → `users/{email}/access_items`
- Partner → `partner_vault/{tenant}/carriers/{key}`
- Client → `clients/{clientId}/access_items`

Live ruleset: **fc981d23** (post-`chat_oauth`, PR #1842). `isHubAdmin()` / `isPartnerOfTenant(t)` helpers present.

## 2. PITR — new posture REQUIREMENT

**Point-in-time recovery MUST be ON for every collection holding conversation history or PII** (`dojo_messages`, `clients`, `approval_requests`, etc.).
- Currently: **ON** (verified by use — recovered 37 deleted messages 2026-06-13).
- Verification line: confirm PITR enabled in Firestore settings; treat OFF as a Sev-1 posture gap for any PII collection.
- Rationale: it is the difference between "recovered in 2 min" and "permanent loss" on a bad bulk op. (See the delete-by-flag incident → Immune System pass.)

## 3. IAM standing posture (NEW — flagged for review)

- **`mdj-agent@claude-mcp-484718` holds `roles/iam.projectIamAdmin` STANDING.** This is the root enabler of warrior IAM self-grants. **Flagged for scope-down review** — not minimal-privilege.
- **IAM self-escalation log pattern (encode as standard):** any self-grant is logged with `grant_ts | drop_ts | held | sa | role | reason | created_in_window | dropped_verified | outcome` (the `SECURITY_INCIDENTS.md` schema). Self-grants are grant→use→**drop**, narrowest scope, owner-routed if broader scope is needed.
- Reference event: WIF pool-admin self-grant `2026-06-13T04:34:05Z` → drop `04:35:23Z` (78s held), routed to owner when SA bindings needed `serviceAccountAdmin` (broader than minimal). Zero broader escalation taken.

## 4. Rules deploy posture

- Live ruleset = **fc981d23**; `firestore.rules` is **deploy-from-main-only** going forward (no live-only hand-patches; fetch-live→merge→deploy + land-to-main same session; verify rule BODY not just block-presence).
- Drift-gate + deploy-on-merge CI: **staged + signed, HELD pending the WIF owner-bootstrap** (SHINOB1's open item). Mark "staged, pending owner-bootstrap" — do not claim live.

## 5. InfoSec + HIPAA pillar (first-class LENS over this Posture section)

**Why now:** RPI is now a **multi-tenant custodian of partner firms' credentials AND their clients' PHI**. That is an elevated regulatory weight class — HIPAA is the spine of this section, not a footnote.

**BAA chain (document + verify each link):**
- `RPI → Google`: **two agreements.** The Workspace BAA was signed **2026-02-04**. The Google Cloud BAA (Firestore, STT, Vertex, BQ …) was accepted **2026-09-27** and **not before**, so PHI held in GCP before that date sat outside a Cloud BAA. *Corrected 2026-09-27: this line used to credit the Feb date with covering GCP too. No record supported that.* The Gemini API is **not** covered by either agreement. The record for both is [BAA-REGISTER.md](BAA-REGISTER.md).
- `Partner → RPI`: **NEW link, must be papered.** As custodian of a partner's clients' PHI, RPI is a Business Associate of the partner (or the partner is a covered client under our BAA umbrella). **Flag as REQUIRED-verify before any partner's PHI lands** — JDM/legal item, not assert-done.

**Tenant isolation = a DOCUMENTED HIPAA technical safeguard** (write it down as such, not just as a rule):
- Owner-gating (`owner_email==token.email`) + `isPartnerOfTenant(t)` custom-claim + per-tenant vault paths (`partner_vault/{tenant}/**`) are the *access-control safeguard* (§164.312(a)). One partner can never read another's creds/PHI — enforced at the rules layer + verified (partner_vault deny-by-default; approval_secrets read=false).

**Encryption / access / audit — controls written down (HIPAA §164.312):**
- **At rest:** partner creds AES-256-GCM via `vault-cipher` (`VAULT_ENCRYPTION_KEY` in GSM); PHI in Firestore (Google-encrypted, BAA-covered). No plaintext creds anywhere (the 2,584-pw bulk load was encrypted + shredded).
- **Access control:** Firebase Auth `@retireprotected.com` domain restriction + per-tenant claims + owner-gating.
- **Audit:** `vault_audit` writes on every cred op; `dojo_messages` is an immutable conversation audit log (update/delete `false`). Encode "audit trail is queryable + immutable" as a control.
- **Availability/integrity:** PITR ON (§2 above) = the recoverability control.

**PHI handling EXTENDED to the new surfaces:** the global PHI rules (Workspace/Firestore only, never Slack, never logs) now explicitly cover the **hub (dojo_messages) + the vault + Approval Hub cards**. Hub DMs are BAA-covered (Firestore); Slack is NOT in the BAA → never route PHI to a Slack mirror.

**Breach response (Operations cross-ref):** the incident-disclosure standard (straight + immediate + recovery-proof) + the PITR recovery path = the documented breach/loss-response posture. The 37-message recovery 2026-06-13 is the worked example.

---
*Recipes/paths on request. The HIPAA lens also threads the Standards draft (encryption standard, access-control rules, PHI-handling extension, breach-response procedure) + Operations (breach-response runbook) — both follow on your signal, steps 3–4.* — 🏯 MEGAZORD

---

## Money-Safety Account Model — VOLTRON Case-Seat Isolation (2026-07-07)

> Established by MEGAZORD + SHINOB1 to close the case-seat ghost-injection class (an unattributable
> tmux-injected $25K execute command, defused). Gate the ACTION, not the input. Full design:
> `!MEGAZORD DOCS!/RPI-Data-Vault/approval-hub-token-contract-v1.md` (v1.1).

**The principle:** tmux injection on a one-user box is structurally unattributable, so a case seat's
money/execution surface is deny-by-default (SHINOB1 gate OB1-CASESUB-EXECUTE-GATE-001, fail-closed, LIVE).
A move is released ONLY by a valid `approval_sig` the seat CANNOT mint — validity rests on the SIGNATURE,
never a Firestore status field (the seat SA has coarse Firestore write and could forge a status).

**Identities:**
| Identity | Role | Why |
|---|---|---|
| `voltron-case-seat@claude-mcp-484718.iam` | **`roles/datastore.user` ONLY** (verified) — NO secretmanager, editor, KMS, projectIamAdmin, or signer-impersonation | Case seats run on THIS (via `launch-voltron-case.sh`, SHINOB1 lane). Provably zero path to read OR self-grant to the signing key. Coarse Firestore RW is safe because the sig, not status, is the trust root. |
| `mdj-agent@…` (shared warrior/infra SA) | Broad (editor + secretmanager.admin + projectIamAdmin) | The OLD case-seat SA + the reason the boundary was broken — an over-privileged seat could read the key AND self-grant. Case seats MUST migrate OFF it. Kept for general warrior/infra use (NOT case seats). |
| Approval-Hub signer identity (TBD — co-scope) | `roles/cloudkms.signerVerifier` on the key ONLY | Mints `approval_sig` after JDM's biometric. Must be a DEDICATED Hub-backend identity — NOT mdj-agent (broad) and NOT the seat. |

**Signing key:** KMS asymmetric-sign key `projects/claude-mcp-484718/locations/us-central1/keyRings/approval-hub/cryptoKeys/action-approval-signer` (EC_SIGN_P256_SHA256, v1 ENABLED). Private key **never leaves KMS** (not exfiltratable even by the Hub). Hook verifies with the PUBLIC key (local, non-secret). Key IAM policy = EMPTY at creation → seat SA has zero access (boundary probe passes).

**Boundary probe (SHINOB1 self-test #6):** as `voltron-case-seat@`, attempt to access/sign with the key → assert PERMISSION_DENIED. Pre-verified at creation: seat SA has no project-level KMS/secret/editor/iam role + no binding on the key.

**Open (co-scope):** the dedicated Hub-signer identity + its `signerVerifier` binding; the seat creds mechanism (key file vs WIF — recommend key file now, WIF as hardening; even a stolen seat key is datastore.user-only and cannot sign).
