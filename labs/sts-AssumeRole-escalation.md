# 🔐 STS AssumeRole Privilege Escalation — Security Notes

## 🔎 Exploit Scenario

### 👤 Low Privilege User

A low-privilege identity typically has limited permissions such as read-only access, basic EC2 visibility, or restricted service usage. The user itself does not hold administrative rights.
However, if this identity is allowed to call `sts:AssumeRole`, it can potentially transform into a more privileged role without modifying IAM policies directly.

**Security Perspective:**
Attackers often target low-privilege users because they are easier to compromise and may still provide escalation paths.

---

### 🎭 Over-Trusting Role

An over-trusting role is an IAM role whose **trust policy** allows broader principals than necessary.
Examples of risky configurations include:

* Wildcard principals (`"Principal": "*"`)
* Entire account root trust
* Missing MFA or condition checks

When a role trusts too many identities, it becomes an indirect privilege escalation gateway.

---

### 🧬 STS Session Creation

When `sts:AssumeRole` succeeds, AWS Security Token Service generates temporary credentials:

* AccessKeyId
* SecretAccessKey
* SessionToken

AWS now evaluates permissions based on the **role**, not the original user identity.
This temporary transformation is the moment where privilege elevation occurs.

---

## ⚙️ Console Flow

### 🔄 Switch Role Process

1. User logs into AWS Console using their normal identity.
2. User selects **Switch Role**.
3. AWS internally performs an `AssumeRole` call via STS.
4. A new role session begins.

### 🧾 Role Session Behavior

* Console header changes to `assumed-role/<role-name>`.
* Permissions now come from the role policies.
* The original user identity still exists in audit logs as the session initiator.

---

## 💻 CLI Flow

### 🔑 STS Temporary Credentials

When using CLI, an AssumeRole request generates temporary credentials that are used for API calls.
These credentials are short-lived and automatically expire, reducing long-term exposure.

### 🎭 Role Session Identity

After assuming a role, commands execute under:

```
arn:aws:sts::<account-id>:assumed-role/<role-name>/<session-name>
```

This indicates that the session identity is derived from the role rather than the base IAM user.

---

## 🚨 Security Impact

### ⚡ Temporary Admin Access

Even a restricted user can gain elevated privileges if the role they assume has powerful permissions.

### 🌐 Expanded Blast Radius

If compromised, the attacker’s capabilities expand from user-level access to role-level authority across services and regions.

### 🕵️ Harder to Detect Without Monitoring

AssumeRole events may appear legitimate.
Without proper CloudTrail monitoring, privilege escalation can blend into normal operational activity.

---

## 🔐 Remediation

### ✅ Least Privilege Policies

Limit `sts:AssumeRole` permissions to only required role ARNs. Avoid wildcards such as `"Resource": "*"`.

### 🛡️ Trust Policy Restrictions

Define explicit principals and avoid overly broad trust relationships.
Example defensive practices:

* Restrict by specific IAM users or roles
* Use account-level scoping carefully

### 🔑 MFA Enforcement

Add conditions requiring MFA before role assumption:

```
aws:MultiFactorAuthPresent = true
```

This significantly reduces unauthorized privilege elevation risk.

### 📊 Monitoring AssumeRole Events

Enable CloudTrail and monitor:

* Unexpected role sessions
* Unusual session names
* Role assumptions from low-privileged identities

---

## 🧠 Professional Remediation Statement

Restrict `sts:AssumeRole` permissions to specific role ARNs, enforce MFA conditions within trust policies, and monitor AssumeRole events using CloudTrail. Avoid wildcard principals and implement explicit deny guardrails to reduce privilege escalation risk.

---

## 🧬 Advanced Identity Concept

AssumeRole reflects modern cloud identity architecture:

* **Zero-Trust Identity Model:** Access is temporary and verified continuously.
* **Just-In-Time Privilege Elevation:** Permissions are granted only when needed.
* **Session-Based Authorization:** Temporary identities reduce long-term credential exposure.

Understanding these concepts helps design secure cloud environments aligned with enterprise security practices.
