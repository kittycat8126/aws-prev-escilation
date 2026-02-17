# 🔗 Role Chaining & Session Policies — Advanced IAM Security Notes

## 📌 Overview

Role chaining occurs when an identity assumes one role and then uses that temporary session to assume another role.
This creates a multi-step identity transformation where permissions are inherited through a chain of temporary sessions.

While role chaining is useful for complex cloud workflows, it can introduce privilege escalation risks if roles are not properly restricted.

---

## 🧬 What is Role Chaining?

### 🔄 Identity Transformation Flow

```
IAM User → Assume Role A → Assume Role B
```

Each AssumeRole action generates a new STS session identity.
AWS evaluates permissions based on the **current assumed role**, not the original identity.

### 🧠 Key Concept

Role chaining does not merge permissions — it **replaces** the identity context at each step.

---

## ⚙️ How Role Chaining Works Internally

1. A user assumes Role A using `sts:AssumeRole`.
2. Temporary credentials are issued.
3. Using those temporary credentials, the session assumes Role B.
4. AWS generates another STS session with Role B’s permissions.

Result:

```
arn:aws:sts::<account>:assumed-role/RoleB/session-name
```

Audit logs still track the original principal through CloudTrail.

---

## 🧾 What Are Session Policies?

Session policies are additional policy documents passed during AssumeRole requests.
They **restrict** what the assumed role session can do, even if the role itself has broader permissions.

### 🎯 Purpose

Session policies act as runtime guardrails.

Example concept:

* Role allows `s3:*`
* Session policy restricts to `s3:GetObject`

Effective permission becomes the **intersection** of both.

---

## 🧠 Policy Intersection Model

Effective Permissions =

```
Role Policies
AND
Session Policies
AND
Permission Boundaries
AND
SCPs
```

Even if a role allows an action, session policies can reduce privileges dynamically.

---

## 🔍 Security Risks of Role Chaining

### 🚨 Hidden Privilege Paths

Attackers may move across multiple roles to reach a high-privilege identity without directly assuming an admin role.

### 🌐 Complex Audit Trails

Multiple chained sessions make it harder to identify the true origin of actions.

### ⏳ Longer Exposure Windows

If session durations are long, attackers maintain elevated access longer.

---

## 🖥️ Console Behavior

Role chaining is often invisible to casual users.

* Switching roles repeatedly creates new sessions.
* Console displays only the current assumed role.
* Previous identity context exists only in audit logs.

---

## 💻 CLI Behavior

Conceptual flow:

1. Assume Role A
2. Export temporary credentials
3. Use those credentials to Assume Role B

Each step generates a new `assumed-role` ARN.

---

## 🚨 Security Impact

### ⚡ Privilege Escalation Through Indirect Paths

A user may not be allowed to assume an admin role directly but can reach it through intermediate roles.

### 🔎 Reduced Visibility Without Monitoring

Without CloudTrail analysis, chained role sessions may appear as normal automation.

### 📈 Increased Blast Radius

Chained roles can combine access paths across services and accounts.

---

## 🔐 Remediation Strategies

### ✅ Restrict Trust Policies

Limit which principals can assume roles. Avoid wildcard principals and unnecessary account-wide trust.

### 🛡️ Control Role Chaining Depth

Use conditions and permission boundaries to prevent excessive role chaining.

### 🔑 Enforce MFA

Require MFA for sensitive role assumptions to reduce unauthorized escalation.

### 📊 Monitor AssumeRole Patterns

Use CloudTrail and GuardDuty to detect:

* Rapid role switching
* Unusual session names
* Role assumptions from unexpected identities

---

## 🧠 Professional Security Statement

Role chaining should be carefully controlled using trust policy restrictions, session policies, and monitoring mechanisms. Organizations should enforce least privilege and apply explicit deny guardrails to prevent indirect privilege escalation through chained role assumptions.

---

## 🌐 Advanced Identity Architecture Concepts

Role chaining reflects modern cloud-native identity principles:

* **Zero Trust:** Every role assumption must be verified.
* **Ephemeral Identity:** Temporary sessions replace permanent credentials.
* **Delegated Authorization:** Access flows through trusted relationships rather than static privileges.

Understanding role chaining is essential for designing secure, scalable cloud identity architectures.
