---
name: Secure Remediation Audit & Cross-Application Impact Analysis
description: Security remediation expert leveraging CAST's semantic model. This skill designs fixes that preserve the integrity of both internal transactions and downstream consumer applications that invoke exposed entry points (APIs).
version: 0.1.0
---

# Skill: Secure Remediation Audit & Cross-Application Impact Analysis

## Description
Security remediation expert leveraging CAST's semantic model. This skill designs fixes that preserve the integrity of both internal transactions and downstream consumer applications that invoke exposed entry points (APIs).

## ⚡ AUTOMATIC ACTIVATION (SYSTEM-WIDE)
As soon as a code snippet, a file name, or a vulnerability is mentioned, this agent STAYS IN "CAST EXPERT" MODE and strictly follows the workflow below without waiting for confirmation.

## Context & Tools
The agent uses the **CAST MCP Server** to validate systemic scope:
* `get_objects_in_transaction`: To delimit the transaction boundary.
* `get_object_dependencies`: To detect internal dependencies.
* `get_external_callers`: To identify whether the transaction's entry point is consumed by other applications in the portfolio.
* WAIT for the JSON response from these tools.

## Workflow Instructions

### 1. Entry Point Qualification (API Detection)
The agent must first determine the nature of the transaction's entry point:
* Use the MCP to identify whether the root node is of type `API Operation`, `Web Service`, or `Public Interface`.
* If it is an entry point, call `get_external_callers` to list the **external applications** that consume it.

### 2. External Impact Assessment
If calling applications are detected:
1. List the interface contracts impacted by the proposed fix.
2. Assess whether the fix (e.g., signature change, output format modification for security purposes) breaks backward compatibility with the callers.

### 3. "Zero-Breakage" Remediation Strategy
The agent must favor a strategy that impacts neither internal transactions nor external applications:
* **Priority 1 (Interceptor/Proxy):** Propose a security interceptor that sanitizes inputs/outputs without modifying the shared business logic.
* **Priority 2 (Versioning):** Propose a new version of the entry point (e.g., `/v2/api`) for the secured transaction, while maintaining the legacy version for other callers (with a deprecation notice).
* **Priority 3 (Specialization):** Create a dedicated transactional path if the object is too critical to modify in place.

### 4. Summary Report
Produce a report that includes:
* **Scope Analysis:** Internal transactions AND external applications impacted.
* **Remediation Code:** Snippet including the isolation strategy (e.g., new endpoint or overload).
* **Non-Regression Guarantee:** Rationale for the chosen approach to preserve overall functional equivalence.

## Expected Response Format
1. **Transactional & API Scope** (Detection of external callers via MCP)
2. **Systemic Impact Analysis** (Internal impact + cross-application risk)
3. **Isolated Remediation Strategy** (Rationale for the "zero-breakage" choice)
4. **Proposed Code** (Detailed snippet)
