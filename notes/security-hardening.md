# Windows 11 Security Hardening

This note summarizes a controlled Windows 11 security-hardening exercise performed with **CIS-CAT Lite Assessor** and the **CIS Microsoft Windows 11 Enterprise Benchmark — Level 1**.

The purpose of the exercise was to practice a repeatable security-baseline workflow:

1. assess the endpoint;
2. review failed benchmark controls;
3. select appropriate findings for remediation;
4. apply targeted Windows security-policy changes;
5. run a second assessment;
6. verify that the selected controls changed from **Fail** to **Pass**.

The complete CIS-CAT reports are intentionally not included in this repository because raw assessment output exposes unnecessary endpoint metadata and a large amount of configuration detail.

---

## Scope

The lab focused on **selected Windows security-policy findings**, not on achieving complete CIS Benchmark compliance.

The exercise included:

- CIS-CAT Lite assessment;
- Windows 11 Enterprise Benchmark Level 1;
- interpretation of failed controls;
- review of CIS remediation guidance;
- Windows security-policy changes;
- Local Group Policy / security configuration;
- second-scan verification.

No claim is made that the endpoint became fully CIS-compliant.

---

## Hardening Workflow

```text
Initial CIS-CAT assessment
        ↓
Review failed controls
        ↓
Select remediation targets
        ↓
Apply Windows security-policy changes
        ↓
Run second CIS-CAT assessment
        ↓
Verify selected Fail → Pass transitions
```

A remediation was considered validated only when the second assessment reported the relevant control as **Pass**.

---

## Selected Verified Remediations

The following examples were confirmed by comparing the initial and final CIS-CAT reports.

| CIS Control | Initial State | Final State | Security Objective |
|---|---|---|---|
| 1.1.1 Enforce password history | Fail | Pass | Reduce password reuse |
| 1.1.3 Minimum password age | Fail | Pass | Prevent rapid password cycling |
| 1.2.2 Account lockout threshold | Fail | Pass | Limit repeated authentication attempts |
| 1.2.4 Reset account lockout counter after | Fail | Pass | Apply controlled lockout reset behavior |
| 2.3.2.1 Force audit policy subcategory settings to override category settings | Fail | Pass | Ensure granular Windows audit-policy enforcement |

These examples are a curated subset of the remediated findings and are included to demonstrate the assessment-and-validation process rather than reproduce the complete benchmark report.

---

## Example: Account Lockout Threshold

The initial assessment reported the CIS recommendation for the account lockout threshold as **Fail**.

The control requires a defined threshold rather than leaving account lockout disabled.

After the relevant Windows policy was adjusted, the second CIS-CAT assessment reported the same control as **Pass**.

### Security Rationale

A controlled account lockout threshold can reduce the likelihood of successful online password-guessing attacks.

The setting must still be chosen carefully because an excessively aggressive lockout policy can increase accidental lockouts or enable denial-of-service behavior against user accounts.

---

## Example: Password Policy

Selected password-policy controls were also remediated and revalidated.

Examples include:

- enforcing password history;
- configuring a minimum password age.

These settings work together to reduce immediate password reuse and discourage users from rapidly cycling through password changes to return to a previous password.

---

## Example: Audit Policy Enforcement

The exercise also included the Windows security option that forces advanced audit-policy subcategory settings to override broader audit-policy category settings.

The initial assessment reported this control as **Fail**.

After remediation, the final assessment reported it as **Pass**.

This setting supports more precise audit-policy enforcement and helps avoid conflicts between legacy category-level auditing and granular subcategory policies.

---

## Validation Principle

The exercise separated **configuration change** from **successful validation**.

Changing a policy was not treated as proof that the endpoint met the benchmark recommendation.

The relevant control had to be reassessed by CIS-CAT and return:

```text
Pass
```

This provides a clearer troubleshooting and hardening chain:

```text
Finding
→ benchmark guidance
→ targeted configuration change
→ independent re-scan
→ verified result
```

---

## Publication Boundary

The raw `before` and `after` CIS-CAT HTML reports are deliberately excluded from the public portfolio.

This avoids publishing:

- endpoint hostname information;
- laboratory IP addressing;
- full benchmark failure inventories;
- detailed machine configuration;
- unnecessary assessment metadata.

The repository therefore documents the **method and selected verified outcomes** while keeping the original assessment artifacts private.

---

## What This Lab Demonstrates

- Windows endpoint security assessment;
- CIS Benchmark interpretation;
- security-baseline remediation;
- Local Group Policy and Windows security-policy configuration;
- password-policy hardening;
- account-lockout hardening;
- audit-policy configuration;
- before/after validation;
- evidence-driven endpoint administration.

---

## What This Lab Does Not Claim

This exercise does not claim:

- full CIS Benchmark compliance;
- production endpoint certification;
- organization-wide Group Policy deployment;
- Microsoft Intune or Configuration Manager implementation;
- continuous compliance monitoring;
- enterprise-scale security-baseline management.

It should be read as a **controlled Windows endpoint hardening laboratory** demonstrating assessment, remediation and re-validation of selected security controls.

---

## Key Engineering Principle

A security change is not complete when the setting is modified.

It is complete when the intended state is **independently reassessed and verified**.
