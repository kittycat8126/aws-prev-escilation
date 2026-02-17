# 🛡️ IAM Privilege Escalation Playbook — Cloud Security Operations Guide

## 📌 Purpose

This playbook outlines common AWS IAM privilege escalation paths, detection strategies, and remediation steps.
It is designed from a defensive cloud security perspective to help security engineers identify and mitigate risky authorization patterns involving AssumeRole, PassRole, and policy misconfigurations.

The objective is not exploitation, but understanding how escalation occurs so it can be prevented and monitored effectively.

---

## 🧨 Threat Model Overview

Privilege escalation in AWS often occurs when:

* A low-privileged identity gains indirect access to a more powerful role.
* Trust policies are overly permissive.
* Wildcard permissions (`Action:"*"`, `Resource:"*"`) increase blast radius.
* Role assumption paths are not monitored.

Common escalation vectors:

* `sts:AssumeRole`
* `iam:PassRole`
* Overly permissive IAM policies
* Weak resource-based policies

---

## 🔎 Scenario 1 — AssumeRole Privilege Escalation

### Description

A low-privileged user is allowed to assume a high-privileged role through an overly trusting role policy.

### Indicators

* CloudTrail `AssumeRole` events from unexpected identities
* Sudden appearance of `assumed-role/AdminRole` sessions
* Role switching outside normal workflows

### Detection Steps

* Monitor CloudTrail for `AssumeRole`
* Alert on admin role sessions initiated by non-admin users
* Analyze session names and source IP addresses

### Remediation

* Restrict trust policy principals
* Require MFA conditions
* Limit `sts:AssumeRole` to specific role ARNs

---

## 🔎 Scenario 2 — PassRole Indirect Escalation

### Description

A user with `iam:PassRole` and compute permissions launches a service (EC2/Lambda) with a privileged role attached, indirectly gaining elevated access.

### Indicators

* New EC2 instances launched with high-privilege roles
* Unusual role attachments to compute services
* Increased STS activity from service identities

### Detection Steps

* Monitor `RunInstances` events combined with role attachments
* Track IAM role usage anomalies

### Remediation

* Restrict `iam:PassRole` to approved roles only
* Add explicit deny guardrails for sensitive roles
* Implement least privilege policies

---

## 🔎 Scenario 3 — Wildcard Policy Abuse

### Description

Policies containing `Action:"*"` or `Resource:"*"` enable broad access that can be abused after compromise.

### Indicators

* Identities with Administrator-like permissions
* Access to unexpected resources across services

### Detection Steps

* Audit IAM policies for wildcards
* Use Access Analyzer to identify overly permissive permissions

### Remediation

* Replace wildcards with scoped permissions
* Implement permission boundaries

---

## 🧠 IAM Evaluation Awareness

Security engineers must understand:

```
Explicit Deny  >  Allow  >  Default Deny
```

Privilege escalation often exploits misunderstandings of policy precedence rather than explicit policy changes.

---

## 📊 Monitoring & Detection Strategy

### CloudTrail Monitoring

Track events such as:

* `AssumeRole`
* `RunInstances`
* `AttachRolePolicy`
* `PassRole`

### Behavioral Signals

* Role sessions from unusual IP ranges
* Sudden permission changes
* Rapid role switching

### Security Services

* AWS GuardDuty
* IAM Access Analyzer
* CloudWatch alerts

---

## 🔐 Defensive Architecture Principles

### Least Privilege

Grant only necessary permissions.

### Zero Trust Identity

Require MFA and condition-based access.

### Session-Based Authorization

Use temporary credentials instead of static access keys.

### Explicit Deny Guardrails

Prevent assumption of critical roles by default.

---

## 🧾 Incident Response Workflow (High-Level)

1. Identify suspicious AssumeRole or PassRole activity.
2. Verify session identity using CloudTrail logs.
3. Temporarily revoke role trust relationships if needed.
4. Rotate credentials and review affected resources.
5. Update IAM policies to prevent recurrence.

---

## 🧬 Professional Security Statement

Privilege escalation in AWS is rarely caused by a single misconfiguration. It typically emerges from the interaction of identity policies, trust relationships, and temporary session privileges. Continuous monitoring, strict trust policy design, and least privilege enforcement are essential for maintaining secure cloud identity boundaries.

---

## 🌐 Cloud Security Mindset

This playbook aligns with modern cloud architecture principles:

* Zero Trust Identity Model
* Just-In-Time Privilege Elevation
* Defense-in-Depth Authorization

Understanding escalation pathways enables proactive security design rather than reactive incident response.
