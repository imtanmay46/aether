# Project Aether
### Enterprise AI Governance, Safety & Red-Teaming Gateway

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.10+-blue.svg" />
  <img src="https://img.shields.io/badge/FastAPI-Production-green.svg" />
  <img src="https://img.shields.io/badge/AI%20Safety-Defense%20in%20Depth-critical" />
  <img src="https://img.shields.io/badge/Architecture-Fail--Closed-important" />
  <img src="https://img.shields.io/badge/Status-Active-success" />
  <img src="https://img.shields.io/badge/License-MIT-lightgrey" />
</p>

---

> **Project Status Note**  
> Project Aether is maintained as a **reference architecture and design disclosure**.  
> While the live demonstration backend was initially deployed using a transient n8n/FastAPI setup, the long-term value of this repository lies in its documented safety architecture, threat model, and governance patterns.

## Overview

**Aether** is a low-latency **AI governance gateway** designed to enforce **security, compliance, and auditability** across enterprise LLM workflows.

Inspired by the mythological concept of *Aether* — the medium filling the space above the terrestrial world — this system functions as a **mandatory environment through which every AI interaction must pass** before being considered safe.

Aether does **not** simply sit in front of LLMs.  
It acts as a **governance layer surrounding them**, protecting:

- **LLMs from malicious or unsafe prompts** *(Gate-1)*  
- **Users and systems from harmful or non-compliant outputs** *(Gate-2)*  

> **No request or response bypasses Aether.**

---

## Motivation

Large Language Models are powerful — but **unsafe by default** in enterprise environments.

### Unmitigated risks include:
- Leakage of **PII** into third-party model providers
- **Prompt injection & jailbreak attacks** bypassing policy controls
- **Zero auditability** of AI decision-making

**Aether addresses these risks** by enforcing a **fail-closed, infrastructure-level governance model** that applies safety guarantees **before, during, and after** every LLM interaction.

---

## Architecture at a Glance

<img width="511" height="532" alt="Screenshot 2026-01-16 at 6 21 33 PM" src="https://github.com/user-attachments/assets/c2232c3d-b0a8-44dd-9f33-8ad56b680caa" />

---

## Gate-1: Pre-Inference Red-Teaming & Prompt Safety

Before any prompt reaches an LLM, it is evaluated by a **FastAPI-based security microservice**.

### Security Controls

#### Semantic Toxicity Detection (ML)
- Powered by **Detoxify (Toxic-BERT)**
- Detects implicit abuse, threats, harassment, and contextual toxicity missed by keyword filters

#### Heuristic Defense Layer
- A curated library of **60+ RegEx patterns** designed to intercept:
  - Prompt injections
  - Jailbreak attempts
  - Instruction-override and policy-evasion techniques

#### PII Detection & Redaction
- Built using **Microsoft Presidio + SpaCy**
- Detects and redacts:
  - SSNs, Credit Cards, IBANs
  - OTPs, Emails, Phone Numbers, IP addresses

#### Risk Gate (Fail-Closed)
- Unsafe prompts trigger an immediate **403 Forbidden**
- The LLM is never invoked

> **Guarantee:**  
> No high-risk, PII-bearing, or malicious prompt is ever forwarded to an LLM.

---

## In-Flight: Intelligent Model Orchestration

Aether dynamically routes requests to balance **latency, cost, and reasoning depth**.

### Routing Logic

#### Complexity Classification
- Performed using **Gemini 2.5 Flash-Lite**
- Evaluates intent and reasoning requirements

#### Adaptive Model Routing
- Simple / factual prompts → **Gemini 2.5 Flash**
- Multi-step reasoning / complex tasks → **Gemini Flash-3 Preview**

#### Resilience & Rate-Limit Handling
- Stateful retry counters
- Exponential backoff
- 60-second cooldown window

Ensures **high availability** under API constraints.

---

## Gate-2: Post-Inference Output Safety & QA

Every generated response is validated **before** being returned to the user.

### Output Controls

#### Response Toxicity Audit
- Uses **Google Perspective API**
- Ensures generated outputs remain safe, helpful, and non-harmful

#### Forensic Persistence
- All safety metadata is persisted in a **Supabase audit log**
- Enables compliance audits and incident investigation

---

## Database & Observability

Aether maintains a **full AI safety audit trail**, functioning as a black-box recorder for LLM interactions.

### `audit_logs` Table

| Field | Type | Description |
|------|------|-------------|
| `id` | BIGINT | Primary key |
| `created_at` | TIMESTAMPTZ | Event timestamp |
| `user_prompt` | TEXT | Raw user input (for forensic review) |
| `pii_detected` | JSONB | Detected & redacted PII entities |
| `risk_level` | TEXT | Safety classification |
| `model_used` | TEXT | Model routing decision |
| `response_toxicity` | DOUBLE PRECISION | Perspective API toxicity score |
| `execution_id` | VARCHAR | Workflow execution trace |

### Enables
- Incident reconstruction
- Governance & compliance reporting
- Policy enforcement verification
- Cost and routing analysis

---

## Technology Stack

**Orchestration**
- n8n (Workflow Automation)

**LLMs**
- Gemini 2.5 Flash-Lite  
- Gemini Flash-3 Preview  

**Security & Safety**
- Detoxify (Semantic Toxicity)
- Microsoft Presidio (PII Detection)
- Regex-based Heuristics (Jailbreak Defense)

**Auditing & Storage**
- Supabase (PostgreSQL)

**Evaluation**
- Google Perspective API

**Frontend**
- V0 / Next.js

---

## Security Design Principles

### Defense in Depth
Aether combines deterministic and probabilistic safeguards:
- **Regex** → fast-fail known exploits
- **Detoxify** → semantic abuse detection
- **Presidio** → compliance-grade PII guarantees

### Fail-Closed by Default
- Safety violations return **403 Forbidden**
- Security is treated as **standard infrastructure behavior**, not an exception

---

## Non-Goals & Trust Boundaries

**Aether does NOT:**
- Train on user data
- Modify or “fix” unsafe prompts
- Alter LLM outputs beyond validation

**Aether ONLY:**
- Detects, blocks, routes, audits, and logs
- Enforces safety as deterministic infrastructure logic

---
## Summary

Aether transforms AI safety from an aspirational prompt-engineering goal into a **deterministic infrastructure requirement**.\
By decoupling governance logic from model reasoning, Aether establishes a robust control plane that ensures enterprise compliance without sacrificing LLM utility.

Through its dual-gate architecture, Aether provides:
* **Active Defense:** Real-time interception of injection, jailbreaks, and PII leakage.
* **Operational Resilience:** Intelligent model routing and stateful retry logic to maintain high availability.
* **Absolute Auditability:** A forensic-grade audit trail for every interaction, ensuring full transparency in AI decision-making.

> **Aether treats AI safety as infrastructure — not prompt engineering.**
