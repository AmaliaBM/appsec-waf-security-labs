# appsec-waf-security-labs
Practical security assessment and remediation labs focused on web applications, Apache and AWS WAF


# AppSec & AWS WAF Security Labs

> Practical security assessment and remediation labs focused on web applications, Apache and AWS WAF.

This repository contains **synthetic security findings, evidence, controlled validation scenarios and remediation strategies** designed to demonstrate how application-security findings can be investigated and mitigated at different architectural layers.

The primary focus is the relationship between:

* Application security
* Apache HTTP Server
* HTTP compression
* Rate limiting
* Scraping mitigation
* AWS WAF
* Defense-in-depth
* Security validation and retesting

---

## ⚠️ Disclaimer

**All information contained in this repository is synthetic.**

The following are intentionally fictional:

* Domains
* IP addresses
* Authentication tokens
* CSRF tokens
* HTTP responses
* Logs
* Request counts
* Application names
* Infrastructure identifiers
* AWS resources

The examples are intended exclusively for:

* Authorized security testing
* Security engineering education
* Application security research
* WAF configuration practice
* Defensive security training

Do not reproduce testing techniques against systems without appropriate authorization.

---

## Repository Objectives

This repository demonstrates a repeatable security assessment methodology:

```text
Identification
     │
     ▼
Evidence Collection
     │
     ▼
Controlled Validation
     │
     ▼
Risk Assessment
     │
     ▼
Root Cause Analysis
     │
     ▼
Primary Remediation
     │
     ▼
Defense in Depth
     │
     ▼
Retesting
```

The objective is not to treat AWS WAF as a universal vulnerability-remediation mechanism.

Instead, each control is mapped to the layer where it is technically appropriate.

---

# Repository Structure

```text
.
├── findings/
│   ├── CVE-2013-3587-BREACH.md
│   └── rate-limiting-scraping.md
│
├── labs/
│   ├── breach/
│   └── aws-waf-rate-limiting/
│
├── mitigations/
│   ├── application/
│   ├── apache/
│   └── aws-waf/
│
└── evidence/
```

---

# Findings

## CVE-2013-3587 — Potential BREACH Exposure

The BREACH finding demonstrates how HTTP compression can create a potential compression side-channel when sensitive information and attacker-controlled content coexist within the same compressed response.

[Read the finding](findings/CVE-2013-3587-BREACH.md)

---

## Rate Limiting / Scraping

This finding demonstrates how excessive request rates can facilitate automated scraping, enumeration and resource consumption.

[Read the finding](findings/rate-limiting-scraping.md)

---

# Labs

The labs provide controlled, synthetic scenarios for reproducing the findings and validating remediation.

### BREACH

[Open the BREACH lab](labs/breach/README.md)

### AWS WAF Rate Limiting

[Open the AWS WAF rate-limiting lab](labs/aws-waf-rate-limiting/README.md)

---

# Security Control Mapping

| Finding                   | Application |   Apache |          AWS WAF |
| ------------------------- | ----------: | -------: | ---------------: |
| Excessive requests        |           ✅ |        ✅ |                ✅ |
| Scraping                  |           ✅ |        ✅ |                ✅ |
| Authentication abuse      |           ✅ | Possible |                ✅ |
| HTTP compression          |           ✅ |        ✅ |                ❌ |
| BREACH                    |           ✅ |        ✅ | Defense-in-depth |
| CSRF token implementation |           ✅ |        ❌ |                ❌ |

The important distinction is between **primary remediation** and **defense-in-depth**.

---

# Methodology

Each finding follows the same structure:

```text
Finding
  │
  ├── Description
  ├── Evidence
  ├── Preconditions
  ├── Controlled Validation
  ├── Impact
  ├── Root Cause
  │
  ├── Primary Remediation
  │      ├── Application
  │      └── Server
  │
  ├── AWS WAF Defense-in-Depth
  │
  └── Retesting
```

---

# Synthetic Evidence Policy

Evidence examples use clearly fictional values.

Example:

```http
GET /connexion/ HTTP/1.1
Host: app.example.test
Accept-Encoding: gzip
User-Agent: SecurityLab/1.0
```

Example response:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Encoding: gzip
```

No production data should be committed to this repository.

---

# Recommended Usage

The repository can be used to practice:

* Security finding development
* Evidence analysis
* Application remediation
* Apache hardening
* AWS WAF rule design
* Defense-in-depth architecture
* Security retesting
* Audit report writing

---

## Security Philosophy

A WAF should be treated as an additional security layer, not as a substitute for fixing application vulnerabilities.

```text
Application vulnerability
          │
          ▼
Fix root cause
          │
          ▼
Server hardening
          │
          ▼
WAF defense-in-depth
          │
          ▼
Monitoring + retesting
```

---

## Status

This repository is a continuously evolving security laboratory.

New findings and remediation patterns may be added over time.
