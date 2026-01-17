# Evidence of System Behavior & Controls

## Purpose

This document provides **verifiable evidence** that the system described in this repository is **implemented, operational, and exercised**, not a paper-only design.

The artifacts below demonstrate **real executions**, **enforced controls**, and **logged outcomes** across safety, privacy, and reliability dimensions.

This document intentionally avoids internal secrets, prompts, model instructions, or configuration values, while still proving **actual system behavior**.

---

## 1. Evidence Scope

The evidence focuses on:

- Input classification (Safe / PII / Harmful)
- PII detection and selective redaction
- Risk-based gating and request blocking
- Model routing and fallback behavior
- Toxicity scoring and post-generation enforcement
- Immutable audit logging

Out of scope by design:

- Model weights
- Prompt templates
- Internal workflow JSON
- Secrets, tokens, or credentials

---

## 2. Test Methodology

### 2.1 Test Set Design

A **small, curated test set** was used to exercise all critical control paths without brute-force fuzzing.

The test categories include:

- Safe, non-sensitive input
- Non-sensitive PII (low risk, informational)
- Sensitive PII (high risk, block + redact)
- Harmful or malicious prompts (block)
- Ambiguous / edge-case inputs

Each test was executed through the **same production workflow**, not a mock or test-only pipeline.

The goal is **control coverage**, not volume.

---

## 3. Workflow-Level Evidence

### 3.1 Orchestration & Control Flow

**Artifact**

- High-level workflow overview  
  (`/docs/assets/workflow_overview.png`)

**Demonstrates**

- End-to-end execution from request intake to persistence
- Sequential enforcement of safety gates
- Clear separation between:
  - input inspection
  - risk decisioning
  - model invocation
  - audit logging
- Safety enforced **outside the LLM boundary**

Internal node parameters, workflow JSON, and secrets are intentionally not exposed.

---

## 4. Enforcement Evidence by Risk Category

This section documents **distinct enforcement paths** for different classes of unsafe input.  
Each artifact corresponds to a **real execution** of the system.

---

### 4.1 Harmful / Malicious Prompt (Pre-Model Block)

**Artifact**

- Request blocked with `403 Forbidden`
  <video src="https://github.com/user-attachments/assets/c5112453-9429-4562-ab76-5849328907f0" width="100%" controls></video>

**Demonstrates**

- Harmful intent detected prior to model invocation
- Deterministic fail-closed behavior
- No unsafe input reaches the LLM
- Block decision logged with risk classification

This confirms **pre-inference enforcement**.

---

### 4.2 Sensitive PII Submission (Redaction + Block)

**Artifact**

- Request containing sensitive PII blocked  
  <video src="https://github.com/user-attachments/assets/63787ceb-5a49-477b-ba84-345b4ab6ab19" width="100%" controls></video>

**Demonstrates**

- Sensitive PII detected in-memory
- Prompt redacted prior to any persistence
- Request blocked due to high-risk data exposure
- No raw sensitive identifiers stored

This confirms **privacy-first enforcement at the boundary**.

---

### 4.3 Non-Sensitive PII (Allowed with Redaction)

**Artifact**

- Audit log record showing redacted prompt  
  (`/docs/assets/audit_log_row.png`)

**Demonstrates**

- Selective redaction (not blanket deletion)
- Preservation of semantic intent
- Logging that supports audits without over-collection
- Enables **forensic analysis and trend detection** (e.g., abuse patterns, repeated failure modes) without retaining sensitive identifiers

This confirms **data minimization with forensic utility**, not data suppression.

---

### 4.4 Safe Input (Normal Execution Path)

**Artifact**

- Successful request execution (no block)  
  (`/docs/assets/enforcement_safe.png`)

**Demonstrates**

- Controls do not interfere with benign usage
- Safety system is not overly restrictive
- Model routing and response delivery function normally

This confirms **low false-positive impact**.

---

## 5. Audit Logging Evidence

Each execution produces a **single immutable audit record** containing:

- Timestamp
- Redacted user prompt
- PII detection summary (JSON)
- Risk classification
- Model selected
- Toxicity score (if applicable)
- Execution identifier

**Artifact**

- Single audit log row  
  (`/docs/assets/audit_log_row.png`)

This demonstrates **traceability without excessive data retention**.

---

## 6. Negative Evidence (Intentional Absence)

The following items are intentionally **not present** in any logs or artifacts:

- Raw sensitive PII
- Prompt templates or system instructions
- Model API keys or secrets
- Full workflow exports or node configurations

This absence is deliberate and part of the security posture.

---

## 7. Reproducibility Statement

A reviewer can:

1. Deploy the system using the provided instructions
2. Submit inputs from the documented categories
3. Observe **deterministic enforcement outcomes**
4. Verify redacted persistence in the audit table

Exact model outputs may vary due to nondeterminism, but **risk classification and enforcement behavior remain consistent**.

---

## 8. Reviewer Checklist

- [x] Controls are implemented, not theoretical
- [x] Safety enforced pre- and post-model
- [x] Sensitive PII is never stored raw
- [x] Logging supports audits without over-collection
- [x] Evidence aligns with documented architecture

---

## 9. Closing Note

This evidence set is intentionally concise and selective.  
It demonstrates **operational reality**, not maximal disclosure.

Additional internal artifacts exist but are withheld to avoid expanding the attack surface and to preserve system integrity.
