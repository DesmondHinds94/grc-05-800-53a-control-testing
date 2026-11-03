# 02 — Assessment Workbook

Twelve controls, each assessed by **Examine / Interview / Test** against its 800-53A Rev 5 determination statements. Statement text is paraphrased and organization-defined parameters (ODPs) are filled in from the HAP SSP.

**Legend:** **S** = satisfied · **O** = other than satisfied · *E/I/T* = method used

---

## 1. AC-2(3) — Disable Accounts · **SATISFIED**

| Det. | Statement (ODP filled) | Method & object | Result | |
|---|---|---|---|---|
| AC-02(03)(a) | Accounts are disabled within 24h when they expire | *E* IdP lifecycle policy; *T* query of expired accounts | 0 expired-but-enabled | S |
| AC-02(03)(b) | Disabled when no longer associated with a user | *E* GRC-04 WP-03 (relied upon) | Covered by IA F-01 / POA&M-15 | S¹ |
| AC-02(03)(c) | Disabled when in violation of policy | *I* IAM Engineer | Manual on Security Lead request; 1 instance, done in 2h | S |
| AC-02(03)(d) | Privileged accounts disabled after **35 days** of inactivity; non-privileged after **90** | *E* job config; *T* IdP report of last-login > 35d in privileged groups | 0 privileged > 35d; 0 standard > 90d | S |

¹ *The deprovisioning gap is already a High finding from Internal Audit. Reporting it twice would double-count one risk, so this statement is marked S here and the dependency is noted in the SAR.*

## 2. IA-2(1) — MFA to Privileged Accounts · **SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| IA-02(01) | MFA is implemented for access to privileged accounts | *E* IdP policy `priv-phishing-resistant`; *T* sign-in to the AWS console, SSM and HAP admin with the key removed (3 paths × 2 admins) | 6 of 6 attempts denied without FIDO2 key | S |

## 3. IA-5(1) — Password-Based Authentication · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| IA-05(01)(a) | Compromised-password list maintained and checked when passwords are created or updated | *E* IdP breached-password setting; *T* attempted applicant registration with a known-breached password | Rejected ✓ | S |
| IA-05(01)(a) | Same check applied to **existing** passwords (list updated, re-checked) | *I* IAM Engineer; *E* IdP config | **Existing 38k applicant passwords never screened.** The setting applies only when a password is set | **O** |
| IA-05(01)(b) | Passwords transmitted only over protected channels | *T* TLS scan of login endpoints | TLS 1.2/1.3 only | S |
| IA-05(01)(c) | Passwords stored using an approved salted key derivation function | *E* auth service config | bcrypt cost 12 | S |
| IA-05(01)(h) | Composition and complexity: minimum **12 characters** | *T* registration with an 11-char password | Rejected ✓; existing shorter passwords not forced to rotate | S² |

² *NIST SP 800-63B does not require forced rotation. The gap is the missing breach screen on existing passwords, recorded under IA-05(01)(a).*

## 4. AU-6 — Audit Record Review, Analysis, and Reporting · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| AU-06a. | Audit records reviewed **weekly** for indications of inappropriate activity | *E* review tickets, 6 weeks (09-14 → 10-25) | **2 of 6 weeks have no review record** (09-28, 10-12) | **O** |
| AU-06b. | Findings reported to the **CivicStack Security Lead and CMHA ISSO** | *E* weekly summaries | 4 of 4 completed reviews were sent to both | S |
| AU-06c. | Review level adjusted when risk changes | *I* Security Lead | Review frequency raised to daily during INC-7790 | S |
| (supporting) | P1 alerts triaged within **15 minutes** | *T* injected test P1 alert (`hap-admin` sign-in attempt) on 10-22 14:03 | Acknowledged 14:50, **47 minutes** | **O** |

## 5. AU-11 — Audit Record Retention · **SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| AU-11 | Audit records retained **90 days online + 1 year archive** | *E* SIEM retention policy; S3 Object Lock config (compliance mode, 365d); *T* retrieved a record dated 2026-07-28 from the hot tier and an archived CloudTrail object | Both retrieved; deletion attempt on a locked object denied | S |

## 6. CM-6 — Configuration Settings · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| CM-06a. | Configuration settings established and documented using **CIS Ubuntu 22.04 L1** | *E* IaC repo `baseline/cis-l1.yml` | Present, versioned | S |
| CM-06b. | Settings implemented | *T* audit script against **20 of 52** randomly sampled nodes | **5 of 20 nodes fail ≥ 1 setting**. These are older node-group nodes that the new launch template has not replaced | **O** |
| CM-06c. | Deviations identified, documented and approved | *E* deviation register | **No register exists.** 3 intentional deviations (e.g., `nfs` kernel module needed for EFS) are undocumented | **O** |
| CM-06d. | Changes to settings monitored and controlled | *I* DevSecOps Lead | Drift detection not yet deployed (milestone due 2027-02-05) | **O** |

