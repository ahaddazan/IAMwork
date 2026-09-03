# 🛡️ Sopilot.site — Microsoft Entra ID IAM Hardening & Zero Trust Lab

A hands-on cloud security lab demonstrating **Identity and Access Management (IAM) Hardening** within Microsoft Entra ID. This toolkit covers Privileged Identity Management (PIM), Just-In-Time (JIT) access, Conditional Access (MFA enforcement), group governance, and tenant-wide lockdown for a hypothetical B2B SaaS startup preparing for a SOC 2 compliance audit.

---

## 📋 Table of Contents

1. [Architecture & Scenario Overview](#architecture--scenario-overview)
2. [Task 1 — Baseline Assessment & PIM Audit](#task-1--baseline-assessment--pim-audit)
3. [Task 2 — Enforce Conditional Access (MFA)](#task-2--enforce-conditional-access-mfa)
4. [Task 3 — Dismantle Over-Privileged Groups](#task-3--dismantle-over-privileged-groups)
5. [Task 4 — Implement Just-In-Time (JIT) Access](#task-4--implement-just-in-time-jit-access)
6. [Task 5 — Test the JIT Workflow (Negative & Positive Testing)](#task-5--test-the-jit-workflow-negative--positive-testing)
7. [Task 6 — Tenant-Wide Default Lockdown](#task-6--tenant-wide-default-lockdown)
8. [Key Cloud Identity Concepts](#key-cloud-identity-concepts)
9. [Configuration Quick Reference](#configuration-quick-reference)

---

## Architecture & Scenario Overview

**Sopilot (sopilot.site)** is a rapidly scaling, fully remote SaaS startup with 150 employees operating on Azure. A critical SOC 2 audit has flagged their identity perimeter as high-risk.

**Initial Vulnerabilities (The "Before" State):**

| Vulnerability | Business Risk |
|------|--------|
| **Standing Global Admins** | Several accounts hold permanent 24/7 Global Administrator rights. |
| **No MFA Enforcement** | Administrative accounts can bypass Multi-Factor Authentication. |
| **Group Bloat** | A generic `IT Department` group grants sweeping, unmonitored privileges to its members. |
| **Open Tenant Defaults** | Standard users can register unverified OAuth apps and scrape the directory. |

---

## Task 1 — Baseline Assessment & PIM Audit

### Objective
Audit the current Privileged Identity Management (PIM) assignments to identify which accounts hold standing (permanent) high-level privileges.

### Implementation Steps
1. Navigate to **Identity > Privileged Identity Management > Microsoft Entra roles**.
2. Select **Global Administrator** and view the **Active assignments** tab.

### Observation
The audit reveals two active Global Administrators (`ahad@sopilot.site` and `breakglass@sopilot.site`) with permanent assignments (no start/end time limits). This violates the Principle of Least Privilege.

![Initial PIM Active Assignments](NoITGA)
![Secondary PIM Active Assignments](NoITGA_2)

---

## Task 2 — Enforce Conditional Access (MFA)

### Objective
Ensure that any administrative login requires Multi-Factor Authentication before we begin stripping standing access. 

### How It Works
Conditional Access Policies (CAP) act as if/then statements for authentication. By targeting directory roles rather than individual users, the policy dynamically applies MFA to anyone who elevates their privileges, regardless of their daily user account settings.

### Implementation Steps
1. Navigate to **Protection > Conditional Access > Policies**.
2. Create policy `CAP-MFA-COMPULSORY-ADMIN`.
3. **Users:** Include specific directory roles (Global Admin, User Admin, Security Admin, Exchange Admin). Exclude the emergency access account (`breakglass@sopilot.site`).
4. **Target resources:** Select **All cloud apps**.
5. **Grant:** Select **Require multifactor authentication**.

![Configuring Conditional Access](CAPMFA)

### Result & Analysis
The policy is enabled. Sign-in logs confirm that interactive logins by targeted roles are successfully evaluated and challenged for MFA.

![Policy Evaluation in Logs](CAPAPPLIEDREPORTR)

---

## Task 3 — Dismantle Over-Privileged Groups

### Objective
Eliminate the `IT Department` security group, which was previously used to grant broad, unmonitored access to multiple staff members.

### How It Works
Assigning high-level roles to standard security groups masks individual accountability. Deleting the group forces a transition to individual, role-based PIM assignments where every action is tied to a specific identity.

### Implementation Steps
1. Navigate to **Identity > Groups > All groups**.
2. Locate `IT Department`.
3. Select and Delete.

![Group Before Deletion](BeforeITDEPT)
![Deleting the Over-Privileged Group](deletingITDEPT)

---

## Task 4 — Implement Just-In-Time (JIT) Access

### Objective
Transition IT staff (`chris@sopilot.site`, `JJ@sopilot.site`, `BruceWayne@sopilot.site`) from permanent active privileges to **Eligible** assignments for the `User Administrator` role.

### How It Works
Instead of having admin rights 24/7, users operate as standard employees. When they need to perform admin tasks, they must request temporary elevation (JIT) via PIM, providing a business reason.

### Implementation Steps
1. Navigate to **Privileged Identity Management > Microsoft Entra roles > Roles**.
2. Select **User Administrator** > **Add assignments**.
3. Select the users and set the Assignment type to **Eligible** (instead of Active).

### Result & Analysis
Zero permanent active assignments remain for these users. They are now listed under the **Eligible assignments** tab.

![Eligible Roles for IT](onlyeligiblerolesit)
![JJ's Eligible Status](JJGA)
![Chris's Eligible Status](ChrisGA)

---

## Task 5 — Test the JIT Workflow (Negative & Positive Testing)

### Objective
Verify that un-elevated users are blocked from admin tasks, and test the PIM activation and approval workflow.

### How It Works
If JIT is working correctly, a user attempting an admin action without elevating will hit a permissions wall. Once they elevate and are approved, the portal unlocks for a limited time.

### Implementation Steps & Results
1. **The Block (Negative Test):** `JJ@sopilot.site` attempts to access restricted Azure panels without activating PIM. Result: **Error 401: You don't have access**.
   ![Access Blocked](JJSigninblocked)

2. **The Request:** JJ navigates to PIM > **My roles** and clicks **Activate** for `User Administrator`. They enter the required justification: *"I want to create new onboarding employees"* and request a 2-hour window.
   ![PIM Activation Screen](PIMJJ)

3. **The Approval:** The Global Admin reviews the pending request in the **Approve requests** queue and clicks Approve.
   ![PIM Approval Queue](GlobalAdminAPPROVE)

4. **The Execution (Positive Test):** JJ's session elevates. They successfully access the `Create new user` panel.
   ![Successful Admin Access](AfterApproval)

---

## Task 6 — Tenant-Wide Default Lockdown

### Objective
Harden default Entra ID user settings to prevent standard users from expanding the attack surface via shadow IT or directory reconnaissance.

### How It Works
By default, Microsoft Entra ID is highly permissive to allow easy onboarding. Restricting these settings stops standard compromised accounts from moving laterally or deploying malicious OAuth apps.

### Implementation Steps
Navigate to **Identity > Users > User settings** and configure the following:
* `Users can register applications` ➔ **No**
* `Restrict access to Microsoft Entra admin center` ➔ **Yes**
* `Users can create security groups` ➔ **No**
* `Guest user access restrictions` ➔ **Restricted to properties and memberships of their own directory objects**

![Hardened User Settings](Usersettings)

---

## Key Cloud Identity Concepts

| Concept | Description |
|---------|-------------|
| **Zero Trust** | Security model assuming breach; requires strict identity verification for every person and device. |
| **PoLP (Least Privilege)** | Giving a user only the minimum levels of access necessary to complete their job functions. |
| **PIM (Privileged Identity Mgmt)** | Azure service that manages, controls, and monitors access to important resources. |
| **JIT (Just-In-Time) Access** | Providing privileged access only when needed, and removing it immediately after (e.g., a 2-hour window). |
| **Conditional Access** | If-Then policies that evaluate signals (user, location, device) to enforce MFA or block access. |
| **Break-Glass Account** | A highly secure, excluded emergency access account used only if the primary authentication systems fail. |

---

## Configuration Quick Reference
```text
! View Active vs. Eligible Admin Roles
Search > Privileged Identity Management > Microsoft Entra roles > Assignments

! Create a Conditional Access Policy
Identity > Protection > Conditional Access > Policies > + New policy

! Restrict App Registrations & Guest Visibility
Identity > Users > User settings

! Delete or Modify Security Groups
Identity > Groups > All groups

! Approve a PIM Request
Search > Privileged Identity Management > Approve requests > Microsoft Entra roles
