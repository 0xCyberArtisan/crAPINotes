# OWASP crAPI — Web & API VAPT Portfolio

> **Offensive Security | Web Application Security | API Security | Authentication & Authorization**

**Chukwunonso Jude Aneke**
*VAPT Engineer / Offensive Security Engineer*

---

## Overview

This repository documents a hands-on **Web and API Vulnerability Assessment and Penetration Testing (VAPT)** engagement against the **OWASP crAPI (Completely Ridiculous API)** vulnerable application.

The objective was not simply to identify known vulnerabilities, but to demonstrate a practical offensive-security workflow:

* Attack-surface discovery
* Authentication and session-security assessment
* Authorization and access-control testing
* API manipulation
* Input-validation testing
* Injection testing
* Business-logic assessment
* Manual exploitation
* Impact validation
* Proof-of-concept development
* Risk analysis
* Remediation and retesting

The repository is structured to distinguish between **assessment methodology, technical evidence, and individual vulnerability findings**.

---

## Security Assessment Focus

### Web Application Security

* Application reconnaissance
* Attack-surface enumeration
* Authentication testing
* Session management
* Access-control testing
* Input validation
* Business-logic testing
* Security misconfiguration
* Manual exploitation

### API Security

* API endpoint enumeration
* Authentication mechanisms
* Authorization enforcement
* BOLA / IDOR testing
* Mass assignment
* Parameter manipulation
* Injection testing
* Business-logic abuse
* API response analysis
* Error handling

### Identity & Cryptographic Security

* JWT implementation analysis
* JWT algorithm validation
* Signature verification
* Token manipulation
* Claim manipulation
* Authentication bypass
* Identity impersonation
* Authorization enforcement

---

# Assessment Methodology

The assessment follows a structured offensive-security methodology designed to move from **discovery → validation → exploitation → impact → remediation**.

```text
┌──────────────────────┐
│  Reconnaissance      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Attack Surface       │
│ Enumeration           │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Authentication &     │
│ Session Assessment   │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Authorization &      │
│ Access Control       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Input / Injection    │
│ Testing              │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Business Logic       │
│ Assessment           │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Manual Exploitation  │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Impact Validation    │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Evidence & PoC       │
│ Development          │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│ Remediation &        │
│ Retesting            │
└──────────────────────┘
```

The assessment is aligned with established application-security practices, including:

* OWASP API Security principles
* OWASP Web Security Testing methodology
* CWE classifications
* CVSS-based risk assessment
* Manual vulnerability validation

---

# Findings

The following vulnerabilities have been identified and documented through manual assessment and proof-of-concept validation.

| ID        | Vulnerability                        | Category                          | Evidence                                                     |
| --------- | ------------------------------------ | --------------------------------- | ------------------------------------------------------------ |
| CRAPI-001 | JWT `alg:none` Authentication Bypass | Authentication / JWT              | [PoC](crAPINotes/PoCs/JWT-Alg-None-Authentication-Bypass.md) |
| CRAPI-002 | Authentication Token Vulnerability   | Authentication                    | [PoC](crAPINotes/PoCs/Auth-Vuln-Token.md)                    |
| CRAPI-003 | Contact Mechanic SSRF                | SSRF                              | [PoC](crAPINotes/PoCs/Contact-Mechanic-SSRF-Vuln.md)         |
| CRAPI-004 | Coupon Code NoSQL Injection          | Injection                         | [PoC](crAPINotes/PoCs/Coupon-Code-NoSQL-Injection-Vuln.md)   |
| CRAPI-005 | Unauthorized Private Video Access    | Authorization                     | [PoC](crAPINotes/PoCs/Delete-Private-Video.md)               |
| CRAPI-006 | Improper Inventory Management        | Business Logic                    | [PoC](crAPINotes/PoCs/Improper-Inventory-Managment.md)       |
| CRAPI-007 | Product Mass Assignment              | Authorization / Mass Assignment   | [PoC](crAPINotes/PoCs/List-Products-Mass-Assignment.md)      |
| CRAPI-008 | Vehicle Location BOLA                | Broken Object Level Authorization | [PoC](crAPINotes/PoCs/Vehicle-Location-vuln-BOLA.md)         |