*Hands-on hardening and evidence capture for this control: [GRC-09](https://github.com/DesmondHinds94/grc-09-cis-hardening-cm-2).*

## 7. CP-9(1) — System Backup: Testing for Reliability and Integrity · **SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| CP-09(01) | Backups tested **annually** to verify media reliability and information integrity | *E* restore test record 2026-09-30; *I* SRE; *T* re-ran checksum comparison on a restored table sample in staging | Full restore completed in **6h 40m** (RTO 24h). Point-in-time recovery to 15 minutes before the cut. 200 of 200 sampled row hashes matched | S |

## 8. IR-8 — Incident Response Plan · **SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| IR-08a. | Plan provides a roadmap, structure, metrics, and **named roles** | *E* IR Plan v3.0 (2026-09-20) | Named IC, deputy, comms lead, legal, CMHA liaison | S |
| IR-08a. | Defines reportable incidents and **notification of CMHA within 24h** | *E* §6 severity matrix | Present | S |
| IR-08b. | Distributed to named personnel | *E* acknowledgment records | 14 of 14 acknowledged | S |
| IR-08c. | Updated after testing/incidents | *E* change log | Updated after INC-7790; tabletop pending (IR-3, see [GRC-10](https://github.com/DesmondHinds94/grc-10-ir-tabletop)) | S |
| IR-08e. | Protected from unauthorized disclosure and modification | *E* document repo permissions | Edit restricted to Security Lead + CISO | S |

## 9. RA-5 — Vulnerability Monitoring and Scanning · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| RA-05a. | Scan **monthly** and when new vulnerabilities are reported | *E* scanner schedule + ad-hoc scan after a vendor OpenSSH advisory (10-07) | Monthly + ad hoc ✓ | S |
| RA-05(05) | Privileged (authenticated) access for scanning | *E* scan coverage report | **49 of 52 nodes authenticated (94%)**. 3 nodes missing the agent, the same older node group as in CM-6 | **O** |
| RA-05b. | Tools/techniques that enumerate platforms, flaws and configurations | *E* scanner policy | Network + agent + container image scanning | S |
| RA-05d. | Remediate per **SI-2 SLAs** (Critical 15d, High 30d) | *E* finding age report 10-27 | **2 critical findings open 41 days** (both on the 3 unagented nodes, found by network scan only) | **O** |
| RA-05(02) | Vulnerabilities to scan for updated before each scan | *E* feed update log | Daily | S |
| (POA&M-03) | CI image scan **blocks** the build | *T* pushed a test image with a known critical CVE | Build **passed with a warning**. The gate is in `warn` mode | **O** |

*Independent scan validation of this control: [GRC-06](https://github.com/DesmondHinds94/grc-06-ra-5-vulnerability-scanning).*

## 10. SC-7(5) — Deny by Default, Allow by Exception · **SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| SC-07(05) | Outbound traffic from the app tier denied by default; allowed by exception | *E* egress firewall policy (6 allow-listed FQDNs); *T* from an app pod: `curl https://example.org` and `curl https://<state-IV-api>` | First blocked and logged; second allowed | S |

## 11. SC-28(1) — Protection of Information at Rest: Cryptographic Protection · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| SC-28(01) | Cryptographic mechanisms prevent unauthorized disclosure of **SSN** at rest (field-level, beyond volume encryption) | *E* tokenization service design; *T* read-only query on the primary DB `applicant.ssn` column (n=500 random rows) | New records tokenized ✓. **39% of legacy rows still in cleartext** (migration at 61%) | **O** |
| SC-28(01) | Same, for **all stores** in the boundary | *T* schema scan of every datastore for SSN-pattern columns | **Analytics read replica `hap_reporting` holds a full cleartext `ssn` column** fed by a nightly ETL that predates tokenization. 12 analysts can read it | **O** |

## 12. SI-2 — Flaw Remediation · **OTHER THAN SATISFIED**

| Det. | Statement | Method & object | Result | |
|---|---|---|---|---|
| SI-02a. | Flaws identified, reported, corrected | *E* ticket flow from scanner | Automated ticketing ✓ | S |
| SI-02b. | Updates tested for effectiveness and side effects before install | *E* staging promotion records | ✓ | S |
| SI-02c. | Updates installed within **SLA** (Critical 15d / High 30d / Moderate 90d) | *E* 90-day closure report | High-or-above closed in SLA: **81%** (target 95%). Criticals: 2 overdue (see RA-5) | **O** |
| SI-02d. | Flaw remediation in configuration management | *E* patches delivered through the AMI pipeline | ✓ | S |
