# sql-iam-access-audit
SQL security audit and QA validation queries for detecting orphan database accounts and attempted login abnormalities
# SQL Security Audit & User Access Reconciliation (T-SQL / SSMS)

## Overview
In regulated healthcare and enterprise SaaS environments handling sensitive PII and compliance records, enforcing **Separation of Duties (SoD)** and **Least Privilege** is critical. This project demonstrates how relational SQL Server (T-SQL) queries and QA change-control checks are used to audit active database permissions against HR employment statuses, detect brute-force authentication anomalies, and execute verified remediation updates.

## Database Architecture
Built in **Microsoft SQL Server 2025 / SSMS** with enforced referential integrity (`PRIMARY KEY` and `FOREIGN KEY` constraints):
* **`hr_roster`**: Authoritative employee/contractor employment statuses (`Active` vs. `Terminated`).
* **`db_permissions`**: Active database privilege levels (`BIT` active flags and modification timestamps).
* **`auth_logs`**: High-precision (`DATETIME2`) authentication events with auto-incrementing `IDENTITY` log IDs.

## Key Security & QA Workflows
1. **Orphan Account Detection (`INNER JOIN`):** Cross-references `db_permissions` against `hr_roster` to flag terminated employees or external contractors who still retain active database access (`account_active = 1`).
2. **Brute-Force Anomaly Hunting (`GROUP BY` / `HAVING`):** Aggregates failed authentication attempts by user, source IP, and date to surface accounts experiencing 5 or more consecutive failures.
3. **Controlled Remediation & QA Verification (`BEGIN TRANSACTION`):** Executes a transactional `UPDATE` to revoke terminated permissions (`account_active = 0`) and runs a post-remediation QA check confirming zero unauthorized accounts remain active while valid staff remain untouched.

## Execution & Results

### 1. Pre-Remediation Audit Findings (Orphan Accounts & Brute-Force Spike)
![Pre-Remediation Audit Findings](audit_findings.png)

### 2. Controlled Remediation & Post-Remediation QA Verification
![Post-Remediation QA Verification](qa_verification.png)