> **Note:** Severity ratings are intentionally maintained within the individual vulnerability reports rather than inferred at the portfolio level.

---

# Featured Finding

## CRAPI-001 — JWT `alg:none` Authentication Bypass

**Severity:** Critical
**CWE:** CWE-347 — Improper Verification of Cryptographic Signature

A critical JWT implementation weakness was identified in which the application accepted a token using the attacker-controlled:

```text
alg: none
```

The issue was further validated by modifying the JWT subject claim to another controlled account and successfully accessing that account's authenticated dashboard.

### Impact demonstrated

The vulnerability allowed an attacker to:

* Bypass JWT signature verification
* Forge authentication tokens
* Manipulate identity claims
* Impersonate another application user
* Access another user's authenticated data

The finding demonstrates why JWT authentication must enforce a server-side cryptographic algorithm policy rather than trusting the algorithm specified by an attacker-controlled token.

**Technical evidence:**
[JWT `alg:none` Authentication Bypass PoC](crAPINotes/PoCs/JWT-Alg-None-Authentication-Bypass.md)

---

# Technical Capabilities Demonstrated

This engagement demonstrates practical experience across multiple areas of offensive security.

### Reconnaissance & Enumeration

* Application discovery
* API endpoint enumeration
* Endpoint behavior analysis
* Parameter discovery
* Authentication-flow analysis

### Authentication

* JWT analysis
* Token manipulation
* Signature-validation testing
* Algorithm-confusion testing
* Authentication bypass testing
* Identity impersonation

### Authorization

* BOLA / IDOR
* Horizontal privilege escalation
* Object-level access-control testing
* Unauthorized resource access
* Mass-assignment testing

### Injection

* NoSQL injection
* Parameter manipulation
* Server-side request forgery
* Input-validation bypass

### Business Logic

* Workflow manipulation
* Application-state abuse
* Inventory manipulation
* Privilege-boundary testing
* Abuse-case identification

### Exploitation & Validation

* Manual request construction
* Burp Suite interception
* API replay
* Token crafting
* Python-based testing
* `curl`-based PoCs
* Controlled exploitation
* Evidence preservation

---

# Tooling

The assessment environment makes use of industry-standard offensive-security tooling.

| Tool           | Purpose                                                    |
| -------------- | ---------------------------------------------------------- |
| **Burp Suite** | HTTP interception, request manipulation and manual testing |
| **jwt_tool**   | JWT security analysis and token manipulation               |
| **curl**       | API interaction and PoC validation                         |
| **Python**     | Automation, token processing and security testing          |
| **Postman**    | API discovery and functional testing                       |
| **Docker**     | Isolated vulnerable application deployment                 |
| **Kali Linux** | Offensive-security testing environment                     |
| **Git**        | Version control and security research documentation        |

Tools are treated as **means of validation rather than the methodology itself**. Findings are manually verified wherever practical.

---

# Lab Environment

The assessment was performed against a locally deployed instance of OWASP crAPI.

```text
Operating Environment
        │
        ▼
┌──────────────────────┐
│      Kali Linux      │
│                      │
│  Burp Suite          │
│  curl                │
│  Python              │
│  jwt_tool            │
│  Postman             │
└──────────┬───────────┘
           │
           │ HTTP
           ▼
┌──────────────────────┐
│      OWASP crAPI     │
│                      │
│  Web Application     │
│  REST APIs           │
│  Authentication      │
│  Business Logic      │
└──────────────────────┘
```

The vulnerable application is isolated within a controlled laboratory environment.

---

# Evidence-Driven Testing

A vulnerability is not considered complete merely because a scanner or tool reports it.

Each finding is approached through a validation cycle:

```text
Potential Issue
      │
      ▼
Manual Verification
      │
      ▼
Exploitability Assessment
      │
      ▼
Controlled Proof of Concept
      │
      ▼
Impact Validation
      │
      ▼
Evidence Collection
      │
      ▼
Risk Classification
      │
      ▼
Remediation Recommendation
      │
      ▼
Retest
```

