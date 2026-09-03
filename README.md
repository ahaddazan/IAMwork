# Cloud Identity & Access Management (IAM) Portfolio

This repository serves as a centralized portfolio for hands-on engineering projects focused on Cloud Identity, Access Management (IAM), and Zero Trust security architectures. 

The primary objective of these labs is to move beyond theoretical concepts and demonstrate practical, real-world deployment of identity lifecycle management, privileged access governance, and directory hardening.

---

**Core Security Competencies Demonstrated**
* **Role-Based Access Control (RBAC):** Implementing the Principle of Least Privilege (PoLP) across tenant resources and administrative roles.
* **Privileged Identity Management (PIM):** Eliminating standing access by deploying Just-In-Time (JIT) elevation workflows and approval gates.
* **Conditional Access & Authentication Rules:** Designing network, device, and role-based policies to enforce Multi-Factor Authentication (MFA) and block high-risk sign-ins.
* **Directory Governance:** Hardening default tenant configurations, securing third-party app registrations, and auditing guest access.

---

**Repository Structure**

This portfolio is divided into modular project directories. Each folder contains a detailed incident report, architecture breakdown, and verification artifacts.

| Project Directory | Cloud Provider | Key Technologies | Focus Area |
| :--- | :--- | :--- | :--- |
| **`/Entra-ID-IAM-Hardening`** | Microsoft Azure | Entra ID, PIM, Conditional Access | Remediation of standing Global Admin privileges, JIT access enforcement, and tenant default hardening.|
---

**How to Navigate This Repository**
Each project folder contains its own self-contained `README.md` that acts as an executive summary and technical walk-through. 
1. Navigate to a specific project folder from the root directory.
2. Review the **Executive Summary** for the business context and security objectives.
3. Examine the **Implementation Details** and **Evidence Table** to see exactly how controls were applied and verified through cloud audit logs and portal configurations.
