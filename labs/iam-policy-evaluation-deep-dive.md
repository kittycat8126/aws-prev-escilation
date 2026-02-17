# 🧠 IAM Policy Evaluation Deep Dive — Authorization Logic & Security Analysis

## 📌 Overview

AWS Identity and Access Management (IAM) does not evaluate a single policy in isolation.
Instead, AWS combines multiple policy layers to determine whether a request is allowed or denied.
Understanding this evaluation logic is critical for both secure architecture design and identifying privilege escalation risks.

This document explains how AWS authorization works internally from a cloud security engineer’s perspective.

---

## ⚙️ The IAM Evaluation Model

When a request is made, AWS evaluates permissions across several layers:

1. **Service Control Policies (SCPs)** – Organization-level guardrails
2. **Permission Boundaries** – Maximum permissions for identities
3. **Session Policies** – Temporary restrictions during STS sessions
4. **Identity-Based Policies** – Policies attached to users, groups, or roles
5. **Resource-Based Policies** – Policies attached directly to resources

Final decision logic:

```
Explicit Deny  >  Allow  >  Default Deny
```

If any policy explicitly denies an action, the request fails immediately.

---

## 🔴 Explicit Deny — The Highest Priority

Explicit deny acts as a security override mechanism.

Example concept:

* Identity policy allows `s3:*`
* Bucket policy denies `s3:DeleteObject`

Result:

```
DeleteObject = DENIED
```

Security teams use explicit deny to enforce strong guardrails around sensitive resources such as IAM roles or production databases.

---

## 🟢 Allow — Only Valid When No Deny Exists

Allow statements grant permissions, but they are not absolute.

AWS evaluates whether:

* The action is allowed by at least one policy
* No higher-precedence policy denies it

This explains why users sometimes receive AccessDenied errors despite having an Allow policy.

---

## ⚫ Default Deny — The Baseline Security Model

AWS follows a default deny model:

If no policy explicitly allows an action, access is denied automatically.

This aligns with Zero-Trust principles where access must be explicitly granted.

---

## 🧬 Policy Intersection & Effective Permissions

Effective permissions are determined by combining policy layers:

```
Effective Permission =
(Identity Policies ∩ Permission Boundaries ∩ Session Policies ∩ SCPs)
− Explicit Denies
```

Even if a role allows an action, a permission boundary or SCP may restrict it.

---

## 🔍 Identity Policies vs Resource Policies

### Identity-Based Policies

Attached to users, groups, or roles.
They define what the identity is allowed to do.

### Resource-Based Policies

Attached to resources such as S3 buckets or KMS keys.
They define who can access the resource.

AWS evaluates both together.

Security Insight:
Resource policies can grant access even when identity policies are minimal, making them a common misconfiguration vector.

---

## 🧾 Real “Allowed but Denied” Scenario

Common confusion:

* Role policy allows `ec2:RunInstances`
* Permission boundary does not include EC2 actions

Result:

```
AccessDenied
```

Reason:
Permission boundaries act as a maximum permission ceiling.

---

## 🚨 Security Risks from Misunderstanding Evaluation Logic

### ⚡ Over-Permissive Policies

Using wildcard actions or resources increases blast radius if credentials are compromised.

### 🔗 Indirect Privilege Escalation

Attackers exploit policy intersections to bypass restrictions using AssumeRole or PassRole paths.

### 🕵️ Hidden Deny Conditions

Implicit denies caused by SCPs or boundaries can create operational confusion and security gaps.

---

## 🔐 Defensive Design Strategies

### ✅ Apply Explicit Deny Guardrails

Use explicit deny for critical actions such as:

* IAM modifications
* Role assumption restrictions
* Resource deletion

### 🛡️ Use Permission Boundaries

Prevent privilege escalation by limiting maximum permissions for developers and automation roles.

### 🔑 Enforce Least Privilege

Avoid wildcard permissions and restrict access to required services and resources only.

### 📊 Monitor Policy Changes

Track IAM changes and AssumeRole events using CloudTrail to detect abnormal behavior.

---

## 🧠 Professional Security Statement

Understanding IAM policy evaluation logic is essential for building secure cloud environments. By leveraging explicit deny guardrails, permission boundaries, and least privilege principles, organizations can minimize privilege escalation risks and maintain strong authorization control across AWS services.

---

## 🌐 Advanced Cloud Identity Concepts

IAM policy evaluation supports modern cloud security architecture:

* **Zero Trust Authorization:** Every request must be explicitly allowed.
* **Session-Based Security:** Temporary credentials reduce persistent access risks.
* **Defense-in-Depth:** Multiple policy layers enforce layered security controls.

Mastering this evaluation model enables engineers to predict authorization outcomes and design resilient cloud identity systems.
