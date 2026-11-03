# 01 — Security Assessment Plan (SAP extract)

| | |
|---|---|
| **System** | City of Marlowe Housing Assistance Portal (HAP), GovRAMP Moderate |
| **Assessment type** | Targeted control assessment ahead of the interim ATO decision. Focus: controls tied to open POA&M items |
| **Assessor** | Independent Security Control Assessor (SCA), separate from CivicStack engineering and from Internal Audit |
| **Assessment window** | 2026-10-19 → 2026-10-30 |
| **Reference** | NIST SP 800-53A Rev 5 · NIST SP 800-37 Rev 2 (Assess step, task A-3) |

## 1. Why these controls

The interim ATO depends on the Moderate POA&M items (due 2026-11-15) being on track. This SAP selects the controls whose POA&M milestones were scheduled to finish **before** the assessment window. That makes it possible to test whether claimed progress is real. Access-lifecycle controls (AC-2, AC-6) are excluded here because Internal Audit just tested them in [GRC-04](https://github.com/DesmondHinds94/grc-04-internal-audit); this assessment relies on that work and does not repeat it.

| # | Control | Linked POA&M | Milestone claimed complete |
|---|---|---|---|
| 1 | AC-2(3) Disable accounts | POA&M-01 | 35-day privileged inactivity (09-10) |
| 2 | IA-2(1) MFA, privileged | POA&M-01 / -04 | Hardware MFA on console + SSM (08-25) |
| 3 | IA-5(1) Password-based authentication | POA&M-04 | Breached-password screening + 12-char minimum (09-30) |
| 4 | AU-6 Audit record review | POA&M-05 | Detections + 15-minute P1 triage (10-05) |
| 5 | AU-11 Audit record retention | POA&M-05 | 90 days hot + 1 year immutable (09-20) |
| 6 | CM-6 Configuration settings | POA&M-09 | CIS benchmark in IaC (not due until 11-15; tested early at CivicStack's request) |
| 7 | CP-9(1) Backup reliability testing | POA&M-08 | Full restore to isolated environment (09-30) |
| 8 | IR-8 Incident response plan | POA&M-12 | Revised plan with named roles + 24h CMHA notice (09-20) |
| 9 | RA-5 Vulnerability monitoring | POA&M-03 | Authenticated scanning (09-15); CI gate (09-30); 0 critical (10-20) |
| 10 | SC-7(5) Deny by default | POA&M-06 | Default-deny egress (11-01; early) |
| 11 | SC-28(1) Cryptographic protection at rest | POA&M-10 | Field-level SSN tokenization (10-25) |
| 12 | SI-2 Flaw remediation | POA&M-03 | Patch SLA met |

## 2. Methods (800-53A)

| Method | What it means in this assessment | Depth / coverage |
|---|---|---|
| **Examine** | Review policies, configs, logs, tickets, scan output | *Focused* depth: the specific artifacts that show the control operating, not only that it exists |
| **Interview** | Control owners and operators | *Focused*: the owner plus one operator per control |
| **Test** | Exercise the mechanism directly: attempt the login, query the config, run the restore check, sample the nodes | *Focused* depth; *specific* coverage, with sample sizes stated per test |

## 3. Determination rules

- Each control is broken into its **determination statements** (the lettered parts in 800-53A).
- Every determination statement is rated **S** (satisfied) or **O** (other than satisfied).
- A control is **Satisfied** only if every one of its determination statements is S. Any O makes the control **Other Than Satisfied (OTS)** and produces a finding.
- OTS findings are risk-rated with the GRC-03 5×5 scale and either mapped to an **existing** POA&M item (when that item's milestones would fix it) or opened as a **new** POA&M item.

## 4. Rules of engagement

- Testing used the staging tenant for the restore test and read-only production credentials for everything else.
- No production data left the boundary. Evidence screenshots are redacted to show the last 4 SSN digits at most.
- Assessor access was provisioned 10-19 and removed 10-31 (verified).

## 5. Deliverables

1. Assessment workbook: [02-assessment-workbook.md](02-assessment-workbook.md)
2. Results by determination statement: [03-assessment-results.csv](03-assessment-results.csv)
3. Security Assessment Report findings: [04-sar-findings.md](04-sar-findings.md)
4. POA&M updates for the ISSO: [05-poam-updates.csv](05-poam-updates.csv)
