# 04 — Security Assessment Report: Findings

**System:** HAP · **Assessment:** 2026-10-19 → 10-30 · **Report date:** 2026-11-02 · **To:** CMHA Authorizing Official, CMHA ISSO, CivicStack CISO

## Summary

| | Count |
|---|---|
| Controls assessed | 12 |
| Determination statements | 38 |
| **Satisfied** | **6 controls**: AC-2(3), IA-2(1), AU-11, CP-9(1), IR-8, SC-7(5) |
| **Other than satisfied** | **6 controls** (IA-5(1), AU-6, CM-6, RA-5, SC-28(1), SI-2) → 5 findings; RA-5 and SI-2 share one |
| POA&M items validated closable | 1 (POA&M-08 restore milestone; the exercise milestone remains) |
| POA&M items opened | 2 (POAM-17, POAM-18) |
| POA&M items re-rated | 1 (POAM-09 Low → Moderate) |

**Assessor's view for the AO:** the claimed milestones held up for identity, logging retention, backups, egress and the IR plan. They did **not** hold up for SSN protection: tokenization covers only part of the primary database, and a reporting replica holds every applicant's SSN in cleartext. That replica was outside the scope of POA&M-10, and fixing it is the precondition for any authorization decision.

---

## SAR-01 — Cleartext SSNs in the reporting replica and 39% of the primary table · **HIGH**

| | |
|---|---|
| **Control** | SC-28(1). Related: AC-6, PT-2 (minimization) |
| **Determination** | SC-28(01): O (primary DB, partial) · O (all stores) |
| **Weakness** | Tokenization (POA&M-10) covers new records, and the legacy migration is 61% complete. The nightly ETL to `hap_reporting` was built before tokenization and copies the raw `ssn` column. **About 40,000 cleartext SSNs** are readable by 12 analysts with no business need for the full number. |
| **Evidence** | Query results (n=500, primary): 196 cleartext. Schema scan output for `hap_reporting.applicant_snapshot.ssn` (redacted screenshot). Role grants export |
| **Likelihood / Impact** | 3 / 5 → **15 (High)**. Maps to risk R-10; inherent score returns to 15 because the residual assumption was wrong |
| **Recommendation** | (1) Within 7 days, drop or null the replica column and replace it with the last 4 digits or a token. (2) Finish the legacy migration. (3) Add the replica to the boundary data-flow diagram and to POA&M-10 scope. (4) Add a CI check that fails any schema migration that introduces an SSN-pattern column outside the tokenization service. |
| **POA&M** | **POAM-18 (new, High, 30 days)**; POA&M-10 milestone extended |

## SAR-02 — Unscanned nodes are carrying overdue criticals; CI gate does not block · **MODERATE**

| | |
|---|---|
| **Control** | RA-5, RA-5(5), SI-2 |
| **Determination** | RA-05(05) O · RA-05d. O · CI gate O · SI-02c. O |
| **Weakness** | 3 of 52 nodes in an older node group have no scan agent. The network scan found 2 critical findings on them, now 41 days old against a 15-day SLA. Remediation of High-or-above within SLA is 81% against a 95% target. The CI image gate warns but does not block. |
| **Likelihood / Impact** | 3 / 4 → **12 (High band, rated Moderate)**. The nodes sit in private subnets with default-deny egress, which reduces exploitability. Rated Moderate on that basis, with the rationale recorded |
| **Recommendation** | Retire the old node group (this also fixes SAR-03). Switch the CI gate to `block` for Critical and High. Report SLA attainment monthly in ConMon. |
| **POA&M** | POA&M-03: add milestones; keep the 11-15 date for the gate, 11-30 for node retirement |

## SAR-03 — Configuration baseline not enforced on 25% of sampled nodes; no deviation register · **MODERATE**

| | |
|---|---|
| **Control** | CM-6 (also CM-2, CM-3) |
| **Determination** | CM-06b. O · CM-06c. O · CM-06d. O |
| **Weakness** | 5 of 20 sampled nodes fail at least one CIS L1 setting. 3 intentional deviations are undocumented. No drift detection. |
| **Likelihood / Impact** | 3 / 3 → **9 (Moderate)** |
| **Recommendation** | Retire the old node group. Create a deviation register with an approver and an expiry date for each entry. Bring the drift-detection milestone forward. |
| **POA&M** | POA&M-09 **re-rated Low → Moderate**; completion pulled in to 2027-01-15 |

## SAR-04 — Weekly log review missed twice; P1 triage took 47 minutes against a 15-minute standard · **MODERATE**

| | |
|---|---|
| **Control** | AU-6 (also SI-4, IR-4) |
| **Determination** | AU-06a. O · P1 triage O |
| **Weakness** | The weekly review is done by one analyst, and no review happened in the weeks that analyst was on call for incidents. An injected P1 alert reached the pager, but the on-call engineer was in a change window and nobody escalated. |
| **Likelihood / Impact** | 3 / 3 → **9 (Moderate)** |
| **Recommendation** | Rotate review duty across two named analysts. Set pager escalation to secondary at 10 minutes and the Security Lead at 20. Re-test with a second injected alert. |
| **POA&M** | **POAM-17 (new, Moderate, 90 days)** |

## SAR-05 — Existing applicant passwords never checked against breach lists · **LOW**

| | |
|---|---|
| **Control** | IA-5(1) |
| **Determination** | IA-05(01)(a) O (existing passwords) |
| **Weakness** | Screening runs only when a password is set. About 38,000 existing passwords have never been screened. Applicant MFA is rolling out under POA&M-04 and reduces the impact. |
| **Likelihood / Impact** | 2 / 3 → **6 (Low)** |
| **Recommendation** | Screen each password at next sign-in (the IdP supports a check-on-login hook) and force a reset on a match. |
| **POA&M** | POA&M-04: add milestone, 2026-11-30 |
