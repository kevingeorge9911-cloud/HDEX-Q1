# HDEX Q1
### Hardened Deterministic Execution

HDEX Q1 is a secure, deterministic, and offline execution runtime designed for high-trust workloads.

It enables organizations to execute sensitive jobs with strong security controls, reproducible execution, and verifiable audit evidence.

---
<p align="center">
  <img src="docs/images/HDEX%20Q1%20getting%20started.png"
       alt="HDEX Q1 Getting Started"
       width="100%">
</p>

## Why HDEX Q1?

Traditional execution systems prioritize flexibility and convenience.

HDEX Q1 prioritizes **trust**.

Every execution is designed to be:

- Secure
- Deterministic
- Offline-capable
- Auditable

The runtime follows a fail-closed security model where every important decision can be verified.

---

# Core Guarantees

| Guarantee | Description |
|-----------|-------------|
| 🔐 Secure | Default-deny execution with policy enforcement and sandbox isolation. |
| ♻️ Deterministic | Identical approved inputs produce reproducible execution results. |
| 📦 Offline | Runs without cloud services, telemetry, or runtime downloads. |
| 📋 Auditable | Generates structured evidence for every security-relevant decision. |

---

# Architecture

```
                +-----------------------+
                |   HDEX Q1 Runtime     |
                +-----------------------+
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   Policy Engine     Secure Sandbox     Audit System
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    Worker Execution
```

---

# Key Features

- Policy-based execution
- Secure sandboxing
- Deterministic execution controls
- Offline operation
- Tamper-evident audit logs
- Signed policies and workers
- Preflight validation
- Artifact verification
- Resource isolation
- Reproducible execution

---

# Repository Structure

```
crates/
 ├── hdex-core
 ├── hdex-policy
 ├── hdex-runner
 ├── hdex-sandbox
 ├── hdex-audit
 ├── hdex-artifacts
 ├── hdex-cli
 └── hdex-api

workers/
schemas/
policies/
examples/
tests/
docs/
```

---

# Execution Lifecycle

```
CREATED
    │
VALIDATED
    │
QUEUED
    │
RUNNING
    │
COMPLETED
```

Unsafe executions are blocked before they can run.

---

# Security Principles

HDEX Q1 follows several non-negotiable rules:

- Default-deny security
- Fail-closed policy evaluation
- No implicit trust
- No shell execution
- Explicit filesystem permissions
- Explicit network permissions
- Signed execution components
- Secret redaction
- Deterministic execution
- Offline-first operation

---

# Development

```bash
make check
make test
make security-test
make offline-test
make proof
```

---

# Project Status

HDEX Q1 is currently focused on delivering its proof-first runtime:

- Secure execution sandbox
- Policy enforcement engine
- Deterministic execution
- Offline runtime
- Audit logging
- Secure document pipeline demonstration

---

# License

HDEX Q1 is free-to-use proprietary software.

Official signed binary releases may be used under the project license. The core runtime source code is not publicly licensed.

See the `LICENSE` file for complete terms.