This approach helps distinguish **theoretical weaknesses from demonstrably exploitable vulnerabilities**.

---

# Repository Structure

```text
crAPI/
│
├── README.md
│
├── docker-compose.yml
│
├── crAPINotes/
│   │
│   ├── README.md
│   │
│   ├── note.md
│   │
│   ├── PoCs/
│   │   ├── JWT-Alg-None-Authentication-Bypass.md
│   │   ├── Auth-Vuln-Token.md
│   │   ├── Contact-Mechanic-SSRF-Vuln.md
│   │   ├── Coupon-Code-NoSQL-Injection-Vuln.md
│   │   ├── Delete-Private-Video.md
│   │   ├── Improper-Inventory-Managment.md
│   │   ├── List-Products-Mass-Assignment.md
│   │   └── Vehicle-Location-vuln-BOLA.md
│   │
│   └── wordlists/
│
├── postman_collections/
│
└── keys/
```

### Documentation hierarchy

**`README.md`**
Portfolio landing page — capabilities, findings and assessment overview.

**`crAPINotes/README.md`**
Detailed assessment methodology, environment and testing process.

**`crAPINotes/PoCs/`**
Individual vulnerability reports containing technical evidence and exploitation details.

**`crAPINotes/note.md`**
Working research notes and observations collected during testing.

---

# What This Portfolio Demonstrates

This project is intended to demonstrate more than familiarity with security tools.

It demonstrates the ability to:

* Think from an attacker's perspective
* Identify security boundaries
* Understand application authentication mechanisms
* Test authorization independently of authentication
* Manipulate API requests and application state
* Develop reproducible vulnerability PoCs
* Validate real-world security impact
* Translate technical findings into security risk
* Document vulnerabilities clearly
* Recommend practical remediation
* Retest security controls after remediation

The emphasis is on **manual reasoning, exploitation, evidence and impact**, rather than automated scanner output alone.

---

# Security Testing Principles

### 01 — Understand the application

Before attacking an endpoint, understand its purpose, authentication model, trust boundaries and expected behavior.

### 02 — Test the security boundary

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to access or perform?**

Both are assessed independently.

### 03 — Manipulate assumptions

Security weaknesses frequently occur where developers assume that:

* A token cannot be modified
* A user will not change an object identifier
* A client will send expected parameters
* A workflow will be followed correctly
* An API request originated from a trusted interface

These assumptions are deliberately challenged during testing.

### 04 — Prove impact

A finding becomes significantly more valuable when exploitation can be demonstrated safely and reproducibly.

### 05 — Document for remediation

A useful security report should allow a developer or security team to understand:

* What is wrong
* Why it is exploitable
* What security boundary was bypassed
* What the impact is
* How to reproduce it
* How to fix it
* How to verify the fix

---

# Scope & Ethics

This repository is maintained for **authorized security research, education and professional portfolio development**.

All exploitation demonstrated here is performed against intentionally vulnerable applications or controlled environments.

The techniques and proof-of-concepts documented in this repository should only be applied to systems where explicit authorization has been granted.

---

# About the Researcher

**Chukwunonso Jude Aneke**
**VAPT Engineer | Offensive Security**

Focus areas include:

* Web Application Security
* API Security
* Vulnerability Assessment & Penetration Testing
* Authentication & Authorization
* JWT Security
* Access Control
* Business Logic Testing
* Security Research
* Red Teaming

This repository forms part of a practical offensive-security portfolio focused on **finding, validating, exploiting and clearly communicating application-security vulnerabilities**.

---

## Portfolio Philosophy

> **Don't just identify the vulnerability. Understand the trust boundary, prove the impact, document the attack path, and explain how to fix it.**

---

## Disclaimer

OWASP crAPI is an intentionally vulnerable application designed for security education and testing.

All techniques documented in this repository are intended for **authorized security testing and controlled laboratory environments only**.
