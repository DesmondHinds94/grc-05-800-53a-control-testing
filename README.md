# GRC-05 — Control Testing the 800-53A Way

> **SC-28(1): A reporting replica holds ~40,000 applicant SSNs in cleartext, outside the scope of the tokenization fix. Rated High; new POAM-18, 30 days.**
> **RA-5 / SI-2: 3 unscanned nodes carry 2 critical vulnerabilities 41 days old; the CI image gate warns but does not block. Rated Moderate.**
> **12 controls, 38 determination statements: 6 Satisfied, 6 Other Than Satisfied → 5 findings, 2 new POA&M items, 1 re-rated.**

`Section 02 // Audit & Evidence` · Part of the [GRC Control Lab Docket](https://github.com/DesmondHinds94/grc-portfolio)

---

## What this lab shows

An independent Security Control Assessor tests whether the POA&M milestones CivicStack reported as complete for the fictional **Marlowe Housing Assistance Portal** actually happened. Each control is broken into its NIST SP 800-53A Rev 5 determination statements. Each statement is assessed by **Examine, Interview or Test** and ruled **S** or **O**. A single O makes the control Other Than Satisfied and feeds the POA&M.

```
SAP (scope + methods)  →  workbook (per determination statement)  →  results.csv
        →  SAR findings (risk-rated)  →  POA&M updates (new / update / re-rate / close)
```

## Results

| Control | Result | Key evidence |
|---|---|---|
| AC-2(3) Disable accounts | ✅ Satisfied | 0 privileged accounts inactive > 35 days |
| IA-2(1) MFA, privileged | ✅ Satisfied | 6 of 6 sign-in attempts without FIDO2 denied |
| IA-5(1) Password-based auth | ❌ OTS | Existing passwords never breach-screened |
| AU-6 Audit review | ❌ OTS | 2 of 6 weekly reviews missing; injected P1 took 47 min |
| AU-11 Retention | ✅ Satisfied | Object Lock delete attempt denied |
| CM-6 Configuration settings | ❌ OTS | 5 of 20 nodes fail CIS L1; no deviation register |
| CP-9(1) Backup testing | ✅ Satisfied | Restore 6h 40m vs 24h RTO; 200/200 hashes match |
| IR-8 IR plan | ✅ Satisfied | Named roles, 24h CMHA notice, 14/14 acknowledged |
| RA-5 Vulnerability scanning | ❌ OTS | 94% authenticated coverage; CI gate in warn mode |
| SC-7(5) Deny by default | ✅ Satisfied | Non-allow-listed egress blocked from app pod |
| SC-28(1) Encryption at rest | ❌ OTS | 39% legacy cleartext + cleartext replica |
| SI-2 Flaw remediation | ❌ OTS | 81% High+ closed in SLA vs 95% |

## What's in this repo

| File | What it is |
|---|---|
| [01-security-assessment-plan.md](01-security-assessment-plan.md) | Why these 12 controls, methods and depth, determination rules, rules of engagement |
| [02-assessment-workbook.md](02-assessment-workbook.md) | Every determination statement with method, object, result, S/O |
| [03-assessment-results.csv](03-assessment-results.csv) | 38 rows, one per determination statement, ready to load into a GRC tool |
| [04-sar-findings.md](04-sar-findings.md) | 5 risk-rated SAR findings with evidence and recommendations; assessor's view for the AO |
| [05-poam-updates.csv](05-poam-updates.csv) | POA&M changes in the same columns as [GRC-02](https://github.com/DesmondHinds94/grc-02-gap-analysis-poam/blob/main/02-poam.csv) |

## How it connects

- POA&M baseline → [GRC-02](https://github.com/DesmondHinds94/grc-02-gap-analysis-poam)
- Relied on Internal Audit for AC-2 / AC-6 → [GRC-04](https://github.com/DesmondHinds94/grc-04-internal-audit)
- RA-5 scan validation → [GRC-06](https://github.com/DesmondHinds94/grc-06-ra-5-vulnerability-scanning)
- CM-6 hardening evidence → [GRC-09](https://github.com/DesmondHinds94/grc-09-cis-hardening-cm-2)

---

*Fictional system. Determination statements are paraphrased from NIST SP 800-53A Rev 5; refer to the publication for authoritative text.*
