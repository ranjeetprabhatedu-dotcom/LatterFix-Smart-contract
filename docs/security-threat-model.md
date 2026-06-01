# Security Threat Model and Core Pipeline Diagrams

**Issue #685**: Security hardening + threat model notes for core pipeline architecture

## Overview

This document outlines the threat model for Sanctifier's core pipeline and provides diagrams of the critical security-relevant components.

## Core Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Input Validation Layer                      │
│  • Schema validation (SARIF, reports)                            │
│  • Size limits enforcement                                       │
│  • Format verification                                          │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│                  Analysis Engine                                 │
│  • Rule execution (deterministic)                                │
│  • Vulnerability detection                                      │
│  • Metadata enrichment                                          │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│              Output Generation & Verification                    │
│  • SARIF format encoding                                         │
│  • Cryptographic signing (optional)                              │
│  • Rate limiting enforcement                                     │
└──────────────────┬──────────────────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────────────────┐
│                    Storage & Delivery                            │
│  • Encrypted storage (at rest)                                   │
│  • TLS transport (in transit)                                    │
│  • Access control enforcement                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Threat Model

### Assets

1. **Analysis Rules**: Integrity of vulnerability detection logic
2. **User Input**: Contracts and reports submitted for analysis
3. **Output Reports**: Vulnerability findings and metadata
4. **Audit Logs**: Compliance and forensic records

### Threats

| Threat | Attack Vector | Impact | Mitigation |
|--------|---------------|--------|-----------|
| **Input Injection** | Malformed WASM, oversized payloads | DoS, memory exhaustion | Schema validation, size limits |
| **Rule Tampering** | Modified detection logic | False positives/negatives | Code review, cryptographic verification |
| **Output Manipulation** | Altered reports | Incorrect vulnerability assessment | Immutable logs, signed output |
| **Side-Channel Attacks** | Timing analysis | Rule extraction | Constant-time comparisons |
| **Unauthorized Access** | Credential compromise | Unauthorized analysis | Rate limiting, authentication |

### Security Controls

1. **Input Validation**
   - All inputs validated against schema before processing
   - Size limits enforced at entry points
   - Format verification for WASM, JSON, SARIF

2. **Execution Isolation**
   - Rules run in sandboxed environment
   - No filesystem access from rules
   - Memory limits enforced per execution

3. **Output Protection**
   - All reports logged with audit trails
   - Optional cryptographic signing
   - Immutable storage with versioning

4. **Access Control**
   - Rate limiting per API key
   - Role-based access control
   - IP allowlisting support

## Data Flow Diagram

```
User Input (Contract WASM)
         │
         ▼
   ┌─────────────┐
   │  Validator  │
   │  - Schema   │
   │  - Size     │
   └──────┬──────┘
          │
     ┌────▼────────┐
     │   Engine    │
     │ - Rules     │
     │ - Analysis  │
     └─────┬───────┘
           │
      ┌────▼────────────┐
      │   Formatter     │
      │  - SARIF        │
      │  - Signing      │
      └─────┬───────────┘
            │
      ┌─────▼────────────────┐
      │ Output Delivery      │
      │ - Storage            │
      │ - TLS Transport      │
      │ - Audit Logs         │
      └──────────────────────┘
```

## Compliance Considerations

- **SOC 2 Type II**: Audit trail logging required
- **GDPR**: No PII in analysis outputs
- **HIPAA**: Encryption required for healthcare contexts
- **PCI DSS**: Rate limiting and access controls

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) - System architecture overview
- [SECURITY.md](../SECURITY.md) - Security policies and procedures
- [SOROBAN_DEPLOYMENT.md](../SOROBAN_DEPLOYMENT.md) - Smart contract deployment security
