# AWS Privilege Escalation & IAM Security Research

## Overview

This repository documents practical research into AWS Identity and Access Management (IAM) security, focusing on real-world privilege escalation paths, policy evaluation logic, and defensive remediation strategies.
The goal is to understand **how AWS actually makes authorization decisions**, how misconfigurations create attack surfaces, and how security engineers design least-privilege architectures.

This is not a collection of theoretical notes — it is a hands-on security learning log covering attack paths, detection logic, and secure policy design.

---

## IAM Policy Evaluation Logic

AWS evaluates access requests by combining multiple policy layers including identity-based policies, resource-based policies, permission boundaries, session policies, and service control policies (SCPs).

Key Principles:

* Explicit **Deny** overrides any Allow.
* Permissions are calculated as the union of allows minus explicit denies.
* If no allow exists, access is denied by default.

Understanding this evaluation flow is critical for both defensive architecture and identifying privilege escalation risks. Many real-world security incidents occur due to misunderstood policy precedence rather than intentional misconfiguration.

---

## Real Privilege Escalation Scenarios

This repository explores practical escalation paths commonly observed in cloud environments:

* Over-permissive policies (`Action: "*"`, `Resource: "*"`)
* Abuse of `iam:PassRole` with compute services
* Weak trust policies allowing unintended role assumption
* Misconfigured resource policies that bypass identity restrictions

Each scenario is analyzed from both attacker and defender perspectives, including:

* Initial access vector
* Policy weakness
* Escalation mechanism
* Resulting blast radius

The focus is on understanding *why AWS allows the action* instead of just demonstrating the exploit.

---

## Detection Strategies

Security is not only prevention — it is also visibility.

Detection approaches documented here include:

* Monitoring suspicious role assumptions via CloudTrail
* Identifying abnormal privilege usage patterns
* Detecting wildcard permissions during policy reviews
* Analyzing service role attachments that may enable indirect escalation

The emphasis is on building a **security mindset**, where engineers proactively identify risky permission combinations before they are exploited.

---

## Secure Policy Design & Remediation

For every misconfiguration explored, secure alternatives are documented:

* Replacing wildcard actions with scoped permissions
* Restricting `iam:PassRole` using service and resource conditions
* Applying least-privilege principles to identity and resource policies
* Designing roles that minimize blast radius if compromised

Example secure patterns include:

* Role-based access with temporary STS credentials
* Condition keys such as `aws:SourceIp` and MFA requirements
* Explicit deny strategies to protect sensitive resources

---

## Purpose of This Repository

This project reflects a practical journey into cloud security engineering, focusing on:

* Deep IAM understanding
* Real attack simulations
* Defensive architecture thinking
* Interview-level security explanations

The objective is to develop expertise aligned with real-world cloud security roles rather than certification-only knowledge.
